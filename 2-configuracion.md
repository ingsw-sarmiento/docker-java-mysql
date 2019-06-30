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
