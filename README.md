# IdP Testing Space

This app hub wraps testing deployments of three open source identity providers: Keycloak, Authentik, Hanko.

## How to use:

1. Open the app Hub at [https://ba-pahle.aet.cit.tum.de](https://ba-pahle.aet.cit.tum.de)
2. Select the IdP provider you want to test the auth flow.
3. Click on 'Login' and select 'TUM-Login'
4. In the official TUM-Login authenticate using your TUM credentials
5. After successful login you are redirected back to the Client App
6. You can now register a passkey using the 'Register Passkey' button
7. Follow the steps of your password manager to save the passkey
8. After successful passkey registration you can log out from the Client and relogin using the passkey

You can check out the Admin Dashboards of Keycloak/Authentik using the 'Keycloak Admin'/'Authentik Admin' buttons in
the corresponding client apps. Admin credentials can be found on the Outline page of my thesis.

<details>
<summary>Proxy Documentation</summary>

# VM Reverse Proxy Setup (Traefik + Per-Project Compose)

This repository provides a base Docker Compose stack for running multiple services behind one endpoint on your VM.

## What this contains

- A root compose stack that runs Traefik as reverse proxy.
- A shared Docker network (`proxy`) for all project stacks.
- A `sources/` directory where each project keeps its own `docker-compose.yml`.
- A hub project in `sources/hub` that serves `/` with quick links.

## Folder structure

```text
.
├── docker-compose.yml
├── traefik/
│   ├── traefik.yml
│   └── dynamic.yml
└── sources/
    └── <project>/
        └── docker-compose.yml
```

## Start Traefik

From the repository root:

```bash
docker compose up -d
```

Traefik will listen on:

- `80` (HTTP)
- `443` (HTTPS)

## TLS from `/foo`

Traefik mounts host folder `/foo` as `/certs` (read-only) and uses:

- `/foo/fullchain.pem`
- `/foo/privkey.pem`

If your filenames differ, change `certFile` and `keyFile` in `traefik/dynamic.yml`.

## Add a project

Create a new folder under `sources/`, for example `sources/app1/`, and add a project-level compose file.

Example routing by subpath `/app1`:

```yaml
services:
  app1:
    image: nginx:alpine
    labels:
      - traefik.enable=true
      - traefik.http.routers.app1.rule=Host(`vm.localhost`) && PathPrefix(`/app1`)
      - traefik.http.routers.app1.entrypoints=websecure
      - traefik.http.routers.app1.tls=true
      - traefik.http.services.app1.loadbalancer.server.port=80
      - traefik.http.middlewares.app1-strip.stripprefix.prefixes=/app1
      - traefik.http.routers.app1.middlewares=app1-strip
    networks:
      - proxy

networks:
  proxy:
    external: true
    name: proxy
```

Then start that project from its own directory:

```bash
cd sources/app1
docker compose up -d
```

## Dashboard

Current dynamic config exposes the dashboard at:

- `http://traefik.localhost`
- `https://traefik.localhost`

(From `traefik/dynamic.yml`.)

## Notes

- Make sure each app uses unique router/service/middleware names in labels.
- Use `StripPrefix` only if your app expects `/` internally.
- This setup uses certificate files from `/foo` and does not use Let's Encrypt.

</details>
