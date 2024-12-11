# Projet-E3-3MSCOR
FIGAROLI Romain
Groupe : Kadi Soulaymane / Hambourger Tomy / Ahmadi Rateb / Bernard Jonathan

<!-- TABLE OF CONTENTS --> <details> <summary>Table des matières</summary> <ol> <li><a href="#Environnement de travail">Environnement de travail</a></li> <li><a href="#Initialisation des applications">Initialisation des applications</a></li> 
<li><a href="#Optimisation des images">Optimisation des images</a></li> <li><a href="#Configuration des fichiers pour les applications">Configuration des fichiers pour les applications</a></li> <li><a href="#Configuration de l'application en HTML5">Configuration de l'application en HTML5</a></li> <li><a href="#Build des images">Build des images</a></li> <li><a href="#Tests et preuves">Tests et preuves</a></li> <li><a href="#Questions du projet">Questions du projet</a></li> </ol> </details>

## Environnement de travail

J'ai effectué ce projet depuis une VM Ubuntu créer depuis Multipass.

![image](https://github.com/user-attachments/assets/65a8479f-aaf9-4797-86f2-e08b58b934e7)

Et, j'ai crée un dossier "ProjetRF" dans lequel toutes mes applications seront :

![image](https://github.com/user-attachments/assets/fd93e370-5e03-40d3-9bb0-31a0cbd037d5)


Voici l'arborescence du projet :

![image](https://github.com/user-attachments/assets/3fc26596-03bc-4bc2-afb0-7cc706bba072)

Voici un schéma de l'objectif d'infrastructure final : 

![image](https://github.com/user-attachments/assets/f8cb0478-39d0-4f0f-8765-09c8ffbcc9e8)

## Initialisation des applications

J'ai importé un total de 7 applications dont 1 en HTML.
Pour importer les applications j'ai utilisé la commande : 
```bash
git clone https://github.com/app-generator/django-argon-dashboard
git clone https://github.com/app-generator/django-datta-able
git clone https://github.com/app-generator/django-material-dashboard.git
git clone https://github.com/app-generator/django-soft-ui-design.git
git clone https://github.com/app-generator/django-pixel.git
git clone https://github.com/app-generator/django-coreui.git
```

Et, pour l'application en HTML, j'ai utilisé la commande :
```bash
wget https://html5up.net/massively/download
```
![image](https://github.com/user-attachments/assets/39eebb1e-2702-4c35-9d89-c07ab298dd01)

## Optimisation des images

Une fois les applications importées, j'ai optimisé les images en prenant la version la plus légère et en ajoutant les esperluettes pour limiter les commandes.

Exemple d'un Dockerfile utilisé :

```Dockerfile
FROM python:3.10-alpine

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

COPY requirements.txt .
# install python dependencies
RUN pip install --upgrade pip && pip install --no-cache-dir -r requirements.txt

COPY . .

# running migrations
RUN python manage.py migrate

# gunicorn
CMD ["gunicorn", "--config", "gunicorn-cfg.py", "core.wsgi"]
```

## Configuration du docker-compose

J'ai aussi dû supprimer les différents dossier NGINX et les docker-compose des application pour ne faire qu'un seul docker-compose pour toutes mes applications. 
Voici la configuration de mon docker-compose :

```yaml
version: '3.8'
services:
# Application Interne
  django-argon-dashboard:
    container_name: django-argon-dashboard
    restart: always
    ports :
      - "1001:1001"
    build: "./django-argon-dashboard"
    networks:
      - db_network
      - web_network
# Applications Externes
  django-coreui:
    container_name: django-coreui
    restart: always
    build: "./django-coreui"
    networks:
      - db_network
      - web_network
  django-datta-able:
    container_name: django-datta-able
    restart: always
    build: "./django-datta-able"
    networks:
      - db_network
      - web_network
  django-material-dashboard:
    container_name: django-material-dashboard
    restart: always
    build: "./django-material-dashboard"
    networks:
      - db_network
      - web_network
  django-pixel:
    container_name: django-pixel
    restart: always
    build: "./django-pixel"
    networks:
      - db_network
      - web_network
  django-soft-ui-design:
    container_name: django-soft-ui-design
    restart: always
    build: "./django-soft-ui-design"
    networks:
      - db_network
      - web_network
  nginx:
    container_name: nginx
    restart: always
    image: "nginx:stable-alpine-slim"
    ports:
      - "8087:8087"
      - "80:80"
      - "8082:8082"
      - "8083:8083"
      - "8084:8084"
      - "8085:8085"
      - "8086:8086"
    volumes:
      - ./nginx/html:/usr/share/nginx/html
      - ./nginx:/etc/nginx/conf.d
    networks:
      - web_network
    depends_on: 
      - django-coreui
      - django-datta-able
      - django-material-dashboard
      - django-pixel
      - django-soft-ui-design
networks:
  db_network:
    driver: bridge
  web_network:
    driver: bridge
 ```
Avec ce docker-compose, toutes mes applications sont déployer avec un seul fichier aisni que mon serveur NGINX.

## Configuration des fichiers pour les applications

Voici la configuration de mon fichier appseed-app.conf de mon serveur NGINX :
```conf
upstream django-coreui {
    server django-coreui:1002;
}

server {
    listen 8082;
    server_name 192.168.1.31;

    location / {
        proxy_pass http://django-coreui/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
upstream django-datta-able {
    server django-datta-able:1003;
}
server {
    listen 8083;
    server_name 192.168.1.31;

    location / {
        proxy_pass http://django-datta-able/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
upstream django-material-dashboard {
    server django-material-dashboard:1004;
}

server {
    listen 8084;
    server_name 192.168.1.31;

    location / {
        proxy_pass http://django-material-dashboard/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

upstream django-pixel {
    server django-pixel:1005;
}

server {
    listen 8085;
    server_name 192.168.1.31;

    location / {
        proxy_pass http://django-pixel/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

upstream django-soft-ui-design {
    server django-soft-ui-design:1006;
}

server {
    listen 8086;
    server_name 192.168.1.31;

    location / {
        proxy_pass http://django-soft-ui-design/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

server {
    listen 8087;
    server_name 192.168.1.31;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    error_page 404 /404.html;
}
```
Dans ce fichier de configuration, je précise l'adresse IP de ma VM qui est "192.168.1.31" et les ports que mes applications vont utiliser.

Il faut aussi changer les ports dans le fichier "gunicorn-cfg.py" de chaque application (hors HTML5) à la ligne "Bind" : 
```py
bind = '0.0.0.0:1001'
bind = '0.0.0.0:1002'
bind = '0.0.0.0:1003'
bind = '0.0.0.0:1004'
bind = '0.0.0.0:1005'
bind = '0.0.0.0:1006'
```

## Configuration de l'application en HTML5

Pour l'application en HTML5, je mets le dossier "HTML" dans le dossier NGINX. Et je créais un Dockerfile à la racine de ce dossier : 

![image](https://github.com/user-attachments/assets/3d16d0d5-3109-4001-a918-ff3e79c1174f)

## Build des images

Une fois toutes mes applications correctement configurées, je peux lancer le build des images.
Avec la commande suivantes :
```bash
docker compose up -d
```
![image](https://github.com/user-attachments/assets/bf924242-8dda-49ba-b0a5-7378d42ae0e5)

On peut vérifier que nos applications et serveur sont lancé avec la commande :
```bash 
Docker ps
```
![image](https://github.com/user-attachments/assets/73a59dbb-0b55-4342-8d5c-2c60d9db1a87)

## Tests et preuves

Voici mes applications lancées et accessibles : 

Application accessible depuis le port 1001 en interne :

![image](https://github.com/user-attachments/assets/c7260227-410d-4d55-bd09-c45d0435b416)

Mes 6 applications accessibles depuis le reverse proxy :

![image](https://github.com/user-attachments/assets/ad019230-6808-41d5-a398-b14de1f02010)
![image](https://github.com/user-attachments/assets/fa4eb806-7f83-413d-8e61-b328a80f2864)
![image](https://github.com/user-attachments/assets/74e348c5-a413-4dfe-954f-1f075e17bd93)
![image](https://github.com/user-attachments/assets/4bd3adc5-3456-4089-bf78-e137dc10ccaf)
![image](https://github.com/user-attachments/assets/205f4e35-4739-4cb3-879b-ea1c6fa609d3)

Application en HTML5 :

![image](https://github.com/user-attachments/assets/643b5847-ed65-4d73-a4a5-abd1048b6618)

J'ai aussi fait une requête curl afin de valider l'accessibilité de mes applications : 
```bash
curl http://192.168.1.31:8087
```
![image](https://github.com/user-attachments/assets/74964cfb-f3cd-405b-8d5e-068d801a11c3)

## Mises des images sur Docker Hub

Il faut d'abord mettre un tag sur chaque image avec la commande ci-dessous :
```bash
docker tag <IMAGE ID> romf57/projet3mscor:django-datta-able-latest
```

![image](https://github.com/user-attachments/assets/f9f102e8-6b51-4584-a6c8-07412ddda42a)

Il faut ensuite push nos images dans notre repository avec cette commande :
```bash
docker push romf57/projet3mscor:django-datta-able-latest
```

![image](https://github.com/user-attachments/assets/08365bc7-627c-4404-adbe-a13085f17e6d)

On peux voir ci-dessous que nos images sont disponibles dans notre repository sur Docker Hub. Accessible depuis ce lien : https://hub.docker.com/repository/docker/romf57/projet3mscor/general

![image](https://github.com/user-attachments/assets/7952ac7f-1cec-4e32-a2c5-fc61d877ee9a)

## Questions du projet

-    **Qu'est-ce que le devops ?**

Le DevOps, c’est un ensemble de pratiques qui combinent le développement logiciel. Le but principal est d’améliorer la collaboration entre les équipes pour livrer des projets plus rapidement, tout en garantissant la qualité

-    **En quoi le devops vous a aidé dans cette mission ?**

Le DevOps nous a permis de travailler de façon plus organisée et de gagner du temps

-    **Votre chef de projet vous demande de comparer l'utilisation de Docker avec des machines virtuelles.
  Que répondrez-vous ?**

Docker est plus léger que les machines virtuelles. Une VM embarque tout un système d’exploitation, ce qui consomme beaucoup de ressources.

-    **À l'aide de vos connaissances personnelles, citer un ou plusieurs autres outils similaires à docker qui
  pourrait aider votre équipe à livrer et maintenir ce projet client.**

Il existe **Kubernetes** et **PodMan** comme outils similaires à Docker.


