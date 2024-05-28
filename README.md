# README

## Introduction

This project provides a Docker setup for a PostgreSQL database along with Adminer for database management. Below are the detailed steps to create and manage your PostgreSQL Docker container, set up networking, and ensure data persistence.

## Prerequisites

- Docker installed on your machine
- Terminal or command prompt access

## Step-by-Step Guide

### 1. Create the Dockerfile

Create a `Dockerfile` with the following content:

```dockerfile
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

### 2. Build the Docker Image

Open a terminal and navigate to the directory containing your `Dockerfile`. Run the following command to build the Docker image:

```sh
docker build -t my_postgres .
```

This command will build the image and tag it as `my_postgres`.

### 3. Create a Docker Network

Create a Docker network named `app-network`:

```sh
docker network create app-network
```

### 4. Run the PostgreSQL Container

Run a container from the image you just built. Use the following command to start the container and bind a port on your host to the container's port 5432:

```sh
docker run --name mypostgres -p 5432:5432 -d --network app-network my_postgres
```

To verify that the container is running, use:

```sh
docker ps
```

You should see output similar to this:

```
CONTAINER ID   IMAGE         COMMAND                  CREATED              STATUS              PORTS                  NAMES
971af71c88a0   my_postgres   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:5432->5432/tcp mypostgres
```

### 5. Run the Adminer Container

Run an Adminer container for database management:

```sh
docker run --name my_adminer -p 8080:8080 --network app-network -d adminer
```

You can now access Adminer by navigating to `http://localhost:8080` in your web browser.

### 6. Prepare SQL Files

Create a directory named `data` in the same location as your `Dockerfile`. Place your SQL files inside this `data` directory.

01-CreateScheme.sql:


CREATE TABLE public.departments
(
 id      SERIAL      PRIMARY KEY,
 name    VARCHAR(20) NOT NULL
);

CREATE TABLE public.students
(
 id              SERIAL      PRIMARY KEY,
 department_id   INT         NOT NULL REFERENCES departments (id),
 first_name      VARCHAR(20) NOT NULL,
 last_name       VARCHAR(20) NOT NULL
);

02-InsertData.sql :

INSERT INTO departments (name) VALUES ('IRC');
INSERT INTO departments (name) VALUES ('ETI');
INSERT INTO departments (name) VALUES ('CGP');


INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');

put sql files in /docker-entrypoint-initdb.d on docker;
When rerun, it deletes the files because they are not permanent.

### 7. Use Volumes to Persist Data

To persist data and ensure it is not lost when the container is removed, use volumes. This will map a directory on your host machine to the container:

```sh
docker run --name mypostgres -d --network app-network -v ./data:/docker-entrypoint-initdb.d my_postgres
```

### 8. Verify SQL Files in Container

Ensure your SQL files are located in the `./data` directory on your host. These files will be copied to the container's `/docker-entrypoint-initdb.d` directory when the container starts. You can verify this by checking the contents of the directory within the container.

### 9. Recreate the PostgreSQL Container

If you need to make changes to the SQL files and want them to be reloaded, remove the existing container and recreate it:

```sh
docker rm -f mypostgres
docker run --name mypostgres -d --network app-network -v ./data:/docker-entrypoint-initdb.d my_postgres
```

### 10. Updated Dockerfile with COPY Instruction

Ensure your `Dockerfile` includes the instruction to copy the SQL files:

```dockerfile
FROM postgres:14.1-alpine

# Copy the SQL scripts into the container
COPY ./data /docker-entrypoint-initdb.d

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd
```

### 11. Add SQL Files to Entrypoint

Ensure your SQL files are correctly added to the entrypoint directory `/docker-entrypoint-initdb.d`. This ensures they are executed when the container is started.

### 12. Verify Container and Data Persistence

After recreating the container, verify that the SQL files have been correctly copied into the container and are persistent across container restarts. You can check the container's `/docker-entrypoint-initdb.d` directory.

## Conclusion

You now have a PostgreSQL database running in a Docker container, connected to an Adminer interface for easy database management. The use of Docker volumes ensures that your data persists across container restarts and recreations.


### Backend API

#### Étape 1: Exécution d'une Classe Java Hello World

**1. Création du fichier `Main.java`:**

```java
public class Main {
   public static void main(String[] args) {
       System.out.println("Hello World!");
   }
}
```

**2. Compilation avec la version Java cible:** `javac Main.java`.
Si besoin il faut réinstaller java avec une version plus récente. Dans mon cas, java avait du mal à se mettre sur mon pc.

**3. Création du Dockerfile:**

```Dockerfile
# Utilisation d'une image Java pour exécuter un environnement Java runtime
FROM openjdk:17-rc-oraclelinux8

# Création d'un répertoire de travail
WORKDIR /app

# Copie du fichier .class compilé dans le conteneur
COPY Main.class /app

# Exécution de l'application Java
CMD ["java", "Main"]
```

**Explication:** Dans cette étape, nous avons écrit une classe Java simple qui affiche "Hello World!" sur la console. Ensuite, nous l'avons compilée en utilisant le compilateur Java (`javac`) pour générer un fichier de classe exécutable. Ensuite, nous avons créé un Dockerfile qui utilise une image OpenJDK pour exécuter notre application Java Hello World dans un conteneur Docker.

**4. Construction de l'image Docker:**

```bash
docker build -t java-hello-world .
```

**5. Exécution du conteneur Docker:**

```bash
docker run --name java-hello-world-container --network app-network java-hello-world
```

**Explication:** Nous avons construit une image Docker à partir du Dockerfile et ensuite exécuté un conteneur Docker à partir de cette image. Le message "Hello World!" devrait être affiché dans la sortie de la console.

#### Étape 2: Construction d'une Application Spring Boot avec une API Simple

**1. Création d'une application Spring Boot avec Spring Initializr:**

Nous avons utilisé Spring Initializr pour générer une nouvelle application Spring Boot avec les dépendances nécessaires pour créer une API Web simple.

**2. Implémentation du contrôleur `GreetingController`:**

Nous avons créé une classe `GreetingController` qui définit un point de terminaison REST simple qui renvoie une salutation avec un paramètre optionnel pour le nom.

package fr.takima.training.simpleapi.controller;

import org.springframework.web.bind.annotation.*;

import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {

   private static final String template = "Hello, %s!";
   private final AtomicLong counter = new AtomicLong();

   @GetMapping("/")
   public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
       return new Greeting(counter.incrementAndGet(), String.format(template, name));
   }

   record Greeting(long id, String content) {}

}


**3. Création du Dockerfile pour l'application Spring Boot:**

```Dockerfile
# Étape de construction
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Étape d'exécution
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

**Explication:** Dans ce Dockerfile, nous utilisons une image Maven pour compiler notre application Spring Boot. Nous copions d'abord le fichier `pom.xml` et le répertoire `src` contenant notre code source. Ensuite, nous exécutons la commande Maven `mvn package` pour construire le jar exécutable de notre application. Enfin, nous utilisons une image Amazon Corretto pour exécuter notre application Spring Boot.

**4. Construction de l'image Docker:**

```bash
docker build -f Dockerfile -t simpleapi .
```

**5. Exécution du conteneur Docker:**

```bash
docker run -p 80:8080 simpleapi
```

#### Étape 3: Connexion à la Base de Données

**1. Ajustement de la configuration dans `simple-api/src/main/resources/application.yml`:**

Nous avons ajusté le fichier `application.yml` pour configurer la connexion à la base de données. Nous avons fourni l'URL, le nom d'utilisateur et le mot de passe pour se connecter à la base de données PostgreSQL.

**2. Exécution du conteneur Docker:**

Nous avons exécuté le conteneur Docker avec les modifications apportées à la configuration pour nous assurer que l'application est correctement liée à la base de données. Une fois que tout est correctement lié, nous devrions pouvoir accéder à notre API d'application, par exemple sur : `/departments/IRC/students`.

```json
[
  {
    "id": 1,
    "firstname": "Eli",
    "lastname": "Copter",
    "department": {
      "id": 1,
      "name": "IRC"
    }
  }
]
```

**Explication:** Pour que notre application Spring Boot puisse accéder à la base de données, nous devons configurer les paramètres de connexion dans le fichier `application.yml`. Ensuite, en exécutant le conteneur Docker, nous nous assurons que tout est correctement lié et que notre API fonctionne correctement.


### Http Server

#### Basics

**1. Choose an appropriate base image:**

We'll start by selecting an appropriate base image for our HTTP server. In this case, we'll use the `httpd:2.4-alpine` image, which provides an Apache HTTP server.

**2. Create a simple landing page: `index.html`**

We'll create a simple HTML landing page named `index.html` and place it inside our container. Here's the content of the `index.html` file:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Welcome to My Httpd Docker Container!</title>
</head>

<body>
  <header>
    <h1>Welcome to My Httpd Docker Container!</h1>
  </header>

  <main>
    <p>This is a simple landing page served by the Httpd (Apache) server inside a Docker container.</p>
  </main>
</body>

</html>
```

**3. Dockerfile:**

```Dockerfile
FROM httpd:2.4-alpine

COPY index.html /usr/local/apache2/htdocs/
```

**4. Build the Docker image:**

```bash
docker build -t my-httpd-image .
```

**5. Run the container:**

```bash
docker run --name my-httpd-container -p 82:80 -d my-httpd-image
```

**6. Verify everything is working:**

You can check that everything is working as expected using the following commands:

```bash
docker stats
docker inspect my-httpd-container
docker logs my-httpd-container
```

#### Reverse Proxy

**1. Retrieve the default configuration from the running container:**

We can use `docker exec` to retrieve the default configuration file from the running container at `/usr/local/apache2/conf/httpd.conf`.

```bash
docker exec -it my-httpd-container cat /usr/local/apache2/conf/httpd.conf
```

**2. Modify the default configuration:**

Add the following configuration in  /usr/local/apache2/conf/httpd.conf to enable reverse proxy:

```apache

<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://simple-api-container-student:8080/
ProxyPassReverse / http://simple-api-container-student:8080/
</VirtualHost>
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

```

**3. Update the Dockerfile to include the modified configuration:**

```Dockerfile
COPY httpd.conf /usr/local/apache2/conf/httpd.conf
```

#### Tip: Why do we need a reverse proxy?

A reverse proxy can be useful for several reasons, including:

- Serving front-end applications.
- Configuring SSL termination.
- Handling load balancing.

#### Notes

At this point, we have a working HTTP server serving a simple landing page, and we've configured it as a reverse proxy to forward requests to a backend application.