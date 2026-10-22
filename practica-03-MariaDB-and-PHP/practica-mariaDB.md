# Práctica: MariaDB y PHP en Ubuntu 24.04 (LAMP)

**Ciclo:** SMR · **Módulo:** Aplicaciones Web
**Alumno:** Marcos [apellidos] · **Curso:** [curso] · **Fecha:** 22/10/2026
**Máquina:** `smr-web` (Ubuntu 24.04)

---

## 0. ¿Qué vamos a hacer?

**LAMP** = **L**inux + **A**pache + **M**ariaDB + **P**HP. Ya tenía Linux y Apache; hoy instalo MariaDB y PHP para dejar el servidor listo para WordPress.

```
Navegador ──► Apache ──► PHP ──► MariaDB
              (sirve     (ejecuta   (guarda
               la web)    WordPress) los datos)
```

- **Base de datos:** un archivador ordenado. Tiene **tablas** (hojas), **columnas** (qué datos) y **filas** (cada elemento).
- **WordPress** guarda en ella entradas, páginas, comentarios, usuarios y configuración.
- **MariaDB** es el programa que gestiona la base de datos.

---

## 1. Preparación del sistema

### 1.1 Dejar solo el sitio por defecto

```bash
sudo a2ensite 000-default.conf
sudo systemctl reload apache2
```

- `a2ensite` **activa** un sitio virtual de Apache (para desactivar uno se usa `a2dissite`).
- `reload` hace que Apache lea la configuración nueva sin cortar conexiones.

📷 **IMAGEN 1** (captura de `a2ensite`):

![a2ensite](img/01-a2ensite.png)

### 1.2 Actualizar el sistema

```bash
sudo apt update
sudo apt upgrade -y
```

- `apt update`: descarga la **lista** de paquetes disponibles (no instala nada). Avisa de que hay **24 paquetes** actualizables.
- `apt upgrade -y`: **instala** esas actualizaciones (`-y` = responder "sí" automáticamente).

📷 **IMAGEN 2** (captura de `apt update`):

![apt update](img/02-apt-update.png)

### 1.3 Comprobar que Apache funciona

```bash
systemctl is-active apache2
```

Resultado: `active` → Apache está funcionando.

📷 **IMAGEN 3** (captura de `is-active apache2`):

![apache active](img/03-apache-active.png)

---

## 2. Instalación de MariaDB

```bash
sudo apt install mariadb-server -y
mariadb --version
```

- Instala el servidor de base de datos (37 paquetes nuevos, 19,7 MB).
- `--version` muestra la versión: **MariaDB 10.11.14**.

📷 **IMAGEN 4** (captura de la instalación):

![instalar mariadb](img/04-instalar-mariadb.png)

📷 **IMAGEN 5** (captura de `mariadb --version`):

![version mariadb](img/05-version-mariadb.png)

---

## 3. Comprobación del funcionamiento

### 3.1 Estado del servicio

```bash
sudo systemctl status mariadb
systemctl is-enabled mariadb
```

- `active (running)` → está funcionando **ahora**.
- `enabled` → arrancará **solo** al encender el equipo.
- Ubuntu lo deja configurado automáticamente. Para salir de la pantalla de estado se pulsa `q`.

📷 **IMAGEN 6** (captura de `status mariadb`):

![status mariadb](img/06-status-mariadb.png)

📷 **IMAGEN 7** (captura de `is-enabled mariadb`):

![is-enabled mariadb](img/07-is-enabled-mariadb.png)

### 3.2 Entrar en MariaDB

```bash
sudo mariadb
```

Sale el mensaje de bienvenida (`Welcome to the MariaDB monitor`) y el prompt cambia: estoy dentro de MariaDB y responde correctamente. Para salir: `exit`.

📷 **IMAGEN 8** (captura de la entrada en MariaDB):

![entrar mariadb](img/08-entrar-mariadb.png)

---

## 4. Comandos principales

| Comando | Qué hace |
|---|---|
| `sudo systemctl start mariadb` | **Arranca** el servicio |
| `sudo systemctl stop mariadb` | **Detiene** el servicio |
| `sudo systemctl restart mariadb` | **Reinicia** (para y arranca) |
| `sudo systemctl status mariadb` | Muestra el **estado** |
| `sudo systemctl enable mariadb` | Lo arranca **automáticamente** al encender el equipo |

> `start/stop` actúan **ahora**; `enable/disable` actúan en el **próximo arranque**.

---

## 5. ¿Qué pasa si MariaDB está parado?

```bash
sudo systemctl stop mariadb
systemctl is-active mariadb
sudo mariadb
```

Resultado:

```
inactive
ERROR 2002 (HY000): Can't connect to local server through socket '/run/mysqld/mysqld.sock' (2)
```

Significa: **"no puedo conectar con MariaDB porque no está en marcha"**.

📷 **IMAGEN 9** (captura del error):

![error mariadb parado](img/09-error-mariadb-parado.png)

Lo vuelvo a arrancar:

```bash
sudo systemctl start mariadb
systemctl is-active mariadb
```

Resultado: `active` → todo vuelve a la normalidad.

📷 **IMAGEN 10** (captura de la recuperación):

![mariadb recuperado](img/10-mariadb-recuperado.png)

### ❓ Pregunta de reflexión

**Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos usarías para saber si MariaDB está parado?**

1. `systemctl is-active mariadb` → si responde `inactive`, está parado.
2. `sudo mariadb` → si da el `ERROR 2002`, no hay servidor en marcha.

Solución: `sudo systemctl start mariadb`.

---

## 6. Instalación de PHP

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y
php -v
```

`php -v` muestra **PHP 8.3.6**.

| Paquete | Para qué sirve |
|---|---|
| `php` | El lenguaje PHP |
| `libapache2-mod-php` | Permite a **Apache ejecutar PHP** |
| `php-mysql` | Permite a **PHP hablar con MariaDB** (imprescindible) |
| `php-curl` | Conexiones a otros servidores (actualizaciones, plugins) |
| `php-gd` | Tratamiento de imágenes |
| `php-mbstring` | Textos con tildes y caracteres especiales |
| `php-xml` | Lectura y escritura de ficheros XML |
| `php-zip` | Instalar y actualizar plugins y temas |
| `php-intl` | Idiomas y formatos regionales |

📷 **IMAGEN 11** (captura de la instalación de PHP):

![instalar php](img/11-instalar-php.png)

📷 **IMAGEN 12** (captura de `php -v`):

![php version](img/12-php-version.png)

### 6.1 Comprobar que Apache carga PHP

```bash
sudo systemctl restart apache2
sudo apache2ctl -M | grep -E "mpm|php"
```

Resultado: `mpm_prefork_module` y `php_module` → Apache ya puede ejecutar PHP.

📷 **IMAGEN 13** (captura de los módulos de Apache):

![modulos apache](img/13-apache-modulos.png)

### 6.2 Comprobar que PHP habla con MariaDB

```bash
php -m | grep -i mysql
```

Resultado: `mysqli`, `mysqlnd` y `pdo_mysql` → son las piezas que usa WordPress para conectarse a la base de datos.

📷 **IMAGEN 14** (captura de los módulos de PHP):

![php mysql](img/14-php-mysql.png)

### 6.3 Comprobar que Apache ejecuta PHP

```bash
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php
```

Crea un fichero de prueba. Se usa `sudo tee` porque la carpeta es de `root`. Después abro `http://IP_DEL_SERVIDOR/info.php` en el navegador y aparece la página morada de PHP (Server API: `apache2handler`, Apache 2.4.58, usuario `www-data`).

📷 **IMAGEN 15** (captura de la creación de `info.php`):

![crear info.php](img/15-crear-info-php.png)

📷 **IMAGEN 16** (captura de la página en el navegador):

![info.php navegador](img/16-info-php-navegador.png)

### 6.4 Borrar el fichero de prueba (obligatorio)

```bash
sudo rm /var/www/html/info.php
ls /var/www/html
```

Resultado: solo queda `index.html`.

📷 **IMAGEN 17** (captura del borrado):

![borrar info.php](img/17-borrar-info-php.png)

### ❓ Preguntas de reflexión

**¿Por qué es peligroso dejar `info.php` en un servidor real?**
Porque muestra información interna del servidor (versiones de PHP y Apache, rutas, módulos, configuración). Un atacante puede usarla para buscar vulnerabilidades conocidas.

**¿Para qué sirve `php-mysql`?**
Permite que PHP se comunique con MariaDB. Sin él, WordPress no podría conectarse a la base de datos aunque MariaDB funcionara.

---

## 7. Comprobación final

```bash
systemctl is-active apache2 mariadb
php -v
```

Resultado: `active` dos veces y PHP 8.3.6.

📷 **IMAGEN 18** (captura de la comprobación final):

![comprobacion final](img/18-comprobacion-final.png)

| Pieza | Programa | Estado |
|---|---|---|
| Linux | Ubuntu 24.04 | Instalado |
| Apache | Apache 2.4.58 | Funcionando |
| MariaDB | MariaDB 10.11.14 | Funcionando |
| PHP | PHP 8.3.6 | Funcionando y preparado para MariaDB |

Por último, hago una **instantánea (snapshot)** de la máquina virtual como punto de partida para WordPress.

---

## Conclusiones y problemas

- Ya tengo el **LAMP completo** funcionando.
- Aprendí a diagnosticar un fallo típico: si MariaDB está parado sale el **ERROR 2002** y se arregla con `systemctl start mariadb`.
- Aprendí que hay que **borrar `info.php`** por seguridad.

| Problema | Solución |
|---|---|
| El enunciado dice "deshabilitar", pero `a2ensite` **activa** el sitio | Se activa `000-default.conf`; para desactivar otros se usa `a2dissite` |
| `apt update` solo actualiza la lista | Ejecutar también `apt upgrade -y` |
| ERROR 2002 al parar MariaDB (a propósito) | `sudo systemctl start mariadb` |

---

## Chuleta para el examen

```bash
sudo systemctl start|stop|restart|status|enable mariadb
systemctl is-active mariadb      # ¿funciona ahora?
systemctl is-enabled mariadb     # ¿arranca solo?
sudo mariadb                     # entrar (exit para salir)
php -v                           # versión de PHP
sudo apache2ctl -M | grep -E "mpm|php"   # Apache con PHP
php -m | grep -i mysql           # PHP con MariaDB
```

- **ERROR 2002** = MariaDB parado.
- **php-mysql** = PHP ↔ MariaDB.
- **libapache2-mod-php** = Apache ejecuta PHP.
- **info.php** = se borra siempre.
