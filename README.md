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


### Application overview

Run the application locally
Let’s run the application locally to become familiar with it. Open a terminal to the project’s root directory and execute ./mvnw clean package spring-boot:run on Linux/macOS or mvnw.cmd clean package spring-boot:run on Windows. This will compile the application, run all the tests, and then expose the HTTP endpoints on port 8080.

Now, let’s try interacting with the endpoints by opening a browser window or using the curl command from another terminal. The following shows what the values of some of the endpoints should look like:

- http://localhost:8080/hello
  - Hello World
- http://localhost:8080/hello/Yoda
  - Hello Yoda
- http://localhost:8080/actuator/info
  - {}
- http://localhost:8080/actuator/env/hello.greeting
```bash
{
  "property": {
  "source": "Config resource 'class path resource [application.yml]' via location 'optional:classpath:/'",
  "value": "Hello"
  }
} 
```

NOTE: There is most likely more data in the http://localhost:8080/actuator/env/hello.greeting response. The output is trimmed for brevity.

The /actuator/env/hello.greeting endpoint is the Spring Boot Actuator Environment endpoint. It allows retrieving information about the entire environment or just a single property. This information includes the current value of a property and where it was resolved.

In our case, you can see that the current value of hello.greeting is Hello, and it was resolved from the application.yml file on the classpath, which originated in src/main/resources/application.yml.

### Eclipse JKube
We need a way to deploy and test our application in a Kubernetes environment before committing our code into version control. We will use Eclipse JKube for this.
This link gives more information on Eclipse JKube:
https://www.eclipse.org/jkube/

We add the JKube kubernetes-maven-plugin to our project:
```xml
<profiles>
  <profile>
    <id>k8s</id>
    <properties>
      <jkube.enricher.jkube-namespace.namespace>${project.artifactId}</jkube.enricher.jkube-namespace.namespace>
      <jkube.generator.from>registry.access.redhat.com/ubi8/openjdk-${maven.compiler.release}:latest</jkube.generator.from>
      <jkube.namespace>${project.artifactId}</jkube.namespace>
      <jkube.recreate>true</jkube.recreate>
      <jkube.version>1.10.1</jkube.version>
    </properties>
    <build>
      <plugins>
        <plugin>
          <groupId>org.eclipse.jkube</groupId>
          <artifactId>kubernetes-maven-plugin</artifactId>
          <version>${jkube.version}</version>
          <executions>
            <execution>
              <id>jkube</id>
              <goals>
                <goal>resource</goal>
                <goal>build</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```

We then run:
```bash
./mvnw k8s:deploy -Pk8s
```

We can then see our service running:
```bash
tom@tom-ubuntu:~/Projects/spring-jkube-external-config$ minikube service list
|------------------------------|------------------------------|--------------|---------------------------|
|          NAMESPACE           |             NAME             | TARGET PORT  |            URL            |
|------------------------------|------------------------------|--------------|---------------------------|
| default                      | kubernetes                   | No node port |
| kube-system                  | kube-dns                     | No node port |
| kubernetes-dashboard         | dashboard-metrics-scraper    | No node port |
| kubernetes-dashboard         | kubernetes-dashboard         | No node port |
| spring-jkube-external-config | spring-jkube-external-config | http/8080    | http://192.168.49.2:30526 |
|------------------------------|------------------------------|--------------|---------------------------|
```

### Externalize configuration to ConfigMap
We will use the Spring Cloud Kubernetes project’s PropertySource implementations to help read external configuration. We’ll need to add some dependencies to our project as well as some additional JKube resource fragments, but we won’t need to make any changes to the application’s source code.

Open the project’s pom.xml file again. We first need to add the Spring Cloud Bill of Materials to the project:

In the <properties> section, add <spring-cloud.version>2022.0.0</spring-cloud.version>

After the <properties> section, add a new section:
```xml

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
In the <dependencies>, add the kubernetes-client-config starter:
```xml

<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-kubernetes-client-config</artifactId>
</dependency>
```

We then run ./mvnw verify to make sure everything still works.

We then add rolebinding.yml:

```yaml

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view
subjects:
  - kind: ServiceAccount
    name: default
```

and configmap.yml:

```yaml

data:
  hello.greeting: Hello (from Kubernetes ConfigMap)
```

We then run:
```bash
./mvnw k8s:deploy -Pk8s
```

to redeploy to minikube.

We can then see our service running:
```bash
tom@tom-ubuntu:~/Projects/spring-jkube-external-config$ minikube service list
|------------------------------|------------------------------|--------------|---------------------------|
|          NAMESPACE           |             NAME             | TARGET PORT  |            URL            |
|------------------------------|------------------------------|--------------|---------------------------|
| default                      | kubernetes                   | No node port |
| kube-system                  | kube-dns                     | No node port |
| kubernetes-dashboard         | dashboard-metrics-scraper    | No node port |
| kubernetes-dashboard         | kubernetes-dashboard         | No node port |
| spring-jkube-external-config | spring-jkube-external-config | http/8080    | http://192.168.49.2:30526 |
|------------------------------|------------------------------|--------------|---------------------------|
```

The hello.greeting property is now being read from the ConfigMap.
![image](https://user-images.githubusercontent.com/27693622/236437868-ff528e4d-4096-42d3-8016-59f10aa90132.png)



