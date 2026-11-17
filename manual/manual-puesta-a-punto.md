# Manual de puesta a punto — UCO Kudos

Guía para reconstruir y desplegar el sistema desde cero. No contiene valores secretos: cada secreto se guarda en la plataforma indicada y se copia en el momento de crearlo.

## 1. Piezas y direcciones

| Pieza | Plataforma | Dirección |
| --- | --- | --- |
| Frontend (PWA) y puerta de entrada | Vercel Hobby | `https://uco-kudos.vercel.app` |
| Backend (API) | Render, servicio Docker | `https://ucokudosbackend.onrender.com` |
| Base de datos y adjuntos | Supabase (Postgres y Storage privado) | Panel de Supabase |
| Correo | Brevo | API por `RestClient` |
| WhatsApp | Twilio (sandbox) | API por `RestClient` |
| Repositorios | GitHub, organización `UCO-Kudos` | `UCOKudosBackend` y `UCOKudosFrontend` |
| Calidad | SonarCloud, organización `uco-kudos` | Un proyecto por repositorio |

Flujo de una petición: navegador → Vercel (reglas de firewall y reescritura `/api/*`) → Render (backend) → Supabase.

## 2. Requisitos locales

1. Java 25 y Maven 3.9 o superior para el backend.
2. Node.js (LTS) y npm para el frontend.
3. Git.

## 3. Base de datos (Supabase)

1. Crear el proyecto en Supabase y anotar la cadena de conexión JDBC (usuario, clave y host).
2. Crear el bucket privado de adjuntos y anotar su nombre.
3. Las tablas y los datos iniciales los crea Flyway al arrancar el backend (`V1__esquema_base.sql` y `V2__semilla.sql`). No se ejecutan a mano.
4. Usuarios de la semilla: `admin@uco.net.co`, `autor@uco.net.co` y el destinatario definido por `SEED_DEST_CORREO`. La clave inicial de la semilla se cambia después de la entrega.

## 4. Backend en Render

1. Render → New → Web Service → conectar `UCOKudosBackend`, entorno Docker, plan Free.
2. Render → Environment: crear las variables de la tabla siguiente. Los valores se copian desde cada proveedor.

| Variable | Qué es |
| --- | --- |
| `DB_URL`, `DB_USER`, `DB_PASSWORD` | Conexión JDBC a Supabase |
| `JWT_SECRET` | Texto largo y aleatorio para firmar los JWT |
| `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`, `SUPABASE_BUCKET` | Almacenamiento de adjuntos |
| `BREVO_API_KEY`, `MAIL_FROM` | Correo; `MAIL_FROM` es el remitente verificado en Brevo |
| `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN` | Credenciales de Twilio |
| `TWILIO_WHATSAPP_FROM` | Remitente del sandbox, formato `whatsapp:+17372508034` |
| `TWILIO_CONTENT_SID` | Plantilla del sandbox (empieza por `HX`); ver sección 7 |
| `CORS_ALLOWED_ORIGIN` | `https://uco-kudos.vercel.app` |
| `DOMINIO_INSTITUCIONAL` | `uco.net.co` |
| `SEED_DEST_CORREO`, `SEED_DEST_CELULAR` | Destinatario de la semilla (solo se usan al crear la base) |

3. Guardar. Render construye y despliega. Verificar `https://ucokudosbackend.onrender.com/actuator/health`: debe responder `UP`.
4. Render → Settings → Deploy Hook: copiar la URL para el pipeline (sección 6).

El plan gratis duerme el servicio tras un rato sin tráfico; la primera petición tarda en responder.

## 5. Frontend en Vercel

1. Vercel → Add New → Project → importar `UCOKudosFrontend` (framework Vite).
2. Variable de entorno `VITE_API_URL` = `/api`.
3. El archivo `vercel.json` del repositorio reenvía `/api/*` al backend de Render y el resto a `index.html`. Esa reescritura hace de API Gateway.
4. Vercel → Settings → Git → Deploy Hooks: crear un hook `ci-cd` en la rama `main` para el pipeline.

## 6. Pipelines (GitHub Actions)

El flujo `CI-CD` ejecuta pruebas, SonarQube Cloud, Trivy y, si todo pasa, despliega.

Backend (`UCOKudosBackend`), en Settings → Secrets and variables → Actions:

| Tipo | Nombre | Valor |
| --- | --- | --- |
| Secreto | `SONAR_TOKEN` | Token generado en SonarCloud |
| Secreto | `RENDER_DEPLOY_HOOK_URL` | Deploy Hook de Render |
| Variable | `SONAR_ORG` | `uco-kudos` |
| Variable | `SONAR_PROJECT_KEY` | Clave del proyecto Sonar del backend |
| Variable | `API_URL` | `https://ucokudosbackend.onrender.com` |

Frontend (`UCOKudosFrontend`):

| Tipo | Nombre | Valor |
| --- | --- | --- |
| Secreto | `SONAR_TOKEN` | Token de SonarCloud |
| Secreto | `VERCEL_DEPLOY_HOOK_URL` | Deploy Hook de Vercel |
| Variable | `SONAR_ORG` | `uco-kudos` |
| Variable | `SONAR_PROJECT_KEY` | Clave del proyecto Sonar del frontend |
| Variable | `APP_URL` | `https://uco-kudos.vercel.app` |

Notas:

1. SonarCloud → Administration → Analysis Method: apagar Automatic Analysis en ambos proyectos (el análisis lo hace el pipeline).
2. El token de SonarCloud se muestra una sola vez: copiarlo al crear el secreto.
3. Trivy bloquea el pipeline si encuentra vulnerabilidades CRITICAL. El backend fija `tomcat.version` en `pom.xml` para corregir las de Tomcat.
4. El job `cd` hace una prueba de humo contra la salud del backend.

## 7. WhatsApp (Twilio, plan de prueba)

1. En la consola de Twilio, abrir Home → Try it out → Send a WhatsApp message.
2. Cada teléfono que deba recibir mensajes envía `join twilio-trial` al número del remitente (`+1 737 250 8034`). La unión dura 72 horas.
3. El `ContentSid` se toma de esa misma ventana, pestaña API, línea `ContentSid=HX…`.
4. En la cuenta de prueba solo se entregan plantillas, no mensajes libres (error 21654). La plantilla disponible tiene texto fijo, por lo que el mensaje no incluye el contenido del Kudo.
5. Un destinatario cuyo WhatsApp use identificador de usuario en lugar de número puede no recibir mensajes por la API en cuenta de prueba (error 0).

Para producción haría falta un remitente de WhatsApp Business aprobado por Meta, plantillas propias con variables y una cuenta de pago. El cambio queda en configuración, porque el envío está detrás de `WhatsAppSenderPort`.

## 8. Agregar una persona destinataria

No hay registro en la aplicación; las personas se agregan por SQL en Supabase → SQL Editor.

1. Datos necesarios: nombre, correo `@uco.net.co`, celular en formato `+57…`, rol y programa.
2. La clave se copia del hash de `autor@uco.net.co`.
3. Reglas de roles: una persona tiene un solo rol activo; `RECTOR`, `COORD_ACADEMICO` y `COORD_FACULTAD` son únicos por facultad; `COORD_PROGRAMA`, único por programa.
4. Para recibir WhatsApp, esa persona debe haber enviado el `join` del sandbox.
5. Cambiar solo el celular: `UPDATE persona SET celular = '+57…' WHERE correo = '…';`

## 9. Seguridad en el borde (Vercel Firewall)

Reglas publicadas (plan Hobby, máximo tres reglas personalizadas):

1. Límite de intentos de login: ruta `/api/auth/login`, 10 peticiones por 60 segundos por IP, respuesta 429.
2. Bloqueo de rutas sensibles: ruta que empieza por `/api/auditoria`, acción Deny.

Los cambios se publican con Review Changes → Publish. Limitaciones documentadas en `docs/adr/adr-0012.md`: no es un gateway completo, Render sigue accesible directamente y las reglas OWASP y Bot Protection son de pago.

## 10. Verificación final

1. `https://ucokudosbackend.onrender.com/actuator/health` responde `UP`.
2. Iniciar sesión en `https://uco-kudos.vercel.app` con `autor@uco.net.co`.
3. Publicar un Kudo para el destinatario.
4. Comprobar: Kudo en Mis Kudos, correo recibido, WhatsApp recibido (si el teléfono hizo `join`), aviso en vivo en la sesión del destinatario.
5. Con `admin@uco.net.co`, consultar la auditoría por `traceId`: `KUDO_PUBLICADO` y los eventos de notificación (`NOTIFICACION_CORREO_ENVIADA`, `NOTIFICACION_WHATSAPP_ENVIADA`, `NOTIFICACION_PUSH_ENVIADA`).

## 11. Problemas frecuentes

| Síntoma | Causa y solución |
| --- | --- |
| `cd` falla con `curl: blank argument` | Falta el secreto `RENDER_DEPLOY_HOOK_URL` o la variable `API_URL` |
| Sonar falla con parámetros vacíos | Faltan las variables `SONAR_ORG` y `SONAR_PROJECT_KEY` |
| Trivy bloquea con CRITICAL | Actualizar la dependencia afectada (por ejemplo, `tomcat.version`) |
| Login rechazado | Clave con mayúscula inicial y asterisco final; correo debe terminar en `@uco.net.co` |
| WhatsApp no llega | El teléfono no envió `join` o pasaron más de 72 horas; revisar `NOTIFICACION_WHATSAPP_FALLIDA` en auditoría |
| Primera petición lenta | El servicio gratis de Render estaba dormido |
