Vagrant.configure("2") do |config|
    # Configuración de los servidores web Apache
    (1..2).each do |i|
      config.vm.define "web#{i}" do |web|
        web.vm.box = "ubuntu/bionic64"
        web.vm.hostname = "web#{i}"
        web.vm.network "private_network", ip: "192.168.56.1#{i}"
        web.vm.provider "virtualbox" do |vb|
          vb.memory = "512"
          vb.cpus = 1
        end
  
        # Directorio compartido específico para cada servidor
        web.vm.synced_folder "./www/html/web#{i}", "/var/www/html"
  
        # Instalar Apache y configurar el servidor
        web.vm.provision "shell", inline: <<-SHELL
          apt-get update
          apt-get install -y apache2
          systemctl restart apache2
        SHELL
      end
    end
  
    # Configuración del balanceador de carga NGINX
    config.vm.define "nginx" do |nginx|
      nginx.vm.box = "ubuntu/bionic64"
      nginx.vm.hostname = "nginx"
      nginx.vm.network "private_network", ip: "192.168.56.30"
      nginx.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.cpus = 1
      end
  
      # Configuración de NGINX con balanceo de carga aleatorio
      nginx.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y nginx
        cat <<EOL > /etc/nginx/conf.d/load_balancer.conf
        upstream backend {
          random;
          server 192.168.56.11;
          server 192.168.56.12;
        }
        server {
          listen 80;
          location / {
            proxy_pass http://backend;
          }
        }
        EOL
        systemctl restart nginx
      SHELL
    end
  end
  