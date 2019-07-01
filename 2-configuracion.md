## Creación de la imagen de la aplicación y del entorno para su ejecución

Esta es la segunda entrega del tutorial "Despliegue de una aplicación web Java sobre Docker", y aquí veremos cómo configurar nuestra aplicación para que corra en Docker.

Los ejemplos están basados en la aplicación [Libro Matriz Digital](https://github.com/groupbit/libro-matriz-digital), elaborada por estudiantes de la Universidad Nacional de Quilmes para el ISFDyT nro. 138 de Capitán Sarmiento, pero puede servir para cualquier aplicación Java que use Hibernate y pueda empaquetarse en un WAR.

### Cambios en el código Java

Lo primero que debemos hacer es preparar la aplicación para que poder configurarla según el entorno de ejecución, a través de variables de entorno. En nuestro ejemplo, queremos configurar el acceso a la base de datos y el modo de ejecución de Wicket (desarrollo o producción).

Para evitar trabajar con los valores nulos que devuelve `System.getenv` (lo que Java provee para acceder a las variables de entorno), creé esta pequeña clase que aprovecha el [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) de Java 8:

```java
public class Env {
	public static String getOrElse(String key, String defaultValue) {
		return read(key).orElse(defaultValue);
	}

	public static boolean isPresent(String key) {
		return read(key).isPresent();
	}

	private static Optional<String> read(String key) {
		return Optional.ofNullable(System.getenv(key));
	}
}
```

En el caso de la base de datos, creé cuatro variables: `DB_HOST`, `DB_PORT`, `DB_USERNAME` y `DB_PASSWORD`, dejando el nombre de la base de datos fijo en `libroMatrizDigital`. Además, dejé valores por defecto que nos sirven para no tener que configurar las variables en el ambiente de desarrollo.

El método que crea el `DataSource` quedó así:

```java
public DataSource dataSource() {
  DriverManagerDataSource dataSource = new DriverManagerDataSource();
  dataSource.setDriverClassName("com.mysql.jdbc.Driver");

  String host = Env.getOrElse("DB_HOST", "localhost");
  String port = Env.getOrElse("DB_PORT", "3306");
  dataSource.setUrl("jdbc:mysql://" + host + ":" + port + "/libroMatrizDigital");
  dataSource.setUsername(Env.getOrElse("DB_USERNAME", "root"));
  dataSource.setPassword(Env.getOrElse("DB_PASSWORD", "root"));

  return dataSource;
}
```

Para el modo de ejecución de Wicket elegí crear una variable `WICKET_PRODUCTION`, cuya simple presencia activa el modo productivo. En Java, fue necesario agregar el siguiente método a nuestra `Application`:

```java
@Override
public RuntimeConfigurationType getConfigurationType() {
  return Env.isPresent("WICKET_PRODUCTION")
    ? RuntimeConfigurationType.DEPLOYMENT
    : RuntimeConfigurationType.DEVELOPMENT;
}
```

¡Y listo! Preparada nuestra aplicación para ser configurada a través de variables de entorno. Esto nos sirve para Docker, pero también se podrían configurar a través de Maven o desde la misma consola.

### Creación de la imagen Docker

Como el objetivo final es publicar nuestra aplicación en un entorno productivo, vamos a crear una imagen Docker que la contenga. La imagen contendrá solamente lo necesario para correr la aplicación Java, la base de datos y las migraciones se ejecutarán en otros contenedores.

Partiendo de una imagen base de Jetty, lo mínimo que necesitamos para que crear nuestra imagen es copiar el WAR al directorio correspondiente. Nuestro `Dockerfile` inicial quedaría así:

```docker
FROM jetty:9-jre8-alpine
WORKDIR $JETTY_BASE
COPY target/libro-matriz-digital.war webapps/ROOT.war
```

¡Felicitaciones! Ya tenemos una imagen para ejecutar nuestra aplicación. Si tenemos MariaDB (o MySQL) corriendo con los valores por defecto que espera nuestra aplicación, podríamos levantarla ejecutando lo siguiente:

```
mvn package
docker build mi-app-java .
docker run mi-app-java
```

Para que nos sirva para un entorno productivo, nos faltan dos tareas más:
* crear una carpeta para contener las migraciones, que serán luego ejecutadas por otro contenedor con [FlywayDB](https://flywaydb.org/);
* copiar el script [wait-for](https://github.com/eficode/wait-for), que utilizaremos para esperar a que inicie la base de datos.

Esto lo logramos modificando el `Dockerfile` de la siguiente manera:

```docker
FROM jetty:9-jre8-alpine
WORKDIR $JETTY_BASE
RUN mkdir migrations
COPY scripts/wait-for /wait-for
COPY src/main/resources/db/migration/* migrations/
COPY target/libro-matriz-digital.war webapps/ROOT.war
```

**Ojo:** el orden de los comandos no es casual, está pensado para optimizar el tiempo de `build`: Docker construye las imágenes por capa, y ante modificaciones recrea aquellas partes que cambiaron _y todas las que le siguen_. Por esto, conviene ordenar los comandos según la probabilidad de que cambien; en este caso lo que más seguido cambiará es el WAR y por eso está último, seguido por las migraciones.

### Creación del `docker-compose`

Ya tenemos la imagen de nuestra aplicación, ahora vamos a configurar el resto del entorno para que pueda ejecutarse completamente dentro de Docker.

Crearemos entonces un archivo `docker-compose.yml`, que posteriormente agregaremos al mismo repositorio en el que está el código fuente - así nos queda el código y todo lo necesario para ejecutarlo en un mismo lugar.

Comencemos entonces creando el archivo y un servicio llamado `db` para levantar una base de datos MariaDB:

```yml
version: '3'
services:
  db:
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: MiPasswordSecreto
      MYSQL_DATABASE: libroMatrizDigital
    volumes:
      - data:/var/lib/mysql
    expose:
      - 3306

volumes:
  data:
```

Analicemos un poco las secciones:
* `image:` configura la imagen, en este caso la oficial de `mariadb`;
* `restart:` establece si la máquina debería reinicarse en caso de detenerse. En este caso indicamos que se reinicie siempre (`always`);
* `environment:` aquí van las configuraciones que irán a parar a variables de entorno dentro del contenedor. En este caso configuramos la contraseña del usuario `root` y la creación de la base de datos `libroMatrizDigital`. El listado completo de variables que soporta esta imagen se puede encontrar en [su documentación](https://hub.docker.com/_/mariadb);
* `volumes:` como los contenedores de Docker son descartables, necesitamos una forma de que los datos de la base no se borren. Para ello, configuramos un volúmen llamado `data`, que se sincronizará con el contenido `/var/lib/mysql`, directorio donde MariaDB guarda los datos;
* `expose:` indicamos que se exponga el pueto `3306`, para que la base de datos pueda ser accedida desde otros contenedores dentro del mismo `docker-compose`.

Lista la base de datos, agreguemos nuestra aplicación en un nuevo servicio llamado `app`:

```yml
version: '3'
services:
  db:
    # ya configurado

  app:
    image: mi-app-java
    restart: always
    ports:
      - "80:8080"
    environment:
      DB_HOST: db
      DB_PASSWORD: MiPasswordSecreto
      WICKET_PRODUCTION: 'true'
    depends_on:
      - db
    entrypoint: /wait-for db:3306 -- /docker-entrypoint.sh

volumes:
  data:
```

Analicemos nuestro nuevo servicio:
* `image:` configura la imagen, en este caso la que creamos anteriormente;
* `restart:` igual que para la base de datos;
* `ports:` a diferencia de `expose`, esta configuración publica el servicio fuera de la red de `docker-compose` para que pueda ser accedido desde afuera del contenedor. En este caso, vinculamos el puerto 80 de nuestra PC con el 8080 del contenedor, que es donde corre el Jetty;
* `environment:` configuramos las variables que creamos anteriormente en la aplicación Java. Algo interesante: en `DB_HOST` el valor es `db`, porque corresponde con el nombre que le dimos al servicio que levanta la base de datos. Esto funciona porque compose se encarga de crear una red local entre todos los contenedores y asignar los nombres de host correspondientes;
* `depends_on:` indica que este contenedor depende de `db`, y que no debe empezar hasta que aquel esté listo;
* `entrypoint:` aunque configuramos el `depends_on`, esto no es suficiente en todos los casos. Cuando el contenedor de la base de datos se crea por primera vez, tarda un tiempito en crear el esquema y nuestra aplicación puede llegar a fallar si se inicia antes que ese proceso termine. Para evitar eso, utilizamos el script `wait-for`, que espera a que haya un servicio disponible en `db:3306` antes de ejecutar el script que efectivamente levanta el Jetty (ubicado en `/docker-entrypoint.sh` por herencia de la imagen de Jetty).

Nos queda finalmente agregar el servicio que ejecutará las migraciones, y elegimos para ello una imagen creada por Flyway:

```yml
version: '3'
services:
  db:
    # ya configurado

  app:
    # agregamos un volúmen para compartir las migraciones:
    volumes:
      - migrations:/var/lib/jetty/migrations      

  flyway:
    image: boxfuse/flyway:5.2.4-alpine
    restart: on-failure
    command: -url=jdbc:mysql://db -schemas=libroMatrizDigital -user=root -password=MiPasswordSecreto -connectRetries=30 migrate
    volumes:
      - migrations:/flyway/sql
    depends_on:
      - db
      - app

volumes:
  migrations:
  data:
```

Analicemos nuestro nuevo servicio:
* `restart:` a diferencia de los otros dos servicios, este está pensado para ejecutarse una vez y salir. Por lo tanto, solo nos interesa que se reinicie si falló y utilizamos para ello la opción `on-failure`;
* `command:` copiado de la [documentación de la imagen](https://github.com/flyway/flyway-docker) y configurado esquema, usuario y password.

Como se ve, agregamos también un nuevo volumen, `migrations`, ya que las migraciones son parte de la imagen de nuestra aplicación y necesitamos de alguna manera compartirlas con el servicio que las ejecuta.

### Ejecución de la aplicación

Configurado todo, nos resta solamente probarlo. Para esto, abriremos una terminal dentro de nuestro  repositorio y ejecutaremos `docker-compose up`. Hay esperar a que descargue todo - puede tardar un raaaato largo la primera vez. :hourglass:

Para comprobar si funciona, abrimos un navegador y vamos a http://localhost. Debería aparecer nuestra aplicación. :smiley:

Luego, para salir ejecutamos <kbd>Ctrl</kbd> + <kbd>C</kbd>. Algunos otros comandos útiles de `docker-compose`:

* `docker-compose up db`: levanta únicamente un servicio, `db` en este caso. Si tiene dependencias, también las levantará previamente;
* `docker-compose up -d`: igual que `up`, pero devuelve el control de la consola;
* `docker-compose down`: baja todos los contenedores y los borra;
* `docker-compose down -v`: igual que `down`, pero borra también los volúmenes (en este ejemplo, la base de datos y la carpeta de migraciones).
