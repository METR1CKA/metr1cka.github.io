---
title: Practica 5 - Cloudflare
date: 2025-02-10
categories: [Cloudflare]
tags: [Cloudflare]
---

## Introducción

En este artículo te explicaré cómo configurar Cloudflare SSL para tu dominio. Verás cómo crear una cuenta en Cloudflare, agregar tu dominio, configurar el modo SSL en **Full (strict)** y generar un certificado en Origin Server para autenticar los orígenes. Además, aprenderás a copiar los server names a los DNS de tu dominio y a obtener las claves PEM y PRIVATE KEY necesarias para la configuración de Nginx.

---

## Cloudflare

**Cloudflare** es una empresa de servicios de red que proporciona una red de entrega de contenido, servicios de seguridad de Internet y servicios de servidores de nombres distribuidos. Cloudflare es conocido por su servicio de DNS, pero también ofrece una serie de servicios de seguridad y rendimiento, como la protección contra DDoS y la aceleración de sitios web.

## 1. Crear Cuenta en Cloudflare

1. **Acceder a Cloudflare:**

   - Visita el sitio oficial para crear tu cuenta:  
     [Crear cuenta](https://www.cloudflare.com/)

---

## 2. Tener un Dominio ya Comprado

Asegúrate de contar con un dominio previamente adquirido, ya que será necesario para agregarlo a Cloudflare.

---

## 3. Agregar el Dominio a Cloudflare

1. **Agregar el sitio:**

   - Inicia sesión en Cloudflare y selecciona **Add site** para agregar tu dominio.

2. **Verificar DNS:**

   - Sigue las instrucciones para que Cloudflare detecte y configure los registros DNS de tu dominio.

---

## 4. Configurar SSL en Cloudflare

### 4.1 Copiar los Server Names a los DNS de tu Dominio

1. **Obtener Server Names:**

   - Una vez agregado el dominio, copia los server names que Cloudflare te proporciona.
   
2. **Actualizar DNS:**

   - Ingresa al panel de control de tu registrador de dominio y reemplaza los DNS actuales con los server names de Cloudflare.

### 4.2 Configurar el Modo de SSL

1. **Ir a la sección SSL/TLS:**

   - Dentro del panel de Cloudflare, selecciona la pestaña **SSL/TLS**.

2. **Configurar el Modo de SSL:**

   - Cambia el modo de SSL a **Full (strict)** para garantizar conexiones seguras entre Cloudflare y tu servidor.

### 4.3 Configurar Authenticated Origin Pulls

1. **Acceder a Origin Server:**

   - Dentro de la sección **SSL/TLS**, busca la opción **Origin Server** y selecciona **Authenticated Origin Pulls**.

2. **Crear un Certificado en Origin Server:**

   - Haz clic en **Create Certificate** para iniciar la generación del certificado.

3. **Generar el Certificado:**

   - Selecciona RSA 2048 como algoritmo.
   - Lista los dominios que abarcará el certificado (por ejemplo, `*.dominio` y `dominio`).

4. **Guardar las Claves:**

   - Copia y guarda las claves PEM y la PRIVATE KEY que se generen, ya que las necesitarás para la configuración en Nginx.

---

## 5. Integración con Nginx

Para ver un ejemplo de configuración SSL en Nginx utilizando el certificado generado, consulta el siguiente enlace:

[SSL NGINX](https://metr1cka.github.io/posts/servidor-web/#6-archivos-adicionales-de-configuraci%C3%B3n-de-nginx)

---

## Conclusiones

En este artículo has aprendido a configurar Cloudflare SSL para tu dominio. Has visto cómo crear una cuenta en Cloudflare, agregar tu dominio, configurar el modo SSL en **Full (strict)** y generar un certificado en Origin Server para autenticar los orígenes. Además, has aprendido a copiar los server names a los DNS de tu dominio y a obtener las claves PEM y PRIVATE KEY necesarias para la configuración de Nginx.

> Cualquier duda o comentario, agregarla en la sección de comentarios abajo de cada publicación.
{: .prompt-tip }
