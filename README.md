# Sistemas Clientes-Servidor con PostgreSQL18 en Ubuntu 24.04

## Introducción

En la siguiente documentación, pondremos en práctica las arquitecturas de los sistemas gestores de las bases de datos y la comunicación entre los clientes y los servidores. En este caso, continuaré utilizando las dos máquinas que ya tenía creadas y configuradas con anterioridad. Una de ellas es un **ubuntu 24.04 desktop** y la otra es un **ubuntu 24.04 server**, la desktop mantendrá la IP **192.168.1.16** y se llamará “alumne@dosoriocliente” y la server seguirá con la IP **192.168.1.15** y de nombre “isard@dosorio-server”.
Esta **práctica** la llevaremos a cabo utilizando **PostgreSQL 18** que ya teníamos instalado y configurado de la tarea anterior: ([https://github.com/dhayanandresosorio/Instalacion-y-configuraci-n-PostgreSQL-18](https://github.com/dhayanandresosorio/Instalacion-y-configuraci-n-PostgreSQL-18)).

---

## Carga de la base de datos `dvdrental.tar`

Una vez en claro sobre qué trabajaremos, lo primero que queremos hacer es cargar la base de datos `dvdrental.tar` que se nos ha proporcionado. Para ello, descargamos el archivo en la **máquina server**. De manera opcional, he decidido que haré un **SSH** desde la máquina cliente para poder trabajar de manera más ágil y cómoda. Para ello, simplemente desde la terminal de mi cliente pondré el siguiente comando:

```bash
alumne@dosoriocliente:~$ ssh isard@192.168.1.15
```

Una vez hecho lo anterior, ahora sí, procedo a descargar la base de datos del link que se me ha proporcionado utilizando las siguientes órdenes:

```bash
isard@dosorio-server:~$ cd /tmp
isard@dosorio-server:/tmp$ wget https://github.com/mvm-classroom/mvm-recursos/raw/main/cicles/ASIX/ASGBD/Recursos/dvdrental.tar
```

Comprobaremos que efectivamente se haya descargado correctamente con la siguiente orden:

```bash
isard@dosorio-server:/tmp$ ls -lh /tmp/dvdrental.tar
```

Aquí nos debería devolver el archivo en cuestión con sus permisos y dueño:

```
-rw-rw-r-- 1 isard isard 2,8M nov 10 23:26 /tmp/dvdrental.tar
```

---

## Creación y restauración de la base de datos y conexión con cliente local

Una vez descargada la base de datos dentro del servidor, necesitaremos entrar al servicio **PostgreSQL** con la siguiente orden:

```bash
isard@dosorio-server:/tmp$ sudo -u postgres psql
[sudo] password for isard:
psql (18.0 (Ubuntu 18.0-1.pgdg24.04+3))
Digite «help» para obtener ayuda.
postgres=#
```

Ahora bien, una vez dentro, lo que debemos hacer es la **creación de una base de datos nueva y vacía** que es donde cargaremos la base de datos que hemos descargado con anterioridad. Usaremos el siguiente comando:

```sql
postgres=# CREATE DATABASE dvdrental;
CREATE DATABASE
```

Ahora saldremos de **PostgreSQL** para poder cargar/restaurar el contenido que tenemos en `dvdrental.tar`:

```sql
postgres=# \q
```

El siguiente paso es hacer uso del comando `pg_restore` para cargar todos los datos. Lo usaremos de la siguiente manera:

```bash
isard@dosorio-server:/tmp$ sudo -u postgres pg_restore -d dvdrental /tmp/dvdrental.tar
```

Con este comando lo que hemos logrado es **descomprimir y restaurar** las tablas, datos y relaciones de la base de datos.

---

## Verificación de la carga (tablas y datos)

Para comprobar que efectivamente se han cargado todos los datos de manera correcta, entraremos de nuevo a PostgreSQL y conectaremos a la base de datos. Para ello haremos uso de los siguientes comandos:

```bash
isard@dosorio-server:/tmp$ sudo -u postgres psql
psql (18.0 (Ubuntu 18.0-1.pgdg24.04+3))
Digite «help» para obtener ayuda.
postgres=#
postgres=# \c dvdrental
Ahora está conectado a la base de datos «dvdrental» con el usuario «postgres».
dvdrental=#
```

Una vez conectados con la base de datos, **listamos las tablas** para ver si efectivamente se ha cargado correctamente:

```sql
dvdrental=# \dt
```

```
Listado de tablas
 Esquema |     Nombre     | Tipo  |  Dueño
---------+-----------------+-------+----------
 public  | actor          | tabla | postgres
 public  | address        | tabla | postgres
 public  | category       | tabla | postgres
 public  | city           | tabla | postgres
 public  | country        | tabla | postgres
 public  | customer       | tabla | postgres
 public  | film           | tabla | postgres
 public  | film_actor     | tabla | postgres
 public  | film_category  | tabla | postgres
 public  | inventory      | tabla | postgres
 public  | language       | tabla | postgres
 public  | payment        | tabla | postgres
 public  | rental         | tabla | postgres
 public  | staff          | tabla | postgres
 public  | store          | tabla | postgres
(15 filas)
```

Y como era de esperar, se han cargado todas de manera correcta. Aun así, haremos una consulta rápida para ver que no solo las tablas, sino que **sus datos** también se han cargado correctamente:

```sql
dvdrental=# select * from actor limit 5;
```

```
 actor_id | first_name |  last_name   |        last_update
----------+------------+--------------+----------------------------
        1 | Penelope   | Guiness      | 2013-05-26 14:47:57.62
        2 | Nick       | Wahlberg     | 2013-05-26 14:47:57.62
        3 | Ed         | Chase        | 2013-05-26 14:47:57.62
        4 | Jennifer   | Davis        | 2013-05-26 14:47:57.62
        5 | Johnny     | Lollobrigida | 2013-05-26 14:47:57.62
(5 filas)
```

Con esto, damos por concluida la parte de la **carga de los datos** y la **conectividad desde la máquina local**, ya que efectivamente, todo ha sido un éxito.

---

## Conexión local simulando acceso remoto (localhost)

El siguiente punto a tratar será probar la conexión al servicio desde la misma máquina servidor, pero usando el parámetro `-h` para **simular una conexión remota local**. Usaremos la orden siguiente:

```bash
isard@dosorio-server:~$ psql -h localhost -U postgres
psql: error: falló la conexión al servidor en «localhost» (127.0.0.1), puerto 5432: Connection refused
¿Está el servidor en ejecución en ese host y aceptando conexiones TCP/IP?
```

El error que me ha saltado indica que PostgreSQL **no escucha** en el puerto 5432 o que hay algún tipo de problema con la configuración para aceptar algunas conexiones, incluido el `localhost`. Para solucionar esto, simplemente he de ir al archivo de configuración principal de PostgreSQL y cambiar que pueda escuchar por el `localhost`, ya que lo tenía configurado para conexiones remotas a través de la IP del servidor:

Lo tenía así:

```
listen_addresses = '192.168.1.15'
```

Lo dejo así para hacer solo esta prueba con el `localhost`:

```
listen_addresses = 'localhost'
```

**Pruebo nuevamente la conexión:**

```bash
isard@dosorio-server:~$ psql -h localhost -U postgres
Contraseña para usuario postgres:
psql (18.0 (Ubuntu 18.0-1.pgdg24.04+3))
Conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, compresión: desactivado, ALPN: postgresql)
Digite «help» para obtener ayuda.
postgres=#
```

Y efectivamente, esta vez no he tenido problemas para entrar. Ahora probaré a hacer alguna consulta básica de la base de datos para comprobar que puedo verlo de manera normal:

```sql
postgres=# \c dvdrental
Conexión SSL (protocolo: TLSv1.3, cifrado: TLS_AES_256_GCM_SHA384, compresión: desactivado, ALPN: postgresql)
Ahora está conectado a la base de datos «dvdrental» con el usuario «postgres».
dvdrental=# select count(*) from actor;
```

```
 count
-------
   200
(1 fila)
```

```sql
dvdrental=# select * from language limit 7;
```

```
 language_id |   name    |     last_update
-------------+-----------+---------------------
           1 | English   | 2006-02-15 10:02:19
           2 | Italian   | 2006-02-15 10:02:19
           3 | Japanese  | 2006-02-15 10:02:19
           4 | Mandarin  | 2006-02-15 10:02:19
           5 | French    | 2006-02-15 10:02:19
           6 | German    | 2006-02-15 10:02:19
(6 filas)
```

Una vez comprobada la conexión y funcionalidad de la tabla desde el `localhost` de manera remota, procederé a **cambiar de nuevo** el archivo de configuración principal de PostgreSQL para que pueda escuchar de nuevo **conexiones remotas**, dejándolo así:

```
listen_addresses = '192.168.1.15'
```

---

## Conexión remota desde el cliente

Una vez hecho esto, lo siguiente será conectar desde el **cliente** de manera remota al servicio PostgreSQL. Para ello usaré la orden siguiente:

```bash
alumne@dosoriocliente:~$ psql -h 192.168.1.15 -U postgres
```

Donde me solicitará una contraseña para poder entrar que le hemos indicado previamente en la documentación anterior a esta; pongo la contraseña y, efectivamente, estamos dentro:

```bash
alumne@dosoriocliente:~$ psql -h 192.168.1.15 -U postgres
Password for user postgres:
psql (18.0 (Ubuntu 18.0-1.pgdg22.04+3))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.
postgres=#
```

Lo siguiente será entrar a la base de datos y probar alguna consulta para comprobar que todo funciona de la manera correcta:

```sql
postgres=# \c dvdrental
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
You are now connected to database "dvdrental" as user "postgres".
dvdrental=# select * from city limit 3;
```

```
 city_id |        city         | country_id |     last_update
---------+---------------------+------------+---------------------
       1 | A Corua (La Corua)  |         87 | 2006-02-15 09:45:25
       2 | Abha                |         82 | 2006-02-15 09:45:25
       3 | Abu Dhabi           |        101 | 2006-02-15 09:45:25
(3 rows)
```

```sql
dvdrental=# select title from film limit 3;
```

```
       title
--------------------
 Chamber Italian
 Grosse Wonderful
 Airport Pollock
(3 rows)
```

Tal y como hemos podido comprobar, **todo funciona perfectamente**.

---

## PgAdmin4

El siguiente paso en esta **práctica** será el hecho de **instalar pgAdmin 4** en la máquina servidor. **pgAdmin 4** es una herramienta gráfica muy útil, sobre todo para la administración de bases de datos PostgreSQL; sirve para hacer consultas, gestionar bases de datos, usuarios, etc., de manera gráfica. Para poder comenzar con dicha tarea, lo primero que haremos será **actualizar** el sistema y los repositorios con las órdenes siguientes:

```bash
isard@dosorio-server:~$ sudo apt update
isard@dosorio-server:~$ sudo apt upgrade
```

Una vez hecho esto, debemos **añadir la clave pública** para el repositorio de pgAdmin a nuestra máquina server. Para ello usaremos el siguiente comando, indicado desde la web oficial de pgAdmin:

```bash
isard@dosorio-server:~$ curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /usr/share/keyrings/packages-pgadmin-org.gpg
```

Seguidamente, **crearemos el archivo de configuración** del repositorio:

```bash
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'
```

Por último, instalaremos el **modo web** de pgAdmin, ya que es el que se recomienda para servidores:

```bash
isard@dosorio-server:~$ sudo apt install pgadmin4-web
```

Lo siguiente será **iniciar pgAdmin 4 en modo web**. Una vez realizada la instalación, el sistema nos pedirá configurar el acceso inicial, creando un usuario administrativo para acceder a la interfaz de administración. Este usuario servirá para autenticarse en la aplicación. Durante el proceso de configuración, nos pedirá un correo electrónico y una contraseña. Aquí usaremos el correo que queramos y lo mismo con la contraseña. Para todo lo anterior, haremos uso del siguiente comando:

```bash
isard@dosorio-server:~$ sudo /usr/pgadmin4/bin/setup-web.sh
```

El paso siguiente para poder **acceder correctamente** a la página de pgAdmin desde el navegador del cliente **por vía SSH**, debemos redirigir el puerto con **SSH tunneling**. Con el siguiente comando desde la terminal del cliente, básicamente hacemos un **túnel** para redirigir el puerto 80 (donde se ve pgAdmin) del servidor a la máquina cliente:

```bash
alumne@dosoriocliente:~$ ssh -L 8080:localhost:80 isard@192.168.1.15
```

Y una vez hecho el túnel, si accedemos a la web desde el navegador del cliente, utilizando el puerto 80, nos debería dejar ver la web de pgAdmin:

```
http://localhost:8080/pgadmin4
```

Otra manera de hacerlo es utilizando directamente en el navegador la IP del servidor en lugar del `localhost`, aunque es **menos seguro**:

```
http://192.168.1.15/pgadmin4
```

Ambas direcciones nos redirigen a pgAdmin, donde nos **logueamos** utilizando el correo y contraseña que habíamos dicho antes.

Una vez hemos conseguido entrar, **incluiremos / registraremos** nuestro servidor en pgAdmin. Para ello haremos clic derecho en el apartado “Servers”, seleccionamos **Register**, seguidamente **Server** y nos saldrá una ventana como la siguiente, donde le pondremos un **nombre/apodo** al servicio que queremos añadir en el apartado **General**.

Seguidamente, en el apartado **Connection**, añadiremos la **dirección IP** de nuestro servidor, el **usuario** y la **contraseña** que tenga permisos de administrador en nuestra base de datos `dvdrental`. En este caso, usaremos el usuario `postgres`.

El problema que me ha salido significa que PostgreSQL **sí está en funcionamiento y escuchando**, pero **no está permitiendo el acceso** desde la dirección IP `192.168.1.15` según el archivo de control de accesos `pg_hba.conf`. Por lo tanto, debemos añadir en dicho archivo que nos deje acceso:

```bash
isard@dosorio-server:~$ sudo nano /etc/postgresql/18/main/pg_hba.conf
```

Y añadiremos esta línea:

```
host all all 192.168.1.15/32 scram-sha-256
```

Reiniciamos el servicio:

```bash
isard@dosorio-server:~$ sudo systemctl restart postgresql
```

Y comprobamos que esta vez sí que nos deja conectar: y efectivamente, **así ha sido**.

Hacemos una **consulta** en la base de datos `dvdrental` para comprobar que pilla bien la base de datos. Con la imagen anterior, podemos comprobar que la **implementación de pgAdmin** ha sido un **éxito**.

---

## DBeaver

El siguiente paso es la **implementación e instalación de DBeaver** en su versión más reciente. En esta ocasión utilizaremos la **máquina cliente** para llevar esto a cabo. El método que utilizaré será **instalarlo directamente** con los comandos y repositorios **oficiales** de la web de DBeaver. Lo primero que debemos hacer es **añadir la clave pública** del repositorio de DBeaver a nuestra máquina. Esta clave es necesaria para que nuestro sistema pueda verificar que los paquetes que descargamos del repositorio son **legítimos** y **no han sido modificados**. Para hacerlo, usamos el siguiente comando:

```bash
alumne@dosoriocliente:~$ sudo wget -O /usr/share/keyrings/dbeaver.gpg.key https://dbeaver.io/debs/dbeaver.gpg.key
[sudo] contrasenya per a alumne: 
--2025-11-12 00:26:09-- https://dbeaver.io/debs/dbeaver.gpg.key
S'està resolent dbeaver.io (dbeaver.io)… 209.38.51.239, 2604:a880:800:14:0:1:958:c000
S'està connectant a dbeaver.io (dbeaver.io)|209.38.51.239|:443… conectat.
HTTP: s'ha enviat la petició, s'està esperant una resposta… 200 OK
Mida: 3120 (3,0K) [application/octet-stream]
S'està desant a: ‘/usr/share/keyrings/dbeaver.gpg.key’

/usr/share/keyrings 100%[===================>] 3,05K --.-KB/s in 0s
2025-11-12 00:26:10 (446 MB/s) - s'ha desat ‘/usr/share/keyrings/dbeaver.gpg.key’ [3120/3120]
```

Una vez que hemos añadido la clave, el siguiente paso es **configurar el repositorio** desde el cual instalaremos DBeaver. Esto se hace mediante el siguiente comando:

```bash
alumne@dosoriocliente:~$ echo "deb [signed-by=/usr/share/keyrings/dbeaver.gpg.key] https://dbeaver.io/debs/dbeaver-ce /" | sudo tee /etc/apt/sources.list.d/dbeaver.list
deb [signed-by=/usr/share/keyrings/dbeaver.gpg.key] https://dbeaver.io/debs/dbeaver-ce /
```

Lo siguiente, una vez que hemos añadido el repositorio, es hacer un **update** de la lista de paquetes y seguidamente podremos **instalar** el servicio:

```bash
alumne@dosoriocliente:~$ sudo apt update
alumne@dosoriocliente:~$ sudo apt install -y dbeaver-ce
```

Una vez instalado DBeaver, queremos hacer una **conexión remota** a PostgreSQL. Para ello, **abriremos la aplicación** de DBeaver en nuestra máquina cliente.

Una vez dentro, debemos **establecer una nueva conexión** con PostgreSQL. Para ello, simplemente le daremos al **clic derecho** en el panel de la izquierda, seleccionaremos **Create** y **Connection**.

Seguidamente, se nos abrirá una ventana como la siguiente donde **elegiremos PostgreSQL**.

Inmediatamente después de elegir PostgreSQL, le daremos a **Next** y nos saldrá una ventana como la siguiente, donde **indicaremos que se conecte vía host**, indicando que el **host** sea la IP de la máquina servidor donde tenemos nuestro PostgreSQL y la base de datos `dvdrental`.

Le damos a **Finish** y, si todo ha ido bien, nos muestra la **conexión hecha** con nuestra base de datos `dvdrental` en el panel de la izquierda.

El siguiente paso es comprobar que, efectivamente, no solo haya hecho conexión, sino que **pueda interactuar con los datos** y ver qué tablas y valores hay. Para ello, usaremos **SQL Editor**, que lo tenemos en la parte superior de la app de DBeaver.

Una vez le hemos dado a **Open SQL script**, DBeaver **descargará automáticamente** el **driver JDBC** de PostgreSQL, ya que es el que usa por defecto.

Una vez se ha descargado, podemos proceder a **hacer alguna que otra consulta** para ver si se ha implementado correctamente.

En este caso, **todo ha funcionado correctamente**, haciendo que esta implementación **haya sido un éxito**.

---

## Contesta las siguientes preguntas para relacionar la práctica con la teoría

**¿Consideras PostgreSQL como un SGBD centralizado o basado en el sistema cliente/servidor? Argumenta tu respuesta.**


Una vez terminada la práctica y como ya hemos tratado en clase, digo que es un sistema **cliente-servidor**. Esto se debe a que el **servidor** es quien gestiona las bases de datos, y los **clientes**, como `psql`, **pgAdmin** o **DBeaver**, se conectan a él desde otra máquina cliente mediante la red. Durante la práctica, hemos visto que el servidor se **instala y ejecuta** en una máquina, mientras que desde el cliente **podemos conectarnos de forma remota** y hacer consultas.

**Si has clasificado PostgreSQL como un sistema cliente/servidor, dentro de qué sistema de capas consideras que encaja mejor: ¿2 capas o 3 capas? Argumenta tu respuesta.**


Como hemos visto en clase, **PostgreSQL** funciona con una **arquitectura de tres capas**. Existe una parte que usa el **usuario (cliente)**, otra que se encarga de **comunicar y gestionar las peticiones**, y una última donde **se guardan los datos**, que vendría siendo el servidor. Así, todo el sistema **trabaja de forma ordenada y separada**.
