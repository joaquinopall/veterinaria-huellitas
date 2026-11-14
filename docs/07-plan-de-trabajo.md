# 07 · Plan de trabajo

> **Qué es este documento:** el orden en que se construye el proyecto, en etapas chicas y verificables.
> Cada etapa termina con algo que **funciona y se puede ver** (en Vercel), no con "la mitad de todo".
> Marcá `[x]` a medida que avanzás: este archivo es el tablero de estado del proyecto.

---

## Flujo de Git

```
main  ──●──────────────●──────────────●──►   producción (Vercel la publica sola)
         \            /              /
dev       ●──●──●──●─●──●──●──●──●──●──►    integración
              \    /      \     /
feat/xxx       ●──●        ●──●             una rama por tarea
```

- Cada tarea: rama `feat/...`, `fix/...`, `docs/...` o `chore/...` desde `dev` → Pull Request a `dev` (con CI en verde).
- Al cerrar cada etapa: PR de `dev` a `main` → se publica en producción.
- Commits con **Conventional Commits**, descripción en español:
  `feat(turnos): validar fecha según horarios de la sucursal`
  Tipos: `feat` (funcionalidad), `fix` (corrección), `docs`, `test`, `refactor`, `chore` (configuración), `style`, `perf`, `ci`.
- Commits **chicos y con sentido** (uno por paso lógico), nunca "avances varios".

---

## Etapa 0 · Especificaciones ✅
- [x] Requisitos, arquitectura, modelo de datos, seguridad, calidad, decisiones, plan.
- [x] `CLAUDE.md`, `README.md`, `.env.example`, `.gitignore`.

## Etapa 1 · Esqueleto del proyecto
**Objetivo:** un Next.js vacío pero profesional, publicado en Vercel, con CI.
- [ ] Crear proyecto Next.js (App Router, TS, Tailwind, ESLint, `src/`). *Nota: `create-next-app` no acepta carpetas con archivos; se genera en una carpeta temporal y se mueve.*
- [ ] TypeScript `strict` + `noUncheckedIndexedAccess`; Prettier; scripts `lint`, `typecheck`, `test`, `format`.
- [ ] Inicializar shadcn/ui; paleta de colores y tipografía con contraste verificado.
- [ ] Vitest configurado con un test de ejemplo.
- [ ] `src/lib/env.ts` con validación Zod de variables.
- [ ] GitHub Actions: lint → typecheck → test → build.
- [ ] Conectar repo a Vercel; Dependabot y escaneo de secretos activados en GitHub.
- **Verificación:** la URL de Vercel muestra la página base; el CI está en verde.

## Etapa 2 · Base de datos
**Objetivo:** tablas creadas, protegidas y con datos de demo.
- [ ] Crear proyectos Supabase `huellitas-dev` y `huellitas-prod` (región São Paulo, la más cercana).
- [ ] Prisma: `schema.prisma` según [03-modelo-de-datos](03-modelo-de-datos.md); conexión pooler + directa.
- [ ] Migración inicial + migración SQL que activa RLS en todas las tablas.
- [ ] `seed.ts` con sucursal, servicios, horarios y solicitudes de ejemplo.
- [ ] Test manual: con la clave pública, la API de Supabase **no** devuelve datos.
- **Verificación:** Prisma Studio muestra los datos; la consulta con la clave pública devuelve vacío/error.

## Etapa 3 · Sitio público
**Objetivo:** las 5 páginas públicas, rápidas, accesibles y entendibles por Google. (RF-01–04, RF-08, RNF-03–06)
- [ ] Layout: header con navegación accesible (menú móvil), footer con NAP + aviso de sitio ficticio, "Saltar al contenido".
- [ ] Home, Servicios, Nosotros, Contacto (mapa con *facade*), Privacidad.
- [ ] SEO: metadatos por página, JSON-LD `VeterinaryCare`, `sitemap.ts`, `robots.ts`.
- **Verificación:** Lighthouse mobile ≥ 90 en las 4 categorías; Rich Results Test válido; navegación completa con teclado.

## Etapa 4 · Formulario de solicitud de turno
**Objetivo:** el flujo central del negocio, blindado. (RF-05–07, RNF-01, RNF-11)
- [ ] Esquema Zod compartido (cliente + servidor) + lógica de disponibilidad (fechas/franjas válidas) con tests.
- [ ] Formulario accesible (errores anunciados, foco al primer error).
- [ ] Server Action: honeypot → Turnstile → rate limit → Zod → guardar → email.
- [ ] Plantilla de email (React Email) y pantalla de confirmación.
- [ ] Tests de integración de la acción (todos los caminos de falla).
- **Verificación:** enviar desde el celular → llega el email y la solicitud está en la base; spam simulado rechazado.

## Etapa 5 · Login y protección del panel
**Objetivo:** solo personal autorizado entra. (RF-10, RNF-02)
- [ ] Clientes Supabase (`@supabase/ssr`), página `/admin/login`, cierre de sesión, recuperación de contraseña.
- [ ] `getCurrentStaff()` + `permissions.ts` (`can()`) con tests de la matriz completa.
- [ ] Layout protegido + proxy/middleware que redirige al login.
- [ ] Crear usuarios de prueba (uno por rol) — credenciales **fuera del repo**.
- **Verificación:** sin sesión → redirige; usuario inactivo → no entra; cada rol ve solo su menú.

## Etapa 6 · Gestión de solicitudes
**Objetivo:** la recepcionista trabaja desde el panel. (RF-11–15)
- [ ] Lista con filtros (estado, fechas) en la URL y paginación; vista de tarjetas en celular.
- [ ] Detalle: datos, cambio de estado (`canTransition`), asignar veterinario, nota interna, historial.
- [ ] Vista del VET: solo asignadas.
- [ ] Tests de `status.ts` y del caso "VET pide solicitud ajena → 404".
- **Verificación:** recorrido completo con los 3 usuarios de prueba.

## Etapa 7 · Edición de contenido y usuarios
**Objetivo:** Laura es autónoma. (RF-16–18)
- [ ] ABM de servicios (crear, editar, ordenar, destacar, desactivar) + revalidación.
- [ ] Editor de horarios semanales y días especiales + revalidación.
- [ ] Gestión de usuarios: invitar, cambiar rol, desactivar (con protección anti-autobloqueo).
- **Verificación:** cambiar un precio en el panel → se ve en `/servicios` en segundos.

## Etapa 8 · Operación y endurecimiento
**Objetivo:** que "no se caiga ni la hackeen". (RNF-10, A10–A13)
- [ ] Cabeceras de seguridad y CSP (ADR-08).
- [ ] Cron diario de mantenimiento (retención + mantener activa la base).
- [ ] UptimeRobot sobre la home.
- [ ] Decidir e implementar backups (ver decisiones pendientes).
- [ ] `/security-review` sobre todo el proyecto.
- **Verificación:** [securityheaders.com](https://securityheaders.com) con nota A; el cron corre (logs de Vercel).

## Etapa 9 · Cierre y entrega
- [ ] Tests E2E con Playwright (flujos críticos + axe).
- [ ] Lighthouse final y puntajes en el README.
- [ ] README final: arquitectura, decisiones, limitaciones, cómo correrlo.
- [ ] Manual de la recepcionista (`docs/manual-recepcion.md`) con capturas.
- [ ] Revisión final del historial de commits, PR `dev` → `main`.
- [ ] Entregar: repo + URL + credenciales de prueba **por chat**.
