# pasos seguidos para configurar los servicios y comandos aplicados
Repositorio 
README.md
pasos_dhcp_windows.md
comandos.txt
# Documentación del Proyecto - Servicios en CentOS 10

## Descripción
Este repositorio contiene el respaldo de los pasos realizados para la instalación y configuración de los servicios:
- Odoo
- WordPress
- Asterisk

Los procedimientos fueron ejecutados en el sistema operativo CentOS 10 (Stream) como parte del proyecto de laboratorio.

## Contenido del repositorio
- odoo.md: Pasos de instalación y configuración de Odoo
- wordpress.md: Pasos de instalación y configuración de WordPress
- asterisk.md: Pasos de instalación y configuración de Asterisk

## Autor
Fabian Rodriguez  
Fecha: 02/02/2026
# Instalación de Odoo en CentOS 10

Se realizó la actualización del sistema y la instalación de dependencias necesarias.  
Posteriormente, se configuró PostgreSQL como base de datos y se descargó Odoo desde el repositorio oficial.  
Se creó un servicio para iniciar Odoo automáticamente y se verificó el acceso al sistema mediante el navegador web en el puerto 8069.

Pasos para instalar odoo
1.-Actualizar el sistema
sudo dnf update -y
2.-Instalar dependencias básicas
sudo dnf install -y epel-release
sudo dnf install -y python3 python3-pip git wget nodejs npm

3.-Instalar PostgreSQL
sudo dnf install -y postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
sudo systemctl enable postgresql
sudo systemctl start postgresql
Crear usuario para Odoo:
sudo -u postgres createuser -s odoo
4.-Crear usuario del sistema para Odoo
sudo useradd -m -U -r -s /bin/bash odoo
sudo su - odoo
git clone https://www.github.com/odoo/odoo --depth 1 --branch 17.0
exit

sudo dnf install -y python3-devel libxml2-devel libxslt-devel openldap-devel libsasl2-devel gcc gcc-c++
sudo pip3 install -r /home/odoo/odoo/requirements.txt

6.-Crear archivo de configuración
sudo nano /etc/odoo.conf
Pega esto:
[options]
admin_passwd = admin123
db_host = False
db_port = False
db_user = odoo
db_password = False
addons_path = /home/odoo/odoo/addons
xmlrpc_port = 8069
Guardar: Ctrl + O → Enter → Ctrl + X

7.-Crear servicio de Odoo
sudo nano /etc/systemd/system/odoo.service
[Unit]
Description=Odoo
After=network.target postgresql.service

[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/usr/bin/python3 /home/odoo/odoo/odoo-bin -c /etc/odoo.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target

8.-Iniciar Odoo
sudo systemctl daemon-reload
sudo systemctl enable odoo
sudo systemctl start odoo
sudo systemctl status odoo

9.-Abrir Odoo en el navegador
En tu PC o desde otra máquina:
http://IP_DEL_SERVIDOR:8069


# Instalación de WordPress en CentOS 10

Se instaló el servidor web Apache, PHP y MariaDB.  
Se creó la base de datos para WordPress y se descargó el CMS desde el sitio oficial.  
Finalmente, se configuraron los permisos necesarios y se verificó el acceso al sitio web desde el navegador.
Pasos para descargar el wordpress
sudo dnf update -y
sudo dnf install epel-release -y
________________________________________
sudo dnf install nginx -y
sudo systemctl enable --now nginx
________________________________________
WordPress necesita PHP + módulos.
sudo dnf install php php-fpm php-mysqlnd php-gd php-xml php-mbstring php-json php-curl php-zip -y
Activar php-fpm:
sudo systemctl enable --now php-fpm
________________________________________
sudo dnf install mariadb-server -y
sudo systemctl enable --now mariadb
________________________________________
Ejecuta el script de seguridad:
sudo mysql_secure_installation
Cuando pregunte:
•	Set root password → Y
•	Remove anonymous users → Y
•	Disallow root login remotely → Y
•	Remove test database → Y
•	Reload privilege tables → Y
Listo.
________________________________________
Entrar al servidor MySQL:
sudo mysql
Crear BD:
CREATE DATABASE wordpress;
Crear usuario:
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'clave123';
Dar permisos:
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
________________________________________
Instalar wget y unzip:
sudo dnf install wget unzip -y
Descargar paquete oficial desde WordPress.org:
wget https://wordpress.org/latest.zip
Descomprimir:
unzip latest.zip
Mover archivos a Nginx:
sudo mv wordpress/ /var/www/html/
Dar permisos correctos:
sudo chown -R nginx:nginx /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress
________________________________________
Copiar archivo de ejemplo:
sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
Editar:
sudo nano /var/www/html/wordpress/wp-config.php
Modificar estas líneas:
define('DB_NAME', 'wordpress');
define('DB_USER', 'wpuser');
define('DB_PASSWORD', 'clave123');
define('DB_HOST', 'localhost');
Guardar.
________________________________________
Crear archivo:
sudo nano /etc/nginx/conf.d/wordpress.conf
Pegar:
server {
    listen 80;
    server_name _;
    root /var/www/html/wordpress;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
Guardar.
________________________________________
sudo systemctl restart php-fpm
sudo systemctl restart nginx
________________________________________
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
________________________________________
En tu PC Windows abre:
http://IP_DEL_CENTOS


# Instalación de Asterisk en CentOS 10

Se descargó e instaló Asterisk desde el código fuente.  
Se instalaron las dependencias requeridas para la compilación.  
El servicio fue habilitado para iniciar automáticamente y se verificó su funcionamiento accediendo a la consola de Asterisk.

Pasos para instalar asterisk 
Actualizar el sistema
sudo dnf update -y

Instalar dependencias
sudo dnf install -y epel-release
sudo dnf install -y wget git nano gcc gcc-c++ make ncurses-devel libxml2-devel sqlite-devel openssl-devel jansson-devel uuid-devel

Descargar Asterisk 
cd /usr/src
sudo wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-20-current.tar.gz
sudo tar -xvzf asterisk-20-current.tar.gz
cd asterisk-20.*/

Instalar dependencias extra del script
sudo contrib/scripts/install_prereq install

Compilar e instalar Asterisk
sudo ./configure
sudo make
sudo make install
sudo make samples
sudo make config
sudo ldconfig

Iniciar Asterisk
sudo systemctl enable asterisk
sudo systemctl start asterisk
sudo systemctl status asterisk

Entrar a la consola de Asterisk
sudo asterisk -rvvv


Comandos importantes:
dnf update -y
dnf install -y epel-release
systemctl start httpd
systemctl start mariadb
systemctl start postgresql
systemctl start asterisk
