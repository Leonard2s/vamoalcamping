# VAMOALCAMPING — Cotizador de reservas + CRM de visitantes

Sistema web para VAMOALCAMPING (Playa El Valle, Samaná, R.D.) que resuelve su
problema operativo: **no hay señal celular en la zona**, todo depende de
internet satelital. El sitio actúa como recepción digital 24/7: los clientes
cotizan y pre-reservan en línea, y el negocio gestiona todo desde un CRM liviano.

## Stack

- **Next.js 16** (App Router, TypeScript, Tailwind v4, Cache Components)
- **PostgreSQL 16** (vía `postgres.js`, esquema en `db/schema.sql`)
- **TanStack Query** para data fetching en componentes cliente
- Sesiones admin en base de datos + cookie httpOnly (sin dependencias externas)

## Módulos

| Módulo | Ruta | Descripción |
|---|---|---|
| Landing | `/` | Aviso "sin señal", catálogo real del negocio (modalidades, carpas, equipos/renta, cocina, excursiones, reglas, ubicación), cómo reservar, contacto (estática, caché 1h). Cada modalidad enlaza directo a su reserva (`/reservar?paquete=<id>`) |
| Catálogo | `/` + `/admin/config` | Equipos, comida y excursiones en `catalog_items` — visibles en la landing, editables en el CMS. No se reservan en línea: se coordinan por WhatsApp |
| Cuenta visitante | `/acceso`, `/mi-cuenta` | Registro/login con WhatsApp + contraseña (scrypt). Dashboard con historial, perfil y progreso de fidelidad |
| Cotizador | `/reservar` | 3 pasos: fechas+grupo → experiencia (+carpa) → datos. Verifica disponibilidad real por noche. Prellena datos y aplica descuento si hay sesión |
| Confirmación | `/reservar` (paso final) | Código `VAC-XXXX`, datos bancarios y deep-link a WhatsApp con comprobante |
| Kanban | `/admin` | Pipeline: Esperando 50% → Confirmada → En campamento → Completada/Cancelada + reserva manual (leads de WhatsApp/IG) |
| Detalle | `/admin/reservas/[id]` | Estado, abono, notas internas, WhatsApp al visitante |
| Calendario | `/admin/calendario` | Ocupación por día: personas, reservas, carpas, bote |
| Visitantes | `/admin/visitantes` | CRM: historial, visitas, gasto acumulado, etiquetas |
| Config | `/admin/config` | CMS: cupos, stock/precios de carpas (1p/2p/4p/6p), bote, fidelización, CRUD de experiencias reservables y CRUD del catálogo de artículos |

## Setup local

```bash
docker compose up -d          # PostgreSQL en :5432
npm install
node scripts/migrate.mjs      # crea tablas + catálogo inicial
npm run dev                   # http://localhost:3000
```

Credenciales por defecto (cambiar en `.env.local` antes de producción):
`ADMIN_PASSWORD=vamoal2026`, Postgres `postgres:vamoal_dev`.

## Reglas de negocio implementadas

- **Capacidad por noche**: máx. personas y reservas/noche; stock de carpas por
  tamaño (1p/2p/4p/6p); cupo de bote solo el día de llegada.
- **Modalidades reales** (según el catálogo del negocio): camping tradicional
  RD$1,000/noche por casa de campaña, con vehículo al lado RD$2,000,
  overlanding RD$3,000/vehículo, paquete Playa Ermitaño RD$29,500.
  El precio del camping es por carpa/noche, no por persona.
- **Dos cuentas bancarias** para el depósito (`NEXT_PUBLIC_BANK_*` y
  `NEXT_PUBLIC_BANK2_*`).
- **Retención de cupo**: los estados `esperando_50`, `confirmada` y
  `en_campamento` descuentan disponibilidad → no hay overbooking.
- **Depósito**: siempre 50% del total calculado en servidor.
- **Fidelización**: a partir de `loyalty_min_visits` visitas no canceladas,
  el visitante logueado obtiene `loyalty_discount_pct`% de descuento
  automático (ambos configurables en `/admin/config`). Registrarse con el
  mismo WhatsApp vincula el historial de reservas previas.
- **Cero llamadas**: todo contacto por WhatsApp (`wa.me`) con mensajes
  prellenados que incluyen el código de reserva.
