---
title: Primeros Pasos
date: 2025-01-15
categories: [Digital Ocean, VPS, Linux, SSH]
tags: [Digital Ocean, VPS, Linux, SSH]
---

## Introducción

**SSH** (Secure Shell) es un protocolo de red que permite a los usuarios acceder de forma segura a un servidor remoto a través de una conexión cifrada. SSH se utiliza comúnmente para administrar servidores remotos de forma segura, ya que proporciona una forma segura de autenticación y cifrado de datos. En este artículo, veremos cómo utilizar SSH para acceder a un servidor remoto y ejecutar comandos en él.

## Conexión a un servidor remoto

Una vez creado el droplet en Digital Ocean, se puede acceder a él utilizando SSH para configurarlo y administrarlo.

### Windows
Si lo prefieres, para los usuarios de windows, se puede acceder utilizando Putty, ya hay muchos tutoriales que lo explican. Descarga Putty desde [aquí](https://www.putty.org/).

### Linux y Mac

Para los usuarios de Linux y Mac, se puede acceder al servidor de varias maneras. La forma más común de acceder a un servidor remoto es a través de la terminal utilizando el comando `ssh`.

Algunos ejemplos de cómo se puede acceder a un servidor remoto utilizando SSH son los siguientes:

#### Terminal
Puedes acceder con la terminal de la siguiente manera:

```console
ssh {usuario}@{direccion_ip} -p {puerto} -i {ruta_de_la_clave_privada}
```

O bien puedes crear un archivo de configuración en `~/.ssh/config` y agregar la siguiente configuración:

```console
Host nombre
    HostName {direccion_ip}
    User {usuario}
    Port {puerto}
    IdentityFile {ruta de la clave privada}
```

Luego, puedes acceder al servidor con el siguiente comando:

```console
ssh nombre
```

### Cualquier sistema operativo

#### Termius

Termius es un cliente SSH que permite a los usuarios acceder a servidores remotos de forma segura. Termius es una aplicación multiplataforma que se puede utilizar en Windows, Mac, Linux, iOS y Android. Termius es una excelente alternativa a la terminal de línea de comandos para acceder a servidores remotos de forma segura. Termius es una aplicación de código abierto que se puede descargar de forma gratuita desde su sitio web oficial [aquí](https://termius.com/).

## Primeros pasos

Una vez que se haya conectado al servidor remoto, debera hacer lo siguiente:

1. Actualizar los droplets

```console
sudo dnf update
sudo dnf upgrade

dnf update -y && dnf upgrade -y
```

2. Reiniciar

```console
reboot
```

3. Instalar nano vim y net-tools

```console
dnf install nano vim net-tools -y
```

4. Actualizar despues de instalar

```console
dnf update -y && dnf upgrade -y
```

Después de realizar estos pasos, debe de seguir estos otros pasos:

1. Apagar el droplet desde el panel de Digital Ocean

2. Aumentar la capacidad de memoria ram desde el panel de digital ocean

3. Encender el droplet

4. Conectarse al droplet

Ahora ya puedes seguir con la configuración de tu servidor.

## Conclusión

En este artículo, hemos visto cómo utilizar SSH para acceder a un servidor remoto y ejecutar comandos en él. SSH es una herramienta poderosa que se utiliza comúnmente para administrar servidores remotos de forma segura. Espero que este artículo te haya sido útil y te haya ayudado a aprender cómo utilizar SSH para acceder a un servidor remoto.
