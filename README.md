# rb_lpi_project
Proyecto bus-app vagrant, docker, compose, swarm, stack, scale proxy

# Contenedores en DockerHub
https://hub.docker.com/u/rbq4ds

# PROCESO AUTOMATIZADO CON VAGRANT
0- NODOS --> Manager / Nodo1
1- Vagrantfile --> vagrant validate --> vagrant up > Se generan los nodos
2- Se instala automáticamente Docker, Compose, Swarm
3- Se guarda swarm token en /vagrant
4- El nodo1 se une al clúster mediante el archivo generado con el token (paso3)
5- Se escala el proxy (scale=4)

# Comandos de comprobación de la infraestructura desde manager
vagrant ssh manager
docker stack ls
docker stack inspect rbmstack (nombre del stack, se puede modificar)
docker service ls
docker service inspect rbmstack --pretty
docker ps
docker service ps rbmstack_proxy
docker network ls

# Probar desde navegador o curl Equipo anfitrión (redirección de puertos correcta)
http://localhost:8080



