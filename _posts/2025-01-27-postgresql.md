---
title: Practica 3 - Configuración de PostgreSQL
date: 2025-01-27
categories: [Digital Ocean, VPS, Linux, RockyLinux, PostgreSQL]
tags: [Digital Ocean, VPS, Linux, RockyLinux, PostgreSQL]
---

## Introducción

En este artículo te mostraré de forma muy coloquial cómo instalar y configurar PostgreSQL en un VPS de Digital Ocean. Aprenderás a instalar el servidor y el cliente de PostgreSQL, configurar usuarios y bases de datos, habilitar conexiones seguras mediante SSL, y ajustar las variables de entorno para integrarlo con un proyecto Laravel.

---

## 1. Instalación de PostgreSQL

**PostgreSQL** es un sistema de gestión de bases de datos relacional de código abierto y gratuito. Es muy potente, seguro y altamente extensible, y es una excelente opción para proyectos de cualquier tamaño. En este caso, instalaremos PostgreSQL 15 en un servidor VPS con Rocky Linux.

### 1.1 Instalar el Repositorio y PostgreSQL

1. **Instalar el repositorio de PostgreSQL:**

   ```bash
   sudo dnf install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   ```

2. **Deshabilitar el módulo PostgreSQL por defecto y actualizar repositorios:**

   ```bash
   sudo dnf update -y
   sudo dnf -qy module disable postgresql
   ```

3. **Instalar PostgreSQL (servidor y cliente):**

   - Actualizar repositorios nuevamente
   ```bash
   sudo dnf update -y
   ```

   - Instalación del servidor local de PostgreSQL 15 y sus contribuciones
   ```bash
   sudo dnf install postgresql15 postgresql15-server glibc-all-langpacks postgresql15-contrib -y
   ```

   - Instalación del cliente de PostgreSQL 15
   ```bash
   sudo dnf install postgresql15 glibc-all-langpacks -y
   ```

4. **Inicializar la base de datos:**

   ```bash
   sudo dnf update -y
   sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
   ```

5. **Habilitar y arrancar el servicio de PostgreSQL:**

   ```bash
   sudo systemctl enable postgresql-15
   sudo systemctl start postgresql-15
   ```

6. **Instalar el módulo PDO para PostgreSQL (para PHP):**

   ```bash
   sudo dnf install php-pgsql -y
   ```

---

## 2. Configuración del Usuario PostgreSQL

### 2.1 Configurar Acceso y Roles

1. **Acceder a la consola de PostgreSQL:**

   ```bash
   sudo -i -u postgres psql
   ```

2. **Cambiar la contraseña del usuario `postgres`:**

   ```sql
   ALTER USER postgres PASSWORD 'tucontraseña';
   ```

3. **Crear un nuevo usuario:**

   ```sql
   CREATE USER {user} WITH PASSWORD 'tucontraseña';
   ```

4. **Asignar roles al nuevo usuario:**

   ```sql
   ALTER ROLE {user} WITH LOGIN;
   ALTER ROLE {user} WITH CREATEDB;
   ALTER ROLE {user} WITH SUPERUSER;
   ```

5. **Crear una base de datos y asignarle el dueño:**

   ```sql
   CREATE DATABASE {db} WITH OWNER {user};
   ```

6. **Dar privilegios completos sobre la base de datos al usuario:**

   ```sql
   GRANT ALL PRIVILEGES ON DATABASE {db} TO {user};
   ```

7. **Salir de la consola de PostgreSQL:**

   ```bash
   \q
   ```

---

## 3. Configuración de PostgreSQL

### 3.1 Configurar SSL y Ajustar la Configuración

1. **Instalar OpenSSL (si aún no está instalado):**

   ```bash
   sudo dnf install openssl
   ```

2. **Generar certificados SSL:**

   ```bash
   openssl req -new -x509 -days 365 -nodes -text -out server.crt -keyout server.key -subj "/CN={ip-privada-bd}"
   ```

3. **Ajustar permisos de los archivos (opcional):**

   ```bash
   sudo chmod 644 *.crt
   sudo chmod 600 *.key
   ```

4. **Mover los archivos generados a la carpeta de datos de PostgreSQL:**

   ```bash
   sudo mv * /var/lib/pgsql/15/data/
   ```

5. **Cambiar el propietario de los archivos a `postgres:postgres`:**

   ```bash
   sudo chown -R postgres:postgres /var/lib/pgsql/15/data/*
   ```

6. **Editar el archivo `postgresql.conf`:**

   Ubicado en `/var/lib/pgsql/15/data/postgresql.conf`, ajusta las siguientes líneas:
   
   ```conf
   listen_addresses = 'localhost,{ip-privada},{otra-ip}'
   port = {puerto personalizado}
   max_connections = 100
   password_encryption = scram-sha-256
   ssl = on
   ssl_cert_file = '/var/lib/pgsql/15/data/server.crt'
   ssl_key_file = '/var/lib/pgsql/15/data/server.key'
   ```
   {: file='/var/lib/pgsql/15/data/postgresql.conf'}

7. **Configurar el archivo `pg_hba.conf`:**

   Ubicado en `/var/lib/pgsql/15/data/pg_hba.conf`, añade o ajusta la línea para conexiones SSL:
   
   ```conf
   hostssl    {database}    {userdb}    {ip-privada,127.0.0.1/32,otra-ip}    scram-sha-256
   ```
   {: file='/var/lib/pgsql/15/data/pg_hba.conf'}

8. **Reiniciar el servicio de PostgreSQL para aplicar los cambios:**

   ```bash
   sudo systemctl restart postgresql-15
   ```

### 3.2 Conectar a la Base de Datos y Configurar Certificados en el Lado del Cliente

1. **Conectarse a la base de datos desde la consola de PostgreSQL para verificar la conexión:**

   - Cambiar a usuario postgres y conectarse a la base de datos:
   ```bash
   sudo su - postgres
   psql -h {host} -p {port} -U {userdb} -d {database}
   ```

2. **Conectarse directamente sin cambiar de usuario para verificar la conexión:**

   ```bash
   sudo -i -u postgres psql -h {host} -p {port} -U {userdb} -d {database}
   ```

3. **Copiar el contenido del certificado `server.crt` al droplet web:**

   - **En el servidor de la base de datos (BD):**

     ```bash
     sudo cat /var/lib/pgsql/15/data/server.crt
     ```

   - **En el servidor web (copiando el contenido del archivo .crt de la BD):**
     
     ```bash
     mkdir .postgresql && cd .postgresql
     nano root.crt
     ```
     
     Luego, asigna los permisos adecuados:
     
     ```bash
     sudo chmod 644 *.crt
     sudo setsebool -P httpd_can_network_connect on
     sudo chcon -R -t httpd_sys_rw_content_t /home/{user}/.postgresql
     ```

---

## 4. Agregar Variables de Entorno en Laravel

Configura tu archivo `.env` para conectarte a PostgreSQL:

   ```js
   DB_CONNECTION=pgsql
   DB_HOST={host}
   DB_PORT={port}
   DB_DATABASE={database}
   DB_USERNAME={user}
   DB_PASSWORD={password}
   // Opciones de SSL: "verify-full" para validar todo o "verify-ca" para solo el certificado
   // Recomendable "Verify-ca" para la practica
   DB_SSLMODE={verify-full / verify-ca}
   ```
   {: file='.env'}

---

## 5. Ejecutar Migraciones y Seeders en Laravel

Una vez configurado PostgreSQL y actualizado el archivo `.env`, ejecuta las migraciones y/o seeders:

   ```bash
   php artisan migrate:fresh --seed
   ```

---

## Conclusiones

En este artículo has aprendido a instalar y configurar PostgreSQL en un servidor VPS de Digital Ocean. Hemos configurado usuarios, bases de datos, conexiones seguras mediante SSL, y ajustado las variables de entorno en un proyecto Laravel.

> Cualquier duda o comentario, agregarla en la sección de comentarios abajo de cada publicación.
{: .prompt-tip }
