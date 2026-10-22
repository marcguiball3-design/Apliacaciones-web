**Práctica: Instalación de MariaDB y PHP en Ubuntu 24.04 (completando el LAMP)**  
| | |  
|-|-|  
|   |   |   
| **Alumno** | Marcos |   
| **Ciclo / Módulo** | Sistemas Microinformáticos y Redes (SMR) · Aplicaciones Web |   
| **Curso** | 2026/2027 |   
| **Fecha** | Octubre de 2026 |   
| **Máquina** | smr-web · Ubuntu 24.04 · Apache 2.4.58 |   
   
**Índice**  
1. [Antes de empezar: qué es una base de datos](#anchor-1 "#anchor-1")  
2. [Preparación del sistema](#anchor-2 "#anchor-2")  
3. [Instalación de MariaDB](#anchor-3 "#anchor-3")  
4. [Comprobación del funcionamiento](#anchor-4 "#anchor-4")  
5. [Comandos principales](#anchor-5 "#anchor-5")  
6. [¿Qué pasa si MariaDB está parado?](#anchor-6 "#anchor-6")  
7. [Instalación de PHP](#anchor-7 "#anchor-7")  
8. [Comprobación final](#anchor-8 "#anchor-8")  
9. [Conclusiones y problemas encontrados](#anchor-9 "#anchor-9")  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAAM0lEQVR4nO3OMQ0AIAwAwdIgBKl1gjacsGCAiZDcTT9+q6oRETMAAPjF6ify6QYAADdyA9Y0AypN+bdfAAAAAElFTkSuQmCC)  
**Antes de empezar: qué es una base de datos**  
Una **base de datos** es como un archivador muy ordenado donde un programa guarda su información para encontrarla y usarla rápido. Se organiza en  **tablas** (como las hojas de una hoja de cálculo), con  **columnas** (qué datos se guardan) y  **filas** (cada elemento guardado).  
**¿Para qué la necesita WordPress?** WordPress guarda en la base de datos todo lo que no son ficheros: entradas, páginas, comentarios, usuarios y la configuración del sitio. Cada vez que alguien visita la web, WordPress le pide los datos a la base de datos.  
**¿Qué es MariaDB?** Es el programa (gestor de bases de datos) que guarda los datos, los protege y responde a las peticiones. Viene en los repositorios de Ubuntu y funciona muy bien con WordPress.  
**Cómo encaja todo (el LAMP):**  
Navegador ──► Apache ──► PHP ──► MariaDB  
 (visitante)   (sirve la   (ejecuta   (guarda los  
                web)        WordPress)  datos)  
   
| | | |  
|-|-|-|  
| **Letra** | **Pieza** | **Estado** |   
| **L** | Linux (Ubuntu 24.04) | Ya instalado |   
| **A** | Apache | Ya instalado (práctica anterior) |   
| **M** | MariaDB | **Lo instalo hoy** |   
| **P** | PHP | **Lo instalo hoy** |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSfYxZo/jkUsYQLPJrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA4rDBc72meO5AAAAAElFTkSuQmCC)  
**Apartado 1. Preparación del sistema**  
**1.1. Dejar solo un sitio virtual activo**  
En la práctica anterior creé dos sitios virtuales (sitio1 y sitio2). Ahora vuelvo a activar el sitio por defecto:  
sudo a2ensite 000-default.conf  
 sudo systemctl reload apache2  
   
Salida:  
Enabling site 000-default.  
 To activate the new configuration, you need to run:  
   systemctl reload apache2  
   
a2ensite **activa** un sitio creando un enlace simbólico en sites-enabled. Apache no aplica el cambio solo, por eso hago reload: relee la configuración sin cortar las conexiones activas. (Para desactivar un sitio se usaría a2dissite.)  
*📸 * ***PON AQUÍ LA IMAGEN 1*** * (captura de * *a2ensite 000-default.conf* * y * *reload* *) → * *img/01-a2ensite.png*  
**1.2. Actualizar el sistema**  
sudo apt update  
 sudo apt upgrade -y  
   
- apt update descarga la **lista** de paquetes disponibles (no instala nada). Salida final: *"Se pueden actualizar 24 paquetes"*.  
- apt upgrade -y**instala** esas actualizaciones (-y responde "sí" automáticamente).  
*📸 * ***PON AQUÍ LA IMAGEN 2*** * (captura de * *apt update* *) → * *img/02-apt-update.png*  
*📸 * ***(FALTA – haz esta captura)*** * * *sudo apt upgrade -y* *. En mi captura solo aparece * *apt update* *. Guárdala como * *img/02b-apt-upgrade.png* * y añádela aquí:*  
 *  
 * *![apt upgrade](img/02b-apt-upgrade.png)*  
**1.3. Comprobar que Apache sigue funcionando**  
systemctl is-active apache2  
   
Salida: active → Apache está funcionando.  
*📸 * ***PON AQUÍ LA IMAGEN 3*** * (captura de * *systemctl is-active apache2* *) → * *img/03-apache-active.png*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OMQ2AABAAsSPBCUZfE2IYmVDBhAU2QtIq6DIzW7UHAMBfnGt1V8fXEwAAXrse/xcF7U7sx4wAAAAASUVORK5CYII=)  
**Apartado 2. Instalación de MariaDB**  
sudo apt install mariadb-server -y  
   
Salida relevante:  
Se instalarán los siguientes paquetes NUEVOS:  
   galera-4 ... mariadb-client mariadb-server mariadb-server-core ...  
 0 actualizados, 37 nuevos se instalarán, 0 para eliminar y 24 no actualizados.  
 Se necesita descargar 19,7 MB de archivos.  
   
apt install mariadb-server instala el servidor de base de datos y sus dependencias (37 paquetes nuevos, 19,7 MB). Incluye también el cliente (mariadb-client) para poder conectarme.  
*📸 * ***PON AQUÍ LA IMAGEN 4*** * (captura de la instalación de * *mariadb-server* *) → * *img/04-instalar-mariadb.png*  
Compruebo la versión instalada:  
mariadb --version  
   
mariadb  Ver 15.1 Distrib 10.11.14-MariaDB, for debian-linux-gnu (x86_64) using  EditLine wrapper  
   
Ver 15.1 es la versión del cliente y Distrib 10.11.14 la de MariaDB: tengo **MariaDB 10.11.14**.  
*📸 * ***PON AQUÍ LA IMAGEN 5*** * (captura de * *mariadb --version* *) → * *img/05-version-mariadb.png*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSfYxZo/jkUsYQLPJrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA4rDBc72meO5AAAAAElFTkSuQmCC)  
**Apartado 3. Comprobación del funcionamiento**  
**3.1. Estado del servicio**  
sudo systemctl status mariadb  
 systemctl is-enabled mariadb  
   
Salida de status (resumida):  
● mariadb.service - MariaDB 10.11.14 database server  
      Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; preset: enabled)  
      Active: active (running) since Wed 2026-10-21 07:52:57 UTC; 11min ago  
    Main PID: 27975 (mariadbd)  
      Status: "Taking your SQL requests now..."  
      Memory: 78.6M (peak: 81.9M)  
   
Salida de is-enabled: enabled.  
- **active (running)**: el servicio está funcionando ahora mismo.  
- **enabled**: arrancará solo cada vez que se encienda el equipo.  
- Ubuntu lo deja todo configurado automáticamente al instalar. Para salir de la pantalla de estado se pulsa q.  
*📸 * ***PON AQUÍ LA IMAGEN 6*** * (captura de * *systemctl status mariadb* *) → * *img/06-status-mariadb.png*  
*📸 * ***PON AQUÍ LA IMAGEN 7*** * (captura de * *systemctl is-enabled mariadb* *) → * *img/07-is-enabled-mariadb.png*  
**3.2. Entrar en la base de datos**  
sudo mariadb  
   
Welcome to the MariaDB monitor.  Commands end with ; or \g.  
 Your MariaDB connection id is 31  
 Server version: 10.11.14-MariaDB-0ubuntu0.24.04.1 Ubuntu 24.04  
   
Entro en el monitor de MariaDB: el prompt cambia a MariaDB [(none)]>, lo que demuestra que el servidor **está funcionando y responde**. Con sudo entro como administrador sin contraseña (Ubuntu usa autenticación por socket para root). Para volver al terminal escribo exit.  
*📸 * ***PON AQUÍ LA IMAGEN 8*** * (captura de * *sudo mariadb* *) → * *img/08-entrar-mariadb.png*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OUQmAABBAsSeYxZyXSzCJASxgACv4J8KWYMvMbNURAAB/ca7VXe1fTwAAeO16AKe+BdmJqrPdAAAAAElFTkSuQmCC)  
**Apartado 4. Comandos principales**  
| | | |  
|-|-|-|  
| **Comando** | **Función** | **Explicación con mis palabras** |   
| sudo systemctl start mariadb | Arranca el servicio | Lo pone en marcha si está parado |   
| sudo systemctl stop mariadb | Detiene el servicio | Deja de aceptar conexiones |   
| sudo systemctl restart mariadb | Reinicia el servicio | Lo para y lo arranca de nuevo (útil tras cambiar la configuración) |   
| sudo systemctl status mariadb | Muestra su estado | Dice si funciona, desde cuándo, memoria usada y últimas líneas del log |   
| sudo systemctl enable mariadb | Arranque automático | Se inicia solo al encender el equipo |   
| systemctl is-active mariadb | ¿Funciona ahora? | Responde active o inactive |   
| systemctl is-enabled mariadb | ¿Arrancará solo? | Responde enabled o disabled |   
   
*start* * / * *stop* * actúan * ***ahora*** *; * *enable* * / * *disable* * actúan en el * ***próximo arranque*** * del equipo.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANElEQVR4nO3OUQmAABBAsSeILQSjXgcrmkOs4J8IW4ItM7NXZwAA/MW1Vlt1fBwBAOC9+wEukwQ+V/SggAAAAABJRU5ErkJggg==)  
**Apartado 5. ¿Qué pasa si MariaDB está parado?**  
Más adelante, cuando WordPress muestre *"Error al establecer una conexión con la base de datos"*, lo primero que tendré que mirar es si MariaDB está funcionando. Para ver cómo se nota, lo detengo a propósito:  
sudo systemctl stop mariadb  
 systemctl is-active mariadb  
 sudo mariadb  
   
inactive  
 ERROR 2002 (HY000): Can't connect to local server through socket '/run/mysqld/mysqld.sock' (2)  
   
El estado es inactive y al intentar entrar sale el **ERROR 2002**, que significa *"no puedo conectar con MariaDB porque no está en marcha"* (el fichero socket /run/mysqld/mysqld.sock solo existe cuando el servicio está arrancado).  
*📸 * ***PON AQUÍ LA IMAGEN 9*** * (captura del * *stop* *, * *inactive* * y el ERROR 2002) → * *img/09-error-mariadb-parado.png*  
Lo vuelvo a arrancar y compruebo que todo vuelve a la normalidad:  
sudo systemctl start mariadb  
 systemctl is-active mariadb  
   
Salida: active.  
*📸 * ***PON AQUÍ LA IMAGEN 10*** * (captura de * *start* * y * *active* *) → * *img/10-mariadb-recuperado.png*  
❓ **Si WordPress no pudiera conectar con la base de datos, ¿qué dos comandos de este apartado usarías para saber si el problema es que MariaDB está parado?**  
1. systemctl is-active mariadb: si responde inactive, MariaDB está parado.  
2. sudo mariadb: si da el ERROR 2002 (*Can't connect to local server through socket*), confirma que no hay ningún servidor escuchando.  
Si está parado, se soluciona con sudo systemctl start mariadb.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANElEQVR4nO3OQQmAABRAsad4FCtY9ecwnkms4E2ELcGWmTmrKwAA/uLeqrU6vp4AAPDa/gDzUgM9+S8z3AAAAABJRU5ErkJggg==)  
**Apartado 6. Instalación de PHP**  
WordPress está hecho en PHP, así que instalo PHP y las piezas que necesita:  
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl -y  
 php -v  
   
Salida relevante de la instalación:  
0 actualizados, 22 nuevos se instalarán, 0 para eliminar y 24 no actualizados.  
 Se necesita descargar 5.858 kB de archivos.  
 Se utilizarán 25,3 MB de espacio de disco adicional después de esta operación.  
   
Salida de php -v:  
PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)  
 Copyright (c) The PHP Group  
 Zend Engine v4.3.6, Copyright (c) Zend Technologies  
     with Zend OPcache v8.3.6, Copyright (c), by Zend Technologies  
   
Instalo **PHP 8.3.6**. Para qué sirve cada paquete:  
| | |  
|-|-|  
| **Paquete** | **Función** |   
| php | El lenguaje PHP |   
| libapache2-mod-php | Permite a **Apache ejecutar PHP** |   
| php-mysql | Permite a **PHP hablar con MariaDB** (imprescindible) |   
| php-curl | Conexiones a otros servidores (actualizaciones, plugins) |   
| php-gd | Tratamiento de imágenes |   
| php-mbstring | Manejo de textos con tildes y caracteres especiales |   
| php-xml | Lectura y escritura de ficheros XML |   
| php-zip | Instalación y actualización de plugins y temas |   
| php-intl | Idiomas y formatos regionales |   
   
*📸 * ***PON AQUÍ LA IMAGEN 11*** * (captura de la instalación de PHP) → * *img/11-instalar-php.png*  
*📸 * ***PON AQUÍ LA IMAGEN 12*** * (captura de * *php -v* *) → * *img/12-php-version.png*  
**6.1. Comprobar que Apache ha cargado PHP**  
sudo systemctl restart apache2  
 sudo apache2ctl -M | grep -E "mpm|php"  
   
 mpm_prefork_module (shared)  
  php_module (shared)  
   
Reinicio Apache para que cargue el módulo de PHP (hace falta restart, no reload, al cambiar módulos). apache2ctl -M lista los módulos cargados y grep filtra los que me interesan: php_module (Apache ya ejecuta PHP) y mpm_prefork_module (el modo de trabajo que necesita libapache2-mod-php).  
*📸 * ***PON AQUÍ LA IMAGEN 13*** * (captura de * *apache2ctl -M | grep* *) → * *img/13-apache-modulos.png*  
**6.2. Comprobar que PHP puede hablar con MariaDB**  
php -m | grep -i mysql  
   
mysqli  
 mysqlnd  
 pdo_mysql  
   
php -m lista los módulos de PHP. Aparecen mysqli, mysqlnd y pdo_mysql, las piezas que usa WordPress para conectarse a la base de datos (vienen del paquete php-mysql).  
*📸 * ***PON AQUÍ LA IMAGEN 14*** * (captura de * *php -m | grep -i mysql* *) → * *img/14-php-mysql.png*  
**6.3. Comprobar que Apache ejecuta PHP**  
echo '<?php phpinfo();' | sudo tee /var/www/html/info.php  
   
Creo un fichero de prueba en /var/www/html. Uso sudo tee porque la carpeta es de root y una redirección > normal no tendría permisos. Abro http://IP_DEL_SERVIDOR/info.php en el navegador y aparece la página morada de PHP (Server API: apache2handler, Apache/2.4.58, usuario www-data).  
*📸 * ***PON AQUÍ LA IMAGEN 15*** * (captura del comando * *echo ... | sudo tee* *) → * *img/15-crear-info-php.png*  
*📸 * ***PON AQUÍ LA IMAGEN 16*** * (captura de la página * *info.php* * en el navegador) → * *img/16-info-php-navegador.png*  
**6.4. Borrar el fichero de prueba (obligatorio)**  
sudo rm /var/www/html/info.php  
 ls /var/www/html  
   
Salida: index.html. Borro info.php y compruebo con ls que solo queda index.html.  
*📸 * ***PON AQUÍ LA IMAGEN 17*** * (captura de * *rm* * y * *ls* *) → * *img/17-borrar-info-php.png*  
❓ **¿Por qué es peligroso dejar ** **info.php** ** en un servidor real?**  
Porque phpinfo() muestra información interna del servidor: versión exacta de PHP y de Apache, sistema operativo, rutas de directorios, módulos cargados y toda la configuración de PHP. Un atacante puede usarla para buscar vulnerabilidades conocidas de esas versiones y preparar un ataque. Además, cualquiera que conozca la URL puede verla.  
❓ **¿Para qué sirve el paquete ** **php-mysql** **?**  
Permite que **PHP se comunique con MariaDB/MySQL** (aporta mysqli, mysqlnd y pdo_mysql). Es imprescindible: sin él WordPress no podría conectarse a la base de datos aunque MariaDB estuviera funcionando.  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OQQmAABRAsSfYxZo/jzlMYQLPJrCCNxG2BFtmZquOAAD4i3Ot7mr/egIAwGvXA4q7Bc870TqdAAAAAElFTkSuQmCC)  
**Apartado 7. Comprobación final**  
systemctl is-active apache2 mariadb  
 php -v  
   
active  
 active  
 PHP 8.3.6 (cli) (built: Oct  7 2026 18:06:38) (NTS)  
   
is-active responde active dos veces (primero Apache y luego MariaDB) y PHP muestra la versión 8.3.6.  
*📸 * ***PON AQUÍ LA IMAGEN 18*** * (captura de * *systemctl is-active apache2 mariadb* * y * *php -v* *) → * *img/18-comprobacion-final.png*  
Estado final del servidor:  
| | | |  
|-|-|-|  
| **Pieza del LAMP** | **Programa** | **Estado** |   
| Linux | Ubuntu 24.04 | Instalado |   
| Apache | Apache 2.4.58 | Funcionando |   
| MariaDB | MariaDB 10.11.14 | Funcionando |   
| PHP | PHP 8.3.6 | Funcionando y preparado para MariaDB |   
   
Para terminar, hago una **instantánea (snapshot)** de la máquina virtual con el servidor listo. Será mi punto de partida seguro para instalar WordPress.  
*📸 * ***(Opcional)*** * Captura del snapshot en VirtualBox/VMware.*  
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANUlEQVR4nO3OQQmAABRAsSd40A5GMORPYEt7WMGbCFuCLTNzVFcAAPzFvVZbdX49AQDgtf0BSrIDUgOg4eAAAAAASUVORK5CYII=)  
**Conclusiones y problemas encontrados**  
**Conclusiones**  
Con esta práctica he completado el **LAMP**: ya tengo Linux, Apache, MariaDB y PHP funcionando. He aprendido para qué sirve cada pieza (Apache sirve la web, PHP ejecuta el código y MariaDB guarda los datos), a manejar el servicio con systemctl y a reconocer el error típico cuando MariaDB está parado ( **ERROR 2002**). También he visto que hay que borrar info.php después de usarlo por seguridad.  
*✏️ * *Añade aquí 2–3 frases con tu opinión personal.*  
**Problemas encontrados y cómo los resolví**  
| | |  
|-|-|  
| **Problema** | **Cómo lo resolví** |   
| El enunciado habla de "deshabilitar" sitios, pero el comando es a2ensite (que **activa**) | Entendí que se activa 000-default.conf; para desactivar los otros sitios se usa a2dissite |   
| apt update solo actualiza la lista de paquetes | Ejecuté también apt upgrade -y para instalar los 24 paquetes pendientes |   
| ERROR 2002 al intentar entrar con MariaDB parado (provocado a propósito) | Arranqué el servicio con sudo systemctl start mariadb y comprobé con is-active |   
| ✏️ *Añade aquí cualquier otro problema real que hayas tenido* |   |   
   
![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAnEAAAACCAYAAAA3pIp+AAAABmJLR0QA/wD/AP+gvaeTAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAANklEQVR4nO3OMQ2AABAAsSNBCkLfFR7wwIgHRiywEZJWQZeZ2ao9AAD+4lyruzq+ngAA8Nr1AOIEBeX8aGZPAAAAAElFTkSuQmCC)  
*Memoria enlazada desde el *[ *README.md* * del repositorio.*](README.md "README.md")  
