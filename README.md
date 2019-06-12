## Introducción

En este tutorial veremos cómo instalar [docker](https://www.docker.com/), [docker-compose](https://docs.docker.com/compose/) y [docker-machine](https://docs.docker.com/machine/), y cómo usarlos para poner en producción en Digital Ocean una aplicación hecha en Java que usa MySQL.

## Instalación de Docker

### docker
Seguimos las instrucciones que están en [la guía oficial](https://docs.docker.com/install/linux/docker-ce/ubuntu/). Finalizada la instalación, agregaremos a nuestro usuario actual al grupo de Docker, para no tener que usar `sudo` a cada rato:

```
sudo usermod -aG docker $USER
# Luego de esto hay que reiniciar la computadora para que tenga efecto
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

## Uso

### En la máquina propia

Una vez completada la instalación, ya podemos levantar nuestra aplicación: el [libro matriz](https://github.com/ingsw-sarmiento/libro-matriz-digital) Para esto, previamente ya hemos subido a [Docker Hub](https://hub.docker.com/) la imagen de la aplicación, por lo que lo único necesario es seguir estos pasos:

1. Clonar este repositorio.
1. Abrir una terminal dentro de este repositorio.
1. Ejecutar `docker-compose up` y esperar a que descargue todo - puede tardar un raaaato largo la primera vez. :hourglass:

En formato copiar-pegar sería así:

```
git clone https://github.com/ingsw-sarmiento/docker-java-mysql.git
cd docker-java-mysql
docker-compose up
```

Para comprobar si funciona, abrimos un navegador y vamos a http://localhost. Debería aparecer nuestra aplicación. :smiley:

Luego, para salir ejecutamos <kbd>Ctrl</kbd> + <kbd>C</kbd>. Algunos comandos más de `docker-compose`:

* `docker-compose up db`: levanta únicamente un servicio, `db` en este caso. Si tiene dependencias, también las levantará previamente;
* `docker-compose up -d`: igual que `up`, pero devuelve el control de la consola;
* `docker-compose down`: baja todos los contenedores y los borra;
* `docker-compose down -v`: igual que `down`, pero borra también los volúmenes (en este ejemplo, la base de datos y la carpeta de migraciones).

### En Digital Ocean

Primero hay que crearse una cuenta en Digital Ocean y obtener un token, para esto ultimo seguir [este tutorial](https://docs.docker.com/machine/examples/ocean/).

```
# Creamos la máquina, por única vez. Este comando asume que en $DO_TOKEN hay un access token de Digital Ocean y crea la máquina más barata (u$s 5 al mes):
docker-machine create --driver digitalocean --digitalocean-size s-1vcpu-1gb --digitalocean-image ubuntu-18-04-x64 --digitalocean-access-token $DO_TOKEN libro-matriz-digital

# Configura la consola actual para ejecutar los comandos en Digital Ocean
eval $(docker-machine env libro-matriz-digital)

# Levanta el servidor y devuelve el control a la consola
docker-compose up -d

# Nos muestra información sobre las máquinas que tenemos
docker-machine ls

# Nos permite borrar una máquina
docker-machine rm libro-matriz-digital

# Vuelve la consola "a la normalidad": los comandos de aquí en adelante se ejecutarán en nuestra máquina. Otra forma es directamente cerrar la consola actual y abrir una nueva
eval $(docker-machine env -u)
```

## Créditos

Toda la documentación que utilicé para crear este tutorial está linkeada en cada paso. Me inspiré además en las siguientes publicaciones:
* [un tutorial de CD con Heroku, Travis y Docker](https://medium.com/@javierfernandes/continuous-deployment-con-docker-travis-heroku-c24042fb830b), escrito por el colega docente y programador Javier Fernándes;
* https://medium.com/containers-101/using-docker-from-maven-and-maven-from-docker-1494238f1cf6
* http://geekyplatypus.com/packaging-and-serving-your-java-application-with-docker/
* https://www.digitalocean.com/community/tutorials/how-to-provision-and-manage-remote-docker-hosts-with-docker-machine-on-ubuntu-16-04
