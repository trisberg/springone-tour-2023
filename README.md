# springone-tour-2023

SpringOne Tour 2023 :: Scaling Your Spring Boot App to Zero

## Link to session

[https://springonetour.io/sessions/scaling-your-spring-boot-app-to-zero](https://springonetour.io/sessions/scaling-your-spring-boot-app-to-zero)


## Slides

> TBD

## Tools to install

1. [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)
1. [Carvel](https://carvel.dev/)
1. [Helm](https://helm.sh/docs/intro/install/)
1. [Knative CLI](https://knative.dev/docs/client/install-kn/)
1. [Knative quickstart CLI plugin](https://github.com/knative-extensions/kn-plugin-quickstart#kn-plugin-quickstart)
1. [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

## Demos

### GraalVM Native

1. [Spring PatClinic as native build as Knative service](PetClinic-native-on-knative.md)
1. [Native build with TAP](TAP-native-build.md)

## Challenge

1. [Tanzu Academy - Developer Sandbox Challenge](Sandbox-challenge.md)

### JVM Checkpoint/Restore

1. [Getting started with JVM Checkpoint/Restore builds](JVM-checkpoint-restore.md)

## Links

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
