
при наличии интернета образ можно загрузить и запустить из репозитария докера:

docker run -d --restart=always --network=host --name=emz-mvs emz13/debian-xfcs-vnc-mvs

#Сборка:

git clone https://github.com/emz13666/docker-headless-vnc-container

создаем ярлык для MVS:
echo "[Desktop Entry]" > ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Version=1.0" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Type=Application" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Name=HIKROBOT-MVS" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Comment=" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Exec=/opt/MVS/bin/MVS.sh" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Icon=camera-photo" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Path=/opt/MVS/" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "Terminal=false" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop
echo "StartupNotify=false" >> ~/docker-headless-vnc-container/src/common/xfce/Desktop/HIKROBOT-MVS.desktop

cp ~/MVS-3.0.1_x86_64_20241128.deb ~/docker-headless-vnc-container/src/debian/install/
rm ~/docker-headless-vnc-container/src/common/xfce/Desktop/chromium-browser.desktop
rm ~/docker-headless-vnc-container/src/common/xfce/Desktop/firefox.desktop
cd ~/docker-headless-vnc-container/
cp Dockerfile.debian-xfce-vnc Dockerfile
nano Dockerfile
меняем пароль в переменной VNC_PW=
Вставляем строку после установки UI, установку браузеров комментируем:
RUN dpkg -i $INST_SCRIPTS/MVS-3.0.1_x86_64_20241128.deb
или вот эти 2 для не-debian:
RUN tar -xzvf $INST_SCRIPTS/MVS-3.0.1_x86_64_20241128.tar.gz
RUN $INST_SCRIPTS/MVS-3.0.1_x86_64_20241128/setup.sh


Вставляем строчки в конец группы Install xfce UI
RUN chown 1000 $HOME/Desktop/HIKROBOT-MVS.desktop
RUN chmod 777 $HOME/Desktop/HIKROBOT-MVS.desktop

Вставляем перед USER 1000:
###Cleaning $INST_SCRIPTS
RUN rm -rf $INST_SCRIPTS

Сохраняем файл, выходим.
Создаем образ:
sudo docker build -t emz-mvs-debian-xfce .
При возникновении ошибки типа E: Release file for ... is not valid yet ... выставляем часы и пробуем создавать ещё раз:
sudo hwclock --hctosys 

Первый запуск:
docker run -d --restart=always --network=host --name=emz-mvs emz-mvs-debian-xfce
Второй и последующие запуски (ну вообще он сам должен перезапускаться, но если остановили то да.):
docker start emz-mvs
Остановка:
docker stop emz-mvs

После внесения изменений образ можно закоммитить из контейнера
docker commit emz-mvs emz-mvs-debian-xfce1

Сохранить образ
docker save -o <output-file.tar> <image-name>
или через portrainer

Загрузить образ
docker load < image.tar

Подключиться к работающему контейнеру:
docker exec -it emz-mvs bash
Подключиться к работающему контейнеру под root-ом:
docker exec -it -u 0 emz-mvs bash


# Docker container images with "headless" VNC session

This repository contains a collection of Docker images with headless VNC environments.

Each Docker image is installed with the following components:

* Desktop environment [**Xfce4**](http://www.xfce.org) 
* VNC-Server (default VNC port `5901`)
* [**noVNC**](https://github.com/novnc/noVNC) - HTML5 VNC client (default http port `6901`)
* Browsers:
  * Mozilla Firefox
  * Chromium

![Docker VNC Desktop access via HTML page](.pics/vnc_container_view.png)

## Build Status
[![Build Status MASTER](https://github.com/ConSol/docker-headless-vnc-container/actions/workflows/nightly.yml/badge.svg)](https://github.com/ConSol/docker-headless-vnc-container/actions/workflows/nightly.yml)

## Current provided OS & UI sessions:
* `consol/debian-xfce-vnc`: __Debian 11 with `Xfce4` UI session__

## OpenShift / Kubernetes

It's also possible to run the images in container orchestration platforms like [Kubernetes](https://kubernetes.io) or [OpenShift](https://openshift.io/). For more information how to deploy containers in the cluster, take a look at:

* [Kubernetes usage of "headless" VNC Docker images](./kubernetes/README.md)
* [OpenShift usage of "headless" VNC Docker images](./openshift/README.md)

## Usage
Usage is **similar** for all provided images, e.g. for `consol/rocky-xfce-vnc`:

- Print out help page:

      docker run emz13/debian-xfcs-vnc-mvs --help

- Run command with mapping to local port `5901` (vnc protocol) and `6901` (vnc web access):

      docker run -d -p 5901:5901 -p 6901:6901 emz13/debian-xfcs-vnc-mvs

- Change the default user and group within a container to your own with adding `--user $(id -u):$(id -g)`:

      docker run -d -p 5901:5901 -p 6901:6901 --user $(id -u):$(id -g) emz13/debian-xfcs-vnc-mvs

- If you want to get into the container use interactive mode `-it` and `bash`

      docker run -it -p 5901:5901 -p 6901:6901 emz13/debian-xfcs-vnc-mvs bash

- Build an image from scratch:

      docker build -t image-name .

# Connect & Control
If the container is started like mentioned above, connect via one of these options:

* connect via __VNC viewer `localhost:5901`__, default password: `vncpassword`
* connect via __noVNC HTML5 full client__: [`http://localhost:6901/vnc.html`](http://localhost:6901/vnc.html), default password: `vncpassword`
* connect via __noVNC HTML5 lite client__: [`http://localhost:6901/?password=vncpassword`](http://localhost:6901/?password=vncpassword)


## Hints

### 1) Extend a Image with your own software
Since version `1.1.0` all images run as non-root user per default, so if you want to extend the image and install software, you have to switch back to the `root` user:

```bash
## Custom Dockerfile
FROM consol/rocky-xfce-vnc
ENV REFRESHED_AT 2022-10-12

# Switch to root user to install additional software
USER 0

## Install a gedit
RUN yum install -y gedit \
    && yum clean all

## switch back to default user
USER 1000
```

### 2) Change User of running VNC Container

Per default, since version `1.3.0` all container processes will be executed with user id `1000`. You can change the user id as follows:

#### 2.1) Using root (user id `0`)
Add the `--user` flag to your docker run command:

    docker run -it --user 0 -p 6911:6901 consol/rocky-xfce-vnc

#### 2.2) Using user and group id of host system
Add the `--user` flag to your docker run command:

    docker run -it -p 6911:6901 --user $(id -u):$(id -g) consol/rocky-xfce-vnc

### 3) Override VNC environment variables
The following VNC environment variables can be overwritten at the `docker run` phase to customize your desktop environment inside the container:
* `VNC_COL_DEPTH`, default: `24`
* `VNC_RESOLUTION`, default: `1280x1024`
* `VNC_PW`, default: `my-pw`
* `VNC_PASSWORDLESS`, default: `<not set>`

#### 3.1) Example: Override the VNC password
Simply overwrite the value of the environment variable `VNC_PW`. For example in
the docker run command:

    docker run -it -p 5901:5901 -p 6901:6901 -e VNC_PW=my-pw image-name

#### 3.2) Example: Override the VNC resolution
Simply overwrite the value of the environment variable `VNC_RESOLUTION`. For example in
the docker run command:

    docker run -it -p 5901:5901 -p 6901:6901 -e VNC_RESOLUTION=800x600 image-name

#### 3.3) Example: Start passwordless
Set `VNC_PASSWORDLESS` to `true` to disable the VNC password.
It is highly recommended that you put some kind of authorization mechanism
before this. For example in the docker run command:

    docker run -it -p 5901:5901 -p 6901:6901 -e VNC_PASSWORDLESS=true image-name
### 4) View only VNC
Since version `1.2.0` it's possible to prevent unwanted control via VNC. Therefore you can set the environment variable `VNC_VIEW_ONLY=true`. If set, the startup script will create a random password for the control connection and use the value of `VNC_PW` for view only connection over the VNC connection.

     docker run -it -p 5901:5901 -p 6901:6901 -e VNC_VIEW_ONLY=true image-name

### 5) Known Issues

#### 5.1) Chromium crashes with high VNC_RESOLUTION ([#53](https://github.com/ConSol/docker-headless-vnc-container/issues/53))
If you open some graphic/work intensive websites in the Docker container (especially with high resolutions e.g. `1920x1080`) it can happen that Chromium crashes without any specific reason. The problem there is the too small `/dev/shm` size in the container. Currently there is no other way, as define this size on startup via `--shm-size` option, see [#53 - Solution](https://github.com/ConSol/docker-headless-vnc-container/issues/53#issuecomment-347265977):

    docker run --shm-size=256m -it -p 6901:6901 -e VNC_RESOLUTION=1920x1080 consol/rocky-xfce-vnc chromium-browser http://map.norsecorp.com/

Thx @raghavkarol for the hint!

## How to release
See **[how-to-release.md](./how-to-release.md)**

## Contributors

At this point we want to thank all contributors, which helped to move this great project by submitting code, writing documentation, or adapting other tools to play well together with the docker headless container.

* [Sven Nierlein](https://github.com/sni)
* [Tobias Schneck](https://github.com/toschneck)
* [Robert Bohne](https://github.com/rbo) - IceWM images
* [hsiaoyi0504](https://github.com/hsiaoyi0504) - PR [#66](https://github.com/ConSol/docker-headless-vnc-container/pull/66)
* [dmhumph](https://github.com/dmhumph) - PR [#44](https://github.com/ConSol/docker-headless-vnc-container/issue/44)
* [Simon Hofmann](https://github.com/s1hofmann)

## Changelog

The current changelog is provided here: **[changelog.md](./changelog.md)**
