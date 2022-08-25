<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Self Hosting](#self-hosting)
  - [Vaultwarden](#vaultwarden)
    - [Usage](#usage)
      - [Configure Rclone (⚠️ MUST READ ⚠️)](#configure-rclone--must-read-)
        - [Configure and Check](#configure-and-check)
      - [Backup](#backup)
      - [Restore](#restore)
      - [Environment Variables](#environment-variables)
        - [RCLONE_REMOTE_NAME](#rclone_remote_name)
        - [RCLONE_REMOTE_DIR](#rclone_remote_dir)
        - [RCLONE_GLOBAL_FLAG](#rclone_global_flag)
        - [CRON](#cron)
        - [ZIP_ENABLE](#zip_enable)
        - [ZIP_PASSWORD](#zip_password)
        - [ZIP_TYPE](#zip_type)
        - [BACKUP_KEEP_DAYS](#backup_keep_days)
        - [BACKUP_FILE_DATE_SUFFIX](#backup_file_date_suffix)
        - [TIMEZONE](#timezone)
        - [PING_URL](#ping_url)
        - [MAIL_SMTP_ENABLE](#mail_smtp_enable)
        - [MAIL_SMTP_VARIABLES](#mail_smtp_variables)
        - [MAIL_TO](#mail_to)
        - [MAIL_WHEN_SUCCESS](#mail_when_success)
        - [MAIL_WHEN_FAILURE](#mail_when_failure)
        - [DATA_DIR](#data_dir)
        - [DATA_DB](#data_db)
        - [DATA_RSAKEY](#data_rsakey)
        - [DATA_ATTACHMENTS](#data_attachments)
        - [DATA_SENDS](#data_sends)
    - [Use `.env` file](#use-env-file)
    - [Docker Secrets](#docker-secrets)
    - [About Priority](#about-priority)
    - [Mail Test](#mail-test)
    - [Migration](#migration)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Self Hosting

This is a repo for stuff that I host myself locally.

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

#### Environment Variables

> **Note:** All environment variables have default values, you can use the docker image without setting any environment variables.

##### RCLONE_REMOTE_NAME

Rclone remote name, which needs to be consistent with the remote name in the rclone config.

You can view the current remote name with the following command.

```shell
docker run --rm -it \
  --mount type=volume,source=vaultwarden-rclone-data,target=/config/ \
  ttionya/vaultwarden-backup:latest \
  rclone config show

# [BitwardenBackup] <- this
# ...
```

Default: `BitwardenBackup`

##### RCLONE_REMOTE_DIR

Folder for storing backup files in the storage system.

Default: `/BitwardenBackup/`

##### RCLONE_GLOBAL_FLAG

Rclone global flags, see [flags](https://rclone.org/flags/).

**Do not add flags that will change the output, such as `-P`, which will affect the deletion of outdated backup files.**

Default: `''`

##### CRON

Schedule run backup script, based on [`supercronic`](https://github.com/aptible/supercronic). You can test the rules [here](https://crontab.guru/#5_*_*_*_*).

Default: `5 * * * *` (run the script at 5 minute every hour)

##### ZIP_ENABLE

Pack all backup files into a compressed file. When set to `'FALSE'`, each backup file will be uploaded independently.

Default: `TRUE`

##### ZIP_PASSWORD

Password for compressed file. Note that the password will always be used when packing the backup files.

Default: `WHEREISMYPASSWORD?`

##### ZIP_TYPE

Because the `zip` format is less secure, we offer archives in `7z` format for those who seek security.

It should be noted that the password for vaultwarden is encrypted before it is sent to the server. The server does not have plaintext passwords, so the `zip` format is good enough for basic encryption needs.

Default: `zip` (only support `zip` and `7z` format)

##### BACKUP_KEEP_DAYS

Only keep last a few days backup files in the storage system. Set to `0` to keep all backup files.

Default: `0`

##### BACKUP_FILE_DATE_SUFFIX

Each backup file is suffixed by default with `%Y%m%d`. If you back up your vault multiple times a day that suffix is not unique anymore.
This environment variable allows you to append that date (`%Y%m%d${BACKUP_FILE_DATE_SUFFIX}`) suffix in order to create a unique backup name.

Note that only numbers, upper and lower case letters, `-`, `_`, `%` are supported.

Please use the [date man page](https://man7.org/linux/man-pages/man1/date.1.html) for the format notation.

Default: `''`

##### TIMEZONE

You should set the available timezone name.

Here is timezone list at [wikipedia](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

Default: `UTC`

##### PING_URL

Use [healthcheck.io](https://healthchecks.io/) url or similar cron monitoring to perform `GET` requests after a **successful** backup.

##### MAIL_SMTP_ENABLE

The tool uses [heirloom-mailx](https://www.systutorials.com/docs/linux/man/1-heirloom-mailx/) to send mail.

Default: `FALSE`

##### MAIL_SMTP_VARIABLES

Because the configuration for sending emails is too complicated, we allow you to configure it yourself.

**We will set the subject according to the usage scenario, so you should not use the `-s` option.**

When testing, we will add the `-v` option to display detailed information.

```text
# My example:

# For Zoho
-S smtp-use-starttls \
-S smtp=smtp://smtp.zoho.com:587 \
-S smtp-auth=login \
-S smtp-auth-user=<my-email-address> \
-S smtp-auth-password=<my-email-password> \
-S from=<my-email-address>
```

See [here](https://www.systutorials.com/sending-email-from-mailx-command-in-linux-using-gmails-smtp/) for more information.

##### MAIL_TO

Who will receive the notification email.

##### MAIL_WHEN_SUCCESS

Send email when backup is successful.

Default: `TRUE`

##### MAIL_WHEN_FAILURE

Send email when backup fails.

Default: `TRUE`

##### DATA_DIR

The folder where vaultwarden stores its data.

When using `Docker Compose`, you don't need to change it, but when using automatic backup, you need to change it to `/data`.

Default: `/bitwarden/data`

<details>
<summary><strong>※ Other environment variables</strong></summary>

> **You don't need to change these environment variables unless you know what you're doing.**

##### DATA_DB

Set the sqlite database file path.

Default: `${DATA_DIR}/db.sqlite3`

##### DATA_RSAKEY

Set the rsa_key file path.

Default: `${DATA_DIR}/rsa_key`

##### DATA_ATTACHMENTS

Set the attachment folder path.

Default: `${DATA_DIR}/attachments`

##### DATA_SENDS

Set the sends folder path.

Default: `${DATA_DIR}/sends`

</details>

### Use `.env` file

If you prefer to use env file instead of environment variables, you can map the env file containing the environment variables to the `/.env` file in the container.

```shell
docker run -d \
  --mount type=bind,source=/path/to/env,target=/.env \
  ttionya/vaultwarden-backup:latest
```

### Docker Secrets

As an alternative to passing sensitive information via environment variables, `_FILE` may be appended to the previously listed environment variables, causing the initialization script to load the values for those variables from files present in the container. In particular, this can be used to load passwords from Docker secrets stored in `/run/secrets/<secret_name>` files.

```shell
docker run -d \
  -e ZIP_PASSWORD_FILE=/run/secrets/zip-password \
  ttionya/vaultwarden-backup:latest
```

### About Priority

We will use the environment variables first, then the contents of the file ending in `_FILE` as defined by the environment variables, followed by the contents of the file ending in `_FILE` as defined in the `.env` file, and finally the `.env` file values.

### Mail Test

You can use the following command to test the mail sending. Remember to replace your smtp variables.

```shell
docker run --rm -it -e MAIL_SMTP_VARIABLES='<your smtp variables>' ttionya/vaultwarden-backup:latest mail <mail send to>

# Or

docker run --rm -it -e MAIL_SMTP_VARIABLES='<your smtp variables>' -e MAIL_TO='<mail send to>' ttionya/vaultwarden-backup:latest mail
```

### Migration

If you use automatic backups, you just need to replace the image with `ttionya/vaultwarden-backup`. Note the name of your volume.

If you are using `docker-compose`, you need to update `bitwardenrs/server` to `vaultwarden/server` and `ttionya/bitwardenrs-backup` to `ttionya/vaultwarden-backup`.

We recommend re-downloading the `docker-compose.yml` file, replacing your environment variables, and noting the `volumes` section, which you may need to change.