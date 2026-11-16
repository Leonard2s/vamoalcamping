<!-- BEGIN:nextjs-agent-rules -->

## This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

<!-- Project info (dev) -->

## VAMOALCAMPING — Cotizador + CRM

Next.js 16 (App Router, **cacheComponents activado**) + PostgreSQL + Tailwind v4 + TanStack Query (nunca Zustand).

### Comandos
- `docker compose up -d` (o el contenedor `vamoal-pg` ya creado) → Postgres en `:5432`
- `node scripts/migrate.mjs` → aplica `db/schema.sql` + seed de paquetes/inventario
- `npm run dev` / `npm run build` / `npm start -- -p 3100` (el puerto 3000 lo usa otra app local)
- `npx tsc --noEmit` → typecheck; `npx next typegen` → regenera tipos de rutas

### Auth / acceso
- Panel: `/admin` (login en `/admin/login`). Contraseña en `ADMIN_PASSWORD` de `.env.local` (dev: `vamoal2026`).
- Sesiones en tabla `sessions` (cookie httpOnly `vamoal_session`, 7 días). `src/proxy.ts` = chequeo optimista; la verificación real está en `src/lib/auth.ts` (DAL) y en cada route handler admin.

### Reglas de negocio clave
- `src/lib/quote.ts`: cálculo de precios (per_person_night / per_night / fixed_package + alquiler de carpa), noches ocupadas `[check_in, check_out)`, cupo de bote solo el día de check-in, depósito = 50%.
- Retienen capacidad los estados `esperando_50 | confirmada | en_campamento`.
- Inventario (cupos/stock de carpas/precios) editable en `/admin/config` (fila única `inventory_settings`).
- Next 16: `params`/`searchParams`/`cookies()` son async; datos frescos detrás de `<Suspense>` o `instant=false`; catálogo público con `'use cache'`.
