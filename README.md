# Veterinaria Profesional

Proyecto educativo con Python, Django, MariaDB, HTML, CSS y JavaScript. Permite gestionar propietarios, mascotas, veterinarios, citas, diagnósticos, medicamentos y recetas: siete tablas de negocio, además de las tablas internas de Django.

## Instalación para el profesor

Los comandos siguientes se ejecutan en **PowerShell**, desde la terminal de Visual Studio Code. No se necesita Apache ni colocar el proyecto dentro de XAMPP.

### 1. Requisitos

- Git o descargar el repositorio desde **Code > Download ZIP**.
- **Python 3.14 de 64 bits**, para utilizar el entorno verificado.
- **MariaDB 10.11 o superior**. Se verificó con MariaDB 11.4.13.

Las versiones de las dependencias están fijadas en requirements.txt. MariaDB 10.4, incluida en algunas instalaciones de XAMPP, no es compatible con el Django de este proyecto. Puede utilizarse una instancia independiente en otro puerto sin reemplazar las bases existentes.

### 2. Descargar

```powershell
git clone https://github.com/Angel-ETECH/sistema_veterinaria.git
cd sistema_veterinaria
```

Si se descargó el ZIP, descomprimirlo y abrir la carpeta donde está manage.py.

### 3. Crear el entorno e instalar dependencias

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

No es necesario activar el entorno ni cambiar la política de PowerShell. Si py no está disponible, utilizar python -m venv .venv después de comprobar la versión con python --version. No copiar .venv entre computadoras.

### 4. Configurar .env

```powershell
Copy-Item .env.example .env
.\.venv\Scripts\python.exe -c "import secrets; print(secrets.token_urlsafe(48))"
```

Copiar la clave generada al valor DJANGO_SECRET_KEY de .env y completar los datos del servidor MariaDB:

```dotenv
DJANGO_DEBUG=1
DJANGO_SECRET_KEY=pegar-aqui-la-clave-generada
DJANGO_ALLOWED_HOSTS=127.0.0.1,localhost
DJANGO_CSRF_TRUSTED_ORIGINS=
DB_NAME=veterinaria_profesional_db
DB_USER=root
DB_PASSWORD=tu_contrasena_de_mariadb
DB_HOST=127.0.0.1
DB_PORT=3307
REGISTRATION_CODE=
DJANGO_SECURE_SSL_REDIRECT=0
```

- El puerto de esta laptop es 3307. Si su MariaDB usa 3306, cambiar DB_PORT a 3306.
- Si su usuario no tiene contraseña, dejar DB_PASSWORD= vacío.
- Usar los datos reales de conexión. El archivo .env es privado y no está incluido en GitHub.

### 5. Crear la base

Encender MariaDB. Ejecutar database/crear_bd.sql desde su consola SQL, HeidiSQL o phpMyAdmin conectado al servidor compatible:

```sql
CREATE DATABASE IF NOT EXISTS veterinaria_profesional_db
CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

No es necesario importar una base de otro proyecto. Django creará las tablas con las migraciones.

### 6. Crear tablas, recursos y cuenta de revisión

```powershell
.\.venv\Scripts\python.exe manage.py migrate
.\.venv\Scripts\python.exe manage.py collectstatic --noinput
.\.venv\Scripts\python.exe manage.py createsuperuser
.\.venv\Scripts\python.exe manage.py check
```

Elegir usuario, correo y contraseña. Al escribir la contraseña la terminal no muestra caracteres: es normal. La cuenta creada puede ingresar al sistema y al administrador. Los usuarios locales del alumno no se transfieren al descargar el código.

El comando collectstatic prepara los recursos y el manifiesto que utiliza WhiteNoise; también debe ejecutarse antes de las pruebas automáticas.

### 7. Ejecutar

```powershell
.\.venv\Scripts\python.exe manage.py runserver 8002
```

Mantener la terminal abierta y visitar:

| Página | Dirección |
|---|---|
| Sistema | http://127.0.0.1:8002/ |
| Login | http://127.0.0.1:8002/login/ |
| Registro | http://127.0.0.1:8002/registro/ |
| Administrador | http://127.0.0.1:8002/admin/ |

Ingresar con la cuenta creada en el paso anterior. Para detenerlo, presionar **Ctrl + C**. Si el puerto está ocupado, usar runserver 8004 y abrir http://127.0.0.1:8004/.

## Ejecutarlo otro día

Encender MariaDB, abrir una terminal en la carpeta de manage.py y ejecutar:

```powershell
.\.venv\Scripts\python.exe manage.py runserver 8002
```

No se necesita repetir las migraciones ni crear nuevamente la cuenta.

## Recorrido para la revisión

1. Iniciar sesión y registrar un propietario con datos ficticios.
2. Registrar una mascota relacionada con ese propietario.
3. Registrar un veterinario y un medicamento.
4. Programar una cita para la mascota con el veterinario.
5. Registrar un diagnóstico asociado a la cita.
6. Registrar una receta vinculada al diagnóstico y al medicamento.
7. Probar las búsquedas y editar registros en los siete módulos.
8. Revisar el reporte de citas en /reportes/citas/ y filtrar por fechas y estado.
9. Usar Imprimir en citas, diagnósticos o recetas. Para PDF, elegir Guardar como PDF en la ventana del navegador.
10. Intentar borrar un propietario con mascotas para comprobar la protección de relaciones. Las eliminaciones se prueban con registros sin dependencias.

Cada receta contiene un medicamento por registro. Para varios medicamentos se crean varias recetas relacionadas con el mismo diagnóstico.

## Autenticación y alcance

El login permite mantener la sesión abierta durante catorce días. El registro crea usuarios normales sin privilegios del administrador de Django. En desarrollo el registro está habilitado; en producción se requiere REGISTRATION_CODE.

En esta versión educativa, todos los usuarios autenticados pueden consultar y modificar los módulos. Para una clínica real se necesita autorización por puesto, auditoría, HTTPS, DEBUG desactivado, registro controlado y políticas de privacidad y respaldos. La instalación descrita es para revisión local.

## Pruebas

Con la conexión a MariaDB configurada:

```powershell
.\.venv\Scripts\python.exe manage.py collectstatic --noinput
.\.venv\Scripts\python.exe manage.py test apps --noinput
```

Se verificaron nueve pruebas sobre autenticación, registro, formularios de los siete módulos, reportes, impresiones, citas duplicadas y protección de relaciones. Las pruebas crean y eliminan una base temporal test_veterinaria_profesional_db; el usuario de conexión debe tener permisos para esa operación. No usar ese nombre para guardar datos propios.

## Problemas comunes

| Problema | Qué revisar |
|---|---|
| No puede conectar / error 2003 | Encender MariaDB y revisar host y puerto. |
| Access denied / error 1045 | Corregir DB_USER y DB_PASSWORD. |
| Unknown database / error 1049 | Crear la base del paso 5. |
| Versión de MariaDB incompatible | Utilizar MariaDB 10.11 o superior. |
| Missing staticfiles manifest entry | Ejecutar collectstatic --noinput. |
| No module named django | Instalar requirements.txt y usar el Python de .venv. |
| Error instalando mysqlclient | Comprobar Python 3.14 de 64 bits y acceso a internet. |
| No autorizado en /admin/ | Ingresar con la cuenta creada mediante createsuperuser. |

## Organización

- config/: configuración y rutas principales.
- apps/models/: una clase por archivo para los siete modelos.
- apps/forms.py: formularios y validaciones.
- apps/views.py y apps/urls.py: lógica y rutas.
- apps/templates/control/: páginas HTML.
- apps/static/control/: estilos y JavaScript.
- apps/migrations/: creación de tablas.
- database/crear_bd.sql: creación de la base independiente.

El repositorio excluye .env, .venv, respaldos, registros, archivos generados en staticfiles y datos de usuarios.
