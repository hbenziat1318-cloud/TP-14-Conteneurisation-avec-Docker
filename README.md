# TP14 Docker - Application Spring Boot avec MySQL

##  Structure du Projet

```
springdocker/
│
├── src/
│   └── main/
│       ├── java/ma/ens/springdocker/
│       │   ├── SpringdockerApplication.java
│       │   └── controller/
│       │       └── TestController.java
│       └── resources/
│           └── application.properties
│
├── target/
│   ├── springdocker-0.0.1-SNAPSHOT.jar          # Fichier JAR généré
│   └── springdocker-0.0.1-SNAPSHOT.jar.original
│
├── docker-compose.yml                           # Configuration multi-conteneurs
├── Dockerfile                                   # Définition de l'image Spring Boot
├── pom.xml                                      # Configuration Maven
└── README.md                                    # Ce fichier
 ```


## Description
Ce projet démontre la conteneurisation d'une application Spring Boot avec une base de données MySQL utilisant Docker et Docker Compose.

##  Architecture
- *Spring Boot* : Application web sur le port 8080
- *MySQL* : Base de données sur le port 3306
- *Docker Compose* : Orchestration des conteneurs

##  Installation et Démarrage

### Prérequis
- Docker Desktop
- Java 21
- Maven

### Démarrage de l'application
bash
# 1. Se positionner dans le projet
cd springdocker

# 2. Compiler l'application
mvn clean package -DskipTests

# 3. Démarrer avec Docker Compose
docker-compose up -d --build

# 4. Vérifier les conteneurs
docker-compose ps


##  Structure des Fichiers

### Dockerfile
   
```
dockerfile
FROM eclipse-temurin:21-jdk
WORKDIR /app
COPY target/springdocker-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
 ```

### docker-compose.yml
```
yaml
services:
  mysql-db:
    image: mysql:8.0
    container_name: mysql-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 1234
      MYSQL_DATABASE: demo_db
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - spring-mysql-net

  spring-app:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: spring-app
    restart: always
    depends_on:
      - mysql-db
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://mysql-db:3306/demo_db?createDatabaseIfNotExist=true
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=1234
      - SPRING_JPA_HIBERNATE_DDL_AUTO=update
    ports:
      - "8080:8080"
    networks:
      - spring-mysql-net

volumes:
  mysql-data:

networks:
  spring-mysql-net:
    driver: bridge
 ```


### application.properties
 ```
properties
spring.datasource.url=jdbc:mysql://localhost:3306/demo_db?createDatabaseIfNotExist=true
spring.datasource.username=root
spring.datasource.password=1234
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
server.port=8080
  ```


### TestController.java
```
java
package ma.ens.springdocker.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {

    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Boot Docker Application!";
    }

    @GetMapping("/health")
    public String health() {
        return "Application is running!";
    }
}
 ```


##  TEST DE L'APPLICATION

### ✅ Test 1 : Vérification des conteneurs
bash
# Vérifier que tous les conteneurs tournent
docker-compose ps

# Résultat attendu :



<img width="927" height="173" alt="image" src="https://github.com/user-attachments/assets/9c0ada23-2ca9-41c0-be0c-aa3d6bfcd6d8" />



### ✅ Test 2 : Test de l'application Spring Boot

# Tester l'endpoint principal
curl http://localhost:8080

# Résultat attendu :



https://github.com/user-attachments/assets/3ca7843f-9ba7-461e-a295-18d5c1a59445



# Tester le health check
curl http://localhost:8080/health

# Résultat attendu :


https://github.com/user-attachments/assets/6bb86645-f6a8-4607-81a3-de996a06bcbd





### ✅ Test 3 : Test de la base de données MySQL

# Vérifier que la base de données est créée
docker exec mysql-db mysql -u root -p1234 -e "SHOW DATABASES;"

# Résultat attendu :
```
# +--------------------+
# | Database           |
# +--------------------+
# | demo_db            |
# | information_schema |
# | mysql              |
# | performance_schema |
# | sys                |
# +--------------------+
```


### ✅ Test 4 : Vérification des logs

# Voir les logs de l'application Spring Boot
docker-compose logs spring-app



https://github.com/user-attachments/assets/d7324524-4dbe-40c4-a7b6-58b4b7edee65



# Voir les logs de MySQL
docker-compose logs mysql-db



https://github.com/user-attachments/assets/bc0e97c1-f23a-494c-a97d-159a8889f62f



# Voir tous les logs en temps réel
docker-compose logs -f



https://github.com/user-attachments/assets/1f247944-ab99-4ee0-a9ed-dc778024db4f




### ✅ Test 5 : Test de persistance des données

# Arrêter et redémarrer pour vérifier la persistance
docker-compose down
docker-compose up -d


https://github.com/user-attachments/assets/f64cdbe4-9f95-414b-a0cf-b59c9fc60bb5


# Les données doivent être conservées grâce au volume Docker


## Dépannage

### Problème : Application non accessible
bash
# Vérifier les ports
netstat -an | findstr 8080

# Redémarrer les conteneurs
docker-compose restart


### Problème : MySQL non accessible
bash
# Vérifier que MySQL écoute
docker exec mysql-db netstat -tuln | grep 3306

# Vérifier les logs MySQL
docker-compose logs mysql-db


### Problème : Build Docker échoue
bash
# Reconstruire l'image
docker-compose build --no-cache

# Vérifier le JAR
ls -la target/springdocker-0.0.1-SNAPSHOT.jar


##  Validation du TP

### Checklist de réussite
-  Les conteneurs Docker démarrent sans erreur
-  L'application Spring Boot répond sur le port 8080
-  La base de données MySQL est accessible
-  Les deux conteneurs communiquent entre eux
-  Les données persistent après redémarrage
- L'application affiche le message de bienvenue


##  Arrêt de l'application
bash
# Arrêter les conteneurs
docker-compose down

# Arrêter et supprimer les volumes
docker-compose down -v

**BENZIAT hana**
