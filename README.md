# Practica03

# LAMP en dos niveles

## 0.1 Repositorio de github:

* https://github.com/Alexstrachan/Practica03.git

## 1. Creación del vagrantfile y los scripts de apache y mysql

Como ya explicamos en la practica02, debemos crear una nueva máquina con su respectivo directorio.

```ssh
c:\\users\Alex\>cd iaw
c:\\users\Alex\>mkdir practica03
c:\\users\Alex\>cd practica03
c:\\users\Alex\practica03>vagrant init
```
A partir de este punto, nuestro "vagrantfile" se habrá creado tras ejecutar en "init" y crearemos dos script a los que llamaremos

* provision-for-apache.sh
* provision-for-mysql.sh

## 2. Configuración del Vagrantfile

```ssh
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"

#Apache HTTP Server
config.vm.define "web" do |app|
app.vm.hostname = "web"
config.vm.network "private_network", ip: "192.168.33.10"
app.vm.provision "shell", path: "provision-for-apache.sh"


end

#MySQL Server
config.vm.define "db" do |app|
app.vm.hostname = "db"
config.vm.network "private_network", ip: "192.168.33.11"
app.vm.provision "shell", path: "provision-for-mysql.sh"

end

end
```
Tras haber limpiado el código de vagrantfile, tendremos que tener nuestro vagrantfile tal y como se muestra en la foto.

## 3. Script para apache y mysql

Ahora vamos a hacer dos script, uno para apache y otro para sql. Como hemos visto en el apartado 1 estos script los llamaremos: "provision-for-apache.sh" para apache y "provision-for-mysql.sh" para mysql.

#### provision-for-apache.sh

```ssh
#!/bin/bash

#Instalacion de repositorios

apt-get update

# Instalacion de apache

apt-get install -y apache2
apt-get install -y php libapache2-mod-php php-mysql

sudo /etc/init.d/apache2 restart

#clonar repositorio 

apt-get install -y git 
cd /tmp
rm -rf iaw-practica-lamp
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git

#copiar repositorio

cd iaw-practica-lamp
cp src/* /var/www/html

#modificar la base de datos que queremos usar, en ella estableceremos la ip 192.168.33.11 que es la asociada a mysql. Con el comando "sed -i" reemplazamos "localhost" por la ip "192.168.33.11" dentro del directorio "/var/www/html/config.php"

sed -i 's/localhost/192.168.33.11/' /var/www/html/config.php
chown www-data:www-data /var/www/html -R

#borrar el index
rm -rf /var/www/html/index.html
```

#### provision-for-mysql.sh

```ssh
#!/bin/bash

#Instalacion de repositorios

apt-get update

#Instalacion de mysqlserver

apt-get -y install debconf-utils

DB_ROOT_PASSWD=root
debconf-set-selections <<< "mysql-server mysql-server/root_password password $DB_ROOT_PASSWD"
debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DB_ROOT_PASSWD"

apt-get install -y mysql-server

# De igual manera que para el script de apache, otra vez modificamos la ip del mysqlserver para que en lugar de la 127.0.0.1 sea la 0.0.0.0 para no tener ningún problema de acceso.

sed -i 's/127.0.0.1/0.0.0.0/' /etc/mysql/mysql.conf.d/mysqld.cnf

#/etc/init.d/mysql restart

#clonar repositorio (de la misma manera que lo hemos creado dentro del "provision-for-apache.sh)

apt-get install -y git
cd /tmp
rm -rf iaw-practica-lamp
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git

#creamos la base de datos

mysql -u root mysql -p$DB_ROOT_PASSWD < /tmp/iaw-practica-lamp/db/database.sql

#crear usuario global

#mysql -uroot mysql -p$DB_ROOT_PASSWD <<< "GRANT ALL PRIVILEGES ON *.* TO root@'%' IDENTIFIED BY '$DB_ROOT_PASSWD'; FLUSH PRIVILEGES;"
```

## 4. Anexo de comandos de vagrant que hemos usado

Inciar la máquina.
```ssh
vagrant up
```
Reiniciar la máquina. Esto se utiliza cuando hacemos algun cambio en "Vagrantfile" y queremos que se apliquen los cambios.
```ssh
vagrant reload 
```
Ejecutar los scripts "provisioners". Siempre que realicemos algun cambio en estos scripts, debemos hacerlo.
```ssh
vagrant provision
```
Inicializar el vagrantfile.
```ssh
vagrant init
```
Apagar la máquina virtual
```ssh
vagrant halt
```
Elimiar la máquina virtual
```ssh
vagrant destroy
```
Conectar a una máquina por ssh, en este caso, como tenemos dos script:
```ssh
vagrant ssh web --> Conectar con apache
vagrant ssh db --> Conectar con mysql
```
Comprobar estado de la maquina
```ssh
vagrant status
```