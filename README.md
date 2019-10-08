Aria2 + AriaNg + Filebrowser

English | [简体中文](https://github.com/wahyd4/aria2-ariang-docker/blob/master/README.CN.md)

[![](https://images.microbadger.com/badges/image/wahyd4/aria2-ui.svg)](https://microbadger.com/images/wahyd4/aria2-ui "Get your own image badge on microbadger.com")
[![Docker Pulls](https://img.shields.io/docker/pulls/wahyd4/aria2-ui.svg)](https://hub.docker.com/r/wahyd4/aria2-ui/)
[![Github Build](https://github.com/wahyd4/aria2-ariang-docker/workflows/Docker%20Image%20CI/badge.svg)](https://github.com/wahyd4/aria2-ariang-docker/actions)

- [Features](#features)
- [Recommended versions](#recommended-versions)
- [How to run](#how-to-run)
  - [Quick run](#quick-run)
  - [Full features run](#full-features-run)
  - [Run with docker-compose](#run-with-docker-compose)
  - [Docker-compose example](#docker-compose-example)
  - [Supported Environment Variables](#supported-environment-variables)
  - [Supported Volumes](#supported-volumes)
- [Auto SSL enabling](#auto-ssl-enabling)
- [Build the image by yourself](#build-the-image-by-yourself)
- [Docker Hub](#docker-hub)
- [Usage it in Docker compose](#usage-it-in-docker-compose)
- [FAQ](#faq)

One Docker image for all file downloading, managing, playing and evening sharing platforms!

Besides, it's pretty small and ARM CPU supported which means you can run it on Raspberry Pi🍓.

Last but not least, SSL enabling so easy!

AriaNG
![Screenshot](https://github.com/wahyd4/aria2-ariang-x-docker-compose/raw/master/images/ariang.jpg)

File Browser
![File Browser](https://github.com/wahyd4/aria2-ariang-docker/raw/master/filemanager.png)

## Features

  * Aria2 (SSL support)
  * AriaNg
  * Auto HTTPS （Let's Encrypt）
  * Bind non root user into container, so non root user can also manage downloaded files.
  * Basic Auth
  * File indexing and video playing ([File Browser](https://filebrowser.xyz/))
  * Add support for ARM CPUs, please choose correct [docker image TAG](https://cloud.docker.com/repository/docker/wahyd4/aria2-ui/tags)

## Recommended versions

* wahyd4/aria2-ui:latest
* wahyd4/aria2-ui:arm32
* wahyd4/aria2-ui:arm64

## How to run

### Quick run

```shell
  docker run -d --name aria2-ui -p 80:80 wahyd4/aria2-ui
```

* Aria2: <http://yourip/ui/>
* FileManger: <http://yourip>
* Please use `admin`/`admin` as username and password to login for the first time.

### Full features run

```bash
  docker run -d --name ariang \
  -p 80:80 \
  -p 443:443 \
  -e PUID=1000 \
  -e PGID=1000 \
  -e ENABLE_AUTH=true \
  -e RPC_SECRET=Hello \
  -e DOMAIN=https://example.com \
  -e ARIA2_SSL=false \
  -e ARIA2_USER=user \
  -e ARIA2_PWD=pwd \
  -v /yourdata:/data \
  -v /app/a.db:/app/filebrowser.db \
  -v /yoursslkeys/:/app/conf/key \
  -v <the folder of aria2.conf and aria2.session>:/app/conf \
  wahyd4/aria2-ui
```
### Run with docker-compose

If you wanna get rid of those annoying command line things, just put the following sample content into `docker-compose.yaml`
```yaml
version: "3.5"
services:
  aria2-ui:
    restart: unless-stopped
    image: wahyd4/aria2-ui:latest
    environment:
      - ENABLE_AUTH=true
      - ARIA2_USER=hello
      - ARIA2_PWD=world
      - DOMAIN=toozhao.com
    volumes:
      - ./data:/data
```
Then just run `docker-compose up -d`, that's it!

### Docker-compose example

In order to make the project easier to understand, I created an example [docker-compose.armhf.yaml](/docker-compose.armhf.yaml)

   `PGID=1000`,`PUID=1000` is for RaspberyPi system user pi
   
   when create `/home/pi/data`,cmd we use(you can change pi as what you like,but it must the use name of Raspbian system)
   ```
   mkdir /home/pi/data
   chown pi:pi -R /home/pi/data
   ```
   
  `database.db` corresponds to the `./app/filebrowser.db` file of the Raspbian system.
   how to create `./app/filebrowser.db`
   ```
   mkdir app
   touch app/filebrowser.db
   chown pi:pi app/filebrowser.db
   ```
   Bring up raspberry aria2ng just use cmd:
   ```
   docker-compose -f docker-compose.armhf.yaml up -d
   ```
   How to login
   ```
   username = pi
   password = raspberry
   ```

### Supported Environment Variables

  * ENABLE_AUTH whether to enable Basic auth
  * ARIA2_USER Basic Auth username
  * ARIA2_PWD Basic Auth password
  * PUID Bind Linux UID into container which means you can use non `root` user to manage downloaded files, default UID is `1000`
  * PGID Bind Linux GID into container, default GID is 1000
  * RPC_SECRET The Aria2 RPC secret token
  * DOMAIN The domain you'd like to bind, when domain is a `https://` thing, then auto SSL feature will be enabled


### Supported Volumes
  * `/data` The folder which contains all the files you download.
  * `/app/conf/key` The folder which stored Aria2 SSL `certificate` and `key` file. `Notice`: The certificate file should be named `aria2.crt` and the key file should be named `aria2.key`
  * `/app/conf` The Aria2 configuration and file session folder. Make sure you have `aria2.conf` and `aria2.session` file. For the first time `aria2.session` just need to be a empty file can be appended. You can also user the templates for these two file in the `conf` folder of this project.
  * `/app/filebrowser.db` File Browser settings database, make sure you make a empty file first on your host.

## Auto SSL enabling

Make sure you have add proper `A` record point to the host you running to your domain `DNS` record list, then just add `e` option to bind the `https` domain when you run the image

```bash
docker run -d --name aria2-ui -p 80:80 -p 443:443 -e DOMAIN=https://toozhao.com wahyd4/aria2-ui
```
## Build the image by yourself

```
docker build -t aria2-ui .
```

## Docker Hub

  <https://hub.docker.com/r/wahyd4/aria2-ui/>

## Usage it in Docker compose

  Please refer <https://github.com/wahyd4/aria2-ariang-x-docker-compose>

## FAQ
  1. When you running the docker image with non `80` port or you have HTTPS enabled, you will meet the error says `Aria2 Status Disconnected`, then you will need to go to `AriaNg Settings` -> RPC (at the top of the page) to modify the port value in `Aria2 RPC Address` field. Due to AriaNg store all configurations in local browser, so you need to do this per browser.
  2. If there is no speed at all when you downloading a BitTorrent file, please try to use a popular torrent file first to help the application to cache `DHT` file. Then the speed should get fast and fast, as well as downloading other links.
