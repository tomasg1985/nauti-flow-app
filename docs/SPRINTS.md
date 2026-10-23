# NautiFlow — Plan de sprints (desarrollo en solitario)

**Reglas del juego**
- Desarrollo: **14 días** (Día 1 = sáb 17/10 → Día 14 = vie 30/10). **Pruebas: 7 días** (31/10 → 6/11).
- Un solo desarrollador, aprendiendo el stack: cada archivo viene **comentado línea por línea**.
- Orden: backend de cada sprint primero, después su pantalla en el frontend. Cada sprint termina con algo que **se puede usar y mostrar**.
- Si "2 semanas" era el total (7 días de desarrollo + 7 de pruebas), mirá el **plan comprimido** al final: recorta el alcance, no la calidad.

## Prioridades (MoSCoW)

| Prioridad | Qué entra |
|-----------|-----------|
| **MUST** (sin esto no hay MVP) | Login con roles · alta de lancha y datos · subida de documentos · verificación manual · buscador · viajes y asignación · link de reseña con token · promedio de reputación · retorno vacío + columna en dashboard · landing con reseñas |
| **SHOULD** | Fotos de la lancha · disponibilidad horaria · moderación de reseñas · bloqueo por intentos fallidos · mensaje de WhatsApp prellenado |
| **COULD** (solo si sobra tiempo) | Emails reales (Resend) · recuperar contraseña · vencimientos automáticos de documentos · SEO · auditoría completa |

**Simplificaciones para ir rápido (se cambian después sin reescribir nada):**
1. Archivos en **disco local** (`uploads/`) con una capa de almacenamiento intercambiable → R2 al desplegar.
2. Notificación al lanchero = **registro en la base + log**; el email real es COULD.
3. Migraciones en **SQL puro** (ya hecho): ves exactamente lo que pasa en la base.

---

## Sprint 0 — Fundaciones (Días 1–2 · 17–18/10)

| Día | Tareas | Estado |
|-----|--------|:------:|
| 1 | Proyecto Node + Express 5 · validación de `.env` · conexión Sequelize · migración 001 (users, lancheros, zones, boats) · modelos y relaciones · `/health`, `/ready`, `GET /api/v1/zones` · manejo de errores · tests (8 verdes) | ✅ |
| 2 | **Auth**: registro de lanchero, login, logout, `/auth/me` · hash con argon2 · sesión en cookie httpOnly (JWT) · bloqueo por 5 intentos fallidos · límite por IP · middleware `requireAuth` y `requireRole` · seed del usuario ADMIN · 18 tests nuevos (26 en total) | ✅ |

**Listo cuando:** un lanchero se registra, inicia sesión y `/auth/me` devuelve su rol; una ruta de admin devuelve 403 para un lanchero.

## Sprint 1 — Portal del lanchero (Días 3–5 · 19–21/10)

| Día | Tareas |
|-----|--------|
| 3 | Migración 003 (horarios) · API `/me/boats`: crear, ver, editar, zonas (muchos a muchos), horarios semanales, pausar/activar · slug automático · protección contra acceso a lanchas ajenas · 27 tests nuevos (53 en total) ✅ |
| 4 | Subida de archivos con multer: **fotos** (validar tipo real y tamaño) y **documentos** (Licencia, Seguro, Antecedentes) a almacenamiento privado. Migración 004 (photos + documents) |
| 5 | **Frontend arranca**: Vite + React + Router + Tailwind · pantallas Registro, Login y Portal (checklist de documentos y estado) |

**Listo cuando:** un lanchero se registra desde el navegador, carga lancha y 3 documentos, y ve "En revisión".

## Sprint 2 — Verificación y buscador (Días 6–8 · 22–24/10)

| Día | Tareas |
|-----|--------|
| 6 | API admin: cola de pendientes · URL temporal para ver un documento · aprobar/rechazar con motivo · verificar lancha (con precondiciones) |
| 7 | Motor de búsqueda (SQL): verificada + activa + capacidad + zona + horario + sin viaje superpuesto · enlace `wa.me` |
| 8 | Front admin: panel de verificación con visor y **pantalla del buscador** |

**Listo cuando:** el admin aprueba una lancha y la encuentra buscando "5 personas, Primera Sección, 14:00".

## Sprint 3 — Viajes y reputación (Días 9–11 · 25–27/10)

| Día | Tareas |
|-----|--------|
| 9 | Migración 004 (trips, reviews, review_tokens). CRUD de viajes y estados: solicitado → asignado → completado/cancelado. Asignar con detección de conflicto |
| 10 | Token único de reseña (hash SHA-256, vence a los 14 días, un solo uso) · endpoint público de reseña en **transacción** · recálculo del promedio · notificación al lanchero |
| 11 | Front: lista de viajes, botón "Generar link", **formulario de reseña móvil de un paso** (`/review/:token`) |

**Listo cuando:** un viaje completado genera link, se califica desde el celular y cambia el promedio; reusar el link da 404.

## Sprint 4 — Retornos vacíos, landing y cierre (Días 12–14 · 28–30/10)

| Día | Tareas |
|-----|--------|
| 12 | Migración 005 (empty_legs). Botón **"Publicar Retorno Vacío"** + mensaje para copiar + estados. Dashboard del admin con la **columna separada** |
| 13 | Landing pública (lanchas verificadas, promedio, reseñas) y perfil de lancha. Moderación de reseñas (ocultar) |
| 14 | Congelar funcionalidades · corregir bugs · datos de demo · **deploy a Render** · guía de uso para la agencia |

**Listo cuando:** el flujo completo de los 7 puntos corre en la URL de Render.

## Semana de pruebas (Días 15–21 · 31/10–6/11) — sin código nuevo

| Días | Foco |
|------|------|
| 15–16 | Pruebas de seguridad: acceso a datos ajenos, token reutilizado/vencido, archivos falsos, permisos por rol |
| 17–18 | Recorrido completo con la agencia y 3–5 lancheros reales; anotar fricciones |
| 19 | Pruebas en celulares (iOS Safari / Android Chrome), accesibilidad y velocidad |
| 20 | Corregir lo hallado + backups y prueba de restauración |
| 21 | Salida a producción y plan de soporte |

---

## Plan comprimido (si el total son 7 días de desarrollo)

Se mantiene solo lo **MUST**, sin fotos, sin disponibilidad horaria (el admin confirma el horario por WhatsApp), sin moderación y con UI mínima:

| Día | Entrega |
|-----|---------|
| 1 | Fundaciones ✅ |
| 2 | Auth + roles |
| 3 | Lancha + documentos (API) + registro/login/portal |
| 4 | Verificación + buscador (API + pantalla) |
| 5 | Viajes + token + reseña + reputación (API) + formulario móvil |
| 6 | Retorno vacío + dashboard + landing |
| 7 | Deploy + datos de demo |

## Cómo trabajamos cada día

1. Yo escribo el código del día con **un comentario por línea o bloque** explicando qué hace y por qué.
2. Vos lo corrés (`npm run dev`, `npm test`) y probás con `curl` o el navegador.
3. Si algo no se entiende, lo preguntás y lo explico con otro ejemplo antes de avanzar.
4. Al cierre del día: checklist del sprint y qué viene mañana.
