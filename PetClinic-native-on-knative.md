# Spring PetClinic native build as Knative service

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

# Teardown

We can destroy the quickstart cluster using:

```sh
kind delete cluster --name knative
```
