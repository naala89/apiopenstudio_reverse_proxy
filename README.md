ApiOpenStudio Reverse Proxy
=============================

This docker setup is designed for users of ApiOpenStudio on a local machine or a single server, and allows full `https`
traffic through a [Traefik][traefik_docs] reverse proxy for Core and Admin. It is suitable for local development and
small-medium projects in production. It removes the need to deploy Admin and the Admin GUI on separate servers. It uses
the [ApiOpenStudio][api_docker_image], [ApiOpenStudio Admin][admin_docker_image], [Traefik][traefik_docker_image]
and [MariaDB][mariadb_docker_image] docker images.

Installation
============

Clone this repository:

```bash
git clone git@gitlab.com:apiopenstudio/apiopenstudio_reverse_proxy.git
```

SSL Keys
--------

### Local environment

Install `mkcert`, for documentation, see [mkcert][mkcert].

Generate new keys using the following bash commands:

```bash
cd apiopenstudio_reverse_proxy
mkcert -cert-file certs/localhost.crt -key-file certs/localhost.key "*.docker.localhost"
cp "$(mkcert -CAROOT)/rootCA.pem" certs/ca.crt
```

Add the following to your `/etc/hosts` file:

```bash
127.0.0.1 docker.localhost
```

See [mkcert guide][mkcert_guide]

### Production environment

#### .env

Save your certificate key and certificate somewhere safe on your server, and update your `.env` to point to the
directory containing the certificates, e.g.:

```bash
SSL_CERT_DIR=/path/to/cert/dir
```

#### config/dynamic.yml

Update the following entries to point to the correct domains:

* `http.routers.traefik.tls.domains.*.main`
* `http.routers.traefik.tls.domains.*.sans`

Update the following entries to point to the correct filenames (the path to the files should remain `/etc/certs/`):

* `tls.certificates.*.certFile`
* `tls.certificates.*.keyFile`

Configuration
-------------

Copy the example config files:

```bash
cp example.env .env
cp example.settings.api.yml settings.api.yml
cp example.settings.admin.yml settings.admin.yml
sudo chmod 660 .env settings.api.yml settings.admin.yml
sudo chgrp 33 settings.api.yml settings.admin.yml
```

### .env

Update the values in the Database section:

- `MYSQL_ROOT_PASSWORD`
- `MYSQL_DATABASE`
- `MYSQL_USER`
- `MYSQL_PASSWORD`

Update the values in the URLs:

- `API_URL` (without the `https://` prefix)
- `ADMIN_URL` (without the `https://` prefix)

Ensure that you have the correct image tags in:

- `API_IMAGE_TAG`
- `ADMIN_IMAGE_TAG`

### settings.api.yml

Update the following values:

- `db.root_password` (same value as `MYSQL_ROOT_PASSWORD` in `.env`)
- `db.username` (same value as `MYSQL_USER` in `.env`)
- `db.password` (same value as `MYSQL_PASSWORD` in `.env`)
- `api.url` (same value as `API_URL` in `.env`)
- `api.jwt_issuer` (same as `api.url`)
- `api.jwt_permitted_for` (same as `api.url`)

### settings.admin.yml

Update the following values:

- `admin.url` (same value as `ADMIN_URL` in `.env`)

Spin up the docker containers
=============================

Create the docker network and start the images

```bash
docker network create api_network
docker compose up -d
```

Install the systems
===================

### Install ApiOpenStudio Core

```bash
docker exec -it apiopenstudio-api bash
./install.sh
exit
````

### Install ApiOpenStudio Admin

```bash
docker exec -it apiopenstudio-admin bash
./install.sh
exit
```

Useful URLs
===========

Traefik admin page is available on `<API_URL>:8080`

Support
=======

For any bugs or issues found, raise a ticket at the [repository issue tracker][bug_tracker]

Contributing
============

This project is open to contributions.

To contribute:

1. Create a ticket, or work on an existing ticket.
1. Fork the main repository into your own repo.
1. Checkout the fork.
1. Set a new `upstream` remote, pointing to the main repository.
1. Create a feature branch off `develop`.
1. Make updates.
1. Push the changes to your `origin` (fork).
1. Create a merge request to the `upstream/develop` branch.

Clone your fork

```bash
git clone git@gitlab.com:<username>/<repo_name>.git
```

Set upstream

```bash
git remote add upstream https://gitlab.com/apiopenstudio/apiopenstudio_reverse_proxy
git fetch upstream
```

Checkout your feature branch

```bash
git checkout develop
git checkout -b feature/<ticket_id>-ticket-description
git push origin feature/<ticket_id>-ticket-description
```

Code and push changes. After you've finished creating and testing your changes:

```bash
git commit -a -m "#<ticket_number> - short description."
git push origin feature/<ticket_id>-ticket-description
```

Create a merge request. In your repository page in a browser:

* Click on `Merge requests` in the LHS menu.
* Click on `New merge request` button at the top of the page.
* Ensure the `Source branch` is the feature branch on your repository.
* Ensure the target branch is `apiopenstudio/apiopenstudio_docker_dev`
  `develop` branch.
* Click on `Compare branches and continue`.

Follow the rest of the page prompts. If your merge request is accepted, it will
be merged. If there are any updates required, you will be notified through the
issue ticket.

Authors and acknowledgment
==========================

Many thanks to the original developer and Traefik:

* [laughing_man77][laughing_man77]
* [Traefik][traefik_docs]

License
=======

This is available under MIT License.

Links
=====

* [Traefik documentation][traefik_docs]
* [mkcert guide][mkcert_guide]
* [mkcert documentation][mkcert]

[api_docker_image]: https://hub.docker.com/r/naala89/apiopenstudio

[admin_docker_image]: https://hub.docker.com/r/naala89/apiopenstudio_admin

[mariadb_docker_image]: https://hub.docker.com/_/mariadb

[traefik_docker_image]: https://hub.docker.com/_/traefik

[traefik_docs]: https://doc.traefik.io/traefik/

[mkcert_guide]: https://github.com/FiloSottile/mkcert

[mkcert]: https://github.com/FiloSottile/mkcert

[bug_tracker]: https://gitlab.com/apiopenstudio/apiopenstudio_reverse_proxy/-/issues

[laughing_man77]: https://gitlab.com/laughing_man77
