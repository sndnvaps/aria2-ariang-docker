Aria2 + AriaNg

[English](https://github.com/wahyd4/aria2-ariang-docker/blob/master/README.md) | 简体中文

[![](https://images.microbadger.com/badges/image/wahyd4/aria2-ui.svg)](https://microbadger.com/images/wahyd4/aria2-ui "Get your own image badge on microbadger.com")
[![Docker Pulls](https://img.shields.io/docker/pulls/wahyd4/aria2-ui.svg)](https://hub.docker.com/r/wahyd4/aria2-ui/)
[![Github Build](https://github.com/wahyd4/aria2-ariang-docker/workflows/Docker%20Image%20CI/badge.svg)](https://github.com/wahyd4/aria2-ariang-docker/actions)

本镜像包含 Aria2、AriaNg 和File Manager，主要方便那些用户期望只运行一个镜像就能实现图形化下载文件和在线播放文件。（类似离线下载的功能），只使用一个 Docker 镜像也方便用户在群晖NAS 中运行本程序。

- [功能特性](#功能特性)
- [推荐使用的docker image tag](#推荐使用的docker-image-tag)
- [安装于运行](#安装于运行)
  - [快速运行](#快速运行)
  - [开启所有功能](#开启所有功能)
  - [使用docker-compose 运行](#使用docker-compose-运行)
  - [支持的 Docker 环境变量](#支持的-docker-环境变量)
  - [支持的 Docker volume 属性](#支持的-docker-volume-属性)
- [自动 SSL](#自动-ssl)
- [自行构建镜像](#自行构建镜像)
- [Docker Hub](#docker-hub)
- [使用 Docker compose 来运行](#使用-docker-compose-来运行)
- [常见问题](#常见问题)

AriaNG
![Screenshot](https://github.com/wahyd4/aria2-ariang-x-docker-compose/raw/master/images/ariang.jpg)

File Browser
![File Browser](https://github.com/wahyd4/aria2-ariang-docker/raw/master/filemanager.png)

## 功能特性

  * Aria2 (SSL 支持)
  * AriaNg 通过 UI 来操作，下载文件
  * 自动 HTTPS （Let's Encrypt）
  * 支持绑定自定义用户ID，可以主机上的非`root`用户，也可以管理下载的文件
  * Basic Auth 用户认证
  * 文件管理和视频播放 ([File Browser](https://filebrowser.xyz/)，注意默认情况下，只能访问和管理 `/data` 目录下的文件)
  * 支持ARM CPU 架构，因此可以在树莓派中运行，请下载对应的[ARM TAG](https://cloud.docker.com/repository/docker/wahyd4/aria2-ui/tags) 版本, `arm32`或`arm64`

## 推荐使用的docker image tag

* wahyd4/aria2-ui:latest
* wahyd4/aria2-ui:arm32
* wahyd4/aria2-ui:arm64

## 安装于运行

### 快速运行

```shell
  docker run -d --name aria2-ui -p 80:80 wahyd4/aria2-ui
```

* Aria2: <http://yourip/ui/>
* FileManger: <http://yourip>
* 请使用 admin/admin 进行登录
### 开启所有功能
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
### 使用docker-compose 运行

如果你不想记住那些命令行，你也可以使用docker-compose来将配置放在`docker-compose.yaml`文件中
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
然后使用 `docker-compose up -d` 运行即可

为了能让例子更具体，我建立了一个实例[docker-compose.armhf.yaml](/docker-compose.armhf.yaml)

   其中`PGID=1000`,`PUID=1000`指定了用户组和用户ID，都为1000，这个是在树莓派系统里面，用户pi的属性
   
   在建立`/home/pi/data`的时候，需要用到如下命令,其中的用户pi,可以是你自己创建的其它用户名（例如abc)
   ```
   mkdir /home/pi/data
   chown pi:pi -R /home/pi/data
   ```
   `database.db`这个是文件管理器需要用到的数据库，用于存放数据，对应当前目录下面的`./app/filebrowser.db`
   创建./app/filebrowser.db方法比较简单，直接一条touch命令就可以了
   ```
   mkdir app
   touch app/filebrowser.db
   chown pi:pi app/filebrowser.db
   ```
   启动树莓派的实例，需要用到如下命令
   ```
   docker-compose -f docker-compose.armhf.yaml up -d
   ```
   登陆树莓派Aria2的用户为`pi`,密码`raspberry`
   

### 支持的 Docker 环境变量

  * ENABLE_AUTH 启用 Basic auth(网页简单认证) 用户认证
  * ARIA2_USER Basic Auth 用户认证用户名
  * ARIA2_PWD Basic Auth 密码
  * PUID 需要绑定主机的Linux用户ID，可以通过`cat /etc/passwd` 查看用户列表， 默认UID 是`1000`
  * PGID 需要绑定的主机的Linux 用户组ID，默认GID 是`1000`
  * RPC_SECRET Aria2 RPC 加密 token
  * DOMAIN 绑定的域名, 当绑定的域名为`HTTPS`时，即为启用`HTTPS`， 例： `DOMAIN=https://toozhao.com`


### 支持的 Docker volume 属性
  * `/data` 用来放置所有下载的文件的目录
  * `/app/conf/key` 用户来放置 Aria2 SSL `certificate`证书和 `key` 文件. `注意`: 证书的名字必须是 `aria2.crt`， Key 文件的名字必须是 `aria2.key`
  * `/app/conf` 该目录下可以放置你的自定义`aria2.conf`配置文件，`aria2.session`，且必须包含这两个文件。第一次使用`aria2.session`时，创建一个空文件即可，该文件会包含aria2当前的下载列表，这样即使容器被销毁也不用担心文件列表丢失了。你也可以直接拷贝当前项目下`conf`目录中的两个文件并使用。
  * `/app/filebrowser.db` File Browser 的内嵌数据库，升级Docker 镜像也不用担心之前的设置丢失。请确保在宿主机先创建一个空文件再使用。

## 自动 SSL

请在绑定域名前，设置`DNS`的一条`A`记录，将运行docker的主机IP绑定到该域名。然后你仅仅需要在运行时添加`e`设置即可。

```bash
docker run -d --name aria2-ui -p 80:80 -p 443:443 -e DOMAIN=https://toozhao.com wahyd4/aria2-ui
```

## 自行构建镜像

```
docker build -t aria2-ui .
```

## Docker Hub

  <https://hub.docker.com/r/wahyd4/aria2-ui/>

## 使用 Docker compose 来运行

  请参考 <https://github.com/wahyd4/aria2-ariang-x-docker-compose>

## 常见问题
  1.当你以非其他`80` 端口或以启用了HTTPS`443`端口运行程序时，会出现`Aria2 状态 未连接`的错误，这是因为在最新版本里面，我们去掉aria2的独立6800端口，转而使用和网站同一个端口。因为你需要渠道 `AriaNg设置` -> 页面顶端的 `RPC`页面，将你的Aria2 RPC 地址中的端口从`80`改成你使用的正确端口。因为 AriaNg 仅仅将设置保存在浏览器中，因为当你使用不同的浏览器，或者将浏览器清除缓存后，你都需要重新设置一次。
  1. 下载的BT或者磁力完全没有速度怎么办？ 建议先下载一个热门的BT种子文件，而不是磁力链接。这样可以帮助缓存DHT文件，渐渐地，速度就会起来了。比如试试下载树莓派操作系统的BT种子？[前往下载](https://www.raspberrypi.org/downloads/raspbian/)
