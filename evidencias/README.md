# Evidencias

Capturas de las evidencias obligatorias del proyecto. Cada archivo debe tener exactamente el nombre indicado.

| Evidencia | Archivo | Qué debe verse |
| --- | --- | --- |
| CI | `ci-backend.png`, `ci-frontend.png` | Workflow con el job `ci` en verde |
| CD | `cd-backend.png`, `cd-frontend.png` | Workflow con el job `cd` en verde |
| Sonar | `sonar-proyectos.png` | Los dos proyectos analizados en SonarQube Cloud |
| Trivy | `trivy.png` | Paso de Trivy en verde, sin hallazgos CRITICAL |
| Pruebas | `pruebas-backend.png`, `pruebas-frontend.png` | `mvn -B verify` en verde y `npm test` en verde |
| Publicación | `publicacion.png` | Formulario Publicar Kudo y la lista Mis Kudos con el Kudo nuevo |
| Despliegue | `despliegue-render.png`, `despliegue-vercel.png` | Servicios en estado Live y la URL pública |
| Correo | `correo.png` | Correo recibido por el destinatario |
| WhatsApp | `whatsapp.png`, `whatsapp-twilio-logs.png` | Mensaje recibido en el teléfono y el registro Delivered en Twilio |
| Push | `push.png` | Aviso en tiempo real en la sesión del destinatario |
| Auditoría | `auditoria.png` | Eventos `KUDO_PUBLICADO` y `NOTIFICACION_*` consultados por `traceId` |
| Observabilidad | `observabilidad.png` | `/actuator/health` en `UP` y logs con `traceId` |
| Seguridad en el borde | `waf-reglas.png`, `gateway-401.png`, `auditoria-bloqueada.png` | Reglas del firewall, respuesta 401 sin sesión y bloqueo de `/api/auditoria` |

## Nota sobre WhatsApp

El canal usa el sandbox gratuito de Twilio. Las capturas muestran el mensaje entregado a un teléfono unido al sandbox con la plantilla de texto fijo. Las limitaciones están descritas en el [manual de puesta a punto](../manual/manual-puesta-a-punto.md).
