# TP1

## BASE DE DONNEES

Se mettre dans le dossier "BDD"

Constuction de l'image avec le dockerfile: \
**docker build -t my_postgres_image:14.1-alpine .**

_verification avec la commande **docker images**_

On peut preciser le port, mais le port pour postgres est par defaut le port 5432
Demarrer le conteneur avec : \
**docker run --name my_postgres_container -d my_postgres_image:14.1-alpine**

_verifier avec la commande **docker ps**_

**QUESTION** specifier version a chaque fois ??

_pour supprimer et arreter conteneur_ : \
- **docker rm -f CONTAINER ID**

_pour supprimer image :_ \
- **docker rmi IMAGE ID**

Pour ne pas ecrire en dur les données dans le Dockerfile, on pourrait, une fois l'image créée, lancer cette commande : \
**docker run --name my_postgres_container -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -d my_postgres_image:14.1-alpine**

l'option -e permet de spécifier des variables d'environnement pour une sécurité renforcée.

Se connecter sur postgres avec ceci : \
**docker exec -it my_postgres_container psql -U usr -d db**

Création d'un reseau : \
**docker network create app-network**

On relie le reseau qu'on vient de créer avec notre conteneur : \
**docker run --name my_postgres_container --network app-network -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -e POSTGRES_DB=db -d my_postgres_image:14.1-alpine**

On crée notre administrateur : \
**docker run --name my_adminer --network app-network -p 8080:8080 -d adminer**

on se connecte a localhost:8080 et on remplie les champs suivant :
- System : PostgreSQL
- Server : my_postgres_container (le nom de notre conteneur)
- Username : usr
- Password : pwd
- Database : db

on crée un repertoire pour stocker nos fichiers SQL : \
**mkdir init-scripts-db**

On y met alors nos fichiers SQL.\
On modifie le dockerfile pour que ce dernier copies les scipst sql dans le dossier spécifié en ajoutant cette ligne : \
**COPY init-scripts-db/*.sql /docker-entrypoint-initdb.d/**

On reconstuit alors notre image \
**docker build -t my_postgres_image:14.1-alpine .**

on recré le conteneur : \
**docker run --name my_postgres_container --network app-network -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -e POSTGRES_DB=db -d my_postgres_image:14.1-alpine**

puis on redemarre notre administrateur : \
**docker run -p "8080:8080" --network app-network --name=my_adminer -d adminer**

**QUESTION** le recontruire, ou stop and start suffit ??

Se connecter a localhost:8080 pour regarder l'etat de la bdd.
On peut ajouter des données puis supprimer notre conteneur et le recréer. Tout est réinitialisé.

Pour conserver les données mêmes lorsque le conteneur est détruit, lorsque l'on construit le conteneur, on précise le volume : \
**-v /my/own/datadir:/var/lib/postgresql/data**

Les donnees des dockers sont normalement ephemeres, lorsqu'on le supprime, on suppprime les données avec.\
En attachant le volume, on sauvegarde les données sur le disque de l'hôte, donc notre disque, et non pas dans le conteneur. Cela permet alors d'avoir des données persistantes.

On créera alors le conteneur comme ceci : \
**docker run --name my_postgres_container --network app-network -v /my/own/datadir:/var/lib/postgresql/data -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -e POSTGRES_DB=db -d my_postgres_image:14.1-alpine**

Faire l'ensemble des modifications souhaitées. Lorsque l'on supprime le docker, et qu'on le recrée comme ci-dessus, les données ajoutées sont alors encore présentes.

## API back-end

Se mettre dans le dossier "API"

Compiler le fichier \
**javac Main.java**

On ecrit le dockerfile. \
On construit l'image : \ 
**docker build -t hello_word_image .**

On execute le conteneur : \
**docker run --name hello_word_container --network app-network hello_word_image**

### API simple back-end

On crée l'application springboot sur le web

Une construction en plusieurs étapes permet d'optimiser les Dockerfiles. Ils seront plus faciles à lire et à maintenir. Permet egalement dde contruire l'image finale avec que ce dont on a besoin et préciser quelles étapes construire.s

Dans le dockerfile, nous avons deux phases, la phase de construction et la phase d'execution.

- phase de construction :
On précise l'image utilisée, on défini plusieurs variables d'environnement. On copie les fichiers de construction dans le container. On construit ensuite l'application.

- phase d'execution :
On commence par faire les mêmes étapes que dans la premiere phase. On copie ensuite et demarre l'application.

Se placer dans le dossier simpleapi, et créer l'image :\
**docker build -t api_simple_image .**

On execute le conteneur : \
**docker run --name simple_api_container --network app-network -p 8080:8080 api_simple_image**

### API back-end

Copier le dockerfile dans API backend et se positionner dans ce repertoire.\
On modifie _application.yml_ :\

datasource:\
    url: jdbc:postgresql://my_postgres_container:5432/db \
    username: usr\
    password: pwd\

On relance alors l'image : \
**docker build -t api_backend_image .**

On lance le conteneur postgres en se positionnant dans BDD : \
**docker run --name my_postgres_container --network app-network -v /my/own/datadir:/var/lib/postgresql/data -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd -e POSTGRES_DB=db -d my_postgres_image:14.1-alpine**

On lance alors le backend : \
**docker run --name api_backend_container --network app-network -p 8080:8080 api_backend_image**

Se connecter a _http://localhost:8080/departments/IRC/students_
On peut voir : \
[{"id":1,"firstname":"Eli","lastname":"Copter","department":{"id":1,"name":"IRC"}}]

## HTTP serveur

On commence par ecrire un fichier basique en html.
On crée un dockerfile et on crée l'image :\
**docker build -t my_http_image .**

On lance le container avec : \
**docker run --name my_http_container --network app-network -p 8080:8080 my_http_image**

On a alors le contenu de notre html sur localhost
![alt text](image.png)

### Configuration

On va copier le fichier comme ceci : \
**docker cp my_http_container:/usr/local/apache2/conf/httpd.conf ./httpd.conf**

Puis on l'ajoute a notre dockerfile : \
**COPY httpd.conf /usr/local/apache2/conf/httpd.conf**

On relance tout : \

**docker build -t my_new_http_image .**\
**docker run --name my_new_http_container -d -p 80:80 my_new_http_image**

### Proxy inverse

on ajoute les config au fichier httpd.conf, en remplacant YOUR_BACKEND_LINK par localhost.

On s'assure de lancer les containers de la bdd, du backend, de http 

### Docker compose

On complete le fichier docker-compose.yml

On fait ensuite la commande suivante : \
**docker-compose up**

Pour arrêter : \
**docker-compose down**

application.yml, url du database contienne le nom du container de postgres. On peut fixer dans docker-compose le nom des containers.\
Dans le docker-compose, on peut aussi mettre les variables des bdd comme suit : POSTGRES_DB: ${DB_NAME:db} pour que ça prenne la variable d'environnement fixée ou bien db.

## Publish

Une fois le compte créé, entrer sur le terminal : \
**docker login**

Puis tager les images : \
**docker tag my_new_http_image juuug/my_new_http_image:2.4**\
**docker tag api_backend_image juuug/api_backend_image:3.8.6-amazoncorretto-17** \
**docker tag my_postgres_image:14.1-alpine juuug/my_postgres_image:14.1-alpine**

Enfin, pousser les images sur le dépôt, pour pouvoir les réutiliser plus tard : \
**docker push juuug/my_new_http_image:2.4**
**docker push juuug/api_backend_image:3.8.6-amazoncorretto-17**
**docker push juuug/my_postgres_image:14.1-alpine**

On a alors sur docker hub nos trois images
![alt text](image-1.png)