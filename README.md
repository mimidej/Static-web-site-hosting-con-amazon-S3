# 🌐 Hosting de un sitio web estático con Amazon S3

En este tutorial aprenderemos a crear una página web estática y publicarla directamente desde **Amazon S3**, utilizando únicamente la **AWS Management Console**.

El objetivo es demostrar cómo podemos utilizar Amazon S3 no solamente para almacenar archivos, sino también para publicar un sitio web estático accesible desde Internet.

> **Objetivo:** construir una pequeña landing page, almacenarla en un bucket de S3 y acceder a ella mediante el endpoint de alojamiento web estático proporcionado por AWS.

---

## 📋 Contenido

* [¿Qué vamos a construir?](#-qué-vamos-a-construir)
* [Requisitos](#-requisitos)
* [Arquitectura](#-arquitectura)
* [1. Crear la página web](#1-crear-la-página-web)
* [2. Crear el bucket de S3](#2-crear-el-bucket-de-s3)
* [3. Habilitar Static Website Hosting](#3-habilitar-static-website-hosting)
* [4. Configurar el acceso público](#4-configurar-el-acceso-público)
* [5. Agregar la Bucket Policy](#5-agregar-la-bucket-policy)
* [6. Subir los archivos](#6-subir-los-archivos)
* [7. Probar el sitio web](#7-probar-el-sitio-web)
* [8. Verificar que funciona](#8-verificar-que-funciona)
* [💰 Costos](#-costos)
* [🔐 Consideraciones de seguridad](#-consideraciones-de-seguridad)
* [🧹 Limpieza de recursos](#-limpieza-de-recursos)
* [🚨 Problemas frecuentes](#-problemas-frecuentes)
* [📚 Conceptos aprendidos](#-conceptos-aprendidos)
* [🚀 Siguientes pasos](#-siguientes-pasos)

---

# 🎯 ¿Qué vamos a construir?

Crearemos una página web sencilla que funcionará como una pequeña landing page personal.

La estructura será:

```text
static-website/
│
├── index.html
├── error.html
└── styles.css
```

Posteriormente subiremos estos archivos a un bucket de Amazon S3:

```text
                    Internet
                       │
                       ▼
                ┌─────────────┐
                │   Browser   │
                └──────┬──────┘
                       │
                       │ HTTP
                       ▼
              ┌─────────────────┐
              │   Amazon S3     │
              │                 │
              │  Bucket         │
              │  ├ index.html   │
              │  ├ error.html   │
              │  └ styles.css   │
              └─────────────────┘
```

S3 se encargará de almacenar los archivos y, mediante su funcionalidad de **Static Website Hosting**, responderá las solicitudes HTTP utilizando el archivo `index.html` como documento principal.

---

# 📚 Requisitos

Para realizar este laboratorio necesitas:

* Una cuenta de AWS.
* Acceso a la **AWS Management Console**.
* Conocimientos básicos de:

  * AWS Regions.
  * Amazon S3.
  * Buckets.
  * Objects.
  * IAM y políticas.
* Un navegador web.
* Un editor de texto o IDE.

No utilizaremos:

* EC2.
* Linux.
* Apache.
* Nginx.
* Docker.
* AWS CLI.
* Terraform.

Todo el laboratorio se realizará desde la **AWS Management Console**.

---

# 🏗️ Arquitectura

La arquitectura final será muy sencilla:

```text
                         INTERNET
                            │
                            │ HTTP
                            ▼
                 ┌─────────────────────┐
                 │    Amazon S3        │
                 │                     │
                 │  Static Website     │
                 │      Hosting        │
                 │                     │
                 │  ┌───────────────┐  │
                 │  │  index.html   │  │
                 │  ├───────────────┤  │
                 │  │  error.html   │  │
                 │  ├───────────────┤  │
                 │  │  styles.css   │  │
                 │  └───────────────┘  │
                 │                     │
                 │  Public Read        │
                 └─────────────────────┘
```

El bucket tendrá una política que permitirá únicamente:

```text
s3:GetObject
```

a usuarios anónimos.

Esto significa que cualquier persona podrá **leer los archivos publicados**, pero no tendrá permisos para:

* Subir archivos.
* Modificar archivos.
* Eliminar archivos.
* Administrar el bucket.

---

# 1. Crear la página web

Antes de tocar AWS vamos a crear el sitio que posteriormente almacenaremos en S3.

## 1.1 Crear `index.html`

Crea un archivo llamado:

```text
index.html
```

Utiliza el siguiente contenido:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Mi sitio en AWS</title>

    <link rel="stylesheet" href="styles.css">
</head>

<body>

    <main class="container">

        <h1>Hola, AWS 👋</h1>

        <h2>Mi sitio web estático en Amazon S3</h2>

        <p>
            Esta página está almacenada en un bucket de Amazon S3
            y publicada directamente desde AWS.
        </p>

        <p>
            No utilizamos servidores tradicionales para alojar este sitio.
        </p>

        <a href="https://aws.amazon.com/s3/" target="_blank">
            Conocer Amazon S3
        </a>

    </main>

</body>
</html>
```

---

## 1.2 Crear `styles.css`

Ahora crea:

```text
styles.css
```

Con el siguiente contenido:

```css
body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f4f4f4;
}

.container {
    max-width: 800px;
    margin: 100px auto;
    padding: 40px;
    text-align: center;
    background: white;
    border-radius: 10px;
}

h1 {
    font-size: 3rem;
}

h2 {
    font-weight: normal;
}

p {
    line-height: 1.6;
}

a {
    display: inline-block;
    margin-top: 20px;
    text-decoration: none;
}
```

---

## 1.3 Crear `error.html`

También crearemos una página para errores.

Crea:

```text
error.html
```

Con:

```html
<!DOCTYPE html>
<html lang="es">

<head>
    <meta charset="UTF-8">

    <title>Error - Página no encontrada</title>

    <link rel="stylesheet" href="styles.css">
</head>

<body>

    <main class="container">

        <h1>404</h1>

        <h2>Página no encontrada</h2>

        <p>
            Lo sentimos, la página que estás buscando no existe.
        </p>

        <a href="index.html">
            Regresar al inicio
        </a>

    </main>

</body>

</html>
```

---

## 1.4 Probar la página localmente

Antes de subir cualquier cosa a AWS, es recomendable comprobar que nuestro sitio funciona.

La estructura debe quedar:

```text
static-website/
│
├── index.html
├── error.html
└── styles.css
```

Abre `index.html` directamente en tu navegador.

Deberías observar una página similar a:

```text
        Hola, AWS 👋

  Mi sitio web estático
      en Amazon S3

Esta página está almacenada
en un bucket de Amazon S3...
```

Si el HTML y CSS funcionan correctamente localmente, podemos continuar.

---

# 2. Crear el bucket de S3

Ahora vamos a crear el almacenamiento que utilizará nuestro sitio.

Ingresa a la consola de AWS:

https://console.aws.amazon.com/s3/

Busca **S3** y entra al servicio.

---

## 2.1 Crear un bucket

Selecciona:

```text
Create bucket
```

En **General configuration** selecciona la región donde quieras crear el bucket.

Por ejemplo:

```text
US West (Oregon) us-west-2
```

> Utiliza la región que prefieras. El endpoint del sitio dependerá de la región seleccionada.

---

## 2.2 Nombre del bucket

Escribe un nombre globalmente único.

Por ejemplo:

```text
mimidej-static-website-2026
```

El nombre real debe ser único dentro de Amazon S3.

Puedes utilizar algo como:

```text
mi-nombre-static-website-12345
```

---

## 2.3 Object Ownership

Deja:

```text
ACLs disabled
```

y:

```text
Bucket owner enforced
```

Para este laboratorio utilizaremos una **Bucket Policy** para proporcionar acceso público.

No necesitamos ACLs.

---

## 2.4 Block Public Access

Aquí encontraremos una de las partes más importantes del laboratorio.

AWS bloquea el acceso público a los buckets de S3 de forma predeterminada.

Para este laboratorio vamos a publicar deliberadamente el bucket.

Desmarca:

```text
Block all public access
```

AWS mostrará una advertencia.

Lee la advertencia y confirma que entiendes que el contenido podrá ser accesible públicamente.

Después selecciona:

```text
I acknowledge that the current settings might result
in this bucket and the objects within becoming public.
```

> ⚠️ **IMPORTANTE:** no hagas esto con un bucket que contenga información privada, credenciales, respaldos, documentos personales o cualquier otro archivo que no quieras publicar.

Finalmente selecciona:

```text
Create bucket
```

---

# 3. Habilitar Static Website Hosting

Una vez creado el bucket, entra en él.

Selecciona:

```text
Properties
```

Busca la sección:

```text
Static website hosting
```

Selecciona:

```text
Edit
```

---

## 3.1 Activar el alojamiento web

Selecciona:

```text
Enable
```

En:

```text
Hosting type
```

selecciona:

```text
Host a static website
```

En:

```text
Index document
```

escribe exactamente:

```text
index.html
```

El nombre es **sensible a mayúsculas y minúsculas**.

En:

```text
Error document
```

escribe:

```text
error.html
```

Guarda los cambios.

AWS habilitará el alojamiento web estático y mostrará un **Website endpoint**.

Guarda este endpoint porque lo utilizaremos al final.

---

# 4. Configurar el acceso público

Ahora debemos permitir que usuarios de Internet puedan leer los objetos.

Ve a:

```text
Permissions
```

Busca:

```text
Block public access
```

Comprueba que el bloqueo de acceso público esté deshabilitado para este bucket.

Si AWS no permite modificarlo, revisa también la configuración de Block Public Access a nivel de cuenta u organización.

S3 aplica la configuración más restrictiva entre los distintos niveles.

---

# 5. Agregar la Bucket Policy

Ahora necesitamos indicar explícitamente qué puede hacer un usuario público.

Busca:

```text
Bucket policy
```

Selecciona:

```text
Edit
```

Agrega la siguiente política:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::NOMBRE-DE-TU-BUCKET/*"
        }
    ]
}
```

### ⚠️ Importante

Reemplaza:

```text
NOMBRE-DE-TU-BUCKET
```

por el nombre real de tu bucket.

Por ejemplo:

```json
"Resource": "arn:aws:s3:::mimidej-static-website-2026/*"
```

La parte:

```text
/*
```

es importante porque significa que la política se aplica a los objetos contenidos dentro del bucket.

---

## ¿Qué significa esta política?

Vamos a analizarla:

### `Principal`

```json
"Principal": "*"
```

Significa:

> Cualquier usuario.

---

### `Action`

```json
"Action": "s3:GetObject"
```

Significa:

> El usuario puede solicitar/leer objetos.

---

### `Effect`

```json
"Effect": "Allow"
```

Significa:

> Permitimos esta acción.

---

### `Resource`

```json
"Resource": "arn:aws:s3:::NOMBRE-DE-TU-BUCKET/*"
```

Indica sobre qué recursos aplica la política.

---

Por lo tanto:

```text
Internet
   │
   │ GET
   ▼
S3 Bucket
   │
   └── PublicReadGetObject
            │
            └── s3:GetObject
```

El usuario puede leer:

```text
index.html
styles.css
error.html
```

Pero no puede realizar:

```text
PUT
DELETE
LIST
```

ni administrar el bucket.

AWS documenta este patrón específicamente para sitios web estáticos públicos.

---

# 6. Subir los archivos

Ahora regresa a:

```text
Objects
```

Selecciona:

```text
Upload
```

Sube:

```text
index.html
error.html
styles.css
```

No necesitamos crear carpetas.

La estructura dentro del bucket debe ser:

```text
Bucket
│
├── index.html
├── error.html
└── styles.css
```

Selecciona:

```text
Upload
```

y espera a que los tres archivos terminen de cargarse.

---

# 7. Probar el sitio web

Regresa a:

```text
Properties
```

Busca nuevamente:

```text
Static website hosting
```

Ahí encontraremos:

```text
Website endpoint
```

Será una dirección similar a:

```text
http://NOMBRE-DE-TU-BUCKET.s3-website-us-west-2.amazonaws.com
```

El formato exacto depende de la región.

Abre el endpoint en una nueva pestaña.

Si todo está correctamente configurado, deberíamos ver nuestra página:

```text
Hola, AWS 👋

Mi sitio web estático
en Amazon S3
```

🎉 **¡Nuestro sitio web está publicado!**

---

# 8. Verificar que funciona

No nos vamos a quedar únicamente con "la página abrió".

Vamos a comprobar qué configuramos realmente.

## 8.1 Verificar `index.html`

Abre:

```text
http://TU-ENDPOINT/
```

Deberá mostrarse:

```text
index.html
```

porque lo configuramos como:

```text
Index document
```

---

## 8.2 Verificar CSS

Si nuestra página tiene los estilos aplicados:

```text
background
fuente
espaciado
botones
```

significa que el navegador también pudo obtener:

```text
styles.css
```

desde S3.

---

## 8.3 Verificar `error.html`

Prueba agregar una ruta inexistente:

```text
http://TU-ENDPOINT/pagina-inexistente.html
```

S3 debería devolver nuestro documento:

```text
error.html
```

en lugar de la página principal.

---

# 🧠 ¿Qué acabamos de hacer?

Aunque nuestra página es sencilla, acabamos de utilizar varios conceptos importantes de AWS.

Tenemos:

```text
HTML
  │
  ▼
CSS
  │
  ▼
S3 Object
  │
  ▼
S3 Bucket
  │
  ├── Bucket Policy
  │
  └── Static Website Hosting
           │
           ▼
       HTTP Endpoint
           │
           ▼
        Internet
```

No existe:

```text
EC2
Apache
Nginx
Windows Server
Linux Server
Docker
```

S3 proporciona el almacenamiento y el endpoint de alojamiento web estático.

---

# 🔐 Consideraciones de seguridad

Este laboratorio utiliza una configuración que **deliberadamente hace público el contenido del bucket**.

Esto es importante entenderlo antes de utilizar esta arquitectura en un proyecto real.

AWS recomienda mantener habilitado **Block Public Access** siempre que no sea necesario exponer públicamente un bucket.

En este laboratorio lo deshabilitamos únicamente porque queremos aprender cómo funciona el alojamiento web estático directo de S3.

## Nunca publiques información sensible

No subas a este bucket:

```text
.env
credentials
access keys
secret keys
contraseñas
tokens
archivos privados
bases de datos
respaldos
documentos personales
```

Por ejemplo, **nunca** deberías subir algo como:

```text
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
```

Una vez que un objeto es público, cualquier persona con acceso a Internet puede solicitarlo.

---

# ⚠️ HTTP y HTTPS

Hay otra consideración importante.

El endpoint nativo de **Static Website Hosting de S3 utiliza HTTP**.

Por ejemplo:

```text
http://bucket.s3-website-region.amazonaws.com
```

Los endpoints de website hosting de S3 no soportan HTTPS directamente.

Por eso este laboratorio tiene principalmente un objetivo **educativo**.

Para una aplicación real, una arquitectura más recomendable sería:

```text
              Internet
                  │
                HTTPS
                  │
                  ▼
          ┌───────────────┐
          │  CloudFront   │
          └───────┬───────┘
                  │
                  ▼
          ┌───────────────┐
          │      S3       │
          │    privado    │
          └───────────────┘
```

Con CloudFront podemos mantener el bucket privado y utilizar HTTPS mediante una arquitectura más segura. AWS recomienda este enfoque para sitios estáticos que requieren seguridad adicional.

---

# 💰 Costos

Una de las ventajas de este ejercicio es que estamos utilizando un servicio **serverless y basado en consumo**.

No necesitamos pagar por:

```text
Servidor EC2
Sistema operativo
CPU
RAM
Discos EBS
Administración del servidor
```

Sin embargo:

> ⚠️ **S3 no debe considerarse automáticamente "gratis".**

Amazon S3 cobra según el uso. Entre los componentes de costo se encuentran:

* almacenamiento;
* solicitudes;
* transferencia de datos;
* recuperación de datos en determinadas clases de almacenamiento;
* algunas características adicionales de administración.

Para un sitio personal pequeño, con unos cuantos archivos HTML/CSS y poco tráfico, el consumo puede ser extremadamente pequeño.

Además, dependiendo de la cuenta y de la elegibilidad, puede existir **AWS Free Tier** para determinados niveles de uso. Los límites y condiciones pueden cambiar, por lo que siempre debemos comprobar la información actual de AWS.

### La idea correcta es:

```text
S3
 │
 ├── Pagas por lo que utilizas
 │
 └── Un sitio pequeño puede tener
     un costo muy bajo
```

No:

```text
"S3 siempre es gratis"
```

---

# 🧹 Limpieza de recursos

Este paso es especialmente importante cuando estamos aprendiendo AWS.

Si ya no necesitamos el sitio, debemos eliminarlo.

## 1. Eliminar los objetos

Entra al bucket:

```text
S3
→ Buckets
→ TU-BUCKET
→ Objects
```

Selecciona:

```text
index.html
error.html
styles.css
```

y selecciona:

```text
Delete
```

Confirma la eliminación.

---

## 2. Eliminar el bucket

Regresa a:

```text
S3
→ Buckets
```

Selecciona el bucket.

Después:

```text
Delete
```

AWS solicitará confirmar el nombre del bucket.

Escríbelo exactamente y confirma.

---

## ⚠️ No olvides la limpieza

Aunque el sitio sea pequeño, dejar recursos de AWS activos innecesariamente no es una buena práctica.

Como regla general:

> **Si un laboratorio ya terminó, elimina los recursos que ya no necesitas.**

---

# 🚨 Problemas frecuentes

## `403 Access Denied`

Si aparece:

```text
403 Forbidden
```

revisa:

1. Block Public Access.
2. Bucket Policy.
3. Nombre del bucket dentro del ARN.
4. Que el objeto realmente exista.
5. Que `s3:GetObject` esté permitido.

La configuración de Block Public Access a nivel de cuenta puede seguir bloqueando el acceso aunque lo hayas deshabilitado a nivel del bucket.

---

## `404 Not Found`

Comprueba que exista:

```text
index.html
```

y que el nombre coincida exactamente.

Recuerda:

```text
index.html
```

es diferente de:

```text
Index.html
```

El nombre del documento de índice es sensible a mayúsculas y minúsculas.

---

## La página aparece sin estilos

Comprueba que tengas:

```text
index.html
styles.css
```

en el mismo nivel del bucket:

```text
Bucket
├── index.html
├── styles.css
└── error.html
```

Y que dentro de `index.html` tengas:

```html
<link rel="stylesheet" href="styles.css">
```

---

## La Bucket Policy muestra `Invalid Resource`

Comprueba que el ARN tenga exactamente el nombre de tu bucket:

```text
arn:aws:s3:::NOMBRE-DE-TU-BUCKET/*
```

Por ejemplo:

```text
arn:aws:s3:::mimidej-static-website-2026/*
```

---

## El endpoint no funciona

Comprueba que:

1. Static Website Hosting esté habilitado.
2. `index.html` exista.
3. La Bucket Policy permita `s3:GetObject`.
4. Block Public Access no esté bloqueando la política.
5. Estés utilizando el **Website endpoint**, no simplemente el endpoint REST del bucket.

S3 utiliza endpoints diferentes para las API de objetos y para Static Website Hosting.

---

# 🧠 Conceptos aprendidos

Al finalizar este laboratorio deberías poder explicar:

### Amazon S3

Servicio de almacenamiento de objetos que permite almacenar archivos como objetos dentro de buckets.

### Bucket

Contenedor lógico donde almacenamos nuestros objetos.

### Object

Archivo almacenado dentro de S3.

Ejemplos:

```text
index.html
styles.css
error.html
```

### Static Website Hosting

Funcionalidad que permite utilizar un bucket de S3 para servir contenido web estático.

### Bucket Policy

Política basada en recursos que define qué acciones pueden realizar determinados principals sobre los recursos del bucket.

### `s3:GetObject`

Permiso utilizado para permitir la lectura de objetos.

### Block Public Access

Conjunto de controles de S3 diseñados para impedir exposiciones públicas accidentales.

### Website Endpoint

Dirección HTTP proporcionada por S3 para acceder al sitio estático.

---

# 🚀 Siguientes pasos

Una vez que hayas completado este laboratorio, existen varias formas de llevarlo más lejos.

## Nivel 1 — Mejorar el frontend

Puedes agregar:

* JavaScript.
* Imágenes.
* Formularios.
* Animaciones.
* Diseño responsive.
* Bootstrap.
* Tailwind CSS.

Por ejemplo:

```text
static-website/
│
├── index.html
├── error.html
├── styles.css
├── script.js
└── images/
    ├── profile.jpg
    └── project.jpg
```

---

## Nivel 2 — Dominio personalizado

Puedes investigar cómo utilizar:

```text
Route 53
```

para asociar un dominio propio.

Por ejemplo:

```text
https://www.misitio.com
```

---

## Nivel 3 — HTTPS y CloudFront

El siguiente laboratorio recomendado sería:

```text
S3
 │
 │ private
 ▼
CloudFront
 │
 │ HTTPS
 ▼
Internet
```

Esto permite utilizar una arquitectura más adecuada para producción.

AWS recomienda CloudFront cuando se necesita servir un sitio estático de S3 de manera segura mediante HTTPS.

---

## Nivel 4 — Automatizar el despliegue

Después podemos eliminar la carga manual de archivos.

Por ejemplo:

```text
GitHub
   │
   │ push
   ▼
GitHub Actions
   │
   ▼
AWS
   │
   ▼
S3
```

De esta manera, cada vez que actualicemos nuestro código podremos desplegar automáticamente una nueva versión del sitio.

---

# 🎓 Conclusión

En este laboratorio construimos y publicamos un sitio web estático utilizando únicamente:

```text
HTML
CSS
Amazon S3
AWS Management Console
```

Sin configurar servidores.

Sin instalar Apache.

Sin instalar Nginx.

Sin administrar sistemas operativos.

La arquitectura final es sencilla:

```text
                ┌──────────────┐
                │   Usuario    │
                └───────┬──────┘
                        │
                        │ HTTP
                        ▼
                ┌──────────────┐
                │     S3       │
                │              │
                │ index.html    │
                │ styles.css   │
                │ error.html   │
                └──────────────┘
```

Este es uno de los ejemplos más sencillos para entender el concepto de **serverless** en AWS: delegamos la infraestructura al proveedor y nosotros nos concentramos en el contenido de nuestra aplicación.

> **Nota:** este laboratorio utiliza acceso público directo a S3 únicamente con fines educativos. Para sitios reales que requieran HTTPS y una postura de seguridad más robusta, considera utilizar Amazon CloudFront delante de S3.

---

## 📖 Referencias

* [Amazon S3 — Hosting a static website](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteHosting.html)
* [Tutorial — Configuring a static website on Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/HostingWebsiteOnS3Setup.html)
* [Setting permissions for website access](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteAccessPermissionsReqd.html)
* [Blocking public access to Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html)
* [Amazon S3 Pricing](https://aws.amazon.com/s3/pricing/)

---

**Autor:** `mimidej`

**Repositorio:** `https://github.com/mimidej/aws-tutorial`
