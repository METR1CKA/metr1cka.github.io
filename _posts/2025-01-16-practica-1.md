---
title: Practica 1 - Configuración Inicial de SSH en VPS
date: 2025-01-15
categories: [Digital Ocean, VPS, Linux, SSH]
tags: [Digital Ocean, VPS, Linux, SSH]
---

## Introducción

**SSH** (Secure Shell) es un protocolo de red que permite acceder de forma segura a un servidor remoto mediante una conexión cifrada. Se utiliza habitualmente para administrar servidores de manera segura, ofreciendo autenticación y cifrado de datos. En este artículo veremos cómo configurar SSH en dos servidores VPS (o droplets) de DigitalOcean, configurando diferentes puertos de escucha y siguiendo buenas prácticas de seguridad.

> Nota: Recuerda que el comando `sudo` se utiliza para ejecutar acciones que requieren privilegios elevados. No es necesario usarlo siempre, solo cuando no seas usuario root o no tengas privilegios suficientes (por ejemplo, si no estás en el grupo sudo/wheel).
{: .prompt-warning }

## Configuración de SSH

En este tutorial, vamos a aprender a configurar SSH en dos servidores VPS o droplets de **DigitalOcean**, específicamente para permitir conexiones SSH en puertos diferentes para cada servidor, mejorando así la seguridad al evitar ataques de fuerza bruta.

### Cambio de Puertos

1. Edita el archivo de configuración:
   ```console
   nano /etc/ssh/sshd_config
   ```

2. Cambia el puerto para el droplet 1:
   ```console
   Port 55113
   ```

3. Cambia el puerto para el droplet 2:
   ```console
   Port 60112
   ```

### Reiniciar el Servicio SSH

1. Reinicia el servicio:
   ```console
   systemctl restart sshd
   ```
   
   O alternativamente:
   
   ```console
   service sshd restart
   ```

2. Verifica el estado:
   ```console
   systemctl status sshd
   ```
   
   O:
   
   ```console
   service sshd status
   ```

> Ambos métodos son válidos.
{: .prompt-info }

## Creación de Usuarios

Para mayor seguridad, crea un usuario distinto de root para acceder por SSH. Ejecuta estos pasos en ambos droplets:

1. **Crear un usuario:**
   ```console
   adduser {usuario}
   ```

2. **Asignar una contraseña:**
   ```console
   passwd {usuario}
   ```

3. **(Opcional) Borrar un usuario:**
   ```console
   userdel -r {usuario}
   ```

4. **(Opcional) Agregar el usuario al grupo `wheel` para usar `sudo`:**
   ```console
   gpasswd -a {usuario} wheel
   ```

5. **Eliminar el usuario del grupo `wheel`:**
   ```console
   gpasswd -d {usuario} wheel
   ```

6. **Asignar el directorio home (opcional):**
   ```console
   chown -R {usuario}:{usuario} /home/{usuario}
   ```

7. **Cambiar al usuario para probar el acceso:**
   ```console
   su {usuario}
   ```

8. **Ir al directorio home:**
   ```console
   cd
   ```

9. **Salir del usuario:**
   ```console
   exit
   ```

> Configura los accesos y demás ajustes con root; los usuarios creados solo podrán conectarse por SSH.
{: .prompt-warning }

## Configuración de la Carpeta .SSH y Acceso entre Droplets

1. **Crear la carpeta `.ssh` y el archivo `authorized_keys` en ambos droplets:**
   Puedes crear la carpeta de forma personalizada dentro del directorio home del usuario:
   
   ```console
   # (Opcional) Crear una carpeta personalizada
   mkdir ~/{carpeta}
   cd ~/{carpeta}

   # Crear la carpeta .ssh y entrar en ella
   mkdir .ssh
   cd .ssh

   # Crear el archivo authorized_keys
   touch authorized_keys
   ```

   Otra opción es copiar el archivo de root:
   
   ```console
   cat /root/.ssh/authorized_keys > ~/{ruta_ssh}/authorized_keys
   ```
   
   O:
   
   ```console
   cp /root/.ssh/authorized_keys ~/{ruta_ssh}/authorized_keys
   ```
   
   Asigna los permisos adecuados:
   
   ```console
   chmod 700 -R ~/{ruta_ssh}
   chmod 600 ~/{ruta_ssh}/authorized_keys
   ```

2. **Configurar el archivo `sshd_config`:**
   Añade o modifica estas líneas:
   
   ```console
   AddressFamily inet
   PermitRootLogin no
   AllowUsers {usuario}
   MaxAuthTries 3
   MaxSessions 3

   # Ubicación del archivo de claves:
   AuthorizedKeysFile /home/{usuario}/{ruta_ssh}/authorized_keys
   ```

3. **Reinicia el servicio SSH:**
   ```console
   systemctl restart sshd
   ```

## Creación de Llaves SSH

1. **Generar la llave SSH en el droplet 1:**
   ```console
   # Cambiar al usuario creado
   su {usuario}

   # Generar la llave con nombre personalizado o usar la ruta por defecto
   ssh-keygen -f {ruta_ssh/nombre_archivo} -C "{tu-comentario}"
   # O para usar la ruta por defecto:
   ssh-keygen -C "{tu-comentario}"

   # Salir del usuario
   exit

   # Ajustar permisos (como root)
   chmod 644 ~/{ruta_ssh}/nombre_archivo.pub
   chmod 600 ~/{ruta_ssh}/nombre_archivo
   ```

2. **Copiar la llave pública al droplet 2:**
   ```console
   # Cambiar al usuario creado
   su {usuario}

   # Usar ssh-copy-id para copiar la llave
   ssh-copy-id -i ~/{ruta_ssh}/nombre_archivo.pub {usuario}@{ip_privada_droplet_2}

   # O copiar manualmente:
   cat {ruta_ssh}/nombre_archivo.pub
   ```

3. **Reinicia el servicio SSH:**
   ```console
   systemctl restart sshd
   ```

## Configuración del Acceso SSH entre Droplets

1. **Modificar `sshd_config`:**
   En el droplet 1, asegúrate de que SSH escuche en todas las interfaces:
   
   ```console
   # Droplet 1
   #ListenAddress 0.0.0.0
   ListenAddress 0.0.0.0
   #ListenAddress ::
   ```

   En el droplet 2, especifica la IP privada:
   
   ```console
   ListenAddress {ip_privada_droplet_2}
   ```

2. **Reinicia el servicio en ambos droplets:**
   ```console
   systemctl restart sshd
   ```

## Configuración de Google Authenticator

1. **Instalar las dependencias necesarias:**
   ```console
   # Instalar EPEL
   dnf install epel-release

   # Instalar google-authenticator y qrencode
   dnf install google-authenticator qrencode qrencode-libs
   ```

2. **Generar la llave de Google Authenticator:**
   ```console
   google-authenticator -s ~/{ruta_ssh}/google_authenticator
   ```
   
   Durante el proceso se te harán varias preguntas:
   - ¿Deseas que los tokens sean basados en tiempo? (responde: y)
   - ¿Quieres que se actualice el archivo de configuración? (responde: y)
   - ¿Deshabilitar el uso múltiple del mismo token? (responde: y)
   - ¿Aumentar la ventana de tiempo? (responde: n)
   - ¿Habilitar rate-limiting? (responde: y)

> Realiza la autenticación de Google Authenticator solo una vez por usuario. Si se vuelve a ejecutar, deberás crear otro usuario y borrar el anterior.
{: .prompt-warning }

3. **Realizar respaldos y ajustar SELinux:**
   ```console
   # Restaurar el contexto de SELinux en la carpeta .ssh
   restorecon -Rv ~/{ruta_ssh}/

   # Respaldar el archivo PAM para sshd
   cp /etc/pam.d/sshd /etc/pam.d/sshd.bak

   # Respaldar sshd_config
   cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   ```

4. **Configurar PAM para Google Authenticator:**
   Edita `/etc/pam.d/sshd` y reemplaza (o agrega) lo siguiente:
   
   ```console
   #auth       substack     password-auth
   auth       required     pam_google_authenticator.so secret=/home/${USER}/{ruta_ssh}/google_authenticator nullok
   auth       required     pam_permit.so
   ```

> El "secret" debe apuntar a la ubicación (absoluta o relativa) del archivo `google_authenticator`.
{: .prompt-warning }

5. **Configurar SSH para usar Google Authenticator:**
   - Edita `/etc/ssh/sshd_config.d/50-redhat.conf` y ajusta:
   
     ```console
     ChallengeResponseAuthentication yes
     ```
   
   - Luego, en `/etc/ssh/sshd_config`, cambia:
   
     ```console
     PasswordAuthentication no
     KbdInteractiveAuthentication yes
     AuthenticationMethods publickey,password publickey,keyboard-interactive
     ```

6. **Reinicia el servicio SSH y verifica errores:**
   ```console
   sudo journalctl -u sshd
   ```

7. **Prueba el acceso SSH con Google Authenticator:**
   Una vez autenticado, prueba la conexión al droplet 2:
   
   ```console
   ssh {usuario}@{ip_privada_droplet_2} -p {puerto_ssh_droplet_2} -i ~/{ruta_ssh}/nombre_archivo
   ```

   También puedes configurar un archivo `config` en la carpeta `.ssh`:
   
   ```console
   Host *
     ServerAliveInterval 60

   Host {nombre_host}
     HostName {ip_privada_droplet_2}
     Port {puerto_ssh_droplet_2}
     User {usuario}
     IdentityFile ~/{ruta_ssh}/nombre_archivo
   ```

## Configuración Extra

### Firewall

1. **Instalar y habilitar firewalld:**
   ```console
   sudo dnf install firewalld -y
   sudo systemctl enable firewalld
   sudo systemctl start firewalld
   ```

2. **Remover servicios por defecto:**
   ```console
   sudo firewall-cmd --zone=public --remove-service=cockpit --permanent
   sudo firewall-cmd --zone=public --remove-service=ssh --permanent
   sudo firewall-cmd --zone=public --remove-service=dhcpv6-client --permanent
   ```

3. **Agregar servicios personalizados:**
   ```console
   sudo firewall-cmd --zone=public --add-port={port}/tcp --permanent
   sudo firewall-cmd --zone=public --add-service=http --permanent
   sudo firewall-cmd --zone=public --add-service=https --permanent
   ```

4. **Recargar reglas y verificar:**
   ```console
   sudo firewall-cmd --reload
   sudo firewall-cmd --state
   sudo firewall-cmd --list-all
   ```

### Añadir Fail2Ban

1. **Instalar Fail2Ban:**
   ```console
   dnf install epel-release -y
   dnf install fail2ban -y
   ```

2. **Habilitar y verificar el servicio:**
   ```console
   systemctl enable fail2ban.service
   systemctl status fail2ban.service
   ```

3. **Configurar Fail2Ban:**
   ```console
   cd /etc/fail2ban
   cp jail.conf jail.local
   cd jail.d
   ```

4. **Editar el archivo de configuración para sshd:**
   ```console
   nano sshd.conf
   ```

   Inserta lo siguiente:
   
   ```console
   [sshd]
   enabled = true
   filter = sshd
   port = {puerto_ssh}
   bantime = 21600
   maxretry = 3
   ignoreip = 127.0.0.1
   logpath = /var/log/auth.log
   ```

5. **Reiniciar Fail2Ban y verificar:**
   ```console
   fail2ban-client status sshd
   fail2ban-client status
   service fail2ban restart
   fail2ban-client status sshd
   ```

6. **Para desbanear una IP:**
   ```console
   sudo fail2ban-client set sshd unbanip {remote-ip-address}
   ```

