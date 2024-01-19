# API Run UTC H24
## Contenu
- [Projet](#Projet)
- [Dokuwiki](#Dokuwiki)
- [Conduit](#Conduit)
- [Gestionnaire de mdp](#Gestionnaire-de-mots-de-passe)
  
# Projet

Le contenu du projet est disponible ici : https://php.102.picagraine.net/ 

# Dokuwiki

Pour installer un dokuwiki, il faut d'abord télécharger <a href ="https://download.dokuwiki.org/"> la dernière version </a>.
Ensuite, 

# Conduit

Pour la création d'un conduit à l'aide de Docker, on peut s'aider de l'image matrixconduit prééxistante.
```bash
sudo docker image pull docker.io/matrixconduit/matrix-conduit:latest
```
Il faut ensuite modifier la configuration nginx ( /etc/nginx/nginx.conf ) comme suit:
```php
user www-data;
worker_processes  auto;
pid /run/nginx.pid;

error_log  /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;


events {
    worker_connections  1024;
    multi_accept on;
}

http {

    proxy_headers_hash_max_size 1024;
    proxy_headers_hash_bucket_size 128;



    tcp_nopush on;
    types_hash_max_size 2048;
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    access_log stdout;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/access.log;

    gzip on;

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;

}
```
Puis créer un document (ici appelé chat) dans /etc/nginx/sites-available avec les configurations suivantes:

```php
server {
    server_name chat.102.picagraine.net;
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        }

    listen 443 ssl; # managed by Certbot
   ssl_certificate /etc/letsencrypt/live/chat.102.picagraine.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/chat.102.picagraine.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


server {
    if ($host = chat.102.picagraine.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name chat.102.picagraine.net;
    return 404; # managed by Certbot
}
```
**Remarque** : Il faut s'assurer que le site chat.102.picagraine.net est bien en https, pour cela on peut utiliser certbot

Dans le répertoire courant, on crée un document ```docker-compose.yml ```

```php
version: '3'
services:
  registry:
conduit:
    image: matrixconduit/matrix-conduit:latest
    container_name: conduit
    ports:
      - 8080:6167
    volumes:
      - db:/var/lib/matrix-conduit/
    environment:
      CONDUIT_SERVER_NAME: "chat.102.picagraine"
      CONDUIT_DATABASE_BACKEND: "rocksdb"
      CONDUIT_ALLOW_REGISTRATION: true
      CONDUIT_ALLOW_FEDERATION: true
      CONDUIT_MAX_REQUEST_SIZE: 20000000
      CONDUIT_TRUSTED_SERVERS: "[\matrix.org\"]"
      CONDUIT_MAX_CONCURRENT_REQUESTS: "100"
      CONDUIT_LOG: "warn,rocket=off,_=off,sled=off"
```
et on lance le conteneur docker avec la commande: 

```bash
sudo docker run -d --name conduit matrixconduit/matrix-conduit:latest
```
Les noms de domaines certifiés par certbot étant limités, le conduit n'est actuellement pas disponible.

# Gestionnaire de mots de passe
