# Spring Boot based Java web application
 
This is a simple Sprint Boot based Java application that can be built using Maven. Sprint Boot dependencies are handled using the pom.xml 
at the root directory of the repository.

Execute the Maven targets to generate the artifacts

```
mvn clean package
```

The above maven target stroes the artifacts to the `target` directory. You can either execute the artifact on your local machine
(or) run it as a Docker container.

### Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```

### Execute on Docker

Build the Docker Image

```
docker build -t jenkins-cicd-pipeline:v1 .
```

```
docker run -d -p 8010:8080 -t jenkins-cicd-pipeline:v1
```

Access the application on `http://<ip-address>:8010`
