# Self Hosting

![Banner Image](images/banner.png)

This is a repo for stuff that I host myself locally.

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Requirements](#requirements)
- [Localtunnel](#localtunnel)
  - [Localtunnel Usage](#localtunnel-usage)
- [Nginx Proxy Manager](#nginx-proxy-manager)
- [Piping Server](#piping-server)
  - [Transfer](#transfer)
  - [Self-host on Docker](#self-host-on-docker)
- [Portainer](#portainer)
  - [Portainer Templates](#portainer-templates)
- [Vaultwarden](#vaultwarden)
  - [Vaultwarden Usage](#vaultwarden-usage)
    - [Configure Rclone](#configure-rclone)
      - [Configure and Check](#configure-and-check)
    - [Backup](#backup)
    - [Restore](#restore)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Requirements

The requirements to run most of what is in here are as follows:

- [Docker](https://www.docker.com/)
- [Docker-Compose](https://docs.docker.com/compose/install/)
- [Node.js](https://nodejs.org/en/)
- [NPM](https://www.npmjs.com/)

## Localtunnel

[Localtunnel](https://github.com/localtunnel/localtunnel) | [Localtunnel Server](https://github.com/localtunnel/server)

Localtunnel is great if you don't have a static IP address, or if you are behind carrier grade NAT.

### Localtunnel Usage

To quickly set up localtunnel, run the following command:

```bash
npx localtunnel --port 8000
```

Or if you want to install the package:

```bash
npm install -g localtunnel
lt --port 8000
```

- `--subdomain` request a named subdomain on the localtunnel server (default is random characters)
- `--local-host` proxy to a hostname other than localhost

Example:

```bash
lt --port 8000 --subdomain loftwah-sucks --local-host loftwah.local
```

This would expose `loftwah-sucks.localtunnel.me` to `loftwah.local:8000` on your local network.

## Nginx Proxy Manager

[Nginx Proxy Manager](https://nginxproxymanager.com/)

Nginx Proxy Manager is a great way to manage your proxies. Use the Docker Compose template in the `nginx-proxy-manager` directory.

Default credentials are:

| Default User | Default Password |
|--------------|------------------|
| admin@example.com | changeme |

```bash
cd nginx-proxy-manager
docker-compose up -d
```

## Piping Server

[Piping Server](https://github.com/nwtgck/piping-server)

Infinitely transfer between every device over HTTP/HTTPS

### Transfer

Piping Server is simple. You can transfer as follows.

```bash
# Send
echo 'hello, world' | curl -T - https://ppng.io/hello
# Get
curl https://ppng.io/hello > hello.txt
```

Piping Server transfers data to POST /hello or PUT /hello into GET /hello. The path /hello can be anything such as /mypath or /mypath/123/. A sender and receivers who specify the same path can transfer. Both the sender and the recipient can start the transfer first. The first one waits for the other.

You can also use Web UI like [ppng.io](https://ppng.io) on your browser. A more modern UI is found in [piping-ui.org](https://piping-ui.org), which supports E2E encryption.

### Self-host on Docker

Run a Piping Server on [localhost:8080](http://localhost:8080) as follows.

```bash
docker run -p 8080:8080 nwtgck/piping-server
```

Run a server in background and it automatically always restarts.

```bash
docker run -p 8080:8080 -d --restart=always nwtgck/piping-server
```

## Portainer

[Portainer](https://www.portainer.io/) | [Shipwright](https://github.com/SelfhostedPro/Shipwright)

Portainer is a web interface for Docker.

### Portainer Templates

[Selfhosted Pro](https://github.com/SelfhostedPro/selfhosted_templates) | [Technorabilia](https://github.com/technorabilia/portainer-templates/tree/main/lsio/templates) | [Pi-hosted](https://github.com/pi-hosted/pi-hosted)

- [App list](https://github.com/pi-hosted/pi-hosted/blob/master/docs/AppList.md) for Pi-hosted

Portainer templates are great for people who don't want to get their hands dirty in the command line, or don't have the greatest understanding of Docker or Linux. It makes it easier to get it up and running quickly. I'm not sure which templates are the best but here are a couple of repositories that have some good templates.

## Vaultwarden

[Vaultwarden](https://github.com/dani-garcia/vaultwarden) | [Vaultwarden Backup](https://github.com/ttionya/vaultwarden-backup)

Vaultwarden is a self hostable version of Bitwarden. It can be launched via Docker Compose.

### Vaultwarden Usage

> **Important:** We assume you already read the `vaultwarden` [documentation](https://github.com/dani-garcia/vaultwarden/wiki).

#### Configure Rclone

> **For backup, you need to configure Rclone first, otherwise the backup tool will not work.**
> 
> **For restore, it is not necessary.**

We upload the backup files to the storage system by [Rclone](https://rclone.org/).

Visit [GitHub](https://github.com/rclone/rclone) for more storage system tutorials. Different systems get tokens differently.

##### Configure and Check

You can get the token by the following command.

```shell
docker run --rm -it \
  --mount type=volume,source=vaultwarden-rclone-data,target=/config/ \
  ttionya/vaultwarden-backup:latest \
  rclone config
```

**We recommend setting the remote name to `BitwardenBackup`, otherwise you need to specify the environment variable `RCLONE_REMOTE_NAME` as the remote name you set.**

After setting, check the configuration content by the following command.

```shell
docker run --rm -it \
  --mount type=volume,source=vaultwarden-rclone-data,target=/config/ \
  ttionya/vaultwarden-backup:latest \
  rclone config show

# Microsoft Onedrive Example
# [BitwardenBackup]
# type = onedrive
# token = {"access_token":"access token","token_type":"token type","refresh_token":"refresh token","expiry":"expiry time"}
# drive_id = driveid
# drive_type = personal
```

#### Backup

If you are a new user or are rebuilding vaultwarden, it is recommended to use the `docker-compose.yml` from the project.

Download `docker-compose.yml` to you machine, edit environment variables and start it.

You need to go to the directory where the `docker-compose.yml` file is saved.

```shell
# Start
docker-compose up -d

# Stop
docker-compose stop

# Restart
docker-compose restart

# Remove
docker-compose down
```

#### Restore

> **Important:** Restore will overwrite the existing files.

You need to stop the Docker container before the restore.

You also need to download the backup files to your local machine.

Because the host's files are not accessible in the Docker container, you need to map the directory where the backup files that need to be restored are located to the docker container.

**And go to the directory where your backup files to be restored are located.**

If you use the `docker-compose.yml` provided with this project, you can use the following command.

```shell
docker run --rm -it \
  --mount type=volume,source=vaultwarden-data,target=/bitwarden/data/ \
  --mount type=bind,source=$(pwd),target=/bitwarden/restore/ \
  ttionya/vaultwarden-backup:latest restore \
  [OPTIONS]
```
