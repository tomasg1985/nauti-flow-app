# 🛥️ NautiFlow App

**NautiFlow** es el sistema de back-office centralizado para la gestión y optimización de una agencia de lanchas en el Delta, resolviendo la logística fluvial mediante verificación, búsqueda de flota, gestión de viajes y reseñas seguras.

---

## 📋 Documentación y Requisitos
* **Documentación:** Ubicada en la carpeta raíz (PRD, TRD, UX/UI, Schema, Plan) y la hoja de ruta en `docs/SPRINTS.md`.
* **Requisitos:** Node.js (`v22.x`+) y PostgreSQL (`v16.x`, nativo o mediante Docker).

---

## 🚀 Guía de Inicio Rápido

1. **Instalación:** Entra a `apps/api`, instala dependencias con `npm install` y copia `.env.example` a `.env` configurando `DATABASE_URL` y `JWT_SECRET`.
2. **Base de Datos:** Levanta el contenedor con `docker compose up -d` (opcional).
3. **Migraciones y Seeders:** Ejecuta `npm run db:migrate` y `npm run db:seed:admin`.
4. **Desarrollo:** Inicia el servidor con `npm run dev` (disponible en `http://localhost:3000`).

---

## 🧪 Tests y Autenticación
* **Pruebas:** Crea la base de datos `nautiflow_test` y ejecuta `npm test` dentro de `apps/api`.
* **Endpoints:** Puedes validar el funcionamiento utilizando los comandos `curl` provistos en el repositorio para el registro, verificación de sesión y control de accesos.

---

## 📂 Arquitectura de Directorios (`apps/api`)
* `src/server.js`: Punto de entrada del servidor.
* `src/app.js`: Configuración de Express y ruteo global.
* `src/config/`: Configuraciones centrales y variables de entorno.
* `src/models/` y `src/modules/`: Modelos de datos y arquitectura modular por dominios.
* `src/middlewares/`: Filtros de seguridad, validación y autenticación.
* `db/migrations/` y `tests/`: Migraciones SQL y suite de pruebas automatizadas.
