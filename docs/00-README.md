# NautiFlow — Back-Office de Lanchas Verificadas: paquete de documentación técnica

Versión 1.0 · Octubre 2026 · Alcance: MVP

## Contenido

| # | Documento | Para qué sirve |
|---|-----------|----------------|
| 01 | [PRD](01-PRD.md) | Qué se construye y por qué: objetivos, actores, historias, reglas de negocio, alcance |
| 02 | [TRD](02-TRD.md) | Cómo se construye: stack, arquitectura, API, seguridad, despliegue |
| 03 | [UX/UI](03-UX-UI.md) | Mapa de pantallas, wireframes, sistema de diseño, estados |
| 04 | [AppFlow](04-AppFlow.md) | Flujos de usuario y máquinas de estado (Mermaid) |
| 05 | [Backend Schema](05-Backend-Schema.md) | ERD, DDL PostgreSQL, modelos Sequelize, consultas clave |
| 06 | [Plan de implementación](06-Plan-Implementacion.md) | Fases, sprints, tareas, definición de terminado, riesgos |

## Stack recomendado (resumen)

| Capa | Elección | Motivo |
|------|----------|--------|
| Runtime / API | Node.js 22 LTS + Express 5 (ES Modules) | Coincide con el stack actual del equipo |
| ORM / DB | Sequelize 6 + PostgreSQL 16 | Relacional, filtros por capacidad/zona/horario, integridad |
| Frontend | React 18 + Vite + React Router + Tailwind CSS | Un solo SPA para landing, portal lanchero y admin |
| Validación | Zod (API) + React Hook Form | Esquemas compartidos y mensajes claros |
| Auth | Sesión por JWT en cookie httpOnly + argon2 | Sin tokens en localStorage; roles ADMIN / LANCHERO |
| Archivos | Cloudflare R2 o AWS S3 (privado) + URLs firmadas | Documentos sensibles nunca públicos |
| Imágenes | sharp (redimensionado y EXIF strip) | Fotos livianas, sin metadatos de ubicación |
| Email | Resend o Brevo | Notificar al lanchero (reseña, aprobación/rechazo) |
| WhatsApp | Enlaces `wa.me` con mensaje prellenado | Sin API de pago, cumple el MVP |
| Deploy | Render (Web Service + Static Site + Postgres) | Ya es la plataforma del equipo |
| Calidad | Vitest + Supertest, ESLint, Prettier, GitHub Actions | CI mínimo y confiable |

## Decisiones clave

1. **Humano en el medio**: no hay validación automática de documentos ni matching por GPS.
2. **Token opaco para reseñas**: `/review/trip_<token>`, un solo uso, vencimiento y hash en base de datos.
3. **Documentos privados**: bucket privado + URL firmada de corta duración solo para el admin.
4. **Tramos vacíos como registro, no como subasta**: tabla propia + columna en el dashboard + mensaje para copiar.
5. **Reputación derivada**: promedio recalculado en transacción al guardar la reseña.
