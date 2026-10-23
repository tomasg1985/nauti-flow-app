# 02 · TRD — Technical Requirements Document

**Producto:** NautiFlow · **Versión:** 1.0 (MVP) · **Fecha:** 16/10/2026

---

## 1. Stack tecnológico

| Capa | Tecnología | Versión | Justificación |
|------|-----------|---------|---------------|
| Runtime | Node.js | 22 LTS | LTS vigente, ES Modules nativos |
| Framework API | Express | 5.x | Manejo nativo de errores async, ecosistema maduro |
| ORM | Sequelize | 6.x | Migraciones, transacciones, modelos declarativos |
| Base de datos | PostgreSQL | 16 | Integridad relacional, índices parciales, `tstzrange`, JSONB |
| Validación | Zod | 3.x | Validación de body/query/params con tipos |
| Auth | jsonwebtoken + argon2 | — | JWT en cookie httpOnly + SameSite=Lax; roles |
| Uploads | multer (memoria) + sharp + file-type | — | Valida mime real, redimensiona, elimina EXIF |
| Storage | Cloudflare R2 (S3-compatible) vía `@aws-sdk/client-s3` | — | Bucket privado + URLs firmadas; costo bajo, sin egress |
| Email | Resend (o Brevo) | — | Transaccional, plantillas simples |
| Seguridad HTTP | helmet, cors, express-rate-limit, hpp | — | Cabeceras seguras y límites |
| Logging | pino + pino-http | — | Logs JSON estructurados |
| Jobs | node-cron (en proceso) | — | Vencimientos de tokens, documentos y tramos; suficiente para MVP |
| Frontend | React 18 + Vite + React Router 6 | — | SPA única para landing, portal y admin |
| Estado servidor | TanStack Query | 5.x | Cache, reintentos, invalidación |
| Formularios | React Hook Form + Zod resolver | — | Validación compartida |
| Estilos | Tailwind CSS 3 + componentes propios (shadcn/ui opcional) | — | Velocidad y consistencia |
| Tests | Vitest + Supertest + Playwright (smoke) | — | Unit, integración y e2e mínimo |
| CI/CD | GitHub Actions → Render | — | Deploy automático desde `main` |

> **Nota sobre SEO de la landing:** al ser un SPA, las páginas públicas pueden no indexarse óptimamente. Para el MVP se mitiga con `react-helmet-async` y un `sitemap.xml` generado por el backend. Si el tráfico orgánico se vuelve relevante, migrar solo las rutas públicas a Next.js o agregar prerender (Vite SSG) sin tocar la API.

## 2. Arquitectura

```
                      ┌───────────────────────────────┐
 Turista ───────────► │  Static Site (React + Vite)   │
 Lanchero ──────────► │  /  /lanchas  /portal  /admin │
 Admin ─────────────► │  /review/:token               │
                      └──────────────┬────────────────┘
                                     │ HTTPS · JSON · cookie httpOnly
                      ┌──────────────▼────────────────┐
                      │  API Express 5 (Render Web)   │
                      │  routes → controllers →       │
                      │  services → models            │
                      │  middlewares: auth, rbac,     │
                      │  zod, rate-limit, errors      │
                      │  node-cron (jobs)             │
                      └───────┬────────────┬──────────┘
                              │            │
                ┌─────────────▼───┐   ┌────▼────────────────┐
                │ PostgreSQL 16   │   │ R2 / S3 (privado)   │
                │ (Render)        │   │ fotos + documentos  │
                └─────────────────┘   └─────────────────────┘
                              │
                      ┌───────▼────────┐
                      │ Resend (email) │
                      └────────────────┘
```

**Estilo:** monolito modular. Un solo servicio API, módulos por dominio. No se justifican microservicios para este volumen.

### 2.1 Módulos de dominio

| Módulo | Responsabilidad |
|--------|-----------------|
| `auth` | Registro, login, logout, recuperación de contraseña, roles |
| `lancheros` | Perfil del conductor |
| `boats` | Embarcación, fotos, zonas, disponibilidad |
| `documents` | Subida, estados, revisión, URLs firmadas |
| `verification` | Decisión de verificación, suspensión, auditoría |
| `search` | Motor de búsqueda de flota |
| `trips` | Solicitudes, asignación, estados |
| `reviews` | Tokens, formulario público, moderación |
| `reputation` | Recalcular promedio, alertas |
| `emptyLegs` | Tramos vacíos y mensaje |
| `public` | Endpoints de landing (solo lectura, sin datos sensibles) |
| `notifications` | Emails y alertas internas |
| `audit` | Registro de acciones críticas |

## 3. Estructura del repositorio

```
nautiflow/
├── apps/
│   ├── api/
│   │   ├── src/
│   │   │   ├── config/          # env.js, db.js, storage.js
│   │   │   ├── models/          # sequelize models + index.js (asociaciones)
│   │   │   ├── migrations/
│   │   │   ├── seeders/         # zonas, admin inicial
│   │   │   ├── modules/
│   │   │   │   ├── auth/ {routes,controller,service,schema}.js
│   │   │   │   ├── boats/ documents/ search/ trips/ reviews/ ...
│   │   │   ├── middlewares/     # auth, rbac, validate, upload, errorHandler
│   │   │   ├── jobs/            # expireTokens, expireDocuments, expireEmptyLegs
│   │   │   ├── utils/           # token, phone, whatsapp, slug
│   │   │   └── app.js / server.js
│   │   └── tests/
│   └── web/
│       └── src/
│           ├── pages/ {public,portal,admin,review}
│           ├── components/ hooks/ lib/api.js
│           └── main.jsx
├── .github/workflows/ci.yml
└── README.md
```

Se usa `"type": "module"` en ambos paquetes (ES Modules).

## 4. API REST

Base: `/api/v1` · JSON · errores con formato `{ error: { code, message, details? } }`.

### 4.1 Auth

| Método | Ruta | Rol | Descripción |
|--------|------|-----|-------------|
| POST | `/auth/register` | público | Alta de lanchero (rate limit 5/h/IP) |
| POST | `/auth/login` | público | Setea cookie de sesión |
| POST | `/auth/logout` | autenticado | Limpia cookie |
| GET | `/auth/me` | autenticado | Usuario y rol |
| POST | `/auth/forgot` · `/auth/reset` | público | Recuperación con token de un solo uso |

### 4.2 Portal del lanchero (`LANCHERO`)

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET/PUT | `/me/profile` | Datos del conductor |
| GET/POST | `/me/boats` | Listar / crear embarcación |
| GET/PUT | `/me/boats/:id` | Detalle / editar |
| PATCH | `/me/boats/:id/active` | Activar / pausar |
| POST | `/me/boats/:id/photos` | Subir foto (multipart) |
| DELETE | `/me/boats/:id/photos/:photoId` | Borrar foto |
| PUT | `/me/boats/:id/zones` | Reemplaza zonas `{ zoneIds: [] }` |
| PUT | `/me/boats/:id/availability` | Reemplaza franjas semanales |
| POST | `/me/documents` | Subir documento `{ type, boatId?, expiresAt? }` + archivo |
| GET | `/me/documents` | Estado por documento |
| GET | `/me/reviews` | Reseñas recibidas |

### 4.3 Admin (`ADMIN`)

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/admin/verification/queue` | Cola de pendientes |
| GET | `/admin/documents/:id/url` | URL firmada (TTL 300 s); registra acceso en auditoría |
| PATCH | `/admin/documents/:id` | `{ status: APPROVED\|REJECTED, reason? }` |
| POST | `/admin/boats/:id/verify` | Verifica (valida precondiciones) |
| POST | `/admin/boats/:id/reject` | `{ reason }` |
| POST | `/admin/boats/:id/suspend` | `{ reason }` |
| GET | `/admin/search/boats` | Buscador (ver §6) |
| GET/POST | `/admin/trips` | Listar / crear solicitud |
| GET/PATCH | `/admin/trips/:id` | Detalle / editar |
| POST | `/admin/trips/:id/assign` | `{ boatId, force?, note? }` |
| POST | `/admin/trips/:id/complete` | Marca completado |
| POST | `/admin/trips/:id/cancel` | `{ reason }` |
| POST | `/admin/trips/:id/review-link` | Genera token y devuelve `{ url, whatsappUrl }` |
| GET | `/admin/reviews` | Listar con filtros |
| PATCH | `/admin/reviews/:id/visibility` | `{ visible, reason }` |
| POST | `/admin/trips/:id/empty-leg` | Publica retorno vacío (devuelve mensaje) |
| GET | `/admin/empty-legs` | Listar por estado |
| PATCH | `/admin/empty-legs/:id` | Cambiar estado / vincular viaje |
| GET | `/admin/dashboard` | Columnas: solicitudes, asignados, completados, retornos vacíos |
| GET/POST/PUT | `/admin/zones` | Catálogo de zonas |

### 4.4 Público (sin sesión)

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/public/boats` | Lanchas verificadas; filtros `zone`, `minCapacity`, `sort=rating` ; paginado |
| GET | `/public/boats/:slug` | Perfil + fotos + reputación |
| GET | `/public/boats/:slug/reviews` | Reseñas visibles paginadas |
| GET | `/public/reviews/:token` | Valida token; devuelve nombre de lancha y fecha (nada más) |
| POST | `/public/reviews/:token` | Envía reseña `{ rating, comment?, reviewerName? }` |

## 5. Seguridad

### 5.1 Autenticación y autorización

- Contraseñas con **argon2id**. Mínimo 10 caracteres.
- JWT firmado (HS256, 12 h) en cookie `httpOnly; Secure; SameSite=Lax`. Protección CSRF mediante `SameSite` + cabecera personalizada `X-Requested-With` exigida en métodos mutantes.
- RBAC: middleware `requireRole('ADMIN')`. Cada recurso del lanchero se filtra por `lanchero_id` del usuario (previene IDOR).
- Bloqueo temporal tras 5 intentos fallidos de login.

### 5.2 Uploads

1. `multer` en memoria con límites (fotos 8 MB, documentos 10 MB, máx. 8 fotos por lancha).
2. Verificar tipo real con `file-type` (magic bytes), no la extensión ni el `Content-Type` del cliente.
3. Imágenes: `sharp` → rotación por EXIF, **quitar metadatos**, convertir a WebP, generar miniatura.
4. Documentos: aceptar PDF/JPG/PNG; nombre de objeto aleatorio (`uuid`), nunca el original.
5. Fotos de lancha: prefijo público `boats/…` (servidas por CDN o URL pública del bucket de fotos).
6. **Documentos: bucket separado, privado.** Acceso solo vía URL firmada (TTL 5 min) emitida a ADMIN o al dueño. Cada emisión a un admin se registra en `audit_logs`.
7. (Recomendado) Cifrado en reposo del bucket y política de retención/supresión.

### 5.3 Formulario de reseña (anti-spam)

- Token: `trip_` + 32 bytes aleatorios en base64url (`crypto.randomBytes(32)`). Entropía de 256 bits: no adivinable.
- En la base se guarda **solo el hash SHA-256** del token; el valor en claro se muestra una sola vez al admin al generarlo (y puede regenerarse invalidando el anterior).
- Reglas: viaje en estado `COMPLETADO`; `expires_at` = +14 días; `used_at` nulo; `UNIQUE(trip_id)` en reseñas.
- El envío usa transacción: `SELECT … FOR UPDATE` del token → crear reseña → marcar token usado → recalcular reputación.
- Rate limit: 10 req/min/IP para `/public/reviews/*`; respuesta uniforme `404` ante tokens inválidos/vencidos/usados (sin filtrar información).
- Honeypot oculto + longitud máxima del comentario; sin captcha en el MVP (el token ya acota el abuso).
- Cabecera `Referrer-Policy: no-referrer` y `X-Robots-Tag: noindex` en `/review/*`.

### 5.4 Otros

- `helmet` con CSP restrictiva, CORS limitado al dominio del front, `hpp`.
- Sanitizar comentarios (texto plano; React escapa al renderizar).
- Rate limit global 300 req/15 min/IP; más estricto en login y registro.
- Secretos únicamente en variables de entorno de Render.
- Dependencias auditadas con `npm audit` en CI y Dependabot.

### 5.5 Privacidad (Ley 25.326)

- Aviso y consentimiento explícito al subir documentos.
- Datos de turistas (nombre, teléfono) solo accesibles por el admin; no se exponen al lanchero ni en endpoints públicos.
- Procedimiento de baja/supresión: eliminar documentos del bucket y anonimizar el perfil, conservando reseñas anonimizadas.
- Registrar la base de datos ante la autoridad de aplicación si corresponde (consultar con asesoría legal de la agencia).

## 6. Motor de búsqueda

Entrada: `passengers`, `zoneId`, `startsAt` (ISO), `durationMin` (default 120).

Criterios en SQL (parametrizado):

```sql
SELECT b.id, b.name, b.capacity, b.rating_avg, b.rating_count, l.phone,
       (SELECT COUNT(*) FROM trips t WHERE t.boat_id = b.id AND t.status = 'COMPLETED') AS completed_trips
FROM boats b
JOIN lancheros l       ON l.id = b.lanchero_id
JOIN boat_zones bz     ON bz.boat_id = b.id AND bz.zone_id = :zoneId
WHERE b.verification_status = 'VERIFIED'
  AND b.is_active = TRUE
  AND b.capacity >= :passengers
  -- el horario cae dentro de una franja semanal declarada
  AND EXISTS (
        SELECT 1 FROM boat_availability a
        WHERE a.boat_id = b.id
          AND a.day_of_week = EXTRACT(DOW FROM (:startsAt AT TIME ZONE 'America/Argentina/Buenos_Aires'))
          AND (:startsAt AT TIME ZONE 'America/Argentina/Buenos_Aires')::time >= a.start_time
          AND ((:startsAt AT TIME ZONE 'America/Argentina/Buenos_Aires') + make_interval(mins => :durationMin))::time <= a.end_time
  )
  -- sin viaje asignado que se superponga
  AND NOT EXISTS (
        SELECT 1 FROM trips t
        WHERE t.boat_id = b.id
          AND t.status = 'ASSIGNED'
          AND tstzrange(t.starts_at, t.starts_at + make_interval(mins => t.duration_min))
              && tstzrange(:startsAt, :startsAt + make_interval(mins => :durationMin))
  )
  -- documentos vigentes
  AND NOT EXISTS (
        SELECT 1 FROM documents d
        WHERE d.status = 'APPROVED' AND d.expires_at < (:startsAt)::date
          AND (d.boat_id = b.id OR d.lanchero_id = b.lanchero_id)
  )
ORDER BY b.rating_avg DESC NULLS LAST, b.rating_count DESC
LIMIT 50;
```

Opciones de orden: reputación (default), menor capacidad que cumpla (ajuste), más viajes completados. El WhatsApp se arma con `utils/whatsapp.js`:

```js
export const waLink = (phoneE164, text) =>
  `https://wa.me/${phoneE164.replace(/\D/g, '')}?text=${encodeURIComponent(text)}`;
```

## 7. Reputación

```js
// reputation.service.js (dentro de la misma transacción que crea la reseña)
const { avg, count } = await Review.findOne({
  attributes: [
    [fn('ROUND', fn('AVG', col('rating')), 2), 'avg'],
    [fn('COUNT', col('id')), 'count'],
  ],
  where: { boatId, isVisible: true },
  raw: true, transaction: t,
});
await Boat.update({ ratingAvg: avg, ratingCount: count }, { where: { id: boatId }, transaction: t });
```

- Se desnormaliza `rating_avg` y `rating_count` en `boats` para que la landing y el buscador sean rápidos.
- Tras `commit`, se encola la notificación por email al lanchero (no bloquea la respuesta al turista).
- Si `count ≥ 5` y `avg < 3.0`, se crea una alerta interna para el admin.

## 8. Tramos vacíos

- Entidad `empty_legs` ligada a un `trip` y una `boat`.
- Mensaje generado en el backend con plantilla única para evitar inconsistencias:

```js
export const emptyLegMessage = ({ boatName, zoneName, time }) =>
  `🚤 ${boatName} vuelve vacía desde ${zoneName} a las ${time}hs. ` +
  `¿Alguien necesita un rescate o envío en ese trayecto?`;
```

- La respuesta del endpoint incluye `message` para botón "Copiar" y `whatsappGroupUrl` si se configura `FLEET_WHATSAPP_GROUP_URL`.
- Job cada 10 min: `OPEN/OFFERED` con `available_from + 2h < now()` → `EXPIRED`.
- Sin colas, sin websockets: el dashboard usa polling (TanStack Query, `refetchInterval: 30000`).

## 9. Jobs programados (node-cron)

| Job | Frecuencia | Acción |
|-----|-----------|--------|
| `expireReviewTokens` | Cada hora | Marca tokens vencidos |
| `expireDocuments` | Diario 03:00 | Documentos vencidos → `EXPIRED`; lancha → `SUSPENDED`; email al lanchero |
| `docExpiryReminders` | Diario 09:00 | Aviso a 30 y 7 días del vencimiento |
| `expireEmptyLegs` | Cada 10 min | Cierra tramos vencidos |

> En Render con una sola instancia es suficiente. Si se escala a más de una, mover los jobs a un Cron Job de Render o usar un lock advisory de Postgres (`pg_try_advisory_lock`).

## 10. Despliegue e infraestructura

| Componente | Servicio | Notas |
|------------|----------|-------|
| API | Render Web Service (Node 22) | Health check `/health`; `npm run migrate` en pre-deploy |
| Front | Render Static Site | Rewrite `/* → /index.html` |
| Base de datos | Render PostgreSQL | Backups diarios, plan con retención |
| Storage | Cloudflare R2 | 2 buckets: `nautiflow-photos` (público/CDN) y `nautiflow-docs` (privado) |
| Email | Resend | Dominio verificado (SPF/DKIM) |
| Dominio | Cloudflare DNS | HTTPS forzado |

### Variables de entorno

```
NODE_ENV=production
PORT=10000
DATABASE_URL=postgres://...
JWT_SECRET=...
COOKIE_DOMAIN=.tudominio.com
WEB_ORIGIN=https://app.tudominio.com
R2_ACCOUNT_ID= R2_ACCESS_KEY_ID= R2_SECRET_ACCESS_KEY=
R2_BUCKET_PHOTOS= R2_BUCKET_DOCS= R2_PUBLIC_PHOTOS_URL=
RESEND_API_KEY= MAIL_FROM="NautiFlow <no-reply@tudominio.com>"
AGENCY_WHATSAPP=54911XXXXXXXX
FLEET_WHATSAPP_GROUP_URL=https://chat.whatsapp.com/...
REVIEW_TOKEN_TTL_DAYS=14
DOC_URL_TTL_SECONDS=300
```

## 11. Calidad y pruebas

| Nivel | Herramienta | Cobertura mínima |
|-------|-------------|------------------|
| Unit | Vitest | Servicios de reputación, token, disponibilidad, mensajes WhatsApp |
| Integración | Supertest + Postgres de test (Docker) | Auth/RBAC, flujo de verificación, búsqueda, reseña con token, tramo vacío |
| E2E | Playwright | Alta lanchero → verificación → búsqueda → viaje → reseña → landing |
| Seguridad | Casos negativos | IDOR, token reutilizado/vencido, upload de archivo con extensión falsa |

Casos críticos que deben tener test:

1. Un token usado dos veces devuelve 404 y no crea una segunda reseña.
2. Dos envíos concurrentes con el mismo token crean una sola reseña.
3. Una lancha sin Seguro vigente no puede quedar `VERIFIED`.
4. Un lanchero no puede leer documentos ni lanchas de otro.
5. La búsqueda excluye lanchas con viaje superpuesto.
6. El promedio ignora reseñas ocultas.

## 12. Observabilidad

- `pino-http` con `requestId`; nunca loguear contraseñas, tokens ni URLs firmadas.
- Sentry (free tier) para errores en API y front.
- `/health` (liveness) y `/ready` (verifica DB).
- Métricas de negocio consultables por SQL: tiempos de verificación, tasa de respuesta de reseñas, tramos ocupados.

## 13. Decisiones de arquitectura (ADR resumidas)

| # | Decisión | Alternativa descartada | Motivo |
|---|----------|------------------------|--------|
| 1 | Monolito modular | Microservicios | Equipo chico, volumen bajo |
| 2 | Sequelize | Prisma / Knex | Continuidad con el stack del equipo |
| 3 | Cookie httpOnly | JWT en localStorage | Mitiga XSS |
| 4 | Hash del token de reseña | Token en claro | Una filtración de DB no habilita enviar reseñas |
| 5 | URLs firmadas para docs | Servir por la API | Menos carga y control de TTL |
| 6 | Polling 30 s | WebSockets | Suficiente para el MVP |
| 7 | Enlaces `wa.me` | WhatsApp Business API | Sin costo ni aprobación de plantillas |
| 8 | Rating desnormalizado | Calcular en cada lectura | Landing y búsqueda rápidas |
