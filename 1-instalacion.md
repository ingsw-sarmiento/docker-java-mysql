## Instalación de `docker`, `docker-compose` y `docker-machine`

En este tutorial veremos los pasos necesarios para instalar `docker`, `docker-compose` y `docker-machine` sobre un entorno Linux. Lo escribí probando en mi máquina, que ejecuta Ubuntu 18.04; los pasos podrían ser ligeramente distintos si usás otra distribución (y bastante más distintos si usás Windows o Mac).

Lo que se describe aquí no es más que un recorte de las guías oficiales de Docker, traducido al español y comentado para quienes no tienen mucha experiencia en instalar software. En cada paso está el link a la guía oficial, que debería servir de referencia cuando este tutorial quede desactualizado.

### docker
Seguiremos las instrucciones que están en [la guía oficial](https://docs.docker.com/install/linux/docker-ce/ubuntu/), en su sección [Install using the repository](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository).

Lo primero que hay que hacer es agregar el repositorio de Docker e instalar algunas herramientas necesarias para ello:

```bash
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Agregado el repositorio, procederemos a la instalación en sí:

```bash
sudo apt-get update

sudo apt-get remove docker docker-engine docker.io containerd runc # elimina cualquier versión anterior de docker

sudo apt-get install docker-ce docker-ce-cli containerd.io

```

Finalizada la instalación, agregaremos a nuestro usuario actual al grupo de Docker, para no tener que usar `sudo` a cada rato:

```bash
sudo usermod -aG docker $USER
# Luego de esto hay que reiniciar la computadora para que tenga efecto
```

Si todo funcionó deberíamos ver algo como esto al ejecutar `docker run hello-world`:

```console
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

Instalaremos a continuación `docker-compose`, una herramienta que nos permite conectar varios contenedores de forma sencilla. Para instalarlo seguiremos también [la guía oficial](https://docs.docker.com/compose/install/), que puede resumirse en los siguientes comandos:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Si todo fue bien, deberíamos ver algo como esto:

```console
$ docker-compose --version
docker-compose version 1.24.0, build 0aa59064
```

### docker-machine

Una vez que tengamos todo funcionando, necesitaremos una forma sencilla de subirlo a nuestro host, en este caso Digital Ocean. Ahí es cuando entra en acción `docker-machine`, utilidad que instalaremos con la [guía oficial](https://docs.docker.com/machine/install-machine/), , que puede resumirse en el siguiente comando:

```bash
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

Si todo fue bien, deberíamos ver algo como esto:

```console
$ docker-machine version
docker-machine version 0.16.0, build 702c267f
```
