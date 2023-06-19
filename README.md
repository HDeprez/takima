## DEVOPS - 2023 - Hugo Deprez - 71802209 - h.deprez@free.fr

### TP1

#### 1.1

Commande :
```
docker build -t tp1 .
```

Description :
- `docker build` permet de construire l'image à  partir du Dockerfile
- l'option `-t` permet de donner un nom (--tag) à  l'image (ici `tp1`)
- `.` permet de spécifier le path vers le Dockerfile 

Commande :
```
docker run --name db_tp1 --network app-network -e POSTGRES_PASSWORD=pwd -v /./db_data:/var/lib/postgresql/data tp1
```

Description :
- `docker run` permet de lancer un conteneur
- l'option `--name` permet de donner un nom au conteneur (ici `db_tp1`)
- l'option `--network` permet de spécifier le réseau auquel appartient le conteneur (ici `app-network`)
- l'option `-e` permet de spécifier une variable d'environnement (ici `POSTGRES_PASSWORD` avec la valeur `pwd`)
- l'option `-v` permet de setup un volume afin de persister les données (ici on monte un volume entre `./db_data` en local et `/var/lib/postgresql/data` dans le conteneur)

Dockerfile :
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=tp_db \
   POSTGRES_USER=exampleuser

COPY ./docker-entrypoint-initdb.d /docker-entrypoint-initdb.d
```

Description :
- on part d'une image contenant ce dont on a besoin (ici on a besoin de faire tourner postgresql)
- on déclare ensuite les variables d'environnements dont on va avoir besoin (ici `POSTGRES_DB` et `POSTGRES_USER`, et `POSTGRES_USER` est défini avec une option afin de ne pas laisser le password en clair dans le Dockerfile)
- on copie les scripts qui devront s'exécuter sur le conteneur

#### 1.2

Dockerfile :
```
# Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

# Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
```

Description :
- pourquoi a-t-on besoin d'un multistage build ? On utilise une certaine image pour build, puis une certaine image pour le run, de facon a réduire la taille de l'image finale
- dans la phase de build :
    - on part d'une image contenant un jdk ET maven
    - on déclare une variable d'environnement `MYAPP_HOME` avec la valeur `/opt/myapp` puis on défini le répertoire de travail sur la valeur de cette variable (cad `/opt/myapp`)
    - on copie le `pom.xml` (nécessaire pour la commande mvn juste après) et le contenu du répertoire `src` dans le conteneur
    - on run la commande `mvn package` pour build le jar de notre application
- dans la phase de run :
    - on part d'une image contenant simplement un jre
    - on déclare une variable d'environnement `MYAPP_HOME` avec la valeur `/opt/myapp` puis on défini le répertoire de travail sur la valeur de cette variable (cad `/opt/myapp`)
    - on copie le jar généré lors de la phase de build (grâce a l'option `--from=myapp-build`)
    - on lance la commande `java -jar myapp.jar` pour lancer l'application

#### 1.3

La commande `docker-compose up` permet de créer et de lancer plusieurs conteneurs, en suivant les instructions contenues dans un `docker-compose.yml`.

#### 1.4

docker-compose.yml :
```
version: '3.7'

services:
    backend:
        build:
          ./simple-api
        networks:
          - my-network
        depends_on:
         - database

    database:
        build:
          ./database
        networks:
         - my-network
        volumes:
          - data_db:/var/lib/postgresql/data

    httpd:
        build:
          ./HTTP_Server
        ports:
          - 8080:80  
        networks:
          - my-network
        depends_on:
          - backend

volumes:
  data_db:

networks:
    my-network:
```

Il a trois parties importantes :
- networks :
    - permet de lister les networks disponibles (ils seront créés s'il n'existent pas)
    - ici, on crée un network du nom de `my-network` (il sera utilisé plus tard)
- volumes :
    - permet de lister les volumes disponibles (ils seront créés s'il n'existent pas)
    - ici, on crée un volume du nom de `data_db` (il sera utilisé plus tard)
- services :
    - permet de lister les différents services à build / run :
        - database : service de la base de données
        - backend : API java Spring
        - httpd : proxy entre le futur front et l'API
    - chaque service possède des attributs :
        - `build` : répertoire contenant le Dockerfile pour build l'image
        - `networks` : les réseaux dont doit faire partie le conteneur
        - `depends_on service_name` : permet de définir une ordre dans la création des services en précisant des relations de dépendances (par exemple, ici `httpd` doit attendre que le service `backend` ait commencé a être build pour lui même commencer à être build)
        - volumes : spécifie les volumes que le conteneur doit utiliser (ici, on utilise le volume créé dans la partie `volumes`)
        - `ports` : permet de spécifier les ports du conteneur

Pour résumer :
- on crée un volume nommé `data_db`
- on crée un network nommé `my-network`
- on lance la base de données avec :
    - un conteneur nommé `database`
    - dont l'image peut être build via le Dockerfile présent dans `./database`
    - qui utilise le volume `data_db` vers le répertoire `/var/lib/postgresql/data` dans le conteneur
    - et qui est sur le réseau `my-network`
- on lance le backend (qui dépend du service `database`) avec :
    - un conteneur nommé `backend`
    - dont l'image peut être build via le Dockerfile présent dans `./simple-api`
    - et qui est sur le réseau `my-network`
- on lance le proxy (qui dépend du service `backend`) avec :
    - un conteneur nommé `httpd`
    - dont l'image peut être build via le Dockerfile présent dans `./HTTP_Server`
    - qui map le port 8080 (de la host machine) vers le port 80 (du conteneur)
    - et qui est sur le réseau `my-network`

#### 1.5

Commande :
```
docker tag my-database awakeduck/my-database:1.0
```

Description : 
Cette commande permet de créer un tag, c'est-a-dire "figer" une version d'une image docker, afin de la réutiliser plus tard.
Il est possible de préciser l'image a figer (ici `my-database`) et le nom qu'on veut donner au tag (généralement c'est username/nom_du_projet:version, donc ici on a `awakeduck/my-database:1.0`).

Commande :
```
docker push awakeduck/my-database  
```

Description :
Upload du tag vers un répo Docker (meme principe que git : on commit en local puis on push).

Je peux maintenant utiliser mon image dans un Dockerfile, comme on a utilisé l'image `maven:3.8.6-amazoncorretto-17` par exemple.

### TP2

#### 2.1

testcontainers est un framework permettant de lancer des conteneurs super légers pour run les tests unitaires, et ainsi avoir une vraie base de données ou un vrai WEB-Service qui tourne afin d'avoir des tests plus efficaces. Le conteneur ne vivra que pendant la phase de test.

#### 2.2 & 2.3

**Je regroupe les 2 questions puisque mon action `test.yml` comprend la réponse a la question 2.2 et 2.3**

test.yml (action) :
```
name: CI devops 2023 - Test

on:
  push:
    branches:
      - main
      - develop
  pull_request:

jobs:
  test-backend: 
    runs-on: ubuntu-22.04

    steps:
     #checkout your github code using actions/checkout@v2.5.0
      - uses: actions/checkout@v2.5.0

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          #use oracle distribution and java 17
          distribution: 'oracle'
          java-version: 17

      - name: Build and test with Maven
        run: mvn --file simple-api/pom.xml -B verify sonar:sonar -Dsonar.projectKey=takima_takima -Dsonar.organization=takima -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

Description :
- cette action concerne un `push` ou une `pull request` sur les branches `main` ou `develop` (on retrouve ces informations dans la partie `on` en haut de l'action)
- elle comporte une unique job compose de 3 steps :
    - le job s'appelle `test-backend`
    - il est composé de 3 steps :
        - récupération du code grâce a `actions/checkout@v2.5.0`, un module d'action fourni par GitHub
        - mise en place du JDK pour le build de l'application (on utilise la aussi une action, ici `actions/setup-java@v3`, et on précise la distribution, ici `Oracle`, et la version, ici java `17`)
        - on a récupéré le code et steup le JDK, on peut maintenant build & test notre application avec mvn verify
        - on profite également de cette étape pour demander a Sonar un report sur la qualite de notre code (pour ce faire, on doit s'authentifier avec un token secret, stocké dans les action-secrets sur GitHub, on peut ensuite l'utiliser avec la syntaxe `{{ secrets.NOM_DU_SECRET }}`)

Pour résumer : à chaque pull/push/PR sur develop ou sur main, on vérifie que l'application se build, que les tests passent toujours, et on lance un quality check Sonar.

### TP3

#### 3.1

Inventory :
```
all:
 vars:
   ansible_user: centos
   ansible_ssh_private_key_file: /home/hugo/Desktop/Cours/M2-S2/takima/id_rsa
 children:
   prod:
      hosts: h.deprez.takima.cloud
```

Description :
- `all` : groupe par défaut
- `vars` : déclaration des variables ansible, ici :
    - `ansible_user` : notre user ssh, ici `centos`
    - `ansible_ssh_private_key_file` : le chemin vers la clé SSH
- `children` : les enfants du groupe `all`
- `prod` : nom du groupe enfant, on peut imaginer ici que ca correspond a un environnement de production, mais on pourrait avoir plusieurs enfants du style 'developpement', 'preprod', 'prod' par exemple
- `hosts`: liste des adresses de la prod (ici, `h.deprez.takima.cloud`)

Commande :
```
ansible all -i inventories/setup.yml -m ping
```

Description :
- `ansible all` lance un playbook / des "actions" pour le groupe `all`
- `-i` permet de spécifier l'inventiare
- `-m` permet d'utiliser un module (ici, `ping`, pour vérifier que nos hosts répondent)

Commande :
```
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```

Description :
- meme principe mais cette fois-ci avec le module `setup` pour récupérer des informations sur un host
- `-a` pour les arguments du module, ici `filter=ansible_distribution*` pour récupérer les informations de la distribution du host

#### 3.2

Playbook :
```
- hosts: all
  gather_facts: false
  become: yes
  roles:
    - roles/docker
    - roles/network
    - roles/database
    - roles/app
    - roles/proxy
    - roles/front
```

Description :
- `hosts: all` pour préciser le groupe
- `gather_facts: false` pour ne pas récuperer 80000 informations de l'host
- `become: yes` pour exécuter les commandes en superutilisateur
- `roles` les roles concernés

Chaque role contient des taches a réaliser :
- `roles/docker` doit installer docker sur l'host pour qu'on puisse utiliser les commandes par la suite
- `roles/network` doit setup un network docker
- `roles/database` doit lancer la BD
- `roles/app` doit lancer l'API
- `roles/proxy` doit lancer le proxy
- `roles/front` doit lancer.... le front !

#### 3.3

**Je ne rentre pas dans le details de la tache du role `roles/docker`, car elle etait fournie et que je ne l'ai pas modifiée. Elle installe juste Docker & Python3 en plus de clean quelques package et de mettre a jour les repos**

Network :
```
- name: Create a network
    community.docker.docker_network:
      name: app-network
```

Description :
Création d'un network docker nommé `app-network` grâce au module `community.docker.docker_network`

Database :
```
- name: Run Database
    docker_container:
      name: database
      networks: 
      - name: app-network
      env:
        POSTGRES_DB: "tp_db"
        POSTGRES_USER: "exampleuser"
        POSTGRES_PASSWORD: "examplepassword"
      image: awakeduck/takima_database:latest
      volumes:
      - data_db:/var/lib/postgresql/data
```

Description :
Avec le module `docker_container`, lancement d'un container nommé `database`, faisant partie du réseau `app-network`, avec 3 variables d'environnements : `POSTGRES_DB` (`tp_db`), `POSTGRES_USER` (`exampleuser`), et `POSTGRES_PASSWORD` (`examplepassword`). Le conteneur est construit à partir de l'image push par ma CI sur DockerHub. Enfin, on monte un volume `data_db` vers `/var/lib/postgresql/data`.

App :
```
- name: Run API
    docker_container:
      name: backend
      networks:
      - name: app-network
      image: awakeduck/takima_simple-api:latest
```

Description :
Avec le module `docker_container`, lancement d'un container nommé `backend`, faisant partie du réseau `app-network`. Le conteneur est construit à partir de l'image push par ma CI sur DockerHub.

Proxy :
```
- name: Run HTTPD
    docker_container:
      name: httpd
      networks:
      - name: app-network
      image: awakeduck/takima_http_server:latest
      published_ports:
      - 80:80
```
Avec le module `docker_container`, lancement d'un container nommé `httpd`, faisant partie du réseau `app-network`. Le conteneur est construit à partir de l'image push par ma CI sur DockerHub. Il publie également ses ports : `80` du host vers `80` du conteneur.

Description :

Front :
```
- name: Run Front
    docker_container:
      name: front
      networks:
      - name: app-network
      image: awakeduck/takima_front:latest
```

Description :
Avec le module `docker_container`, lancement d'un container nommé `front`, faisant partie du réseau `app-network`. Le conteneur est construit à partir de l'image push par ma CI sur DockerHub.

skhengui@takima.fr