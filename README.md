# About
This is the docker compose setup for Traefik as reverse proxy in production. It handles automatic certificates
using Lets Encrypt and will redirect all incoming HTTP requests to HTTPS. Docker containers can be registered to
receive traffic by labeling them in their own `docker-compose.production.yml`.

# Production setup
1. Copy `.env.example` to `.env` and fill it if not already done.
2. Create docker context for production if not already existent:

   `docker context create production --docker "host=ssh://<username>@<hostname>"`
3. Change docker context to production system:

   `docker context use production`
4. Create an external network if not already existent:

   `docker network create traefik-network`
5. Start docker compose setup:

   `docker compose -f docker-compose.production.yml up -d`
6. When the docker setup is up an running Traefik will automatically connect new and existing Docker containers
   with Traefik labels as shown in the example below.

Make sure every app has a unique name (e.g. `my-app-php`) and host (e.g. `www.example.com`) in its
`docker-compose.production.yml` file to allow Traefik to identify the correct routers and services etc.:
```
version: '3'
services:
  php:
    # More app specific container settings here ...
    labels:
      - 'traefik.enable=true'
      - 'traefik.http.routers.my-app-php.rule=Host(`www.example.com`)'
      - 'traefik.http.routers.my-app-php.entrypoints=websecure'
      - 'traefik.http.routers.my-app-php.tls.certresolver=myresolver'
      - 'traefik.http.services.my-app-php.loadbalancer.server.port=80'
    networks:
      - traefik-network
      
networks:
  traefik-network:
    external: true
```

If you want to add a new docker compose app and use Traefik for certificate handling and traffic redirection you
simply have to define the labels and networks in the projects own `docker-compose.production.yml` file as shown
in the example above and adapt is slightly. When the app is started on the production machine where Traefik is
already running Traefik will automatically detect the relevant docker container, create a certificate using
Lets Encrypt and start redirecting traffic to the newly deployed app. For this to work make sure to set up
corresponding DNS records so that the Lets Encrypt certificates can automatically obtained.

# Staging Setup
If you first want to test this setup without having Traefik aquire real certificates from Lets Encrypt
you can use the `docker-compose.yml` file where the Traefik dashboard and debug logs are enabled and the
staging environment for Lets Encrypt is used.