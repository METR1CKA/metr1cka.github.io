---
title: Práctica 1 - Configuración Inicial
date: 2025-01-16
categories: [Digital Ocean, VPS, Linux, RockyLinux, SSH]
tags: [Digital Ocean, VPS, Linux, RockyLinux, SSH]
---

## Introducción

**SSH** (Secure Shell) es un protocolo de red que permite acceder de forma segura a un servidor remoto mediante una conexión cifrada. Se utiliza habitualmente para administrar servidores de manera segura, ofreciendo autenticación y cifrado de datos. En este artículo veremos cómo configurar SSH en dos servidores VPS (o droplets) de DigitalOcean, configurando diferentes puertos de escucha y siguiendo buenas prácticas de seguridad.

## Configuración de SSH

En este tutorial, vamos a aprender a configurar SSH en dos servidores VPS o droplets de **DigitalOcean**, específicamente para permitir conexiones SSH en puertos diferentes para cada servidor, mejorando así la seguridad al evitar ataques de fuerza bruta.

> **Nota:** Antes de realizar esta configuración, si deseas integrar protección de firewall interno y baneo de ips, deberas de consultar el artículo de [Configuraciones Adicionales](https://metr1cka.github.io/posts/extras/), ya que antes de cambiar los puertos de SSH deberas configurar dichos apartados. De no ser así, podrías quedar bloqueado de tu servidor.
{: .prompt-warning }

### Cambio de Puertos

1. Edita el archivo de configuración:
   ```bash
   nano /etc/ssh/sshd_config
   ```

2. Cambia el puerto para el droplet 1:
   ```bash
   Port 55113
   ```
   {: file='/etc/ssh/sshd_config'}

3. Cambia el puerto para el droplet 2:
   ```bash
   Port 60112
   ```
   {: file='/etc/ssh/sshd_config'}

### Reiniciar el Servicio SSH

1. Reinicia el servicio:
   ```bash
   systemctl restart sshd
   ```
   
   O alternativamente:
   
   ```bash
   service sshd restart
   ```

2. Verifica el estado:
   ```bash
   systemctl status sshd
   ```
   
   O:
   
   ```bash
   service sshd status
   ```

> Ambos métodos son válidos.
{: .prompt-info }

## Creación de Usuarios

Para mayor seguridad, crea un usuario distinto de root para acceder por SSH. Ejecuta estos pasos en ambos droplets:

1. **Crear un usuario:**
   ```bash
   adduser {usuario}
   ```

2. **Asignar una contraseña:**
   ```bash
   passwd {usuario}
   ```

3. **Cambiar al usuario para probar el acceso:**
   ```bash
   su {usuario}
   ```

4. **Ir al directorio home:**
   ```bash
   cd
   ```

5. **Salir del usuario:**
   ```bash
   exit
   ```

6. **Comandos extras y/o opcionales**

    - **Borrar un usuario:**
    ```bash
    userdel -r {usuario}
    ```

    - **Agregar el usuario al grupo `wheel` para usar `sudo`:**
    ```bash
    gpasswd -a {usuario} wheel
    ```

    - **Eliminar el usuario del grupo `wheel`:**
    ```bash
    gpasswd -d {usuario} wheel
    ```

    - **Asignar el directorio home:**
    ```bash
    chown -R {usuario}:{usuario} /home/{usuario}
    ```

> Configura los accesos y demás ajustes con root; los usuarios creados solo podrán conectarse por SSH. Se debera crear un usuario para acceso SSH y otro con permisos de sudo para cada droplet.
{: .prompt-warning }

## Configuración de la Carpeta .SSH y Acceso entre Droplets

1. **Crear la carpeta `.ssh` y el archivo `authorized_keys` en ambos droplets:**

   - Puedes crear la carpeta de `.ssh` dentro del directorio home del usuario:
   ```bash
   mkdir ~/.ssh
   cd ~/.ssh
   ```

   - Otra opción es crear la carpeta de forma personalizada dentro del directorio home del usuario (no recomendado):
   ```bash
   mkdir ~/{carpeta-personalizada}
   cd ~/{carpeta-personalizada}
   mkdir .ssh
   cd .ssh
   ```

   - Luego crear el archivo `authorized_keys` y copiar la clave pública del usuario:
   ```bash
   touch authorized_keys
   ```
   {: file='~/ruta_ssh/'}

   - Una opción es asignar el contenido del archivo `authorized_keys` de root al que acabas de crear:
   ```bash
   cat /root/.ssh/authorized_keys > ~/{ruta_ssh}/authorized_keys
   ```

   - O copiar el archivo `authorized_keys` de root al nuevo usuario y ajustar los permisos:
   ```bash
   cp /root/.ssh/authorized_keys ~/{ruta_ssh}/authorized_keys
   ```

   - Asigna los permisos adecuados:
   ```bash
   chmod 700 -R ~/{ruta_ssh}
   chmod 600 ~/{ruta_ssh}/authorized_keys
   ```

2. **Configurar el archivo `sshd_config`:**

   Añade o modifica estas líneas:
   ```bash
   AddressFamily inet
   PermitRootLogin no
   AllowUsers {usuario-creado-para-ssh}
   MaxAuthTries 3
   MaxSessions 3

   # Ubicación del archivo de claves:
   AuthorizedKeysFile /home/{usuario-creado-para-ssh}/{ruta_ssh}/authorized_keys
   ```
   {: file='/etc/ssh/sshd_config'}

3. **Reinicia el servicio SSH:**
   ```bash
   systemctl restart sshd
   ```

## Creación de Llaves SSH

1. **Generar la llave SSH en el droplet 1:**

   - Ingresa al usuario creado:
   ```bash
   su {usuario}
   ```

   - **(Opcional)** Genera la llave con nombre personalizado:
   ```bash
   ssh-keygen -f {ruta_ssh/nombre_archivo} -C "{tu-comentario}"
   ```

   - Genera la llave por defecto:
   ```bash
   ssh-keygen -C "{tu-comentario}"
   ```

   - Sal de la sesión del usuario y ajusta los permisos:
   ```bash
   exit
   chmod 644 ~/{ruta_ssh}/nombre_archivo.pub
   chmod 600 ~/{ruta_ssh}/nombre_archivo
   ```

2. **Copiar la llave pública al droplet 2:**

   - Ingresa al usuario creado:
   ```bash
   su {usuario}
   ```

   - **(Opcional)** Usar ssh-copy-id para copiar la llave
   ```bash
   ssh-copy-id -i ~/{ruta_ssh}/nombre_archivo.pub {usuario}@{ip_privada_droplet_2}
   ```

   - O copiar manualmente, ejecuta cat para mostrar la llave y copia el contenido:
   ```bash
   cat {ruta_ssh}/nombre_archivo.pub
   ```

3. **Reinicia el servicio SSH:**
   ```bash
   systemctl restart sshd
   ```

## Configuración del Acceso SSH entre Droplets

1. **Modificar `sshd_config`:**

   - En el droplet 1, asegúrate de que SSH escuche en todas las interfaces:
   ```bash
   #ListenAddress 0.0.0.0
   ListenAddress 0.0.0.0
   #ListenAddress ::
   ```
   {: file='/etc/ssh/sshd_config'}

   - En el droplet 2, especifica la IP privada:
   ```bash
   ListenAddress {ip_privada_droplet_2}
   ```
   {: file='/etc/ssh/sshd_config'}

2. **Reinicia el servicio en ambos droplets:**
   ```bash
   systemctl restart sshd
   ```

## Configuración de Google Authenticator

> **Nota:** Antes de continuar, asegúrate de haber aumentado la capacidad RAM a 2GB, ya que de no ser así, la instalación podría fallar y arrojar un mensaje de error como por ejemplo: `Killed`, a causa de la falta de memoria. También asegúrate de tener instalado Google Authenticator en tu dispositivo móvil.
{: .prompt-warning }

1. **Instalar las dependencias necesarias:**
   ```bash
   dnf install epel-release && dnf update -y
   ```

   ```bash
   dnf install google-authenticator qrencode qrencode-libs
   ```

2. **Generar la llave de Google Authenticator:**
   ```bash
   google-authenticator -s ~/{ruta_ssh}/google_authenticator
   ```
   
   Durante el proceso se te harán varias preguntas:
   - ¿Deseas que los tokens sean basados en tiempo? (responde: y)
   - ¿Quieres que se actualice el archivo de configuración? (responde: y)
   - ¿Deshabilitar el uso múltiple del mismo token? (responde: y)
   - ¿Aumentar la ventana de tiempo? (responde: n)
   - ¿Habilitar rate-limiting? (responde: y)

    > Es recomendable realizarla solo una vez por usuario, aunque no hay problema si eliminas el archivo `google_authenticator` y lo vuelves a generar. Verifica que SELinux no esté bloqueando el acceso a la carpeta `.ssh`.
    {: .prompt-warning }

3. **Realizar respaldos y ajustar SELinux:**

   - Restaurar el contexto de SELinux en la carpeta .ssh
   ```bash
   restorecon -Rv ~/{ruta_ssh}/
   ```

   - Respaldar el archivo PAM para sshd
   ```bash
   cp /etc/pam.d/sshd /etc/pam.d/sshd.bak
   ```

   - Respaldar sshd_config
   ```bash
   cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   ```

4. **Configurar PAM para Google Authenticator:**

   - Edita `/etc/pam.d/sshd` y reemplaza (o agrega) lo siguiente:
   ```bash
   #auth       substack     password-auth
   auth       required     pam_google_authenticator.so secret=/home/${USER}/{ruta_ssh}/google_authenticator nullok
   auth       required     pam_permit.so
   ```
   {: file='/etc/pam.d/sshd'}

    > El **"secret"** debe apuntar a la ubicación (absoluta o relativa) del archivo `google_authenticator`.
    {: .prompt-warning }

5. **Configurar SSH para usar Google Authenticator:**
   - Edita `/etc/ssh/sshd_config.d/50-redhat.conf` y ajusta:
     ```bash
     ChallengeResponseAuthentication yes
     ```
     {: file='/etc/ssh/sshd_config.d/50-redhat.conf'}
   
   - Luego, en `/etc/ssh/sshd_config`, cambia:
   
     ```bash
     PasswordAuthentication no
     KbdInteractiveAuthentication yes
     AuthenticationMethods publickey,password publickey,keyboard-interactive
     ```
     {: file='/etc/ssh/sshd_config'}

6. **Reinicia el servicio SSH y verifica errores:**
   ```bash
   sudo journalctl -u sshd
   ```

7. **Prueba el acceso SSH con Google Authenticator:**

   Una vez autenticado, prueba la conexión al droplet 2:
   ```bash
   ssh {usuario}@{ip_privada_droplet_2} -p {puerto_ssh_droplet_2} -i ~/{ruta_ssh}/nombre_archivo
   ```

   También puedes configurar un archivo `config` en la carpeta `.ssh`:
   ```bash
   Host *
     ServerAliveInterval 60

   Host {nombre_host}
     HostName {ip_privada_droplet_2}
     Port {puerto_ssh_droplet_2}
     User {usuario}
     IdentityFile ~/{ruta_ssh}/nombre_archivo
   ```
   {: file='~/.ssh/ssh_config'}

---

## Conclusiones

En este tutorial, hemos configurado SSH en dos servidores VPS de DigitalOcean, cambiando los puertos de escucha y siguiendo buenas prácticas de seguridad. También hemos creado usuarios, configurado la carpeta `.ssh` y el acceso entre droplets, y configurado Google Authenticator para autenticación de dos factores del primer droplet, puedes integrar eso mismo al segundo si gustas.

> Cualquier duda o comentario, agregarla en la sección de comentarios abajo de cada publicación.
{: .prompt-tip }
