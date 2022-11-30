# Traefik

[Traefik](https://traefik.io/) is a reverse proxy that can provide TLS termination makes running many
services locally painless.

# Traefik Setup for Shub

1. Install [`mkcert`](https://github.com/FiloSottile/mkcert) on your host OS and create a set of keys for \*.shub.localhost:

```
mkcert "*.shub.localhost"
```

> NOTE: You don't need to make any edits to your local /etc/hosts file: any name with TLD of .localhost should resolve to the loopback address.

2. Copy the _.pem_ output files into the _configuration_ folder next to
   _certs.toml_.

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

## Configure other apps with Traefik

> NOTE: this section refers to your app's _docker-compose.yml_ file, not THIS
> repository's compose file.

1. Add a `networks` section to your _docker-compose.yml_:

```
networks:
  default:
    name: <your service>
  shub:
    external: true
```

This creates a _default_ network that will be attached to your containers by default,
and also lets compose know about the _shub_ network.

2. Add the `shub` network to exposed services.

```
services:
  <your web service>:
    networks:
      - default
      - shub
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

Restart your app's containers (`docker compose restart`). No
need to restart Traefik; it will detect the new labels for your services automatically.
