# traefik-docker

Traefik docker-compose stack for local development usage with HTTPS

## Set up

First, you need to install mkcert to generate your local certificates.
See [mkcert#Installation](https://github.com/FiloSottile/mkcert#installation)

Then, install the mkcert root CA
```sh
$ mkcert -install
```

Generates certificates for your local domain
```sh
$ mkcert -key-file certs/key.pem -cert-file certs/cert.pem domain.local \*.domain.local
```

Create traefik docker network
```sh
$ docker network create traefik
```

Create and start the docker-compose stack
```sh
$ docker-compose up -d
```

Your local traefik is now up and ready! You can access to dashboard at http://localhost:8080

## Plug a container to traefik

In order to plug a container to traefik reverse proxy, you need to use docker labels.
There is an example with a whoami docker-compose stack.

```yaml
version: '3'

services:
  whoami:
    image: 'containous/whoami'
    networks: 
      - traefik
    labels: 
      # expose container to traefik
      - traefik.enable=true
      # set up http host
      - traefik.http.routers.whoami-http.rule=Host(`whoami.domain.local`)
      - traefik.http.routers.whoami-http.entrypoints=http
      # set up middleware to redirect http to https
      - traefik.http.routers.whoami-http.middlewares=redirect-to-https@file
      # set up https host
      - traefik.http.routers.whoami-https.rule=Host(`whoami.domain.local`)
      - traefik.http.routers.whoami-https.entrypoints=https
      - traefik.http.routers.whoami-https.tls=true

networks:
  traefik:
    name: traefik
    external: true
```

Don't forget to add your local domain into your /etc/hosts file!  
Now, you're be able to access whoami at https://whoami.domain.local under https!
