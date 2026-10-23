# 01 · PRD — Product Requirements Document

**Producto:** NautiFlow — Back-Office de Lanchas Verificadas
**Versión:** 1.0 (MVP) · **Fecha:** 16/10/2026 · **Estado:** Borrador para validación con la agencia

---

## 1. Resumen ejecutivo

La agencia recibe solicitudes de viaje (por WhatsApp, teléfono, redes) y hoy las resuelve de memoria o con planillas sueltas. No tiene una base propia de lanchas y conductores verificados, no puede filtrar rápido por capacidad / zona / horario, no mide la calidad del servicio y pierde dinero en los retornos vacíos ("fletes muertos") hacia el Delta.

NautiFlow es un **sistema de gestión interna (back-office)** más un **portal público de reputación** que permite:

1. Que los lancheros se registren y suban sus datos y documentación.
2. Que el administrador verifique manualmente esa documentación.
3. Que el administrador encuentre en segundos la lancha adecuada para cada solicitud y la contacte por WhatsApp.
4. Que cada viaje completado genere una reseña verificable del turista, que alimenta la reputación pública de la lancha.
5. Que los retornos vacíos se registren y se ofrezcan a la flota para capitalizarlos.

## 2. Objetivos y métricas de éxito

| Objetivo | Métrica | Meta a 90 días del lanzamiento |
|----------|---------|-------------------------------|
| Acelerar la asignación | Tiempo desde la solicitud hasta lancha contactada | ≤ 3 min (hoy: sin medir) |
| Construir base confiable | Lanchas con estado Verificado | ≥ 30 |
| Ecosistema de confianza | % de viajes completados con reseña | ≥ 40 % |
| Reputación visible | Lanchas con ≥ 3 reseñas en la landing | ≥ 10 |
| Optimizar fletes muertos | % de retornos vacíos publicados que se ocupan | ≥ 20 % |
| Calidad de datos | Documentos vencidos sin revisar | 0 |

## 3. Actores

| Actor | Descripción | Accede a |
|-------|-------------|----------|
| **Lanchero** | Dueño/conductor de una embarcación | Portal propio (alta, edición, estado de verificación, reseñas recibidas) |
| **Administrador** | Personal de la agencia | Back-office completo |
| **Turista** | Cliente final | Solo landing pública y formulario de reseña por token. **No tiene cuenta.** |
| **Sistema** | Procesos automáticos | Cálculo de reputación, emails, vencimiento de tokens y documentos |

## 4. Alcance del MVP

### 4.1 Incluido

- Landing pública con lanchas verificadas, promedio y reseñas.
- Registro y portal del lanchero (datos, fotos, capacidad, zonas, disponibilidad horaria, documentos).
- Panel de verificación: visor de documentos, aprobar/rechazar con motivo.
- Buscador interno de flota (capacidad, zona, fecha/hora, solo verificadas) con botón de WhatsApp.
- Gestión de viajes: crear solicitud, asignar lancha manualmente, registrar completado, cancelar.
- Reseñas por link único con token, calificación 1–5 y comentario.
- Reputación: promedio, contador, actualización de la landing, notificación al lanchero.
- Bolsa de tramos vacíos simplificada (registro + columna en dashboard + mensaje copiable).

### 4.2 Excluido explícitamente (fuera de MVP)

| Excluido | Razón |
|----------|-------|
| Integración con Renaper, Prefectura u otras APIs oficiales | La verificación es visual y humana |
| Cuenta o login del turista, reservas online, pagos | El turista no interactúa con el sistema |
| Matching automático / GPS / "lancha más cercana" | La asignación es humana |
| Subastas en tiempo real, push geolocalizado | Alcanza con registrar y ofrecer manualmente |
| App móvil nativa | Web responsive |
| Integración con WhatsApp Business API | Se usan enlaces `wa.me` |
| Facturación / liquidaciones | Etapa posterior |

## 5. Historias de usuario y criterios de aceptación

### Épica A — Portal de Lancheros

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| A1 | Como lanchero quiero registrarme con email y contraseña | Email único; contraseña ≥ 10 caracteres; queda en estado `PENDIENTE` sin acceso a la landing |
| A2 | Como lanchero quiero cargar los datos de mi embarcación | Nombre, matrícula, capacidad (1–60), descripción, año; hasta 8 fotos (JPG/PNG/WebP ≤ 8 MB c/u); al menos 1 zona de cobertura |
| A3 | Como lanchero quiero declarar mis zonas y horarios | Selección múltiple de zonas; franjas por día de semana (ej. Lun–Dom 08:00–20:00) |
| A4 | Como lanchero quiero subir mis documentos | Licencia, Seguro y Antecedentes Penales (PDF/JPG/PNG ≤ 10 MB); fecha de vencimiento para Licencia y Seguro; estado visible por documento |
| A5 | Como lanchero quiero ver por qué me rechazaron | Motivo del rechazo visible; puedo volver a subir solo el documento observado |
| A6 | Como lanchero quiero pausar mi disponibilidad | Interruptor "Activo/Pausado"; una lancha pausada no aparece en búsquedas |
| A7 | Como lanchero quiero ver mis reseñas y promedio | Listado con estrellas, comentario y fecha; recibo email por cada reseña nueva |

### Épica B — Verificación (Admin)

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| B1 | Como admin quiero ver una cola de lancheros pendientes | Ordenada por antigüedad; contador de documentos pendientes |
| B2 | Como admin quiero revisar cada documento visualmente | Visor con zoom/rotación; PDF embebido; URL firmada con vencimiento ≤ 5 min |
| B3 | Como admin quiero aprobar o rechazar cada documento | Rechazo exige motivo; queda registro (quién, cuándo, qué decidió) |
| B4 | Como admin quiero marcar la lancha como Verificada | Solo permitido si Licencia, Antecedentes (lanchero) y Seguro (lancha) están aprobados y vigentes |
| B5 | Como admin quiero rechazar o suspender | Motivo obligatorio; la lancha sale de búsquedas y de la landing |
| B6 | Como admin quiero ver alertas de vencimiento | Lista de documentos que vencen en ≤ 30 días; al vencer, la lancha pasa a `SUSPENDIDA` |

### Épica C — Motor de búsqueda (Admin)

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| C1 | Como admin quiero filtrar por pasajeros, zona y fecha/hora | Resultado: lanchas `VERIFICADA` + activas + capacidad ≥ pasajeros + cubren la zona + horario dentro de su disponibilidad + sin viaje superpuesto |
| C2 | Como admin quiero ver reputación y contexto en el resultado | Foto, capacidad, promedio y cantidad de reseñas, viajes completados |
| C3 | Como admin quiero contactar por WhatsApp con un clic | Botón abre `wa.me/<tel>?text=<mensaje con zona, hora y pasajeros>` |
| C4 | Como admin quiero asignar el viaje desde el resultado | Un clic crea/actualiza el viaje en estado `ASIGNADO` con la lancha elegida |

### Épica D — Gestión de viajes (Admin)

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| D1 | Cargar solicitud de viaje | Nombre y teléfono del turista, pasajeros, zona, fecha/hora, duración estimada, notas |
| D2 | Asignar lancha manualmente | El sistema advierte si hay conflicto horario, pero el admin puede forzar con nota |
| D3 | Registrar viaje completado | Cambia a `COMPLETADO`; habilita generar el link de reseña |
| D4 | Cancelar viaje | Motivo; libera la lancha |

### Épica E — Reseñas y reputación

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| E1 | Como admin quiero generar el link de reseña | Solo para viajes `COMPLETADO`; genera `/review/trip_<token>`; muestra botón "Enviar por WhatsApp" con mensaje prellenado y botón "Copiar link" |
| E2 | Como turista quiero calificar en un solo paso desde el celular | Sin login; 5 estrellas táctiles + comentario opcional (máx. 600 caracteres) + nombre opcional; envío en un clic; pantalla de gracias |
| E3 | Como agencia quiero que el formulario no se pueda abusar | Token de un solo uso, vence a los 14 días, ligado a un viaje real; un viaje = una reseña; rate limit por IP |
| E4 | Como sistema debo actualizar la reputación | Al guardar: recalcula promedio y cantidad en la misma transacción; la landing refleja el cambio |
| E5 | Como lanchero quiero ser notificado | Email con calificación y comentario; no incluye datos de contacto del turista |
| E6 | Como admin quiero moderar | Puedo ocultar una reseña con motivo (insultos, datos personales); el promedio excluye reseñas ocultas |

### Épica F — Tramos vacíos

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| F1 | Como admin quiero publicar un retorno vacío desde un viaje | Botón "Publicar Retorno Vacío" en el viaje; formulario prellenado (lancha, zona de origen, hora, destino opcional) |
| F2 | Como admin quiero un mensaje para copiar | Texto: "Lancha X vuelve vacía desde la Zona Y a las 15:00hs. ¿Alguien necesita un rescate o envío en ese trayecto?"; botón "Copiar" y alerta interna |
| F3 | Como admin quiero ver los tramos en una columna separada | Dashboard con columna "Retornos vacíos" ordenada por hora; estados Abierto / Ofrecido / Ocupado / Vencido |
| F4 | Como admin quiero cerrar el tramo | Marcar como Ocupado (opcionalmente vinculando un viaje) o Cancelado; los vencidos se marcan solos |

### Épica G — Landing pública

| ID | Historia | Criterios de aceptación |
|----|----------|-------------------------|
| G1 | Como turista quiero ver lanchas verificadas con reputación | Listado con foto, capacidad, zonas, promedio, cantidad de reseñas, insignia "Verificada" |
| G2 | Como turista quiero ver el perfil y las reseñas de una lancha | Galería, descripción, reseñas visibles paginadas; **sin teléfono, documentos ni datos personales** |
| G3 | Como agencia quiero un llamado a contacto | Botón de WhatsApp de la agencia (no del lanchero) |

## 6. Reglas de negocio

| ID | Regla |
|----|-------|
| RN-01 | Una lancha solo aparece en búsquedas y landing si está `VERIFICADA`, activa y con documentos vigentes |
| RN-02 | La verificación la decide siempre una persona; ningún proceso automático aprueba |
| RN-03 | Los documentos son privados: solo el admin y el propio lanchero pueden verlos |
| RN-04 | Un token de reseña pertenece a un único viaje `COMPLETADO` y se consume al enviar la reseña |
| RN-05 | Solo hay una reseña por viaje |
| RN-06 | El promedio se calcula con reseñas visibles; se muestra con 1 decimal y la cantidad |
| RN-07 | Una lancha con una reseña promedio por debajo de 3,0 tras ≥ 5 reseñas dispara una alerta interna al admin (no baja automática) |
| RN-08 | Un retorno vacío vence a su hora de salida + 2 h si no se cerró |
| RN-09 | El lanchero nunca ve datos de contacto del turista |
| RN-10 | Toda decisión de verificación y toda moderación queda en un registro de auditoría |

## 7. Requisitos no funcionales

| Categoría | Requisito |
|-----------|-----------|
| Rendimiento | Búsqueda de flota < 500 ms con 500 lanchas; landing LCP < 2,5 s en 4G |
| Seguridad | HTTPS, cookies httpOnly, hash argon2, URLs firmadas, rate limiting, validación estricta de uploads |
| Privacidad | Cumplimiento de la Ley 25.326 (Protección de Datos Personales): consentimiento al subir documentos, finalidad declarada, derecho de supresión; antecedentes penales son dato sensible y se conservan solo mientras sean necesarios |
| Disponibilidad | 99 % mensual (Render); backups diarios de la base |
| Usabilidad | Formulario del turista usable con una mano en un celular de gama baja; admin optimizado para escritorio y tablet |
| Accesibilidad | WCAG 2.1 AA: contraste, foco visible, etiquetas, objetivos táctiles ≥ 44 px |
| Idioma | Español (es-AR); zona horaria `America/Argentina/Buenos_Aires` |
| Observabilidad | Logs estructurados, alertas ante errores 5xx |

## 8. Supuestos

- El volumen inicial es de decenas de lancheros y cientos de viajes por mes.
- Todos los lancheros tienen WhatsApp.
- La agencia opera un único grupo de WhatsApp para la flota.
- El catálogo de zonas es chico y administrado por el admin (ej. Primera Sección, Segunda Sección, Tercera Sección, Canal Arias, Río Luján, Carapachay, Tigre Centro, San Fernando).

## 9. Riesgos

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| Documentos falsificados | Alto | Revisión humana con checklist, fecha de vencimiento, auditoría; fuera de alcance validar con organismos |
| Fuga de documentos sensibles | Crítico | Bucket privado, URLs firmadas cortas, accesos auditados, minimizar copias |
| Baja respuesta de turistas a la reseña | Medio | Formulario de un paso, envío por WhatsApp con mensaje cálido, recordatorio manual a las 48 h |
| Reseñas abusivas o falsas | Medio | Token atado a viaje real, moderación, un solo uso |
| Lancheros no cargan disponibilidad | Medio | Disponibilidad por defecto 24/7 editable; el admin igual confirma por WhatsApp |
| Admin único = cuello de botella | Medio | Roles preparados para sumar más administradores |

## 10. Preguntas abiertas

1. ¿Qué zonas exactas y qué nombres usa la agencia? (se sembrarán en el seed)
2. ¿Una misma persona puede tener varias lanchas? (el modelo lo soporta; la UI del MVP prioriza una)
3. ¿Cuántos días de vigencia mínima se exige a Licencia y Seguro para aprobar?
4. ¿Se muestra el nombre del lanchero en la landing o solo el de la embarcación?
5. ¿El email de notificación al lanchero es suficiente o prefieren además un mensaje por WhatsApp manual?
