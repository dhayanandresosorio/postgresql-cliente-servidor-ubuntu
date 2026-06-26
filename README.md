# PostgreSQL Cliente-Servidor en Ubuntu

Laboratorio tecnico de instalacion y configuracion de PostgreSQL en un entorno cliente-servidor con Ubuntu 24.04.

El proyecto documenta la configuracion de un servidor PostgreSQL, la preparacion de una base de datos de prueba, la conexion desde un cliente Linux y el acceso mediante pgAdmin 4.

## Objetivo

Montar un entorno cliente-servidor funcional con PostgreSQL, comprobando la conectividad remota, la autenticacion de usuarios y el acceso a una base de datos desde un cliente.

## Tecnologias utilizadas

- Ubuntu Server 24.04
- Ubuntu Desktop 24.04
- PostgreSQL
- psql
- pgAdmin 4
- SSH
- SQL
- Git y GitHub

## Arquitectura del laboratorio

- Servidor PostgreSQL: 192.168.1.15
- Cliente Ubuntu: 192.168.1.16
- Base de datos utilizada: dvdrental
- Herramienta grafica: pgAdmin 4

## Estructura del repositorio

postgresql-cliente-servidor-ubuntu/
|-- README.md
|-- .gitignore
|-- .gitattributes
|-- docs/
|   |-- memoria.md
|-- img/
|   |-- imagenes del proyecto

## Contenido de la practica

La practica incluye:

- Instalacion de PostgreSQL en Ubuntu Server.
- Configuracion del servicio PostgreSQL.
- Modificacion de postgresql.conf.
- Modificacion de pg_hba.conf.
- Conexion desde cliente mediante psql.
- Instalacion y acceso a pgAdmin 4.
- Restauracion de una base de datos de prueba.
- Documentacion tecnica con comandos y evidencias visuales.

## Seguridad

Las contrasenas reales no se publican en el repositorio.

Cuando se documentan credenciales o accesos, se hace de forma generica o mediante placeholders.

## Documentacion completa

La documentacion completa de la practica se encuentra en docs/memoria.md.
