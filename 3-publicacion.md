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

# Indica cuál es la máquina activa
docker-machine active

# Nos permite borrar una máquina
docker-machine rm libro-matriz-digital

# Vuelve la consola "a la normalidad": los comandos de aquí en adelante se ejecutarán en nuestra máquina. Otra forma es directamente cerrar la consola actual y abrir una nueva
eval $(docker-machine env -u)
```
