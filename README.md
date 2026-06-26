# PostgreSQL Cliente-Servidor en Ubuntu

Práctica de configuración de PostgreSQL en un entorno cliente-servidor con dos máquinas Ubuntu 24.04.

En este proyecto he configurado un servidor PostgreSQL en una máquina Ubuntu Server y he realizado las conexiones desde un cliente Ubuntu Desktop, primero desde terminal con `psql` y después desde pgAdmin 4.

La idea de la práctica era entender cómo funciona PostgreSQL cuando no se usa únicamente en local, sino como un servicio de base de datos accesible desde otra máquina de la red.

> [!NOTE]
> Este repositorio no es un despliegue de producción. Es una práctica de laboratorio orientada a administración de sistemas, bases de datos y conexión cliente-servidor.

## Entorno utilizado

* Servidor PostgreSQL: Ubuntu Server 24.04
* Cliente: Ubuntu Desktop 24.04
* IP del servidor: `192.168.1.15`
* IP del cliente: `192.168.1.16`
* Base de datos utilizada: `dvdrental`
* Herramientas principales: PostgreSQL, `psql`, SSH y pgAdmin 4

## Qué se ha trabajado

La práctica documenta el proceso completo para preparar PostgreSQL como servidor accesible desde red.

Entre las tareas realizadas están:

* Instalación de PostgreSQL en Ubuntu Server.
* Comprobación del estado del servicio.
* Acceso local mediante `psql`.
* Configuración de `postgresql.conf`.
* Configuración de `pg_hba.conf`.
* Habilitación de conexiones desde el cliente.
* Prueba de conexión remota desde terminal.
* Instalación y configuración de pgAdmin 4.
* Acceso a pgAdmin desde navegador.
* Registro del servidor PostgreSQL en pgAdmin.
* Restauración y uso de la base de datos `dvdrental`.
* Revisión de errores relacionados con permisos y acceso remoto.

## Configuración principal

Uno de los puntos importantes de la práctica ha sido modificar PostgreSQL para que no escuche únicamente en local.

En el archivo:

```text
/etc/postgresql/16/main/postgresql.conf
```

se configura la escucha del servicio en la IP del servidor:

```text
listen_addresses = '192.168.1.15'
```

Después, en el archivo:

```text
/etc/postgresql/16/main/pg_hba.conf
```

se permite el acceso desde la máquina cliente:

```text
host all all 192.168.1.16/32 scram-sha-256
```

Con esta configuración, el cliente puede conectarse al servidor PostgreSQL usando `psql`:

```bash
psql -h 192.168.1.15 -U postgres
```

> [!TIP]
> En este tipo de prácticas, si la conexión remota falla, normalmente conviene revisar primero tres puntos: que PostgreSQL esté escuchando en la IP correcta, que `pg_hba.conf` permita la IP del cliente y que el servicio se haya reiniciado después de los cambios.

## pgAdmin 4

Además de las pruebas por terminal, también se configura pgAdmin 4 para administrar PostgreSQL desde una interfaz web.

En la documentación se muestra el proceso de acceso, registro del servidor, configuración de conexión y comprobación de la base de datos desde la interfaz gráfica.

Esta parte permite ver de forma más clara cómo se gestiona un servidor PostgreSQL desde una herramienta habitual en administración de bases de datos.

## Estructura del repositorio

```text
postgresql-cliente-servidor-ubuntu/
|-- README.md
|-- .gitignore
|-- .gitattributes
|-- docs/
|   |-- memoria.md
|-- img/
|   |-- imágenes de la práctica
```

## Notas sobre credenciales

Durante la práctica se usan contraseñas para acceder a PostgreSQL y a pgAdmin, pero no forman parte del contenido que quiero publicar en GitHub.

En la documentación se mantiene la explicación del proceso, pero sin dejar contraseñas reales escritas en el repositorio.

> [!IMPORTANT]
> Si se reutiliza este laboratorio, cada persona debería definir sus propias credenciales en local y no subirlas al repositorio.

## Documentación completa

La explicación completa paso a paso, con comandos, configuración, errores revisados y evidencias visuales, está en:

```text
docs/memoria.md
```

Las imágenes utilizadas en la documentación están organizadas en:

```text
img/
```
