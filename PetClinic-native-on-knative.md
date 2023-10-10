# Spring PetClinic native build running as Knative service

This doc will help you get started building native images based on your Spring Boot based apps. This demo uses the Spring PetClinic sample app. We build it as a native image and then run it in a local Knative cluster.
    
# Installation

1. install Knative CLI - https://knative.dev/docs/client/install-kn/
1. install `quickstart` CLI plugin - https://github.com/knative-extensions/kn-plugin-quickstart#kn-plugin-quickstart
1. install Kind - https://kind.sigs.k8s.io/docs/user/quick-start/

# Starting a Knative quickstart cluster

To create a Kind cluster using Knative quickstart with Knative Serving, run this command:

```sh
kn quickstart kind --registry --install-serving
```

# Building and deploying PetClinic

## Clone the repo

Run the following command to clone the PetClinic repo:

```sh
git clone https://github.com/spring-projects/spring-petclinic
```

We then change to the `spring-petclinic` directory.

```sh
cd spring-petclinic
```

## Building the app

We want to use a Prostgres database so we need to set that profile when building.

Set the registry and repo to use for the built image:

```sh
export IMAGE_NAME=docker.io/springdeveloper/spring-petclinic:3.1.0-SNAPSHOT
```

### Change builder when using a Mac with Apple Silicon

If you are using a Mac with Apple Silicon then you need to use a builder that can handle building an image for the ARM architecure. One way of doing that is to add `dashaun/builder:tiny` builder image to the`spring-boot-maven-plugin`.

Add an additional profile to update the entry for `spring-boot-maven-plugin` using the builder image `dashaun/builder:tiny`:

```xml
    <profile>
      <id>arm</id>
      <build>
        <plugins>
          <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
              <image>
                <builder>dashaun/builder:tiny</builder>
              </image>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
```

### Build on Intel (amd64) based system

```sh
./mvnw -Pnative -Dmaven.test.skip=true \
  -Dspring-boot.aot.jvmArguments='-Dspring.profiles.active=postgres' \
  -Dspring-boot.build-image.imageName=$IMAGE_NAME \
  spring-boot:build-image 
```

### Build on ARM (aarch64) based system

```sh
./mvnw -Pnative,arm -Dmaven.test.skip=true \
  -Dspring-boot.aot.jvmArguments='-Dspring.profiles.active=postgres' \
  -Dspring-boot.build-image.imageName=$IMAGE_NAME \
  spring-boot:build-image 
```

### Push image to registry

```sh
docker push $IMAGE_NAME
```

## Create the Postgres database

We can use Helm to create a basic database deployment:

```sh
helm install pets oci://registry-1.docker.io/bitnamicharts/postgresql
```

This creates the resources for the detabase and also shows connection information that we can use when we deploy the app. The password for the `postgres` user is provided in a `pets-postgresql` secret.

## Deploying the app

To deploy the function we use the following command:

```sh
cat <<EOF | kubectl -n default create -f -
---
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: spring-petclinic
spec:
  template:
    spec:
      containers:
      - name: workload
        image: $IMAGE_NAME
        imagePullPolicy: Always
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "postgres"
        - name: SPRING_DATASOURCE_URL
          value: "jdbc:postgresql://pets-postgresql.default.svc.cluster.local:5432/postgres"
        - name: SPRING_DATASOURCE_USERNAME
          value: "postgres"
        - name: SPRING_DATASOURCE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pets-postgresql
              key: postgres-password
EOF
```

# Access the App

Once the app gets deployed we can access it with the URL shown with this command:

```sh
kn service list
```

# Check the startup log

We can view the startup log by running:

```sh
kubectl logs deployment/spring-petclinic-00001-deployment -c workload
```

It should look like this:

```

              |\      _,,,--,,_
             /,`.-'`'   ._  \-;;,_
  _______ __|,4-  ) )_   .;.(__`'-'__     ___ __    _ ___ _______
 |       | '---''(_/._)-'(_\_)   |   |   |   |  |  | |   |       |
 |    _  |    ___|_     _|       |   |   |   |   |_| |   |       | __ _ _
 |   |_| |   |___  |   | |       |   |   |   |       |   |       | \ \ \ \
 |    ___|    ___| |   | |      _|   |___|   |  _    |   |      _|  \ \ \ \
 |   |   |   |___  |   | |     |_|       |   | | |   |   |     |_    ) ) ) )
 |___|   |_______| |___| |_______|_______|___|_|  |__|___|_______|  / / / /
 ==================================================================/_/_/_/

:: Built with Spring Boot :: 3.1.3


2023-10-10T18:00:01.028Z  INFO 1 --- [           main] o.s.s.petclinic.PetClinicApplication     : Starting AOT-processed PetClinicApplication using Java 17.0.8.1 with PID 1 (/workspace/org.springframework.samples.petclinic.PetClinicApplication started by cnb in /workspace)
2023-10-10T18:00:01.028Z  INFO 1 --- [           main] o.s.s.petclinic.PetClinicApplication     : The following 1 profile is active: "postgres"
2023-10-10T18:00:01.052Z  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2023-10-10T18:00:01.053Z  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2023-10-10T18:00:01.053Z  INFO 1 --- [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.12]
2023-10-10T18:00:01.058Z  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2023-10-10T18:00:01.058Z  INFO 1 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 30 ms
2023-10-10T18:00:01.086Z  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2023-10-10T18:00:01.118Z  INFO 1 --- [           main] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@700854de
2023-10-10T18:00:01.118Z  INFO 1 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2023-10-10T18:00:01.158Z  INFO 1 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
2023-10-10T18:00:01.160Z  INFO 1 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate ORM core version 6.2.7.Final
2023-10-10T18:00:01.160Z  WARN 1 --- [           main] org.hibernate.orm.deprecation            : HHH90000029: The [hibernate.bytecode.use_reflection_optimizer] configuration is deprecated and will be removed. Set the value to [true] to get rid of this warning
2023-10-10T18:00:01.172Z  INFO 1 --- [           main] o.h.b.i.BytecodeProviderInitiator        : HHH000021: Bytecode provider name : none
2023-10-10T18:00:01.181Z  INFO 1 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
2023-10-10T18:00:01.181Z  INFO 1 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
2023-10-10T18:00:01.197Z  INFO 1 --- [           main] o.s.d.j.r.query.QueryEnhancerFactory     : Hibernate is in classpath; If applicable, HQL parser will be used.
2023-10-10T18:00:01.304Z  INFO 1 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 13 endpoint(s) beneath base path '/actuator'
2023-10-10T18:00:01.309Z  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2023-10-10T18:00:01.309Z  INFO 1 --- [           main] o.s.s.petclinic.PetClinicApplication     : Started PetClinicApplication in 0.293 seconds (process running for 0.305)
```

**Notice** the startup time: `Started PetClinicApplication in 0.293 seconds (process running for 0.305)`

# Teardown

We can destroy the quickstart cluster using:

```sh
kind delete cluster --name knative
```
