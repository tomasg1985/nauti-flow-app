# 03 · UX/UI Design

**Producto:** NautiFlow · **Versión:** 1.0 (MVP) · **Fecha:** 16/10/2026

---

## 1. Principios de diseño

1. **Velocidad del admin primero.** Cada tarea frecuente (buscar, contactar, asignar) debe resolverse en pocos clics y sin cambiar de pantalla.
2. **Confianza visible.** La insignia "Verificada" y las estrellas son los elementos más prominentes de la landing.
3. **Fricción cero para el turista.** Un solo paso, mobile-first, sin cuentas.
4. **Estados siempre claros.** Cada documento, lancha, viaje y tramo muestra su estado con color + texto + ícono (nunca solo color).
5. **Revisión humana asistida.** El visor de documentos prioriza legibilidad: pantalla grande, zoom, rotación y checklist.
6. **Sobrio y marítimo.** Paleta azul profundo + verde agua; mucho aire; fotografía real de lanchas.

## 2. Mapa del sitio

```
PÚBLICO
├── /                       Landing (hero, lanchas destacadas, cómo funciona, reseñas)
├── /lanchas                Listado filtrable (zona, capacidad, orden por reputación)
├── /lanchas/:slug          Perfil público de la lancha + reseñas
├── /review/:token          Formulario de reseña (un paso, mobile)
├── /ingresar               Login
└── /registro-lancheros     Alta de lanchero

PORTAL LANCHERO  (/portal)
├── /portal                 Resumen: estado de verificación + pendientes
├── /portal/perfil          Datos del conductor
├── /portal/lancha          Datos, fotos, capacidad, zonas, horarios
├── /portal/documentos      Licencia · Seguro · Antecedentes
└── /portal/resenas         Promedio y reseñas recibidas

ADMIN  (/admin)
├── /admin                  Dashboard (kanban de viajes + columna Retornos Vacíos)
├── /admin/buscar           Motor de búsqueda de flota
├── /admin/viajes           Lista y detalle de viajes
├── /admin/verificacion     Cola + visor de documentos
├── /admin/flota            Todas las lanchas y estados
├── /admin/resenas          Moderación
├── /admin/retornos         Bolsa de tramos vacíos
└── /admin/configuracion    Zonas, usuarios admin
```

## 3. Sistema de diseño

### 3.1 Color

| Token | Hex | Uso |
|-------|-----|-----|
| `--navy-900` | `#0B2545` | Texto principal, encabezados, sidebar admin |
| `--navy-700` | `#13315C` | Botón primario, enlaces |
| `--teal-500` | `#1E9E94` | Acento, insignia Verificada, estados positivos |
| `--teal-100` | `#D9F2EF` | Fondos suaves |
| `--sand-50` | `#F7F5F0` | Fondo de páginas |
| `--amber-500` | `#E9A21A` | Estrellas, pendientes |
| `--red-600` | `#C62F2F` | Rechazado, errores |
| `--gray-600` | `#5B6770` | Texto secundario |
| `--white` | `#FFFFFF` | Tarjetas |

Contraste mínimo 4.5:1 en texto. Estados con ícono y etiqueta además del color.

### 3.2 Tipografía

- **Títulos:** Plus Jakarta Sans 600/700.
- **Cuerpo / UI:** Inter 400/500.
- Escala: 12 · 14 · 16 · 20 · 24 · 32 · 44. Base 16 px (15 px en tablas densas del admin).

### 3.3 Espaciado y forma

- Grilla de 4 px (4, 8, 12, 16, 24, 32, 48).
- Radio: 8 px en controles, 12 px en tarjetas.
- Sombra suave en tarjetas; foco con anillo de 2 px `teal-500`.
- Objetivo táctil mínimo 44 × 44 px en móvil.

### 3.4 Componentes base

| Componente | Variantes / notas |
|-----------|-------------------|
| Botón | Primario, secundario, fantasma, destructivo, **WhatsApp** (verde `#25D366` con ícono) |
| Badge de estado | `Pendiente` (ámbar) · `Verificada` (teal) · `Rechazada` (rojo) · `Suspendida` (gris) · `Vencido` (rojo outline) |
| Rating | Estrellas de solo lectura (1 decimal) y estrellas interactivas (táctiles, grandes) |
| Tarjeta de lancha | Foto 4:3, nombre, capacidad, zonas, ★ promedio (n), badge |
| Tabla | Cabecera fija, filas clicables, paginación, vacíos con mensaje |
| Visor de documento | Pantalla dividida: imagen/PDF + panel de decisión |
| Toast | Confirmaciones y errores no bloqueantes |
| Empty state | Ilustración simple + acción sugerida |
| Skeleton | En listas y tarjetas mientras carga |

## 4. Pantallas clave (wireframes)

### 4.1 Landing pública `/`

```
┌──────────────────────────────────────────────────────────────┐
│ ⚓ NautiFlow          Lanchas   Cómo funciona   [WhatsApp]    │
├──────────────────────────────────────────────────────────────┤
│  Navegá el Delta con lanchas verificadas y reseñas reales    │
│  [ Hablar con la agencia por WhatsApp ]   [ Ver lanchas ]    │
│  ✔ Documentos revisados  ✔ Reseñas de viajes reales          │
├──────────────────────────────────────────────────────────────┤
│  Lanchas mejor calificadas                                    │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐                  │
│  │ foto   │ │ foto   │ │ foto   │ │ foto   │                  │
│  │Delfín  │ │Aurora  │ │Nautilus│ │Cormorán│                  │
│  │★4.9(23)│ │★4.8(17)│ │★4.8(12)│ │★4.7(9) │                  │
│  │Hasta 8 │ │Hasta 12│ │Hasta 6 │ │Hasta 10│                  │
│  │✔Verif. │ │✔Verif. │ │✔Verif. │ │✔Verif. │                  │
│  └────────┘ └────────┘ └────────┘ └────────┘                  │
├──────────────────────────────────────────────────────────────┤
│  Cómo funciona:  1 Nos contás tu viaje · 2 Elegimos la lancha │
│                  verificada · 3 Navegás y nos contás cómo fue │
├──────────────────────────────────────────────────────────────┤
│  Lo que dicen los turistas  (carrusel de reseñas recientes)   │
├──────────────────────────────────────────────────────────────┤
│  ¿Sos lanchero? Sumá tu embarcación →   Footer · Privacidad   │
└──────────────────────────────────────────────────────────────┘
```

**Notas:** el CTA principal lleva al WhatsApp de la agencia (la reserva la opera el admin). Nunca se muestran teléfonos de lancheros.

### 4.2 Perfil público `/lanchas/:slug`

```
┌──────────────────────────────────────────────────────────────┐
│ ← Volver                                                      │
│ [ Galería: foto grande + miniaturas ]                         │
│ Delfín del Delta   ✔ Verificada desde 03/2026                 │
│ ★ 4.9  (23 reseñas)   Capacidad: 8 · Zonas: Primera Sección…  │
│ [ Consultar disponibilidad por WhatsApp ]                     │
│ ─ Sobre la embarcación ──────────────────────────────────────│
│ ─ Distribución de estrellas ─  5★ ████████ 20                 │
│                                4★ ██ 3                        │
│ ─ Reseñas ───────────────────────────────────────────────────│
│  ★★★★★ Martina · 12/09/2026                                   │
│  "Muy amable y puntual…"                                      │
│  [ Ver más ]                                                  │
└──────────────────────────────────────────────────────────────┘
```

### 4.3 Formulario de reseña `/review/:token` (móvil, un solo paso)

```
┌─────────────────────────┐
│ ⚓ NautiFlow              │
│                         │
│  ¿Cómo estuvo tu viaje  │
│  con Delfín del Delta?  │
│  Viaje del 12/09/2026   │
│                         │
│   ☆  ☆  ☆  ☆  ☆        │   ← estrellas grandes (56 px), tocar para elegir
│   "Excelente"           │   ← etiqueta dinámica según valor
│                         │
│  Contanos más (opcional)│
│ ┌─────────────────────┐ │
│ │                     │ │
│ └─────────────────────┘ │
│  0/600                  │
│  Tu nombre (opcional)   │
│ ┌─────────────────────┐ │
│ │                     │ │
│ └─────────────────────┘ │
│                         │
│ [   Enviar reseña   ]   │   ← deshabilitado hasta elegir estrellas
│  Tu reseña será pública │
│  junto a la lancha.     │
└─────────────────────────┘
```

**Estados:** cargando (skeleton) → formulario → enviando → **gracias** ("¡Gracias! Tu opinión ayuda a otros viajeros") · **enlace no válido/usado/vencido** (mensaje neutro: "Este enlace ya no está disponible", con botón al sitio).
**Detalles:** teclado numérico no requerido; el campo de texto no se abre automáticamente; botón fijo en la parte inferior; funciona con una mano; carga < 50 KB gracias a la ruta liviana.

### 4.4 Portal del lanchero — Resumen `/portal`

```
┌──────────────────────────────────────────────────────────────┐
│ Hola, Carlos                                  [Pausar lancha] │
│ Estado: ⏳ En revisión                                         │
│ ──────────────────────────────────────────────────────────── │
│ Checklist para ser verificado                                 │
│  ✔ Datos de la lancha                                         │
│  ✔ Fotos (5)                                                  │
│  ✔ Zonas y horarios                                           │
│  ⏳ Licencia      — en revisión                               │
│  ✖ Seguro        — Rechazado: "Foto borrosa"  [Volver a subir]│
│  ⬜ Antecedentes  — falta subir               [Subir]         │
│ ──────────────────────────────────────────────────────────── │
│ Reputación: ★ — (aún sin reseñas)                             │
└──────────────────────────────────────────────────────────────┘
```

Registro y edición usan un asistente por pasos (Datos → Fotos → Zonas y horarios → Documentos) con guardado automático de borrador.

### 4.5 Panel de verificación `/admin/verificacion`

```
┌───────────────┬──────────────────────────────────────────────┐
│ COLA (12)     │  Carlos Pérez · "Delfín del Delta"            │
│ ───────────── │  Licencia  Seguro  Antecedentes   (pestañas)  │
│ ● Carlos P.   │ ┌──────────────────────────┬─────────────────┐│
│   3 docs pend.│ │                          │ Checklist       ││
│ ○ Marta G.    │ │   VISOR (imagen / PDF)   │ ☐ Nombre coincide││
│   1 doc pend. │ │   [+][-][↻][⤢]           │ ☐ Vigente        ││
│ ○ Luis R.     │ │                          │ ☐ Legible        ││
│   2 docs pend.│ │                          │ Vence: 12/2027  ││
│               │ │                          │ [Aprobar]        ││
│               │ │                          │ [Rechazar…]      ││
│               │ └──────────────────────────┴─────────────────┘│
│               │ Estado de lancha: 2/3 docs aprobados          │
│               │ [ Verificar lancha ] (deshabilitado hasta 3/3)│
│               │ [ Rechazar lancha… ]                          │
└───────────────┴──────────────────────────────────────────────┘
```

**Detalles:** atajos de teclado (A aprobar, R rechazar, ←/→ cambiar documento); al rechazar se abre un modal con motivos frecuentes (ilegible, vencido, no coincide el nombre, otro) + texto libre; el botón "Verificar lancha" solo se habilita con los tres documentos aprobados y vigentes.

### 4.6 Buscador de flota `/admin/buscar`

```
┌──────────────────────────────────────────────────────────────┐
│ Pasajeros [ 5 ]  Zona [ Primera Sección ▾ ]  Fecha [16/10]    │
│ Hora [ 14:00 ]  Duración [ 2 h ▾ ]            [ Buscar ]      │
├──────────────────────────────────────────────────────────────┤
│ 4 lanchas disponibles                    Ordenar: Reputación ▾│
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ [foto] Delfín del Delta   ★4.9 (23)  · Cap. 8 · 41 viajes │ │
│ │        Zonas: 1ª Sección, Canal Arias                      │ │
│ │        [ 💬 WhatsApp ]  [ Asignar a viaje ]                │ │
│ └──────────────────────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │ [foto] Aurora             ★4.8 (17)  · Cap. 12 · 30 viajes│ │
│ │        [ 💬 WhatsApp ]  [ Asignar a viaje ]                │ │
│ └──────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

**Detalles:** los filtros se recuerdan; el botón WhatsApp abre un mensaje prellenado ("Hola Carlos, ¿podés hacer un viaje de 5 pax en Primera Sección hoy 14:00 hs por 2 h?"); "Asignar a viaje" abre un panel lateral con el viaje o crea uno nuevo; si hay conflicto de horario se advierte y se pide nota para forzar; resultados vacíos sugieren ampliar zona u horario.

### 4.7 Dashboard del admin `/admin` (columna de retornos vacíos)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│ Hoy 16/10 · [ + Nueva solicitud ]                                             │
├──────────────┬──────────────┬──────────────┬──────────────┬───────────────────┤
│ SOLICITADOS  │ ASIGNADOS    │ EN CURSO / OK │ COMPLETADOS  │ 🔄 RETORNOS VACÍOS │
│ (3)          │ (5)          │ (2)           │ (8)          │ (2)               │
│ ───────────  │ ───────────  │ ───────────   │ ───────────  │ ───────────────── │
│ 14:00 · 5pax │ 11:00 Delfín │ 10:00 Aurora  │ 09:00 Nautil.│ Delfín · 15:00    │
│ 1ª Sección   │ → 2ª Sección │               │ [Link reseña]│ Desde 2ª Sección  │
│ [Buscar]     │ [Completar]  │               │ [Link reseña]│ [Copiar mensaje]  │
│              │ [Retorno     │               │              │ [Ocupado][Cancelar]│
│              │  vacío]      │               │              │ ───────────────── │
│              │              │               │              │ Aurora · 17:30    │
│              │              │               │              │ Desde Tigre       │
└──────────────┴──────────────┴──────────────┴──────────────┴───────────────────┘
```

**Detalles:** columna "Retornos vacíos" separada visualmente (fondo `teal-100`, ícono ↺); tarjetas con cuenta regresiva a la hora de salida; estados Abierto / Ofrecido / Ocupado; actualiza cada 30 s; las tarjetas de viaje `ASIGNADO` tienen el botón rápido **"Publicar Retorno Vacío"**.

### 4.8 Modal "Publicar Retorno Vacío"

```
┌──────────────────────────────────────────────┐
│ Publicar retorno vacío                    ✕  │
│ Lancha:  Delfín del Delta (del viaje)        │
│ Sale desde: [ 2ª Sección ▾ ]                 │
│ Hora de salida: [ 15:00 ]                    │
│ Hacia (opcional): [ Tigre Centro ▾ ]         │
│ Nota: [______________________________]       │
│                                              │
│ Vista previa del mensaje:                    │
│ ┌──────────────────────────────────────────┐ │
│ │ 🚤 Delfín del Delta vuelve vacía desde la │ │
│ │ 2ª Sección a las 15:00hs. ¿Alguien       │ │
│ │ necesita un rescate o envío en ese       │ │
│ │ trayecto?                                │ │
│ └──────────────────────────────────────────┘ │
│ [ Cancelar ]  [ Publicar y copiar mensaje ]  │
└──────────────────────────────────────────────┘
```

Al confirmar: se crea el registro, aparece en la columna del dashboard, el mensaje queda copiado al portapapeles y se muestra un toast "Mensaje copiado — pegalo en el grupo de la flota".

### 4.9 Link de reseña desde un viaje completado

```
Viaje #482 — COMPLETADO
Turista: Martina R. · +54 9 11 …
[ Generar link de reseña ]
→ https://app.tudominio.com/review/trip_Zk3…   [Copiar]
[ 💬 Enviar por WhatsApp al turista ]
Estado: Enviado hoy 18:20 · Vence 30/10 · Sin responder
```

Mensaje prellenado: "¡Hola Martina! Gracias por navegar con nosotros. ¿Nos contás cómo te fue con Delfín del Delta? Solo toma 30 segundos: {link}".

## 5. Estados y manejo de errores

| Situación | Tratamiento |
|-----------|-------------|
| Cargando listas | Skeleton, no spinner a pantalla completa |
| Sin resultados en búsqueda | Mensaje + sugerencia (ampliar zona/horario) |
| Error de red | Banner con "Reintentar" |
| Upload fallido | Mensaje específico (tipo no permitido, tamaño) y conservación del resto del formulario |
| Conflicto horario al asignar | Modal de advertencia con detalle del viaje en conflicto |
| Token inválido | Pantalla neutral, sin pistas |
| Sesión vencida | Redirección a login conservando la ruta de retorno |

## 6. Responsive

| Pantalla | Móvil | Tablet | Escritorio |
|----------|-------|--------|------------|
| Landing | Prioridad (tráfico del turista) | Sí | Sí |
| Formulario de reseña | **Diseñado solo para móvil** | Centrado | Centrado |
| Portal lanchero | Prioridad móvil (suben fotos desde el celular) | Sí | Sí |
| Admin | Usable en tablet; dashboard en columnas deslizables | Sí | **Optimizado** |

## 7. Accesibilidad

- Contraste AA, foco visible, orden de tabulación lógico.
- Estrellas interactivas implementadas como `radiogroup` con etiquetas ("1 estrella"… "5 estrellas") y operables con teclado.
- Textos alternativos en fotos de lanchas (nombre + zona).
- Mensajes de error asociados a campos con `aria-describedby`.
- Soporte de zoom hasta 200 % sin pérdida de funcionalidad.

## 8. Microcopy (tono)

- Cercano y claro, voseo argentino ("Subí tu documento", "Contanos cómo te fue").
- Estados del lanchero en positivo: "En revisión", "Necesitamos que vuelvas a subir tu seguro".
- Rechazos con motivo y siguiente paso concreto.
- Evitar jerga técnica en el portal del lanchero y en el formulario del turista.

## 9. Entregables de diseño sugeridos

1. Wireframes de baja fidelidad (los de este documento) → validación con la agencia.
2. UI kit en Figma con tokens y componentes de la sección 3.
3. Prototipo clickeable de: búsqueda → asignación, verificación, reseña móvil.
4. Prueba de usabilidad con 2 lancheros y 1 admin antes del desarrollo del front.
