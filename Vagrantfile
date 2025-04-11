Vagrant.configure("2") do |config|
  # Configuración común para todas las máquinas
  config.vm.box = "bento/ubuntu-22.04"
  config.vm.synced_folder ".", "/vagrant"

  # Script común para instalar Docker
  DOCKER_INSTALL_SCRIPT = <<-SCRIPT
    # Evitar advertencias de dpkg-preconfigure
    export DEBIAN_FRONTEND=noninteractive

    # Actualizar el sistema
    echo "Actualizando el sistema..."
    sudo apt-get update -y || { echo "Error al actualizar el sistema"; exit 1; }

    # Instalar dependencias necesarias para Docker
    echo "Instalando dependencias necesarias para Docker..."
    sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common || { echo "Error al instalar dependencias"; exit 1; }

    # Agregar la clave GPG oficial de Docker
    echo "Agregando la clave GPG oficial de Docker..."
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg || { echo "Error al agregar la clave GPG"; exit 1; }

    # Agregar el repositorio de Docker
    echo "Agregando el repositorio de Docker..."
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null || { echo "Error al agregar el repositorio de Docker"; exit 1; }

    # Actualizar nuevamente para incluir el repositorio de Docker
    echo "Actualizando nuevamente el sistema..."
    sudo apt-get update -y || { echo "Error al actualizar el sistema después de agregar el repositorio"; exit 1; }

    # Instalar Docker
    echo "Instalando Docker..."
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io || { echo "Error al instalar Docker"; exit 1; }

    # Verificar la instalación de Docker
    echo "Verificando la instalación de Docker..."
    docker --version || { echo "Error: Docker no está instalado correctamente"; exit 1; }

    # Añadir el usuario 'vagrant' al grupo 'docker'
    echo "Añadiendo el usuario 'vagrant' al grupo 'docker'..."
    sudo usermod -aG docker vagrant || { echo "Error al añadir el usuario al grupo 'docker'"; exit 1; }
  SCRIPT

  # Nodo Manager
  config.vm.define "manager" do |manager|
    manager.vm.network "private_network", ip: "192.168.33.10"
    manager.vm.network "forwarded_port", guest: 80, host: 8080
	manager.vm.network "forwarded_port", guest: 9090, host: 9090
    manager.vm.provision "shell", inline: <<-SHELL
      #{DOCKER_INSTALL_SCRIPT}

      # Inicializar Docker Swarm
      echo "Inicializando Docker Swarm..."
      sudo docker swarm init --advertise-addr 192.168.33.10 || { echo "Error al inicializar Docker Swarm"; exit 1; }

      # Guardar el comando docker swarm join en /vagrant/swarm_join.sh
      echo "Guardando el comando docker swarm join en /vagrant/swarm_join.sh..."
      sudo docker swarm join-token worker | grep "docker swarm join" > /vagrant/swarm_join.sh || { echo "Error al guardar el comando swarm join"; exit 1; }
      chmod +x /vagrant/swarm_join.sh || { echo "Error al cambiar los permisos del archivo swarm_join.sh"; exit 1; }

      # Copiar docker-compose.yml al nodo manager
      echo "Copiando docker-compose.yml al nodo manager..."
      cp /vagrant/bus-ticket-app/docker-compose.yml /home/vagrant/docker-compose.yml || { echo "Error al copiar docker-compose.yml"; exit 1; }

      # Desplegar la aplicación con Docker Swarm
      echo "Desplegando la aplicación con Docker Swarm..."
      sudo docker stack deploy -c /home/vagrant/docker-compose.yml rbmstack --detach=false || { echo "Error al desplegar la aplicación"; exit 1; }

      # Escalar el servicio proxy a 4 réplicas
      echo "Escalando el servicio proxy a 4 réplicas..."
      sudo docker service scale rbmstack_proxy=4 || { echo "Error al escalar el servicio proxy"; exit 1; }

      echo "Provisionamiento del manager completado correctamente."
    SHELL
  end

  # Nodo Worker
  config.vm.define "nodo1" do |nodo1|
    nodo1.vm.network "private_network", ip: "192.168.33.11"
    nodo1.vm.provision "shell", inline: <<-SHELL
      #{DOCKER_INSTALL_SCRIPT}

      # Unirse al cluster usando el comando almacenado en /vagrant/swarm_join.sh
      echo "Uniendo el nodo al cluster..."
      if [ -f /vagrant/swarm_join.sh ]; then
        sudo bash /vagrant/swarm_join.sh || { echo "Error al unir el nodo al cluster"; exit 1; }
      else
        echo "Error: No se encontró el archivo swarm_join.sh. Asegúrate de que el manager ha inicializado Swarm."
        exit 1
      fi

      echo "Provisionamiento del nodo1 completado correctamente."
    SHELL
  end
end
