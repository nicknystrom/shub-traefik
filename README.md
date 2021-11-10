Traefik Setup for Shub
===

1. Install [`mkcert`](https://github.com/FiloSottile/mkcert) on your host OS and create a set of keys for *.shub.localhost:

```
mkcert "*.shub.localhost"
```

2. Copy the *.pem* output files into the *configuration* folder next to
*certs.toml*.

```
shub-traefik/
  docker-compose.yml
  configuration/
    certs.toml
    _wildcard.shub.localhost-key.pem
    _wildcard.shub.localhost.pem
```

3. Start Traefik

```
docker compose up
```

  Add a `-d` flag to run in the background.

Configure other services with Traefik
---

1. At the bottom of your app's *docker-compose.yml*, set a name for the compose
file's *default network*. This forces Docker Compose to run your app's containers
on the same network as Traefik:

```
networks:
  default:
    name: shub
```

2. Add labels to your services so that Traefik can route to them.

```
services:
  <your service>:
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.<your service>.loadbalancer.server.port=<port your app listens on>"
      - "traefik.http.routers.<your service>.rule=Host(`<your service>.shub.localhost`)"
      - "traefik.http.routers.<your service>.tls=true"
```

Restart your app's containers (`docker compose stop; docker compose start`). No
need to restart Traefik; it will detect the new labels for you services automatically.
