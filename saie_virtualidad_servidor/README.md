# SAIE · Virtualidad compartida

Aplicación multiusuario con Python 3.12, SQLite y sin paquetes externos. Cada profesional ingresa con su usuario, registra inicio y fin, escribe resumen y acta con guardado automático. Administración puede consultar todas las sesiones, abrir actas, filtrar horas por mes o rango y exportar CSV.

## Opción guiada: Railway

1. Descomprimir el ZIP. Crear un repositorio **privado** en GitHub y subir **el contenido de la carpeta `saie_virtualidad_servidor` en la raíz** del repositorio (`Dockerfile`, `app.py` y `static/`). No subir un archivo `.env` real ni contraseñas.
2. En Railway, crear un proyecto y elegir **Deploy from GitHub repo**. Seleccionar ese repositorio. Detectará el `Dockerfile`.
3. En el servicio, **Variables**, agregar `SAIE_ADMIN_PASSWORD` con una contraseña única de al menos 12 caracteres, `SAIE_ADMIN_USER=admin`, `SAIE_HTTPS=1`, `SAIE_DB=/app/data/saie.sqlite3` y `RAILWAY_RUN_UID=0`. El volumen de Railway se monta con propiedad de root. No hace falta copiar `PORT`; Railway lo asigna.
4. **Attach Volume** al servicio con ruta de montaje `/app/data` antes de cargar datos reales. Sin el volumen, la base SQLite podría perderse al redesplegar.
5. En **Settings → Networking**, usar **Generate Domain**. Abrir la URL HTTPS, ingresar con `admin` y la clave del paso 3, y crear los profesionales.
6. Para correos, agregar las variables `SAIE_SMTP_*` según el proveedor del correo. Hacer una prueba de finalización y comprobar que llegue el mensaje. Activar respaldos del volumen.

Railway puede tener costos de servicio y almacenamiento; revisar el plan vigente antes de usar datos reales.

## Instalación con Docker en un servidor

1. Copiar esta carpeta al servidor.
2. Copiar `.env.example` a `.env` y cambiar `SAIE_ADMIN_PASSWORD` por una clave única de 12 caracteres o más. No publicar `.env`.
3. Ejecutar `docker compose up -d --build`.
4. Configurar un dominio con HTTPS y un proxy inverso que apunte a `127.0.0.1:8080`. El puerto 8080 queda accesible solo desde el servidor. Para pruebas locales sin HTTPS, usar `SAIE_HTTPS=0` en `.env`; en producción usar `1`.
5. Entrar con el usuario `admin` y la clave configurada. Crear cada profesional con un usuario y contraseña propios. Entregar esas credenciales en privado.

El archivo SQLite vive en el volumen `saie_data`. Realizar copias de seguridad periódicas de ese volumen, preferentemente con la función de respaldo de SQLite. No borrar el volumen al actualizar el contenedor. Las sesiones del HTML anterior guardadas en navegadores no se migran automáticamente.

## Correo automático

Completar las variables `SAIE_SMTP_*` en `.env` con datos de un servidor SMTP. Al crear cada profesional, cargar el correo de coordinación que recibirá el aviso. Al finalizar se envía inicio, fin, resumen y acta. Si SMTP falla, la sesión queda guardada y la pantalla indica el problema. Si el acta contiene información sensible, verificar con coordinación si debe enviarse íntegra por correo o consultarse solo dentro de la aplicación.

## Alcances

- Solo administración ve todas las sesiones. Cada profesional ve las suyas.
- La duración se calcula con las fechas del servidor, incluso si se cierra la pestaña. La sesión se cierra al pulsar Finalizar.
- El filtro de meses usa la fecha local del navegador correspondiente al inicio de cada sesión. Los tiempos son reales entre inicio y fin; una pestaña abierta sin actividad también cuenta hasta que se cierre la sesión.
- Si dos profesionales usan el mismo usuario, compartirán la misma sesión; entregar cuentas individuales.
- Este paquete está preparado para desplegar, pero no trae un dominio, un servidor ni credenciales SMTP.
