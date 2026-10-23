# 04 · AppFlow — Flujos de la aplicación

**Producto:** NautiFlow · **Versión:** 1.0 (MVP) · **Fecha:** 16/10/2026

Los diagramas están en sintaxis **Mermaid** (se renderizan en GitHub, GitLab, Notion, VS Code con extensión, mermaid.live).

---

## 1. Vista general del ecosistema

```mermaid
flowchart LR
    L[Lanchero] -->|1. Alta + docs| P[Portal Lancheros]
    P --> DB[(PostgreSQL)]
    A[Administrador] -->|2. Revisa y verifica| V[Panel de Verificación]
    V --> DB
    T[Turista] -->|Contacta| AG[Agencia vía WhatsApp]
    AG --> A
    A -->|3. Busca flota| S[Motor de Búsqueda]
    S --> DB
    A -->|4. Contacta y asigna| W[WhatsApp wa.me]
    W --> L
    A -->|5. Completa viaje + link| R[Link /review/token]
    R --> T
    T -->|6. Califica| F[Formulario de reseña]
    F --> REP[Reputación]
    REP --> DB
    REP -->|7. Email| L
    DB --> LP[Landing pública]
    T -.->|Lee reseñas| LP
    A -->|8. Retorno vacío| TV[Bolsa de tramos vacíos]
    TV --> DB
```

## 2. Flujo A — Alta y verificación del lanchero

```mermaid
sequenceDiagram
    actor L as Lanchero
    participant W as Web (Portal)
    participant API as API
    participant S as Storage (R2)
    participant DB as PostgreSQL
    actor A as Admin

    L->>W: Registro (email + contraseña)
    W->>API: POST /auth/register
    API->>DB: crea user + lanchero (PENDING)
    L->>W: Completa datos, fotos, zonas, horarios
    W->>API: PUT/POST /me/boats...
    API->>S: guarda fotos (sin EXIF, WebP)
    L->>W: Sube Licencia, Seguro, Antecedentes
    W->>API: POST /me/documents
    API->>S: guarda en bucket privado
    API->>DB: documents = PENDING · boat = PENDING_REVIEW
    A->>W: Abre cola de verificación
    W->>API: GET /admin/verification/queue
    A->>API: GET /admin/documents/:id/url
    API->>S: genera URL firmada (5 min)
    API->>DB: audit_logs (acceso a documento)
    A->>API: PATCH documento APPROVED / REJECTED(+motivo)
    alt Los 3 documentos aprobados y vigentes
        A->>API: POST /admin/boats/:id/verify
        API->>DB: boat = VERIFIED · audit_logs
        API-->>L: Email "Tu lancha fue verificada"
    else Algún documento rechazado
        API-->>L: Email con motivo del rechazo
        L->>W: Re-sube solo el documento observado
    end
```

### Máquina de estados — Documento

```mermaid
stateDiagram-v2
    [*] --> PENDING: Lanchero sube archivo
    PENDING --> APPROVED: Admin aprueba
    PENDING --> REJECTED: Admin rechaza (motivo)
    REJECTED --> PENDING: Lanchero re-sube
    APPROVED --> EXPIRED: Pasa fecha de vencimiento
    EXPIRED --> PENDING: Lanchero sube versión nueva
```

### Máquina de estados — Lancha (verificación)

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Registro
    DRAFT --> PENDING_REVIEW: Completa datos y 3 documentos
    PENDING_REVIEW --> VERIFIED: Admin verifica (docs 3/3 vigentes)
    PENDING_REVIEW --> REJECTED: Admin rechaza (motivo)
    REJECTED --> PENDING_REVIEW: Lanchero corrige y reenvía
    VERIFIED --> SUSPENDED: Admin suspende / documento vencido
    SUSPENDED --> PENDING_REVIEW: Regulariza documentación
```

## 3. Flujo B — Búsqueda y asignación

```mermaid
flowchart TD
    A[Llega solicitud por WhatsApp / teléfono] --> B[Admin crea viaje: pax, zona, fecha/hora, duración]
    B --> C[Admin abre Buscador]
    C --> D{Filtros del sistema}
    D -->|Verificada + activa| E
    D -->|Capacidad ≥ pasajeros| E
    D -->|Cubre la zona| E
    D -->|Horario dentro de disponibilidad| E
    D -->|Sin viaje superpuesto| E
    D -->|Docs vigentes| E[Lista de lanchas candidatas]
    E --> F{¿Hay resultados?}
    F -->|No| G[Ampliar zona/horario o ofrecer otra fecha]
    G --> C
    F -->|Sí| H[Admin elige lancha y pulsa WhatsApp]
    H --> I[Abre wa.me con mensaje prellenado]
    I --> J{¿Lanchero acepta?}
    J -->|No| K[Elige otra candidata]
    K --> H
    J -->|Sí| L[Admin pulsa Asignar]
    L --> M{¿Conflicto horario?}
    M -->|Sí| N[Advertencia: forzar con nota o cancelar]
    N --> L
    M -->|No| O[Viaje = ASSIGNED]
    O --> P[Admin confirma al turista por WhatsApp]
```

## 4. Flujo C — Ciclo de vida del viaje

```mermaid
stateDiagram-v2
    [*] --> REQUESTED: Admin carga solicitud
    REQUESTED --> ASSIGNED: Asigna lancha
    ASSIGNED --> REQUESTED: Desasigna / lanchero declina
    ASSIGNED --> COMPLETED: Registra viaje completado
    REQUESTED --> CANCELLED: Cancela (motivo)
    ASSIGNED --> CANCELLED: Cancela (motivo)
    COMPLETED --> [*]
    CANCELLED --> [*]
```

Acciones habilitadas por estado:

| Estado | Acciones del admin |
|--------|-------------------|
| `REQUESTED` | Buscar lancha, editar, cancelar |
| `ASSIGNED` | Completar, desasignar, cancelar, **Publicar retorno vacío** |
| `COMPLETED` | **Generar link de reseña**, publicar retorno vacío (si aún aplica) |
| `CANCELLED` | Solo lectura |

## 5. Flujo D — Reseña con token y reputación

```mermaid
sequenceDiagram
    actor A as Admin
    actor T as Turista
    participant API as API
    participant DB as PostgreSQL
    participant M as Email
    actor L as Lanchero

    A->>API: POST /admin/trips/:id/review-link
    API->>DB: valida trip = COMPLETED
    API->>DB: guarda hash(token), expira en 14 días
    API-->>A: url /review/trip_xxx + enlace wa.me
    A->>T: Envía link por WhatsApp
    T->>API: GET /public/reviews/trip_xxx
    API->>DB: busca hash(token) vigente y sin usar
    alt Token inválido, usado o vencido
        API-->>T: 404 "Este enlace ya no está disponible"
    else Token válido
        API-->>T: Nombre de lancha + fecha
        T->>API: POST rating (1–5), comentario, nombre
        API->>DB: BEGIN · bloquea token
        API->>DB: inserta review · marca token usado
        API->>DB: recalcula boats.rating_avg y rating_count
        API->>DB: COMMIT
        API-->>T: "¡Gracias!"
        API->>M: Email al lanchero con la reseña
        M-->>L: Nueva reseña recibida
        Note over DB: Landing pública ya refleja el nuevo promedio
    end
```

### Estados del token

```mermaid
stateDiagram-v2
    [*] --> ACTIVE: Admin genera link
    ACTIVE --> USED: Turista envía reseña
    ACTIVE --> EXPIRED: Pasan 14 días
    ACTIVE --> REVOKED: Admin regenera link
    USED --> [*]
    EXPIRED --> [*]
    REVOKED --> [*]
```

## 6. Flujo E — Moderación de reseñas

```mermaid
flowchart TD
    A[Nueva reseña publicada] --> B{¿Contiene datos personales / insultos?}
    B -->|No| C[Permanece visible]
    B -->|Sí| D[Admin oculta con motivo]
    D --> E[Se recalcula promedio sin esa reseña]
    E --> F[Auditoría registrada]
    C --> G{¿Promedio < 3.0 con ≥ 5 reseñas?}
    G -->|Sí| H[Alerta interna al admin para revisar a la lancha]
    G -->|No| I[Sin acción]
```

## 7. Flujo F — Bolsa de tramos vacíos

```mermaid
flowchart TD
    A[Viaje ASIGNADO hacia el Delta] --> B[Admin pulsa Publicar Retorno Vacío]
    B --> C[Formulario: zona origen, hora de salida, destino opcional]
    C --> D[Sistema crea empty_leg = OPEN]
    D --> E[Genera mensaje: Lancha X vuelve vacía desde Zona Y a las HH:MMhs...]
    E --> F[Copia al portapapeles + alerta interna]
    F --> G[Aparece en columna Retornos Vacíos del dashboard]
    G --> H[Admin pega el mensaje en el grupo de WhatsApp]
    H --> I[empty_leg = OFFERED]
    I --> J{¿Alguien responde?}
    J -->|Sí| K[Admin coordina y marca OCCUPIED, vincula viaje]
    J -->|No| L{¿Pasó la hora + 2 h?}
    L -->|Sí| M[Job lo marca EXPIRED]
    L -->|No| J
    K --> N[Se crea nuevo viaje ASIGNADO con la misma lancha]
```

### Estados del tramo vacío

```mermaid
stateDiagram-v2
    [*] --> OPEN: Se publica
    OPEN --> OFFERED: Admin lo ofrece al grupo
    OPEN --> OCCUPIED: Se concreta
    OFFERED --> OCCUPIED: Se concreta
    OPEN --> CANCELLED: Admin cancela
    OFFERED --> CANCELLED: Admin cancela
    OPEN --> EXPIRED: Job (salida + 2 h)
    OFFERED --> EXPIRED: Job (salida + 2 h)
```

## 8. Flujo G — Vencimiento de documentación (automático)

```mermaid
flowchart TD
    A[Job diario 03:00] --> B{¿Documento vence en ≤ 30 días?}
    B -->|Sí| C[Email recordatorio al lanchero 30 y 7 días]
    A --> D{¿Documento venció?}
    D -->|Sí| E[Documento = EXPIRED]
    E --> F[Lancha = SUSPENDED]
    F --> G[Sale de búsquedas y landing]
    G --> H[Email al lanchero + alerta al admin]
    H --> I[Lanchero sube documento nuevo]
    I --> J[Vuelve a cola de verificación]
```

## 9. Matriz de permisos

| Recurso / Acción | Público | Lanchero | Admin |
|------------------|:------:|:--------:|:-----:|
| Ver landing y reseñas visibles | ✔ | ✔ | ✔ |
| Enviar reseña (con token válido) | ✔ | — | — |
| Registrarse / editar su lancha | — | ✔ (propia) | ✔ |
| Subir documentos | — | ✔ (propios) | ✔ |
| Ver documentos | — | ✔ (propios) | ✔ |
| Aprobar / rechazar documentos | — | — | ✔ |
| Buscar flota, ver teléfono | — | — | ✔ |
| Crear / asignar / completar viajes | — | — | ✔ |
| Generar link de reseña | — | — | ✔ |
| Ver reseñas recibidas | — | ✔ (propias) | ✔ |
| Moderar reseñas | — | — | ✔ |
| Publicar / gestionar retornos vacíos | — | — | ✔ |
| Ver datos de contacto del turista | — | — | ✔ |
