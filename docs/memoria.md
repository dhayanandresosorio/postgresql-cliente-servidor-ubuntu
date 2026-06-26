# Memoria técnica - PostgreSQL Cliente-Servidor en Ubuntu 24.04

## 1. Resumen de la práctica

En esta práctica he configurado PostgreSQL en un entorno cliente-servidor utilizando dos máquinas Ubuntu 24.04.

El objetivo ha sido trabajar PostgreSQL como un servicio de base de datos accesible desde red, no solo como una instalación local. Para ello, he usado una máquina Ubuntu Server como servidor PostgreSQL y una máquina Ubuntu Desktop como cliente desde la que se realizan las conexiones, pruebas y administración gráfica.

También he utilizado pgAdmin 4 para comprobar la administración del servidor desde una interfaz web y para validar que la base de datos restaurada podía consultarse correctamente.

> [!NOTE]
> Esta práctica está enfocada a entender la configuración real de un servicio PostgreSQL accesible desde red: instalación, permisos, conexión remota, configuración de acceso, pruebas desde terminal y administración desde pgAdmin.

---

## 2. Entorno utilizado

El laboratorio está formado por dos máquinas Ubuntu 24.04.

```text
Servidor PostgreSQL
- Sistema operativo: Ubuntu Server 24.04
- Usuario: isard
- Hostname: dosorio-server
- IP: 192.168.1.15
- Servicio principal: PostgreSQL

Cliente
- Sistema operativo: Ubuntu Desktop 24.04
- Usuario: alumne
- Hostname: dosoriocliente
- IP: 192.168.1.16
- Herramientas: psql, SSH, navegador y pgAdmin 4
```

Base de datos utilizada durante la práctica:

```text
dvdrental
```

La idea general del laboratorio es la siguiente:

```text
Cliente Ubuntu Desktop
192.168.1.16
        |
        | conexión por red
        v
Servidor Ubuntu Server
192.168.1.15
PostgreSQL - puerto 5432
```

---

## 3. Conexión inicial al servidor

Desde la máquina cliente, primero accedo al servidor mediante SSH.

```bash
alumne@dosoriocliente:~$ ssh isard@192.168.1.15
```

Una vez dentro del servidor, se puede comprobar que se está trabajando en la máquina correcta.

```bash
isard@dosorio-server:~$ hostname
dosorio-server
```

También se puede revisar la configuración de red.

```bash
isard@dosorio-server:~$ ip a
```

En esta práctica, el servidor mantiene la IP:

```text
192.168.1.15
```

Esta comprobación es importante porque más adelante esa IP será la que se usará tanto en la configuración de PostgreSQL como en las conexiones desde el cliente y desde pgAdmin.

---

## 4. Instalación de PostgreSQL en el servidor

En el servidor se actualizan los repositorios.

```bash
isard@dosorio-server:~$ sudo apt update
```

Después se instala PostgreSQL junto con los paquetes adicionales.

```bash
isard@dosorio-server:~$ sudo apt install postgresql postgresql-contrib
```

Una vez instalado, se comprueba el estado del servicio.

```bash
isard@dosorio-server:~$ sudo systemctl status postgresql
```

La salida esperada debe indicar que PostgreSQL está activo.

```text
Active: active (exited)
```

También se puede comprobar la versión instalada.

```bash
isard@dosorio-server:~$ psql --version
```

Ejemplo de salida:

```text
psql (PostgreSQL) 16.x
```

> [!TIP]
> Antes de modificar archivos de configuración, conviene comprobar que PostgreSQL está instalado y que el servicio responde correctamente. Si falla en local, todavía no tiene sentido probar conexiones remotas.

---

## 5. Acceso local a PostgreSQL

PostgreSQL crea por defecto el usuario del sistema `postgres`.

Para acceder a la consola de PostgreSQL desde el servidor:

```bash
isard@dosorio-server:~$ sudo -u postgres psql
```

Dentro de PostgreSQL se puede comprobar la versión del servidor:

```sql
SELECT version();
```

También se pueden listar las bases de datos existentes:

```sql
\l
```

Y salir de la consola:

```sql
\q
```

En este punto ya se confirma que PostgreSQL funciona correctamente de forma local en el servidor.

---

## 6. Configuración de contraseña para el usuario postgres

Para poder conectar posteriormente desde el cliente y desde pgAdmin, se configura una contraseña para el usuario `postgres`.

```bash
isard@dosorio-server:~$ sudo -u postgres psql
```

Dentro de PostgreSQL:

```sql
ALTER USER postgres WITH PASSWORD 'CHANGE_ME_POSTGRES_PASSWORD';
```

Después se sale de la consola:

```sql
\q
```

> [!IMPORTANT]
> La contraseña real usada durante la práctica no se publica en el repositorio. En la memoria se mantiene el proceso técnico, pero usando un valor placeholder.

---

## 7. Configuración de PostgreSQL para aceptar conexiones remotas

Por defecto, PostgreSQL suele escuchar únicamente en local. Para permitir conexiones desde otra máquina de la red, se modifica el archivo `postgresql.conf`.

```bash
isard@dosorio-server:~$ sudo nano /etc/postgresql/16/main/postgresql.conf
```

Dentro del archivo se busca la directiva `listen_addresses`.

En esta práctica se configura con la IP del servidor:

```text
listen_addresses = '192.168.1.15'
```

Con esto se indica a PostgreSQL que escuche en la interfaz de red del servidor, no únicamente en `localhost`.

Después se guarda el archivo y se reinicia el servicio.

```bash
isard@dosorio-server:~$ sudo systemctl restart postgresql
```

Se vuelve a comprobar el estado:

```bash
isard@dosorio-server:~$ sudo systemctl status postgresql
```

Y también se puede comprobar que PostgreSQL escucha en el puerto 5432.

```bash
isard@dosorio-server:~$ sudo ss -tulnp | grep 5432
```

Ejemplo esperado:

```text
tcp LISTEN 0 244 192.168.1.15:5432 0.0.0.0:*
```

El puerto por defecto de PostgreSQL es:

```text
5432
```

---

## 8. Configuración de acceso en pg_hba.conf

Además de escuchar en la IP correcta, PostgreSQL necesita permitir explícitamente qué clientes pueden conectarse.

Para ello se modifica el archivo `pg_hba.conf`.

```bash
isard@dosorio-server:~$ sudo nano /etc/postgresql/16/main/pg_hba.conf
```

Se añade una regla para permitir conexiones desde el cliente Ubuntu.

```text
host all all 192.168.1.16/32 scram-sha-256
```

Esta línea significa:

```text
host              conexión TCP/IP
all               cualquier base de datos
all               cualquier usuario
192.168.1.16/32   solo el cliente con esa IP
scram-sha-256     autenticación mediante contraseña cifrada
```

Después se reinicia PostgreSQL.

```bash
isard@dosorio-server:~$ sudo systemctl restart postgresql
```

> [!WARNING]
> En un laboratorio se podría permitir un rango más amplio, pero para esta práctica tiene más sentido permitir únicamente la IP del cliente. Así se evita abrir PostgreSQL a más máquinas de las necesarias.

---

## 9. Prueba de conexión desde el cliente

En la máquina cliente se instala el cliente de PostgreSQL si no está instalado.

```bash
alumne@dosoriocliente:~$ sudo apt update
alumne@dosoriocliente:~$ sudo apt install postgresql-client
```

Después se prueba la conexión remota al servidor.

```bash
alumne@dosoriocliente:~$ psql -h 192.168.1.15 -U postgres
```

PostgreSQL solicita la contraseña del usuario `postgres`.

```text
Password for user postgres:
```

Si la configuración es correcta, se entra en la consola de PostgreSQL.

```text
postgres=#
```

Dentro de la consola se puede comprobar el usuario actual:

```sql
SELECT current_user;
```

Salida esperada:

```text
 current_user
--------------
 postgres
```

También se pueden listar las bases de datos:

```sql
\l
```

Y salir:

```sql
\q
```

Con esta prueba se confirma que la conexión cliente-servidor funciona correctamente desde terminal.

---

## 10. Base de datos de prueba: dvdrental

Para trabajar con una base de datos más realista, se utiliza la base de datos de ejemplo `dvdrental`.

Primero se crea la base de datos desde el servidor.

```bash
isard@dosorio-server:~$ sudo -u postgres createdb dvdrental
```

Después se restaura el contenido desde el archivo correspondiente.

```bash
isard@dosorio-server:~$ pg_restore -U postgres -d dvdrental dvdrental.tar
```

Una vez restaurada, se accede a la base de datos.

```bash
isard@dosorio-server:~$ psql -U postgres -d dvdrental
```

Dentro de PostgreSQL se comprueban las tablas.

```sql
\dt
```

También se puede ejecutar una consulta simple.

```sql
SELECT title, release_year
FROM film
LIMIT 5;
```

Ejemplo de salida esperada:

```text
        title         | release_year
----------------------+--------------
 Academy Dinosaur     |         2006
 Ace Goldfinger       |         2006
 Adaptation Holes     |         2006
 Affair Prejudice     |         2006
 African Egg          |         2006
```

Esta parte permite comprobar que el entorno no se limita a levantar PostgreSQL, sino que también permite trabajar con una base de datos real restaurada.

---

## 11. Instalación de pgAdmin 4

Además de la conexión por terminal, la práctica incluye pgAdmin 4 para administrar PostgreSQL desde una interfaz web.

Durante la configuración inicial, pgAdmin solicita crear un usuario administrativo.

```bash
alumne@dosoriocliente:~$ sudo /usr/pgadmin4/bin/setup-web.sh
```

Durante el proceso se solicita:

```text
Email address:
Password:
Retype password:
```

Estos datos son propios de pgAdmin y se usan para acceder a la interfaz web.

> [!IMPORTANT]
> Las credenciales de acceso a pgAdmin no se dejan escritas en el repositorio. Solo se documenta el proceso necesario para configurarlo.

---

## 12. Acceso a pgAdmin desde navegador

Una vez configurado pgAdmin, se accede desde el navegador.

En el laboratorio se prueba el acceso a la interfaz web mediante:

```text
http://192.168.1.15/pgadmin4
```

También se puede usar un túnel SSH si se quiere acceder de forma más controlada.

```bash
alumne@dosoriocliente:~$ ssh -L 8080:localhost:80 isard@192.168.1.15
```

Y después abrir en el navegador:

```text
http://localhost:8080/pgadmin4
```

La primera captura muestra el acceso a pgAdmin.

![Acceso a pgAdmin](<../img/Captura de pantalla 2025-11-11 151028.png>)

Después se inicia sesión con el usuario creado durante la configuración inicial.

![Inicio de sesión en pgAdmin](<../img/Captura de pantalla 2025-11-11 151854.png>)

---

## 13. Registro del servidor PostgreSQL en pgAdmin

Una vez dentro de pgAdmin, se registra el servidor PostgreSQL.

Desde la interfaz se crea un nuevo servidor.

![Creación de servidor en pgAdmin](<../img/Captura de pantalla 2025-11-11 152158.png>)

En el apartado de conexión se indican los datos principales.

```text
Host name/address: 192.168.1.15
Port: 5432
Maintenance database: postgres
Username: postgres
Password: CHANGE_ME_POSTGRES_PASSWORD
```

![Configuración de conexión en pgAdmin](<../img/Captura de pantalla 2025-11-11 152257.png>)

Al probar la conexión aparece un problema de acceso.

![Error de conexión en pgAdmin](<../img/Captura de pantalla 2025-11-11 152404.png>)

El error indica que PostgreSQL está funcionando, pero no está permitiendo todavía el acceso desde el cliente. En este punto se revisa el archivo `pg_hba.conf`.

La regla correcta para esta práctica es:

```text
host all all 192.168.1.16/32 scram-sha-256
```

Tras añadir o corregir la regla, se reinicia PostgreSQL.

```bash
isard@dosorio-server:~$ sudo systemctl restart postgresql
```

Después se vuelve a probar la conexión desde pgAdmin.

![Conexión corregida en pgAdmin](<../img/Captura de pantalla 2025-11-11 153048.png>)

A partir de aquí, el servidor queda registrado correctamente dentro de pgAdmin.

![Servidor registrado](<../img/Captura de pantalla 2025-11-11 153104.png>)

---

## 14. Revisión de la base de datos desde pgAdmin

Con el servidor ya conectado, se navega por la estructura de PostgreSQL desde pgAdmin.

Se comprueba que la base de datos `dvdrental` está disponible.

![Base de datos disponible en pgAdmin](<../img/Captura de pantalla 2025-11-11 153343.png>)

Después se revisa la estructura de la base de datos.

![Exploración de la base de datos](<../img/Captura de pantalla 2025-11-12 003331.png>)

También se revisan tablas y objetos de la base de datos.

![Tablas de dvdrental](<../img/Captura de pantalla 2025-11-12 003602.png>)

Con esto se confirma que pgAdmin se conecta correctamente al servidor PostgreSQL y que la base de datos restaurada está accesible desde la interfaz gráfica.

---

## 15. Consultas y comprobaciones desde pgAdmin

Una vez conectada la base de datos en pgAdmin, se realizan comprobaciones desde la interfaz.

![Consulta desde pgAdmin](<../img/Captura de pantalla 2025-11-12 003756.png>)

Se ejecuta una consulta sobre la base de datos.

![Ejecución de consulta](<../img/Captura de pantalla 2025-11-12 003910.png>)

Después se revisa el resultado obtenido.

![Resultado de consulta](<../img/Captura de pantalla 2025-11-12 004117.png>)

También se comprueba la navegación por los datos desde la interfaz.

![Datos consultados en pgAdmin](<../img/Captura de pantalla 2025-11-12 004331.png>)

Esta parte es útil porque permite validar que la conexión no solo funciona a nivel de login, sino que permite consultar datos reales de la base de datos.

---

## 16. Evidencias finales del procedimiento

Las últimas capturas documentan la parte final de la comprobación en pgAdmin.

![Revisión final en pgAdmin](<../img/Captura de pantalla 2025-11-12 004416.png>)

![Comprobación adicional](<../img/Captura de pantalla 2025-11-12 004709.png>)

![Estado final de la práctica](<../img/Captura de pantalla 2025-11-12 004751.png>)

Con estas evidencias se deja constancia de que el servidor PostgreSQL quedó accesible desde el cliente, que pgAdmin pudo conectarse y que la base de datos restaurada se podía consultar correctamente.

---

## 17. Problemas revisados durante la práctica

Durante la práctica se revisan varios puntos típicos cuando PostgreSQL no permite conexión remota.

Los más importantes son:

* El servicio PostgreSQL no está activo.
* `listen_addresses` sigue configurado solo para conexiones locales.
* El puerto 5432 no está escuchando en la IP del servidor.
* Falta una regla correcta en `pg_hba.conf`.
* La IP permitida en `pg_hba.conf` no corresponde al cliente real.
* No se ha reiniciado PostgreSQL después de modificar la configuración.
* La contraseña del usuario no es correcta.
* pgAdmin intenta conectar al host equivocado.
* El cliente no tiene conectividad con el servidor.

> [!TIP]
> Cuando falla la conexión desde pgAdmin, es recomendable probar primero con `psql` desde terminal. Si `psql` tampoco conecta, el problema suele estar en PostgreSQL, red o permisos. Si `psql` conecta pero pgAdmin no, normalmente el problema está en los datos introducidos en la conexión gráfica.

---

## 18. Comandos de comprobación utilizados

Comprobar el estado del servicio:

```bash
sudo systemctl status postgresql
```

Reiniciar PostgreSQL:

```bash
sudo systemctl restart postgresql
```

Comprobar escucha en el puerto 5432:

```bash
sudo ss -tulnp | grep 5432
```

Conectarse desde el cliente:

```bash
psql -h 192.168.1.15 -U postgres
```

Listar bases de datos:

```sql
\l
```

Conectarse a `dvdrental`:

```sql
\c dvdrental
```

Listar tablas:

```sql
\dt
```

Consulta de prueba:

```sql
SELECT title, release_year
FROM film
LIMIT 5;
```

---

## 19. Organización del repositorio

El repositorio se ha reorganizado para separar la presentación corta de la documentación completa.

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

La carpeta `img/` contiene las evidencias visuales utilizadas en esta memoria técnica.

---

## 20. Valor técnico de la práctica

Esta práctica me ha servido para trabajar varios puntos importantes de administración de sistemas y bases de datos:

* Instalación y gestión de PostgreSQL en Linux.
* Configuración de servicios en Ubuntu Server.
* Conexión cliente-servidor por red.
* Edición de archivos de configuración reales.
* Control de acceso mediante `pg_hba.conf`.
* Uso de `psql` desde terminal.
* Administración gráfica con pgAdmin 4.
* Restauración y revisión de una base de datos de ejemplo.
* Documentación técnica con comandos, salidas y capturas.

Es una práctica sencilla, pero útil para afianzar la base de PostgreSQL en un entorno de red y entender mejor cómo se comporta un SGBD cuando se administra como servicio.
