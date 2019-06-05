## Introducción

En este tutorial veremos cómo instalar [docker](https://www.docker.com/), [docker-compose](https://docs.docker.com/compose/) y [docker-machine](https://docs.docker.com/machine/), y cómo usarlos para poner en producción en Digital Ocean una aplicación hecha en Java que usa MySQL.

## Instalación de Docker

### docker
Seguimos las instrucciones que están en [la guía oficial](https://docs.docker.com/install/linux/docker-ce/ubuntu/). Finalizada la instalación, agregaremos a nuestro usuario actual al grupo de Docker, para no tener que usar `sudo` a cada rato:

```
sudo usermod -aG docker $USER
```

Si todo funcionó deberíamos ver algo como esto al ejecutar `docker run hello-world`:

```
$ docker run hello-world                                                                                                           

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

### docker-compose

Para este ejemplo utilizaremos `docker-compose`, una herramienta que nos permite conectar varios contenedores de forma sencilla. Para instalarlo seguiremos también [la guía oficial](https://docs.docker.com/compose/install/), que puede resumirse en los siguientes comandos:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Si todo fue bien, deberíamos ver algo como esto:

```
$ docker-compose --version
docker-compose version 1.24.0, build 0aa59064
```

### docker-machine

Una vez que tengamos todo funcionando, necesitaremos una forma de subirlo a nuestro host, en este caso Digital Ocean. Ahí es cuando entra en acción `docker-machine`, utilidad que instalaremos con la [guía oficial](https://docs.docker.com/machine/install-machine/), , que puede resumirse en el siguiente comando:

```
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

Si todo fue bien, deberíamos ver algo como esto:

```
$ docker-machine version
docker-machine version 0.16.0, build 702c267f
```

```
docker-machine create --driver digitalocean --digitalocean-size s-1vcpu-1gb --digitalocean-image ubuntu-18-04-x64 --digitalocean-access-token $DO_TOKEN libro-matriz-digital
eval $(docker-machine env libro-matriz-digital)
docker-compose up -d
docker-machine ip libro-matriz-digital
```

## Créditos

Toda la documentación que utilicé para crear este tutorial está linkeada en cada paso. Me inspiré además en las siguientes publicaciones:
* [un tutorial de CD con Heroku, Travis y Docker](https://medium.com/@javierfernandes/continuous-deployment-con-docker-travis-heroku-c24042fb830b), escrito por el colega docente y programador Javier Fernándes;
* https://medium.com/containers-101/using-docker-from-maven-and-maven-from-docker-1494238f1cf6
* http://geekyplatypus.com/packaging-and-serving-your-java-application-with-docker/
* https://www.digitalocean.com/community/tutorials/how-to-provision-and-manage-remote-docker-hosts-with-docker-machine-on-ubuntu-16-04
