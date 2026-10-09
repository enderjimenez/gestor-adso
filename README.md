# Gestor ADSO (Kubuntu / Linux)

## 1. Nombre y propósito del proyecto

**Gestor ADSO** es el proyecto integrador ABP (Aprendizaje Basado en Proyectos) del programa ADSO, SENA Regional Antioquia · CTMA, desarrollado con **PHP y Laravel**.

Esta primera semana tiene como propósito dejar un **entorno de desarrollo reproducible en Kubuntu**: instalar las herramientas, crear el proyecto Laravel `gestor-adso`, conectarlo a una base de datos y registrar el primer commit. Este mismo proyecto seguirá creciendo en las siguientes semanas.

- **Resultado de aprendizaje:** RA EL02 · Norma 220501094, *Elaborar propuesta técnica del software según especificaciones*.

---

## 2. Herramientas y versiones reales

> Ejecuta cada comando en tu equipo y pega el resultado real. No escribas versiones inventadas.

| Herramienta | Comando de verificación | Versión real instalada |
|---|---|---|
| Kubuntu | `lsb_release -a` | _(pegar versión)_ |
| PHP (CLI) | `php -v` | _(pegar versión)_ |
| Ruta de PHP | `which php` | `/usr/bin/php` |
| Composer | `composer -V` | _(pegar versión)_ |
| Laravel | `php artisan --version` | _(pegar versión)_ |
| MariaDB | `mariadb --version` | _(pegar versión)_ |
| Git | `git --version` | _(pegar versión)_ |
| Visual Studio Code | `code --version` | _(pegar versión)_ |

**Extensiones de PHP requeridas:** `openssl`, `PDO`, `pdo_mysql`, `mbstring`, `fileinfo`, `zip` (y `xml`, `curl`, que Laravel/Composer también usan).

En Linux se usa el **PHP instalado con `apt`** (`/usr/bin/php`), de modo que la terminal, Composer y Laravel trabajan con la misma instalación. No se necesita configurar el PATH manualmente.

---

## 3. Pasos de instalación y configuración de `.env`

### 3.1 Instalación del entorno

Orden de instalación: **Entorno → Herramientas → Proyecto → Versionar**. No se continúa si la verificación del paso anterior falla.

**1. Actualizar el sistema**

```bash
sudo apt update && sudo apt upgrade -y
```

**2. Instalar PHP y sus extensiones**

```bash
sudo apt install -y php-cli php-mbstring php-xml php-curl php-zip php-mysql unzip curl
```

`openssl`, `PDO` y `fileinfo` vienen incluidos en el paquete base de PHP. Verificar:

```bash
which php
php -v
php --ini
php -m | grep -Ei "openssl|pdo|pdo_mysql|mbstring|fileinfo|zip"
```

- `which php` debe mostrar `/usr/bin/php`.
- `php --ini` muestra el `php.ini` que PHP lee realmente (normalmente `/etc/php/X.Y/cli/php.ini`).
- Si falta una extensión, instálala con `sudo apt install php-NOMBRE`. No se editan líneas a mano ni se duplican extensiones.

**3. Instalar Composer (instalador oficial de `getcomposer.org`)**

```bash
cd /tmp
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
sudo mv composer.phar /usr/local/bin/composer
rm composer-setup.php
cd ~
```

Cerrar y abrir la terminal, y verificar:

```bash
composer -V
which composer
```

> Se evita `sudo apt install composer` porque suele instalar una versión desactualizada.

**4. Instalar Git**

```bash
sudo apt install -y git
git --version
```

Configurar la identidad (necesaria para el primer commit):

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@ejemplo.com"
```

**5. Instalar Visual Studio Code**

Descargar el paquete `.deb` desde `code.visualstudio.com` e instalarlo:

```bash
sudo apt install ./code_*.deb
code --version
```

(Alternativa: `sudo snap install code --classic`.)



**Verificación global antes de crear el proyecto:**

```bash
php -v
composer -V
git --version
code --version
which php
which composer
systemctl is-active mariadb
```

### 3.2 Creación del proyecto

```bash
cd ~/Documentos        # si tu sistema está en inglés: cd ~/Documents
composer create-project laravel/laravel gestor-adso
cd gestor-adso
php artisan --version
```

### 3.3 Configuración de `.env`

Crear el archivo (si no existe) y generar la clave de la aplicación:

```bash
cp .env.example .env
php artisan key:generate
```

Editar `.env` (por ejemplo con `code .env` o `kate .env`):

```env
DB_CONNECTION=mariadb
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=gestor_adso
DB_USERNAME=gestor
DB_PASSWORD=gestor1234
```


## 4. Creación de la BD y ejecución de migraciones

1. Verificar que MariaDB esté activo:
   ```bash
   sudo systemctl status mariadb
   ```
2. Crear la base de datos y el usuario:
   ```bash
   sudo mariadb
   ```
   Dentro de la consola de MariaDB:
   ```sql
   CREATE DATABASE gestor_adso CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   CREATE USER 'gestor'@'localhost' IDENTIFIED BY 'gestor1234';
   CREATE USER 'gestor'@'127.0.0.1' IDENTIFIED BY 'gestor1234';
   GRANT ALL PRIVILEGES ON gestor_adso.* TO 'gestor'@'localhost';
   GRANT ALL PRIVILEGES ON gestor_adso.* TO 'gestor'@'127.0.0.1';
   FLUSH PRIVILEGES;
   EXIT;
   ```
3. Probar la conexión con el nuevo usuario:
   ```bash
   mariadb -h 127.0.0.1 -P 3306 -u gestor -p gestor_adso
   ```
4. Limpiar la caché de configuración y ejecutar las migraciones (dentro de `gestor-adso`):
   ```bash
   php artisan config:clear
   php artisan migrate
   ```
5. Verificar que las tablas se crearon:
   ```bash
   mariadb -u gestor -p gestor_adso -e "SHOW TABLES;"
   ```

---

## 5. Cómo ejecutar el servidor

1. Verificar que MariaDB esté activo (`systemctl is-active mariadb`).
2. Dentro de la carpeta `gestor-adso`, ejecutar:
   ```bash
   php artisan serve
   ```
3. Abrir en el navegador la dirección que muestre la terminal (normalmente `http://127.0.0.1:8000`). Debe aparecer la pantalla de bienvenida de Laravel.

> La terminal queda ocupada mientras el servidor esté activo. Para otros comandos, abrir una **segunda terminal** (Konsole) dentro de `gestor-adso`.

**Problemas comunes**

| Problema | Solución |
|---|---|
| `php: command not found` | Reinstalar `php-cli` y abrir una terminal nueva. |
| Extensión faltante (`ext-xml`, `ext-zip`…) | `sudo apt install php-xml php-zip` |
| Error de conexión a la BD | Revisar que MariaDB esté activo y los valores `DB_*` de `.env`. |
| Puerto 3306 ocupado | `sudo ss -tulpn \| grep 3306` y reflejar el puerto real en `.env`. |
| Puerto 8000 ocupado | Leer la dirección real que muestra `php artisan serve`. |
| `Access denied for user` | Usar el usuario `gestor`, no `root`, y revisar la contraseña. |

---

## 6. Evidencias del proceso

Resumen del proceso realizado en la Semana 1, en orden:

1. Se actualizó Kubuntu con `apt update` y `apt upgrade`.
2. Se instaló **PHP** con `apt` y se verificó con `which php`, `php -v` y `php --ini`.
3. Se comprobó con `php -m` que las extensiones `openssl`, `PDO`, `pdo_mysql`, `mbstring`, `fileinfo` y `zip` estaban cargadas.
4. Se instaló **Composer** con el instalador oficial y se verificó con `composer -V` y `which composer`.
5. Se instaló **Git**, se configuró la identidad con `git config` y se verificó con `git --version`.
6. Se instaló **Visual Studio Code** y se verificó con `code --version`.
7. Se instaló **MariaDB**, se inició el servicio con `systemctl` y se verificó con `mariadb --version`.
8. Se hizo el control de calidad del entorno completo antes de crear el proyecto.
9. Se creó el proyecto con `composer create-project laravel/laravel gestor-adso` y se comprobó con `php artisan --version`.
10. Se creó `.env` con `cp .env.example .env`, se generó la `APP_KEY` con `php artisan key:generate` y se configuraron las variables `DB_*`.
11. Se creó la base de datos `gestor_adso` (`utf8mb4_unicode_ci`) y el usuario `gestor` desde la consola de MariaDB.
12. Se ejecutaron `php artisan config:clear` y `php artisan migrate`, y se verificaron las tablas con `SHOW TABLES;`.
13. Se levantó el servidor con `php artisan serve` y se confirmó la pantalla de bienvenida en `http://127.0.0.1:8000`.
14. Se abrió el proyecto en VS Code con `code .`.
15. Se inicializó el repositorio y se registró el primer commit:
    ```bash
    git status
    git init
    git add .
    git commit -m "Semana 1: Entorno listo + Laravel base"
    ```



**Autor:** Ender Alexander Jimenez Sanchez · Programa ADSO · SENA Regional Antioquia · CTMA
**Instructor:** Gustavo Bolaños