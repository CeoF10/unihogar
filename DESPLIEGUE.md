# Publicar UniHogar (gratis) y subirlo a Google Play

Todo el código ya está preparado. Faltan las cuentas, que solo puedes crear tú. El único pago es la cuenta de desarrollador de Google Play (25 USD, una sola vez).

| Pieza | Servicio gratis | Límite que importa |
|---|---|---|
| Base de datos | [Neon](https://neon.tech) | 0,5 GB. Las fotos y documentos se guardan ahí: alcanza para unos cientos de fotos |
| API | [Render](https://render.com) (Web Service, plan Free) | Se duerme tras 15 min sin uso; la primera petición tarda ~50 s. La app ya espera hasta 75 s |
| Panel admin | Render (Static Site) | Sin límites relevantes |
| Correos | [Brevo](https://www.brevo.com) | 300 correos al día |
| Política de privacidad y eliminación de cuenta | La propia API: `/privacidad` y `/eliminar-cuenta` | — |

## 1. Base de datos en Neon

1. Crea una cuenta en Neon y un proyecto (región **US East** o la más cercana a Render Oregon).
2. Copia la *connection string* (termina en `?sslmode=require`).
3. En tu PC, crea el esquema **sin datos de demostración**:

```bash
cd unihogar-database
DATABASE_URL='postgres://...neon.tech/neondb?sslmode=require' bash produccion/crear-base.sh
```

4. Crea la cuenta del administrador de Bienestar UEB:

```bash
cd ../unihogar-api
DATABASE_URL='postgres://...neon.tech/neondb?sslmode=require' node src/scripts/crear-admin.js correo@ueb.edu.ec 'UNA_CLAVE_LARGA'
```

## 2. Correos con Brevo

1. Crea la cuenta en Brevo.
2. **Senders & IP → Senders → Add a sender**: agrega el correo que enviará los mensajes (puede ser tu Gmail) y confírmalo.
3. **SMTP & API → API Keys → Generate a new API key**. Guárdala: se pega en Render.

## 3. API y panel en Render

1. Crea la cuenta en Render con tu GitHub y dale acceso a los repos `unihogar-api` y `unihogar-admin-web`.
2. **New → Blueprint** → elige el repo `unihogar-api`. Render lee `render.yaml` y crea:
   - `unihogar-api` (la API)
   - `unihogar-admin` (el panel)
3. Te pedirá los valores secretos:

| Variable | Valor |
|---|---|
| `DATABASE_URL` | La connection string de Neon |
| `PUBLIC_API_URL` | `https://unihogar-api.onrender.com/api` (usa la URL real que te asigne Render) |
| `BREVO_API_KEY` | La clave de Brevo |
| `MAIL_FROM` | El remitente verificado en Brevo |
| `CONTACTO_EMAIL` | Correo de contacto que verá la gente en la política de privacidad |

`JWT_SECRET` se genera solo. `ALMACENAMIENTO=db` y `TRUST_PROXY=1` ya vienen puestos.

4. Comprueba:
   - `https://unihogar-api.onrender.com/api/health` → `{"ok":true}`
   - `https://unihogar-api.onrender.com/privacidad`
   - `https://unihogar-api.onrender.com/eliminar-cuenta`
   - El panel en `https://unihogar-admin.onrender.com` → inicia sesión con el admin del paso 1.

**Si Render te da otra URL** (por ejemplo `unihogar-api-x1y2.onrender.com`), cámbiala en:
- `PUBLIC_API_URL` en Render.
- `unihogar-admin-web/index.html` → `API_PRODUCCION`.
- La app: en `unihogar-guaranda/local.properties` agrega `API_BASE_URL_RELEASE=https://TU-URL/api/` y vuelve a generar el `.aab`.

## 4. Probar antes de subir

Instala la versión de producción en tu teléfono y prueba: registro (te llega el correo), confirmar, iniciar sesión, publicar un anuncio con fotos, subir el expediente, aprobarlo desde el panel y eliminar una cuenta.

```bash
cd unihogar-guaranda
./gradlew :app:assembleRelease
adb install app/build/outputs/apk/release/app-release.apk
```

## 5. Google Play

### Firma (muy importante)

La clave de subida está en `~/unihogar-firma/` (`upload.jks` y `keystore.properties` con la contraseña). **Haz una copia de esa carpeta en un lugar seguro** (USB o Drive privado). Si la pierdes, no podrás publicar actualizaciones sin pedir un cambio de clave a Google. Nunca la subas a GitHub. En Play Console activa **Firma de apps de Google Play** (viene activada por defecto).

### Generar el paquete

```bash
cd unihogar-guaranda
./gradlew :app:bundleRelease
# → app/build/outputs/bundle/release/app-release.aab
```

Para cada actualización sube `versionCode` en `app/build.gradle.kts` (1 → 2 → 3…).

### Ficha y formularios

- **Nombre del paquete:** `io.github.ceof10.unihogar` (no se puede cambiar después de publicar).
- **Política de privacidad:** `https://unihogar-api.onrender.com/privacidad`
- **Eliminación de cuenta (URL):** `https://unihogar-api.onrender.com/eliminar-cuenta`
- **Gráficos:** en `~/unihogar-play/` están el ícono de 512×512, el gráfico destacado de 1024×500 y capturas.
- **Acceso a la app (para revisores):** crea en producción una cuenta de estudiante y otra de propietario de prueba y dalas en *Contenido de la app → Acceso a la app*.
- **Anuncios:** No contiene anuncios.

**Seguridad de los datos** (respuestas según lo que hace la app):

| Dato | Se recopila | Se comparte | Para qué | ¿Obligatorio? |
|---|---|---|---|---|
| Nombre | Sí | No | Funcionalidad de la app, gestión de cuenta | Sí |
| Correo electrónico | Sí | No | Gestión de cuenta | Sí |
| Teléfono | Sí | No* | Funcionalidad de la app | Opcional (estudiantes) |
| Ubicación precisa | Sí | No | Funcionalidad (ubicación del inmueble) | Opcional (solo propietarios) |
| Fotos | Sí | No | Funcionalidad (fotos del anuncio) | Opcional |
| Archivos y documentos | Sí | No | Verificación de identidad del propietario | Opcional (solo propietarios) |
| Otros datos personales (cédula) | Sí | No | Verificación del propietario | Opcional (solo propietarios) |

\* Mostrar el teléfono del anuncio a otros usuarios es parte de la función de la app; Google no lo cuenta como "compartir" con terceros.

- Los datos **se cifran en tránsito** (HTTPS): Sí.
- **Los usuarios pueden pedir que se borren:** Sí (en la app y en la URL de eliminación).

### Cuentas personales nuevas

Si tu cuenta de desarrollador es **personal** y la creaste después de noviembre de 2023, Google exige una **prueba cerrada con al menos 12 testers durante 14 días seguidos** antes de publicar en producción. Crea la prueba cerrada, invita a 12+ compañeros (sus correos de Gmail) y pídeles que instalen la app y la mantengan instalada esos 14 días.
