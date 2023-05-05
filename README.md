### Deploying Spring Boot to Kubernetes with Eclipse JKube and ConfigMaps

In this lesson, we will learn how to deploy a Spring Boot application to Kubernetes using the kubernetes-maven-plugin from the Eclipse JKube project.
We will then learn how to inject configuration from a ConfigMap into our application using Spring Cloud Kubernetes Config.
This link is useful for an overview of the process:
https://kubebyexample.com/learning-paths/developing-spring-boot-kubernetes/lesson-4-deploying-spring-boot-kubernetes-eclipse

Here we use minikube as the Kubernetes distribution, but the same steps apply to any Kubernetes distribution.

We also require Jave runtime version >= 17. Spring Boot 3 requires Java 17.

This is the repository backing the [Spring Boot externalized Configuration](https://kubebyexample.com/en/learning-paths/developing-spring-boot-kubernetes/lesson-4-deploying-spring-boot-kubernetes-eclipse) lesson for the [Developing with Spring Boot on Kubernetes](https://kubebyexample.com/en/learning-paths/developing-spring-boot-kubernetes) learning path on [kube by example](https://kubebyexample.com).

This project was generated using [start.spring.io](https://start.spring.io). The [`HelloController`](src/main/java/org/acme/externalconfig/rest/HelloController.java) and [`HelloControllerTests`](src/test/java/org/acme/externalconfig/rest/HelloControllerTests.java) classes were added as a starting point for the lesson.

The completed solution for the lesson can be found on the [`solution`](https://github.com/RedHat-Middleware-Workshops/spring-jkube-external-config/tree/solution) branch of the repo.


