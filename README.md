# Self Hosting

This is a repo for stuff that I host myself locally.

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Requirements](#requirements)
- [Localtunnel](#localtunnel)
  - [Localtunnel Usage](#localtunnel-usage)
- [Vaultwarden](#vaultwarden)
  - [Vaultwarden Usage](#vaultwarden-usage)
    - [Configure Rclone (⚠️ MUST READ ⚠️)](#configure-rclone--must-read-)
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

## Vaultwarden

[Vaultwarden](https://github.com/dani-garcia/vaultwarden) | [Vaultwarden Backup](https://github.com/ttionya/vaultwarden-backup)

Vaultwarden is a self hostable version of Bitwarden. It can be launched via Docker Compose.

### Vaultwarden Usage

> **Important:** We assume you already read the `vaultwarden` [documentation](https://github.com/dani-garcia/vaultwarden/wiki).

#### Configure Rclone (⚠️ MUST READ ⚠️)

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
