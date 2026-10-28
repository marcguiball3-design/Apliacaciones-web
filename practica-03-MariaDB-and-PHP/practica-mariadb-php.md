#  Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04 

-  **Alumno:** Marcos Guillen B
-  **Ciclo / Módulo:** Sistemas Microinformáticos y Redes (SMR) · Aplicaciones Web
-  **Curso:** 2026/2027 
-  **Fecha:** Octubre de 2026
-  **Máquina:** `smr-web` · Ubuntu 24.04 · Apache 2.4.58

---

##  Índice


1. [Apartado 1. Preparación del sistema](#apartado-1-preparación-del-sistema)
2. [Apartado 2. Instalación de MariaDB](#apartado-2-instalación-de-mariadb)
3. [Apartado 3. Comprobación del funcionamiento](#apartado-3-comprobación-del-funcionamiento)
4. [Apartado 4. Comandos principales](#apartado-4-comandos-principales)
5. [Apartado 5. ¿Qué pasa si MariaDB está parado?](#apartado-5-qué-pasa-si-mariadb-está-parado)
6. [Apartado 6. Instalación de PHP](#apartado-6-instalación-de-php)
7. [Apartado 7. Comprobación final](#apartado-7-comprobación-final)


---


## Apartado 1. Preparación del sistema

### 1.1. Dejar solo un sitio virtual activo

En la práctica anterior creé **dos sitios virtuales** (`sitio1` y `sitio2`). Ahora quiero dejar de nuevo **solo el sitio por defecto** (`000-default.conf`).

```bash
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
```

**Salida:**

```text
Enabling site 000-default.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

**Qué hace cada comando:**

- `a2ensite` (*Apache 2 enable site*): **activa** un sitio virtual creando un enlace simbólico desde `sites-available` hacia `sites-enabled`.
- `systemctl reload apache2`: **recarga la configuración sin cortar** las conexiones que ya hay abiertas. Apache no aplica el cambio por sí solo, por eso este paso es necesario.

> [!IMPORTANT]
> `a2ensite` **activa** un sitio, no lo desactiva. El comando contrario, para **desactivar** un sitio, es `a2dissite`. Si quedara activo algún sitio de la práctica anterior (`sitio1.conf`, `sitio2.conf`), se desactivaría con `sudo a2dissite sitio1.conf sitio2.conf`.


![a2ensite 000-default](imagenes/1e.png)

### 1.2. Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

**Salida final de `apt update`:**

```text
Descargados 7.440 kB en 1s (5.118 kB/s)
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Se pueden actualizar 24 paquetes. Ejecute «apt list --upgradable» para verlos.
```

**Qué hace cada comando:**

- `apt update` → **solo descarga la lista** de paquetes disponibles en los repositorios. **No instala nada.** Aquí me avisa de que hay **24 paquetes actualizables**.
- `apt upgrade -y` → **instala las actualizaciones** de esos 24 paquetes. La opción `-y` contesta "sí" automáticamente a la pregunta de confirmación.

> [!WARNING]
> No hay que confundirlos: `update` actualiza la **lista**, `upgrade` actualiza los **programas**. Se hacen siempre en ese orden para no instalar versiones antiguas o con fallos de seguridad ya corregidos.


![apt update](imagenes/e2.png)


### 1.3. Comprobar que Apache sigue funcionando

```bash
systemctl is-active apache2
```

**Salida:**

```text
active
```

`systemctl is-active` responde solo una palabra: **`active`** si el servicio está funcionando o **`inactive`** si está parado. Aquí responde `active`, así que **Apache funciona** y puedo seguir.


![Apache activo](imagenes/e3.png)

---

## Apartado 2. Instalación de MariaDB

```bash
sudo apt install mariadb-server -y
```

**Salida relevante:**

```text
Se instalarán los siguientes paquetes NUEVOS:
  galera-4 ... mariadb-client mariadb-server mariadb-server-core ...
0 actualizados, 37 nuevos se instalarán, 0 para eliminar y 24 no actualizados.
Se necesita descargar 19,7 MB de archivos.
```

**Explicación:** `apt install mariadb-server` instala el **servidor de base de datos** y todas sus dependencias:

-  **37 paquetes nuevos**.
-  **19,7 MB** de descarga.
- Incluye también el **cliente** (`mariadb-client`), que es el programa con el que me conecto al servidor.


![Instalación de MariaDB](imagenes/e4.png)

### Comprobar la versión instalada

```bash
mariadb --version
```

**Salida:**

```text
mariadb  Ver 15.1 Distrib 10.11.14-MariaDB, for debian-linux-gnu (x86_64) using  EditLine wrapper
```

**Cómo leerla:**

- `Ver 15.1` → versión del **cliente** de línea de comandos.
- `Distrib 10.11.14-MariaDB` → versión de **MariaDB instalada**.

 Tengo instalado **MariaDB 10.11.14**.



![Versión de MariaDB](imagenes/e5.png)

---

## Apartado 3. Comprobación del funcionamiento

### 3.1. Estado del servicio

```bash
sudo systemctl status mariadb
systemctl is-enabled mariadb
```

**Salida de `status` (resumida):**

```text
● mariadb.service - MariaDB 10.11.14 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-10-21 07:52:57 UTC; 11min ago
   Main PID: 27975 (mariadbd)
     Status: "Taking your SQL requests now..."
     Memory: 78.6M (peak: 81.9M)
```

**Salida de `is-enabled`:**

```text
enabled
```

**Qué significa cada cosa:**

-  **`active (running)`** → el servicio **está funcionando ahora mismo**.
-  **`enabled`** → el servicio **arrancará solo** cada vez que se encienda el equipo.
- `Status: "Taking your SQL requests now..."` → MariaDB está listo para recibir consultas.
- `Main PID: 27975` → número del proceso principal (`mariadbd`).
- `Memory: 78.6M` → memoria RAM que usa.

> [!TIP]
> Ubuntu deja MariaDB **configurado y arrancado automáticamente** al instalarlo: no hay que hacer nada más. Para salir de la pantalla de estado se pulsa la tecla **`q`**.


![Estado de MariaDB](imagenes/e6.png)


![MariaDB enabled](imagenes/e7.png)

### 3.2. Entrar en la base de datos

```bash
sudo mariadb
```

**Salida:**

```text
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 31
Server version: 10.11.14-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04
```

**Explicación:**

- Entro en el **monitor de MariaDB**: el prompt cambia a `MariaDB [(none)]>`.
- Eso demuestra que el servidor **está funcionando y responde**.
- Entro con `sudo` como **administrador sin contraseña**, porque Ubuntu usa autenticación por socket para el usuario `root` de MariaDB.
- Para volver al terminal de Linux escribo **`exit`** y pulso Enter.


![Entrar en MariaDB](imagenes/e8.png)

---

## Apartado 4. Comandos principales

Estos son los comandos que se usan para manejar el servicio, con mis palabras:

-  **`sudo systemctl start mariadb`** → **arranca** el servicio si está parado.
-  **`sudo systemctl stop mariadb`** → **detiene** el servicio; deja de aceptar conexiones.
-  **`sudo systemctl restart mariadb`** → **reinicia** el servicio (lo para y lo vuelve a arrancar). Útil tras cambiar la configuración.
-  **`sudo systemctl status mariadb`** → **muestra el estado**: si funciona, desde cuándo, memoria usada y últimas líneas del log.
-  **`sudo systemctl enable mariadb`** → lo **arranca automáticamente** al encender el equipo.

**Comandos de comprobación relacionados:**

- `systemctl is-active mariadb` → ¿funciona **ahora**? Responde `active` o `inactive`.
- `systemctl is-enabled mariadb` → ¿arrancará **solo** al encender? Responde `enabled` o `disabled`.

> [!IMPORTANT]
> **`start` / `stop`** actúan **ahora mismo**. **`enable` / `disable`** actúan en el **próximo arranque** del equipo. Son cosas distintas.

---

## Apartado 5. ¿Qué pasa si MariaDB está parado?

Más adelante, cuando WordPress muestre el error *"Error al establecer una conexión con la base de datos"*, **lo primero que tendré que mirar es si MariaDB está funcionando**. Para ver cómo se nota, lo detengo a propósito:

```bash
sudo systemctl stop mariadb
systemctl is-active mariadb
sudo mariadb
```

**Salida:**

```text
inactive
ERROR 2002 (HY000): Can't connect to local server through socket '/run/mysqld/mysqld.sock' (2)
```

**Explicación:**

- El estado es **`inactive`**.
- Al intentar entrar sale el **ERROR 2002**, que significa: **"no puedo conectar con MariaDB porque no está en marcha"**.
- El fichero *socket* `/run/mysqld/mysqld.sock` solo existe cuando el servicio está arrancado; por eso no lo encuentra.

> [!WARNING]
> **ERROR 2002 = MariaDB parado.** Es el error típico que se verá en WordPress si la base de datos no está en marcha.


![Error 2002](imagenes/e9.png)

### Recuperación

Lo vuelvo a arrancar y compruebo que todo vuelve a la normalidad:

```bash
sudo systemctl start mariadb
systemctl is-active mariadb
```

**Salida:**

```text
active
```

 Vuelve a estar `active`: el problema está resuelto.


![MariaDB recuperado](imagenes/e10.png)

###  Pregunta de reflexión

**Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos de este apartado usarías para saber si el problema es que MariaDB está parado?**

1. **`systemctl is-active mariadb`** → si responde `inactive`, MariaDB está parado.
2. **`sudo mariadb`** → si da el `ERROR 2002` (*Can't connect to local server through socket*), confirma que no hay ningún servidor escuchando.

 Si está parado, se soluciona con **`sudo systemctl start mariadb`**.

---

## Apartado 6. Instalación de PHP

WordPress está hecho en **PHP**, así que instalo PHP y las piezas que necesita:

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y
php -v
```

**Salida relevante de la instalación:**

```text
0 actualizados, 22 nuevos se instalarán, 0 para eliminar y 24 no actualizados.
Se necesita descargar 5.858 kB de archivos.
Se utilizarán 25,3 MB de espacio de disco adicional después de esta operación.
```

**Salida de `php -v`:**

```text
PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.3.6, Copyright (c) Zend Technologies
    with Zend OPcache v8.3.6, Copyright (c), by Zend Technologies
```

 Tengo instalado **PHP 8.3.6**.

### Para qué sirve cada paquete

-  **`php`** → el lenguaje PHP.
-  **`libapache2-mod-php`** → permite a **Apache ejecutar PHP**.
-  **`php-mysql`** → permite a **PHP hablar con MariaDB** (**imprescindible**).
-  **`php-curl`** → conexiones a otros servidores (actualizaciones, plugins).
-  **`php-gd`** → tratamiento de imágenes.
-  **`php-mbstring`** → manejo de textos con tildes y caracteres especiales.
-  **`php-xml`** → lectura y escritura de ficheros XML.
-  **`php-zip`** → instalación y actualización de plugins y temas.
-  **`php-intl`** → idiomas y formatos regionales.


![Instalación de PHP](imagenes/e11.png)


![php -v](imagenes/e12.png)

### 6.1. Comprobar que Apache ha cargado PHP

```bash
sudo systemctl restart apache2
sudo apache2ctl -M | grep -E "mpm|php"
```

**Salida:**

```text
 mpm_prefork_module (shared)
 php_module (shared)
```

**Qué hace cada parte:**

- `systemctl restart apache2` → reinicia Apache para que **cargue el módulo de PHP**.
- `apache2ctl -M` → lista todos los **módulos cargados** en Apache.
- `| grep -E "mpm|php"` → filtra solo las líneas que contienen `mpm` o `php`.

**Resultado:**

-  `php_module` → Apache ya **puede ejecutar PHP**.
-  `mpm_prefork_module` → es el modo de trabajo de Apache que necesita `libapache2-mod-php`.

> [!NOTE]
> Hace falta **`restart`** (y no `reload`) cuando se activan o instalan **módulos**.


![Módulos de Apache](imagenes/e13.png)

### 6.2. Comprobar que PHP puede hablar con MariaDB

```bash
php -m | grep -i mysql
```

**Salida:**

```text
mysqli
mysqlnd
pdo_mysql
```

**Explicación:** `php -m` lista los módulos de PHP y `grep -i mysql` filtra los que contienen "mysql" (sin distinguir mayúsculas). Aparecen **`mysqli`**, **`mysqlnd`** y **`pdo_mysql`**: son las piezas que usa **WordPress para conectarse a la base de datos**. Vienen del paquete `php-mysql`.


![Módulos MySQL de PHP](imagenes/e14.png)

### 6.3. Comprobar que Apache ejecuta PHP

```bash
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php
```

**Salida:**

```text
<?php phpinfo();
```

**Explicación:**

- Creo un **fichero de prueba** `info.php` en `/var/www/html` (la carpeta web de Apache).
- `echo` escribe el código PHP y `tee` lo guarda en el fichero.
- Uso **`sudo tee`** porque la carpeta es de `root` y una redirección `>` normal **no tendría permisos** con `sudo`.
- `phpinfo()` es una función de PHP que muestra toda la información de PHP.

Abro **`http://IP_DEL_SERVIDOR/info.php`** en el navegador y aparece la **página morada de PHP**, donde se ve:

- **Server API:** `apache2handler` → Apache está ejecutando PHP.
- **Apache Version:** Apache/2.4.58 (Ubuntu).
- **User/Group:** `www-data` → el usuario con el que corre Apache.
- **Loaded Modules:** incluye `mod_php` y `prefork`.


![Crear info.php](imagenes/e15.png)


![info.php en el navegador](imagenes/e16.png)

### 6.4. Borrar el fichero de prueba (obligatorio)

```bash
sudo rm /var/www/html/info.php
ls /var/www/html
```

**Salida:**

```text
index.html
```

Borro `info.php` y compruebo con `ls` que **solo queda `index.html`**.

> [!CAUTION]
> **Borrar `info.php` es obligatorio.** Es un fichero de prueba que nunca debe quedarse en un servidor.


![Borrar info.php](imagenes/e18.png)

###  Preguntas de reflexión

**¿Por qué es peligroso dejar `info.php` en un servidor real?**

Porque `phpinfo()` muestra **información interna del servidor**:

- Versión exacta de PHP y de Apache.
- Sistema operativo.
- Rutas de directorios.
- Módulos cargados y toda la configuración de PHP.

Un atacante puede usar esos datos para **buscar vulnerabilidades conocidas** de esas versiones y preparar un ataque. Además, cualquiera que conozca la URL puede verla.

**¿Para qué sirve el paquete `php-mysql`?**

Permite que **PHP se comunique con MariaDB/MySQL** (aporta `mysqli`, `mysqlnd` y `pdo_mysql`). Es **imprescindible**: sin él, WordPress no podría conectarse a la base de datos aunque MariaDB estuviera funcionando.

---

## Apartado 7. Comprobación final

```bash
systemctl is-active apache2 mariadb
php -v
```

**Salida:**

```text
active
active
PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)
```

`is-active` responde **`active` dos veces** (primero Apache y después MariaDB) y PHP muestra la versión **8.3.6**.


![Comprobación final](imagenes/e18.png)

###  Estado final del servidor

-  **Linux:** Ubuntu 24.04 → instalado.
-  **Apache:** Apache 2.4.58 → funcionando.
-  **MariaDB:** MariaDB 10.11.14 → funcionando.
-  **PHP:** PHP 8.3.6 → funcionando y preparado para MariaDB.

> [!TIP]
> Para terminar, hago una **instantánea (snapshot)** de la máquina virtual con el servidor listo. Será mi **punto de partida seguro** para instalar WordPress.


---


> [!IMPORTANT]
> **Ideas clave para recordar:**
>
> - **ERROR 2002** (`Can't connect to local server through socket`) = MariaDB **parado**.
> - **`php-mysql`** = permite a PHP conectar con MariaDB. Sin él, WordPress no funciona.
> - **`libapache2-mod-php`** = permite a Apache ejecutar PHP.
> - **`info.php`** = se **borra siempre**: revela información sensible del servidor.
> - **`start` / `stop`** = ahora. **`enable` / `disable`** = en el próximo arranque.
> - **`update`** = actualiza la lista. **`upgrade`** = instala las actualizaciones.
> - **LAMP** = Linux + Apache + MariaDB (MySQL) + PHP.

---

*Memoria enlazada desde el [README.md](README.md) del repositorio.*
