---
title: Practica 2 - Configuración de Nginx
date: 2025-01-18
categories: [Digital Ocean, VPS, Linux, RockyLinux, Nginx]
tags: [Digital Ocean, VPS, Linux, RockyLinux, Nginx]
---

## Introducción

En este artículo te explicaré cómo instalar y configurar Nginx en un VPS de Digital Ocean. Además, veremos cómo instalar PHP, Composer y configurar un proyecto Laravel. También incluiremos configuraciones para proxy y load balancing, y te mostraremos ejemplos de archivos de configuración (como *laravel.conf*, *ssl.conf* y *default.conf*).

---

## 1. NGINX

**Nginx** es un servidor web de código abierto que se utiliza para servir contenido web estático y dinámico. Es conocido por su alto rendimiento, estabilidad, bajo uso de recursos y configuración sencilla. En este artículo, aprenderemos a instalar y configurar Nginx en un VPS de Digital Ocean.

> Documentación oficial de [Nginx](https://nginx.org/en/linux_packages.html#RHEL){:target="_blank"}.
{: .prompt-tip }


### 1.1 Instalación de Nginx

Sigue estos pasos para instalar Nginx en tu VPS:

1. **Instalar prerequisitos y crear el repositorio de Nginx:**

   - Instalar los prerequisitos de YUM
   ```bash
   sudo yum install yum-utils
   ```

   - Crear el archivo de repositorio de Nginx
   ```bash
   nano /etc/yum.repos.d/nginx.repo
   ```

2. **Agregar el siguiente contenido al archivo `nginx.repo`:**

   ```conf
   [nginx-stable]
   name=nginx stable repo
   baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=1
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true

   [nginx-mainline]
   name=nginx mainline repo
   baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=https://nginx.org/keys/nginx_signing.key
   module_hotfixes=true
   ```
   {: file='/etc/yum.repos.d/nginx.repo'}

3. **Actualizar repositorios e instalar Nginx:**

   ```bash
   sudo yum update -y && sudo yum install nginx -y
   ```

### 1.2 Reiniciar y Habilitar Nginx

1. **Verificar el estado del servicio:**

   ```bash
   sudo systemctl status nginx
   sudo service nginx status
   ```

2. **Iniciar el servicio:**

   ```bash
   sudo systemctl start nginx
   sudo service nginx start
   ```

3. **Habilitar Nginx para que inicie con el sistema:**

   ```bash
   sudo systemctl enable nginx
   sudo service nginx enable
   ```

### 1.3 Comprobación de Conexiones

1. **Mostrar conexiones activas:**

   ```bash
   sudo netstat -ptona
   ```

2. **Verificar conexión en un puerto específico:**

   ```bash
   sudo netstat -tuln | grep {puerto}
   ```

3. **Cambiar el puerto de escucha de Nginx (si es necesario):**

   ```bash
   sudo semanage port -a -t http_port_t -p tcp {port}
   ```

### 1.4 Archivo de Configuración Default

El archivo por defecto se encuentra hecho de esta forma:
   ```nginx
    server {
      listen       80;
      server_name  localhost;

      #access_log  /var/log/nginx/host.access.log  main;

      location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
      }

      #error_page  404              /404.html;

      # redirect server error pages to the static page /50x.html

      error_page   500 502 503 504  /50x.html;

      location = /50x.html {
        root   /usr/share/nginx/html;
      }

      # proxy the PHP scripts to Apache listening on 127.0.0.1:80
      #
      #location ~ \.php$ {
      #    proxy_pass   http://127.0.0.1;
      #}

      # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
      #
      #location ~ \.php$ {
      #    root           html;
      #    fastcgi_pass   127.0.0.1:9000;
      #    fastcgi_index  index.php;
      #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
      #    include        fastcgi_params;
      #}

      # deny access to .htaccess files, if Apache's document root
      # concurs with nginx's one
      #
      #location ~ /\.ht {
      #    deny  all;
      #}
    }
   ```
   {: file='/etc/nginx/conf.d/default.conf'}

### 1.5 Reiniciar el Servicio de Nginx

Después de checar la configuración, reinicia el servicio:

   ```bash
   sudo systemctl restart nginx
   sudo service nginx restart

   # Verificar la configuración
   sudo nginx -t

   # Recargar la configuración sin reiniciar
   sudo nginx -s reload

   # Agregar al usuario al grupo de Nginx (opcional)
   sudo gpasswd -a {user} nginx
   ```

---

## 2. PHP y Composer

### 2.1 Instalación de PHP y Composer

1. **Actualizar repositorios e instalar repositorios adicionales:**

   ```bash
   sudo dnf update -y
   sudo dnf install epel-release -y
   sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
   ```

2. **Resetear y habilitar el módulo de PHP (ejemplo con PHP 8.2):**

   ```bash
   sudo dnf module list reset php -y
   sudo dnf module enable php:remi-8.2
   ```

3. **Instalar PHP y extensiones necesarias:**

   ```bash
   sudo dnf install php php-common php-xml php-json curl unzip php-fpm php-mysqlnd php-opcache php-gd php-pgsql php-mbstring php-zip -y
   ```

4. **Actualizar cache de repositorios:**

   ```bash
   sudo dnf makecache
   ```

5. **Instalar Composer:**

   ```bash
   sudo curl -sS https://getcomposer.org/installer | php
   sudo mv composer.phar /usr/local/bin/composer
   sudo chmod +x /usr/local/bin/composer
   ```

### 2.2 Configuración de PHP

1. **Editar el archivo `/etc/php.ini`:**

   ```conf
   # Configurar la zona horaria y otros parámetros
   date.timezone = America/Monterrey
   cgi.fix_pathinfo = 0
   expose_php = Off
   # Opcionales:
   memory_limit = 256M
   upload_max_filesize = 5M
   ```
   {: file='/etc/php.ini'}

2. **Editar el archivo de PHP-FPM `/etc/php-fpm.d/www.conf`:**

   ```conf
   user = {user}
   group = nginx

   listen.owner = {user},nginx
   listen.group = nginx
   listen.mode = 0660

   security.limit_extensions = .php .php3 .php4 .php5 .php7
   ```
   {: file='/etc/php-fpm.d/www.conf'}

3. **Reiniciar y habilitar PHP-FPM:**

   ```bash
   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   ```

---

## 3. Laravel

### 3.1 Configuración del Proyecto Laravel

1. **Instalar Node.js (opcional, se puede hacer con NVM o RPM):**

   - **Con NVM (no recomendado):**

     ```bash
     curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
     source ~/.bashrc
     nvm ls-remote
     nvm install {node-version}
     source ~/.bashrc
     ```

   - **O instalar Node.js usando RPM:**

     ```bash
     curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
     sudo dnf install nodejs -y
     ```

2. **Clonar el repositorio de Laravel:**

   ```bash
   sudo dnf install git unzip -y
   git clone {repo}
   ```

3. **Configurar el archivo `.env`:**

   Copia el archivo `.env.example` a `.env` y ajusta las siguientes variables:

   ```js
   APP_ENV=production
   APP_URL=http://{ip}
   ASSET_URL=http://{ip}
   ```
   {: file='.env'}

4. **Instalar dependencias:**

   - Composer
   ```bash
   composer install
   ```

   - Node.js
   ```bash
   npm install
   ```

5. **Optimizar y generar la key de Laravel:**

   ```bash
   php artisan optimize:clear
   php artisan key:generate
   ```

6. **Construir los archivos de Node.js:**

   ```bash
   npm run build
   ```

### 3.2 Configurar Permisos en el Proyecto

1. **Asignar permisos adecuados:**

   ```bash
   sudo chmod 750 /home/{user}
   sudo chown -R {user}:nginx /home/{user}
   ```

2. **Habilitar el acceso a la red para el usuario de Nginx:**

   ```bash
   sudo setsebool -P httpd_can_network_connect on
   ```

3. **Ajustar etiquetas de SELinux para el proyecto Laravel:**

   ```bash
   sudo chcon -R -t httpd_sys_rw_content_t /home/{user}/{project-laravel-folder}
   ```

4. **Reiniciar servicios:**

   ```bash
   sudo systemctl restart nginx
   sudo systemctl restart php-fpm
   ```

5. **Ajustar permisos finales en el proyecto:**

   ```bash
   sudo chmod -R 750 /home/{user}/{project-laravel-folder}
   sudo chmod -R 770 /home/{user}/{project-laravel-folder}/storage
   sudo chmod -R 770 /home/{user}/{project-laravel-folder}/bootstrap/cache
   ```

### 3.3 Comprobar Logs

Verifica los logs para asegurarte de que todo funcione correctamente:

   - Logs de Nginx
   ```bash
   sudo tail -f /var/log/nginx/error.log
   ```

   - Logs de PHP-FPM
   ```bash
   sudo tail -f /var/log/php-fpm/error.log
   ```

   - Logs del sistema
   ```bash
   sudo journalctl -u nginx
   sudo journalctl -u php-fpm
   ```

---

## 4. Proxy

### 4.1 Configuración de Proxy con Nginx

1. **Crear el archivo de configuración `proxy.conf`:**

   Crea el archivo `proxy.conf` y coloca el siguiente contenido:

   ```nginx
   server {
     listen       {port};
     listen       {[::] o host-ip-pub/priv}:{port};
     server_name  {dominio};

     location / {
       proxy_connect_timeout 1s;
       proxy_send_timeout 1s;
       proxy_read_timeout 1s;

       proxy_next_upstream error timeout http_500 http_502 http_503 http_504;

       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;

       proxy_pass http://{upstream-name}$request_uri;
     }

     # Redirigir errores del servidor a /50x.html
     error_page   500 502 503 504  /50x.html;

     location = /50x.html {
       root /usr/share/nginx/html;
       internal;
     }
   }
   ```
   {: file='proxy.conf'}

2. **Permitir el tráfico saliente en SELinux para Nginx:**

   ```bash
   sudo setsebool -P httpd_can_network_connect 1
   ```

3. **Reiniciar Nginx para aplicar los cambios:**

   ```bash
   sudo systemctl restart nginx
   ```

---

## 5. Load Balancer

### 5.1 Configuración del Load Balancer con Nginx

1. **Crear el archivo de configuración `load-balancer.conf`:**

   - Crea el archivo `load-balancer.conf` con el siguiente contenido:

   ```nginx
   upstream {upstream-name} {
     least_conn;
     server {ip-1/host}:{port};
     server {ip-2/host}:{port};
     server {ip-3/host}:{port};
     server {ip-4/host}:{port} backup;
   }
   ```
   {: file='load-balancer.conf'}

2. **Reiniciar Nginx:**

   ```bash
   sudo systemctl restart nginx
   ```

---

## 6. Archivos Adicionales de Configuración de Nginx

### 6.1 laravel.conf

Utiliza este archivo para configurar Nginx con un proyecto Laravel. Crea el archivo `laravel.conf` con el siguiente contenido:

   ```nginx
   server {
     listen        {port};
     listen        {[::] o host-ip-pub/priv}:{port};
     server_name   {host-ip-pub/priv/domain};
     server_tokens off;

     root          /home/{user}/{laravel}/public;
     index         index.php index.html index.htm;

     charset utf-8;
     gzip on;
     gzip_types text/css application/javascript text/javascript application/x-javascript image/svg+xml text/plain text/xsd text/xsl text/xml image/x-icon;

     add_header X-Frame-Options "SAMEORIGIN" always;
     add_header X-XSS-Protection "1; mode=block" always;
     add_header X-Content-Type-Options "nosniff" always;

     location / {
       try_files $uri $uri/ /index.php?$query_string;
     }

     location = /favicon.ico {
       access_log off;
       log_not_found off;
     }

     location = /robots.txt {
       access_log off;
       log_not_found off;
     }

     error_page 404 /index.php;

     location ~* \.php$ {
       fastcgi_pass unix:/var/run/php-fpm/www.sock;
       fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
       include fastcgi_params;
       fastcgi_hide_header X-Powered-By;
     }

     location ~ /\.(?!well-known).* {
       allow all;
     }
   }
   ```
   {: file='laravel.conf'}

### 6.2 ssl.conf

Crea el archivo `ssl.conf` para configurar SSL:

   ```nginx
   server {
     listen        443 ssl http2;
     listen        [::]:443 ssl http2;
     server_name {domain_name};

     ssl_certificate /etc/nginx/ssl/origin_certificate.pem;
     ssl_certificate_key /etc/nginx/ssl/private_key.pem;
   }
   ```
   {: file='ssl.conf'}

---

## Conclusiones

En este artículo hemos aprendido cómo instalar y configurar Nginx en un VPS de Digital Ocean. También hemos visto cómo instalar PHP, Composer y configurar un proyecto Laravel. Además, hemos incluido configuraciones para proxy y load balancing, y te hemos mostrado ejempjsones de archivos de configuración como *laravel.conf*, *ssl.conf* y *default.conf*.

> Cualquier duda o comentario, agregarla en la sección de comentarios abajo de cada publicación.
{: .prompt-tip }
