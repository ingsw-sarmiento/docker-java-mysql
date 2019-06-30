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
