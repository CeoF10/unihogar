# UniHogar Guaranda · Demostración y pruebas

## Preparar la demostración

1. Inicia PostgreSQL con los datos de demostración. En una base existente, aplica en orden las migraciones pendientes hasta `006_visitas_reportes_y_panorama.sql`. Usa una base local de pruebas.
2. Configura e inicia la API. En `unihogar-api`, ejecuta `npm run demo:fotos`. Las imágenes ficticias se cargan en los anuncios #1, #5 y #7. La portada del anuncio #1 se obtuvo del mismo panorama que usa el visor.
3. Abre la app Android y conecta el servidor en la pantalla de acceso. Usa `http://10.0.2.2:3000/api/` desde el emulador para una API local en el puerto 3000. La versión de producción utiliza HTTPS.
4. Abre el panel con la misma API. Los enlaces de confirmación y recuperación se pueden revisar en la salida de la API local o recibir por correo al configurar Brevo.

## Guion

- **Cuentas (RF01–RF02).** Registra un estudiante con correo institucional y un propietario con correo personal. Confirma los correos, inicia sesión y prueba la recuperación. En **Perfil → Activar otro perfil**, confirma el segundo correo y vuelve a iniciar sesión con el perfil elegido.
- **Publicaciones (RF03–RF05).** Crea un borrador con ubicación manual, contacto, costos y fotos. Desde **Mis publicaciones**, edita los datos, gestiona las fotos y adjunta un JPEG 2:1 para el recorrido. Envía a auditoría. Tras aprobar desde el panel, comprueba que pausar o marcar como arrendado lo retire del catálogo.
- **Catálogo, mapa y ficha (RF06–RF09).** Combina tipo, precio, servicios y distancia. Cambia entre lista y mapa y carga otra página. Abre una ficha, comprueba el costo recurrente y el depósito, guarda el anuncio y vuelve a iniciar sesión para revisar Favoritos.
- **Comparador, contacto y visitas (RF10–RF12).** Compara hasta tres anuncios. Prueba compartir, copiar el contacto, abrir WhatsApp y consultar cómo llegar. Solicita una visita desde la ficha. Abre **Perfil → Visitas** con cada rol para proponer otra fecha, responder o cancelar.
- **Reportes (RF13).** Reporta un anuncio con motivo. En el panel, abre **Reportes de anuncios** y elige ocultar o descartar. Revisa quién resolvió el reporte y cuándo.
- **Panorama (RF14).** En el anuncio ficticio #1, abre **Ver recorrido 360°** y prueba giro, zoom y centrado. Abre después el #5 para probar una ficha con galería convencional. El visor requiere un panorama equirectangular JPEG 2:1, de 512 a 4096 píxeles de ancho y de hasta 5 MB.

## Evidencia de la prueba local

- PostgreSQL 16. Se verificó la creación de una base con las migraciones, la actualización de una base de prueba y la restauración de un respaldo. Se compararon cantidad, tamaño y contenido de los archivos restaurados.
- API. Pasaron 38 comprobaciones de la suite del repositorio y 46 solicitudes adicionales con verificaciones de contenido y permisos para panoramas, visitas, reportes y el segundo perfil.
- Panel web. Se comprobó el ingreso, la recepción de un reporte y el registro de su resolución y autor.
- Búsqueda. Con 200 anuncios, 20 clientes concurrentes y 300 solicitudes, el percentil 95 fue **52,7 ms** en la red local de la misma máquina. La meta del informe es menos de 2 s.
- Android. Se generó y se instaló un APK de depuración en el emulador. Pasaron 23 pruebas unitarias. La inestabilidad del emulador impidió validar el recorrido completo en Android. En Chromium, el visor se comprobó a 400 × 700 y 830 × 430, con giro, arrastre, zoom, centrado y error de carga.
- Queda pendiente la prueba completa en dos configuraciones Android, la validación con cinco usuarios si están disponibles y la verificación de la entrega de correo en producción.

La implementación conserva Kotlin, Compose, Express, PostgreSQL y osmdroid. El panorama cubre la mejora visual del informe. Spring Boot, Google Maps, Room y las notificaciones push siguen siendo propuestas de arquitectura del documento académico.
