# NautiFlow

Back-office para una agencia de lanchas del Delta: verificación de lancheros, buscador de flota, viajes, reseñas con token y retornos vacíos.

- Documentación técnica: carpeta de documentos entregada antes (PRD, TRD, UX/UI, AppFlow, Schema, Plan).
- Plan de trabajo: [`docs/SPRINTS.md`](docs/SPRINTS.md)

## Requisitos

- **Node.js 22 o superior** → `node -v`
- **PostgreSQL 16**: instalado, o con Docker (`docker compose up -d` desde esta carpeta)

## Primer arranque (Día 1)

```bash
cd apps/api
npm install                 # descarga las dependencias
cp .env.example .env        # crea tu archivo de configuración (revisá DATABASE_URL)
npm run db:migrate          # crea las tablas en la base
npm run db:seed:admin       # crea el usuario ADMIN con ADMIN_EMAIL / ADMIN_PASSWORD de tu .env
npm run dev                 # arranca la API en http://localhost:3000
```

Probá en otra terminal:

```bash
curl localhost:3000/health          # {"status":"ok",...}
curl localhost:3000/api/v1/zones    # lista de zonas
```

### Probar el login (Día 2)

```bash
# Registrar un lanchero y guardar la cookie de sesión en cookies.txt
curl -c cookies.txt -H "Content-Type: application/json" \
  -d '{"email":"demo@mail.com","password":"MiClaveSegura1","fullName":"Demo","phone":"+54 9 11 5555-0000","acceptTerms":true}' \
  localhost:3000/api/v1/auth/register

curl -b cookies.txt localhost:3000/api/v1/auth/me           # ¿quién soy?
curl -b cookies.txt localhost:3000/api/v1/admin/ping        # 403: un lanchero no es admin
```

> **Importante (Día 2):** tu `.env` necesita `JWT_SECRET` (mínimo 32 caracteres). Copiá las variables nuevas desde `.env.example`.

## Tests

Creá una vez la base de pruebas y migrala:

```bash
# con PostgreSQL local (ajustá el usuario si el tuyo es otro)
psql -U nautiflow -h localhost -d postgres -c "CREATE DATABASE nautiflow_test;"
cd apps/api && npm run db:migrate:test
npm test
```

> Con Docker: `docker compose exec db psql -U nautiflow -c "CREATE DATABASE nautiflow_test;"`

## Mapa de carpetas de la API (`apps/api`)

| Ruta | Para qué sirve |
|------|----------------|
| `src/server.js` | Enciende el servidor (conecta la base y escucha un puerto) |
| `src/app.js` | Arma Express: seguridad, JSON, rutas, errores |
| `src/config/` | Variables de entorno validadas y conexión a la base |
| `src/models/` | Tablas traducidas a JavaScript + relaciones |
| `src/modules/<tema>/` | Cada funcionalidad: `routes` (URLs) → `controller` (lógica) |
| `src/middlewares/` | Pasos intermedios: manejo de errores, (luego) auth |
| `db/migrations/` | Cambios de la base en SQL, numerados |
| `scripts/migrate.js` | Aplica las migraciones pendientes |
| `tests/` | Pruebas automáticas |
=======
