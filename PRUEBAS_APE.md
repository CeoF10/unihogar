# UniHogar Guaranda · Demostración y pruebas

## Preparar la demostración

1. Inicia PostgreSQL con los datos de demostración. En una base existente, aplica en orden las migraciones pendientes hasta `009_indice_fotos.sql`. Usa una base local de pruebas.
2. Configura e inicia la API. En `unihogar-api`, ejecuta `npm run demo:fotos`. Las imágenes ficticias se cargan en los anuncios #1, #5 y #7. La portada del anuncio #1 se obtuvo del mismo panorama que usa el visor.
3. Abre la app Android y conecta el servidor en la pantalla de acceso. Usa `http://10.0.2.2:3000/api/` desde el emulador para una API local en el puerto 3000. La versión de producción utiliza HTTPS.
4. Abre el panel con la misma API. Los enlaces de confirmación y recuperación se pueden revisar en la salida de la API local o recibir por correo al configurar Brevo.

## Guion

- **Cuentas (RF01–RF02).** Registra un estudiante con correo institucional y un propietario con correo personal. Confirma los correos, inicia sesión y prueba la recuperación. En **Perfil → Activar otro perfil**, confirma el segundo correo y vuelve a iniciar sesión con el perfil elegido.
- **Publicaciones (RF03–RF05).** Crea un borrador con ubicación manual, contacto, costos y fotos. Desde **Mis publicaciones**, edita los datos, gestiona las fotos y adjunta un JPEG 2:1 para el recorrido. Envía a auditoría. Tras aprobar desde el panel, comprueba que pausar o marcar como arrendado lo retire del catálogo.
- **Catálogo, mapa y ficha (RF06–RF09).** Combina tipo, precio, servicios y distancia. Cambia entre lista y mapa y carga otra página. Abre una ficha, comprueba el costo recurrente y el depósito, guarda el anuncio y vuelve a iniciar sesión para revisar Favoritos.
- **Comparador, contacto y visitas (RF10–RF12).** Compara hasta tres anuncios. Prueba compartir, copiar el contacto, abrir WhatsApp y consultar cómo llegar. Solicita una visita desde la ficha. Abre **Perfil → Visitas** con cada rol para proponer otra fecha, responder o cancelar.
- **Reportes (RF13).** Reporta un anuncio con motivo. En el panel, abre **Reportes de anuncios** y elige ocultar o descartar. Revisa quién resolvió el reporte y cuándo.
- **Panorama y modelo 3D (RF14).** En el anuncio ficticio #1, abre **Ver recorrido 360°** y prueba giro, zoom y centrado. El visor requiere un panorama equirectangular JPEG 2:1, de 512 a 4096 píxeles de ancho y de hasta 5 MB. Luego toca **Explorar habitación en 3D**. Compara el modelo con las fotos, pulsa **Entrar**, gira con un dedo y acerca con dos. Los anuncios #5 y #7 también traen modelo. Como propietario, abre **Mis publicaciones → Crear modelo 3D**, cambia las medidas y los muebles, revisa la vista previa y guarda.
- **Cuentas desde el panel.** En **Usuarios**, crea un estudiante con correo `@ueb.edu.ec`. Abre el enlace de activación, que en desarrollo aparece en la terminal de la API, elige la clave e inicia sesión en la app.

## Evidencia de la prueba local

- PostgreSQL 16. Se verificó la creación de una base con las migraciones, la actualización de una base de prueba y la restauración de un respaldo. Se compararon cantidad, tamaño y contenido de los archivos restaurados.
- API. Pasaron 38 comprobaciones de la suite del repositorio y 46 solicitudes adicionales con verificaciones de contenido y permisos para panoramas, visitas, reportes y el segundo perfil.
- Panel web. Se comprobó el ingreso, la recepción de un reporte y el registro de su resolución y autor.
- Búsqueda. Con 200 anuncios, 20 clientes concurrentes y 300 solicitudes, el percentil 95 fue **52,7 ms** en la red local de la misma máquina. La meta del informe es menos de 2 s.
- Android. Se generó y se instaló un APK de depuración en el emulador. Pasaron 23 pruebas unitarias. La inestabilidad del emulador impidió validar el recorrido completo en Android. En Chromium, el visor se comprobó a 400 × 700 y 830 × 430, con giro, arrastre, zoom, centrado y error de carga.
- Modelo 3D y cuentas del panel. En el emulador Android se abrió el modelo del anuncio #1 con vista general, vista interior, giro y zoom. Como propietario se creó, previsualizó, guardó y quitó un modelo desde **Mis publicaciones**. Pasaron 28 pruebas unitarias. La API rechazó medidas y muebles inválidos, anuncios ajenos y solicitudes sin sesión. En el panel se creó un estudiante. El enlace reenviado anuló el anterior y el nuevo activó la cuenta. El zoom con dos dedos se probó solo en código, porque el emulador no permite simular el gesto.
- El seguimiento del 21 de noviembre de 2026, descrito abajo, amplía la validación Android y prueba el correo de producción. Las cinco sesiones con participantes reales siguen pendientes.

La implementación conserva Kotlin, Compose, Express, PostgreSQL y osmdroid. El panorama y el modelo 3D ilustrativo cubren la mejora visual del informe. Spring Boot, Google Maps, Room y las notificaciones push siguen siendo propuestas de arquitectura del documento académico.

## Seguimiento del 21 de noviembre de 2026

Se usó una base PostgreSQL temporal e independiente (`unihogar_qa_20261121`) con todas las migraciones, datos ficticios y una API local en el puerto 3107. No se modificaron las cuentas ni los anuncios de la base habitual. La batería de RF01 a RF09 obtuvo **38 comprobaciones aprobadas y ninguna fallida**.

Al terminar se cerraron la API y las sesiones temporales del emulador, se eliminó la base de pruebas y se retiró la credencial temporal de producción. Los registros y las capturas se conservaron en el proyecto.

Se añadieron recorridos instrumentados en `IntegratedFlowTest.kt` y `NativePublishingTest.kt`, activados únicamente con `-e qa true` y una API de bucle local por `adb reverse`. Cubren autenticación, recuperación local, persistencia de favoritos tras cerrar sesión, mapa y filtros, comparación, copia de contacto, enlaces externos, respuesta de visitas, denegación de GPS, ubicación manual, selección de fotos del sistema y envío de una publicación a revisión. Los enlaces de WhatsApp, navegación y compartir se interceptan para comprobar su apertura sin enviar mensajes.

La publicación con foto se comprueba mediante los controles de accesibilidad del sistema, conservando el contexto normal de ejecución de Android. Se verifican el regreso a Mis anuncios, la persistencia de una fotografía y el estado EN_AUDITORIA devuelto por la API. No se considera un error de la app el fallo de mensajes observado en una versión anterior de la automatización que sustituía el contexto de corrutinas.

### Configuraciones y resultados Android

- Android 17 / API 37, 1080 × 2400, densidad 420: siete de los ocho casos de bienvenida, catálogo y recorridos integrados aprobaron. Falló la conservación de los servicios tras girar; se corrigió después (ver la incidencia más abajo). La publicación completa con el selector nativo quedó **sin certificar en este tamaño**: los intentos posteriores tuvieron problemas al marcar el mapa y la última sesión dejó de estar disponible por la conexión de pruebas.
- Android 17 / API 37, 720 × 1280, densidad 320: los mismos ocho casos dieron siete aprobados y el mismo fallo de servicios. En una ejecución separada, la publicación con selector nativo aprobó: fotografía persistente, estado EN_AUDITORIA y regreso al panel del propietario.

Son dos configuraciones de un emulador, no dos teléfonos físicos ni dos versiones de Android. Se probaron giros reales entre vertical y horizontal y la denegación del permiso de ubicación. La inestabilidad del emulador y los ajustes de la automatización no se contabilizan como fallos confirmados de la aplicación.

Se conservaron [los registros y las capturas](pruebas/2026-11-21/). El registro de diagnóstico del tamaño grande incluye una prueba de publicación antigua con un contexto de ejecución inadecuado; ese fallo se descartó y se sustituyó por la prueba nativa que aprobó en el tamaño pequeño. La entrega Android completa todavía no se considera aprobada.

### Incidencia confirmada al girar la pantalla

1. Iniciar sesión como propietario y abrir Publicar inmueble.
2. Completar datos y ubicación manual, seleccionar WiFi y continuar al paso 4.
3. Comprobar que el resumen incluye WiFi.
4. Girar el teléfono a horizontal.

Resultado observado: se mantiene el paso del asistente y los campos principales, pero desaparece WiFi del resumen. La prueba de conservación de servicios falla. En el formulario, la lista de servicios utiliza estado en memoria que no se restaura al recrear la actividad.

**Corrección (21 de noviembre de 2026).** Los servicios, los cargos y las fotos del asistente, y los servicios de la hoja de filtros, ahora se guardan con `rememberSaveable`. En Android 17 / API 37, 1080 × 2400, con una base temporal independiente:

- `ownerCanMarkLocationManuallyAndResumeWizard` aprobó. Con la aplicación reinstalada entre casos, los ocho casos de bienvenida, catálogo y recorridos integrados aprobaron.
- Se publicó un anuncio a mano con toques reales: ubicación marcada en el mapa, WiFi y Luz, un cargo por confirmar y una foto del selector del sistema. Se giró a horizontal en el paso 3 y a vertical en el paso 4. Se conservaron los servicios, el cargo y la foto. La API devolvió EN_AUDITORIA, una foto y ningún total definitivo por el cargo pendiente.
- `NativePublishingTest` sigue sin marcar el mapa en este tamaño con toques sintéticos. Un toque real sí lo marca, así que se considera una limitación de la automatización.

### Correo de producción

Con autorización del titular se registró una cuenta temporal de propietario usando un alias de su Gmail y una contraseña aleatoria exclusiva. La API publicada respondió:

- Salud: HTTP 200.
- Registro: HTTP 201, con mensaje de revisar el correo.
- Solicitud de recuperación: HTTP 200, con respuesta genérica.
- Inicio de sesión posterior: HTTP 403, porque el correo seguía sin confirmar.

Las búsquedas en la bandeja Gmail conectada, incluyendo spam y papelera, no encontraron mensajes de UniHogar durante la comprobación. Por tanto, **la entrega de confirmación y recuperación no está aprobada**. Las respuestas HTTP de registro y recuperación no prueban que el proveedor haya enviado el mensaje; debe revisarse la configuración y los registros del servicio de correo antes de repetir la prueba.

La cuenta temporal pendiente no pudo eliminarse mediante la app: la eliminación exige una sesión y el inicio de sesión exige confirmar el correo. Tras la autorización específica del titular, se eliminó desde Supabase únicamente el registro ID 8, restringiendo la operación también por el correo y el nombre exactos de la prueba. La respuesta de la base confirmó una sola fila eliminada. No se borraron otras cuentas.

**Corrección (21 de noviembre de 2026).** La causa era de configuración: el servicio de Render no tenía `BREVO_API_KEY` ni `MAIL_FROM`, así que la API solo escribía los enlaces en su registro. Se creó una clave de Brevo con permiso únicamente de correo transaccional, se configuró como remitente un Gmail verificado y se volvió a desplegar. Una solicitud de recuperación para una cuenta real del equipo llegó a la bandeja de entrada (no a spam) en pocos segundos. **La entrega de correo en producción queda aprobada.**

### Datos de demostración en producción

`unihogar-api/src/scripts/cargar-demo-produccion.js` cargó dos propietarios con expediente aprobado, una estudiante y cinco anuncios aprobados (#9 a #13). Cada anuncio tiene dos fotos, un panorama 360° de 2048 × 1024 y un modelo 3D. Las imágenes son ilustraciones generadas, guardadas en `unihogar-api/demo/produccion/`. Las descripciones indican que los anuncios son ficticios y los teléfonos de contacto no pertenecen a personas reales. Las cuentas iniciaron sesión con su rol en la API publicada, los cinco anuncios aparecieron en el catálogo y los panoramas se sirvieron correctamente. El anuncio #10 tiene un cargo por confirmar y no muestra total mensual.

### Usabilidad con participantes

Por indicación del usuario, se dejó preparado [el protocolo y la ficha para cinco participantes](PRUEBA_USABILIDAD.md). Todavía no se han realizado esas sesiones. No se atribuyen a personas los resultados de las pruebas automáticas.
