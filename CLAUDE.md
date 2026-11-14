# CLAUDE.md — Veterinaria Huellitas

Sitio público + panel de gestión de solicitudes de turno para una veterinaria de barrio (Villa Urquiza, CABA). Proyecto de **autoevaluación freelance** de Joaquín: el objetivo es que él **aprenda y entienda todo**, no solo terminarlo.

## Reglas de oro (no negociables)

1. **Spec-driven.** Antes de implementar, leé las specs relevantes en `docs/`. Si una tarea no está en [docs/01-requisitos.md](docs/01-requisitos.md) o contradice una spec, **frená y preguntá**; no inventes alcance. Lo que está en "Fuera de alcance" no se construye.
2. **Las decisiones son de Joaquín.** Si aparece una decisión técnica no cubierta por [docs/06-decisiones.md](docs/06-decisiones.md), presentá 2–3 opciones con tu recomendación y esperá su respuesta. Una vez decidida, registrala como ADR nuevo.
3. **Explicá siempre, en español y para alguien no 100 % técnico.** Antes de tocar código: qué vas a hacer y por qué (2–4 líneas). Al terminar cada tarea, con este formato:
   - **Qué hice** (en concreto, archivos y comportamiento).
   - **Por qué así** (vinculado a la spec/ADR).
   - **Alternativa descartada** y por qué.
   - **Conceptos nuevos** (si apareció un término, explicalo y agregalo a [docs/glosario.md](docs/glosario.md)).
   - **Cómo verificarlo vos** (pasos concretos).
4. **Nunca commitees ni pushees sin que Joaquín lo pida.** Proponé el mensaje de commit (Conventional Commits, descripción en español).
5. **Una tarea del plan a la vez.** Seguí el orden de [docs/07-plan-de-trabajo.md](docs/07-plan-de-trabajo.md) y marcá `[x]` al terminar.

## Stack

Next.js (App Router, TypeScript strict) · Tailwind · shadcn/ui · Prisma · Supabase (Postgres + Auth) · Upstash Redis (rate limit) · Cloudflare Turnstile · Resend + React Email · Zod · Vitest · Playwright · Vercel · GitHub Actions.
Versiones exactas: las de `package.json`. Ante dudas de API de una librería, consultá su documentación de la versión instalada; no asumas APIs de versiones viejas.

## Mapa de specs

| Necesito… | Leer |
|---|---|
| Qué construir / criterios de aceptación | `docs/01-requisitos.md` |
| Dónde va cada archivo, flujos, renderizado | `docs/02-arquitectura.md` |
| Tablas, campos, máquina de estados | `docs/03-modelo-de-datos.md` |
| Amenazas, permisos, checklist de seguridad | `docs/04-seguridad.md` |
| Rendimiento, SEO, a11y, tests, Definition of Done | `docs/05-calidad.md` |
| Por qué se eligió cada cosa | `docs/06-decisiones.md` |
| Qué sigue | `docs/07-plan-de-trabajo.md` |

## Convenciones de código

- **Idioma:** identificadores en inglés; textos de UI, URLs públicas y docs en español (ADR-10). Términos de negocio según [docs/glosario.md](docs/glosario.md).
- **Estructura por funcionalidad:** lógica en `src/features/<feature>/` (`schema.ts`, `actions.ts`, `queries.ts`, `components/`). Infra compartida en `src/lib/`. Una feature no importa de otra: lo compartido sube a `lib/`.
- **Server Components por defecto.** `"use client"` solo si hay estado/eventos del navegador.
- **Todo archivo con acceso a DB, secretos o sesión** importa `"server-only"`.
- **Zod en todos los bordes** (formularios, params, searchParams, env). Un esquema por formulario, compartido entre cliente y servidor.
- **TypeScript:** prohibido `any` y `@ts-ignore`. Preferí tipos inferidos de Zod y Prisma.
- **Reglas de negocio puras** (sin React ni DB) en archivos propios y testeados: `status.ts`, `permissions.ts`, `schedule/availability.ts`.
- **Fechas:** se guardan en UTC; se muestran y validan en `America/Argentina/Buenos_Aires` usando `src/lib/dates.ts`. Nunca `new Date()` "suelto" para lógica de calendario.
- **Dinero:** enteros en pesos, formateados con `Intl.NumberFormat("es-AR")`.
- **Contenido del negocio** (dirección, teléfono, horarios, precios) viene de la base vía `getDefaultLocation()`; nunca hardcodeado en componentes.
- **Accesibilidad:** cada input con `<label>`, errores con `aria-describedby`, foco visible, HTML semántico, imágenes con `alt`.
- **Imágenes** con `next/image`; **fuentes** con `next/font`.

## Seguridad (resumen operativo de `docs/04-seguridad.md`)

- **Pipeline obligatorio en cada Server Action del panel:** sesión → `StaffUser` activo → `can(user, action, resource)` → Zod → operación + `RequestEvent` (misma transacción) → revalidación. Nunca confiar en que el proxy/middleware protegió la ruta.
- **Permisos solo vía `src/lib/auth/permissions.ts`.** Nada de `if (role === "ADMIN")` disperso.
- VET que accede a una solicitud no asignada → `notFound()`.
- **RLS:** toda tabla nueva lleva `ENABLE ROW LEVEL SECURITY` en su migración. Nunca deshabilitarlo ni crear políticas permisivas.
- **Secretos:** nunca con prefijo `NEXT_PUBLIC_`; nunca en código, logs, tests ni commits. Variables nuevas → agregar a `src/lib/env.ts` y `.env.example`.
- **Logs sin datos personales** (usar ids, no emails ni teléfonos).
- Prohibido `dangerouslySetInnerHTML`, salvo el JSON-LD (serializado con `JSON.stringify` y escapando `<`).
- No crear usuarios con contraseña en el seed ni en scripts versionados.

## Comandos

```bash
npm run dev          # servidor local
npm run lint         # ESLint
npm run typecheck    # tsc --noEmit
npm test             # Vitest
npm run test:e2e     # Playwright
npm run build        # build de producción
npx prisma migrate dev --name <nombre>   # nueva migración (base dev)
npx prisma db seed   # datos de demo
npx prisma studio    # ver la base
```

## Antes de declarar una tarea terminada

Cumplí la **Definition of Done** de `docs/05-calidad.md` §7. Como mínimo: `lint`, `typecheck`, `test` y `build` en verde; checklist de seguridad si hay Server Actions; prueba con teclado y a 360 px si hay UI pública. Si algo no pasa, decilo claramente con la salida del error; no lo maquilles.

## Git

- Ramas `feat/`, `fix/`, `docs/`, `chore/`, `test/` desde `dev`; PR a `dev`; `dev` → `main` al cerrar cada etapa. `main` = producción.
- Commits chicos y atómicos, Conventional Commits: `feat(turnos): validar franja según horario`.
- Nunca commitear `.env*` (salvo `.env.example`), credenciales ni dumps de base.
