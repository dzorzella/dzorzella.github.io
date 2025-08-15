---
date: '2025-07-16T11:11:48+02:00'
draft: false
title: 'Docker Secrets with swarm'
author: Zorzella Davide
tags: ["secrets", "docker", "swarm"]
categories: ["docker"]

cover:
  image: "swarm-logo.jpg"
  alt: "swarm logo"
  caption: "swarm logo"
---

# Swarm orchestrator

When using docker, especially the orchestrator swarm, you can rely on the use of internally managed secrets. Since this is a cluster management command, it must be executed on the swarm manager node. In the meantime, you need to create the docker swarm cluster if it is not already present

```bash
docker swarm init
```

The following command, while recommended by the docker documentation, presents a hidden danger. If the command were launched from the shell (such as bash) it would be searchable a posteriori via the history command, dangerously exposing the token.

```bash
echo “sUp3R_seCr3t_Pa55w0Rd” | docker secret create db_password – # Non-compliant
```

An alternative would be to enter only the command

```bash
docker secret create db_password – # compliant
```

And then, through interaction with the program itself, enter the value of the secret

```
sUp3R_seCr3t_Pa55w0Rd [CTRL+D]
```

So that only the command without the secret value is kept in the shell history.

# Recover secrets with Java 

To retrieve the secret value from the Java runtime program, you need to take into account that secrets managed by docker swarm automatically end up in /run/secrets/. This allows you to access it as if it were a simple file mounted on the docker file system.

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

public class Secrets {
    public static void main(String[] args) {
        String secret = readDockerSecret("db_password");
        System.out.println("secret: " + secret);
    }

    private static String readDockerSecret(String secretName) {
        String secretPath = "/run/secrets/" + secretName;
        try {
            return new String(Files.readAllBytes(Paths.get(secretPath))).trim();
        } catch (IOException e) {
            System.err.println("Error while reading the secret: " + e.getMessage());
            return null;
        }
    }
}
```

# Dockerfile

The following is the Dockerfile

```Dockerfile
FROM alpine:latest
WORKDIR /work/secrets/
COPY Secrets.java /work/secrets/

# Install JDK
RUN apk add openjdk8
ENV JAVA_HOME /usr/lib/jvm/java-1.8-openjdk
ENV PATH=$PATH:$JAVA_HOME/bin

# Compile
RUN javac Secrets.java

ENTRYPOINT java Secrets
```

# Build the image

It is also of course necessary to build the java-app image via Dockerfile with the command

```bash
docker build -t java-app:latest .
```

It is good practice not to put secrets inside image builds, as it will expose them to anyone who uses it. Instead, secrets should be included with the –secret secret flag as follows. Once the image is built, you can start running it with the following command

```bash
docker service create \
  --name java-app \
  --secret db_password \
  java-app
```

# Secret output

The output containing the secret is successfully retrieved and is visible in the logs

```bash
docker logs $container_id
```

```
secret: sUp3R_seCr3t_Pa55w0Rd
```