# springone-tour-2023

SpringOne Tour 2023 :: Scaling Your Spring Boot App to Zero

## Link to session

[https://springonetour.io/sessions/scaling-your-spring-boot-app-to-zero](https://springonetour.io/sessions/scaling-your-spring-boot-app-to-zero)


## Slides

- [SpringOne Tour 2023: Scaling Your Spring Boot App to Zero](https://github.com/trisberg/springone-tour-2023/blob/main/SpringOne%20Tour%202023_%20Scaling%20Your%20Spring%20Boot%20App%20to%20Zero.pdf)

## Tools to install

1. [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
1. [Helm](https://helm.sh/docs/intro/install/)
1. [Knative CLI](https://knative.dev/docs/client/install-kn/)
1. [Knative quickstart CLI plugin](https://github.com/knative-extensions/kn-plugin-quickstart#kn-plugin-quickstart)
1. [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

## Demos

### GraalVM Native

1. [Spring PetClinic as native build as Knative service](PetClinic-native-on-knative.md)
1. [Native build with TAP](TAP-native-build.md)

## Challenge

1. [Tanzu Academy - Developer Sandbox Challenge](Sandbox-challenge.md)

### JVM Checkpoint/Restore

1. [Getting started with JVM Checkpoint/Restore builds](JVM-checkpoint-restore.md)

## Links

### Project CRaC GitHub docs

- https://github.com/CRaC/docs#crac

### Spring Checkpoint Restore Smoke Tests 

- https://github.com/spring-projects/spring-checkpoint-restore-smoke-tests

### JVM Checkpoint/Restore demo app

- https://github.com/trisberg/hello-world

This app has instructions for running it:
- locally using Docker
- kubernetes deployment/service
    - including [notes](https://github.com/trisberg/hello-world/blob/main/keda/README.md) for scaling with Keda
- knative service

### Tanzu Academy - Developer Sandbox

- https://tanzu.academy/guides/developer-sandbox

<img src="images/bit.ly_TanzuDevTry.png" alt="QR Code" width="300"/>

### This repo

- https://github.com/trisberg/springone-tour-2023

<img src="images/springone-tour-2023.png" alt="QR Code" width="300"/>
