# UCO Kudos — Documentación

Proyecto Integrador Final de Ingeniería del Software II, Universidad Católica de Oriente. Trabajo individual.

Este repositorio reúne toda la documentación en Markdown, el manual de puesta a punto y las evidencias.

## Enlaces

| Elemento | Dirección |
| --- | --- |
| Aplicación publicada | <https://uco-kudos.vercel.app> |
| API publicada (salud) | <https://ucokudosbackend.onrender.com/actuator/health> |
| Repositorio Back-End | <https://github.com/UCO-Kudos/UCOKudosBackend> |
| Repositorio Front-End | <https://github.com/UCO-Kudos/UCOKudosFrontend> |

## Contenido

| Carpeta | Qué contiene |
| --- | --- |
| [`arquitectura/`](arquitectura/) | Arquitectura objetivo, diseño de alto nivel y detallado, seguridad, adjuntos y observabilidad, modelo de datos y API |
| [`adr/`](adr/) | Registros de decisiones de arquitectura (incluye el ADR-0012: API Gateway y WAF en el borde) |
| [`diagramas/`](diagramas/) | Diagramas C4, componentes, paquetes y secuencia |
| [`manual/`](manual/) | [Manual de puesta a punto](manual/manual-puesta-a-punto.md) |
| [`evidencias/`](evidencias/) | Capturas de CI, CD, Sonar, Trivy, publicación, despliegue, correo, WhatsApp, push, auditoría, observabilidad y pruebas |

## Resumen del sistema

UCO Kudos permite publicar un reconocimiento (Kudo) a otra persona de la universidad. Al publicarlo, el sistema:

1. Guarda el Kudo (con adjuntos opcionales).
2. Registra la auditoría.
3. Envía un correo electrónico al destinatario.
4. Envía una notificación de WhatsApp.
5. Envía un aviso en tiempo real si el destinatario está conectado.

Los tres canales son independientes: la falla de uno no deshace el Kudo ni afecta a los demás. Todo el proceso comparte un mismo `traceId`.

## Tecnología

| Capa | Tecnología |
| --- | --- |
| Backend | Java 25, Spring Boot 4.1.1, arquitectura hexagonal con reglas verificadas por ArchUnit, JDBC, Flyway, JWT |
| Frontend | React, Vite y TypeScript, aplicación web progresiva (PWA), Mobile First |
| Datos | Supabase (Postgres y almacenamiento privado) |
| Despliegue | Render (backend) y Vercel (frontend, API Gateway y WAF) |
| Calidad y seguridad | GitHub Actions, SonarQube Cloud, Trivy |

## Limitaciones conocidas

1. WhatsApp usa el sandbox gratuito de Twilio: solo entrega a teléfonos unidos al sandbox y con una plantilla de texto fijo.
2. El aviso en vivo solo llega con la aplicación abierta.
3. El API Gateway y el WAF son los del plan gratuito de Vercel; el detalle está en el ADR-0012.
