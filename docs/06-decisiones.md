# 06 · Decisiones técnicas (ADRs)

> **Qué es un ADR:** *Architecture Decision Record*, un registro corto de una decisión: contexto, qué se eligió, qué se descartó y qué consecuencias tiene.
> Sirve para que otra persona (o vos en 6 meses) entienda **por qué** el sistema es como es, y no lo "arregle" rompiendo algo pensado.
> Formato: Contexto → Decisión → Alternativas descartadas → Consecuencias. Estado: ✅ aceptada.

---

## ADR-01 · Supabase Auth para el login
- **Contexto:** el panel necesita login para 3–4 personas, sin registro público, con invitaciones.
- **Decisión:** Supabase Auth con email + contraseña, sesión en cookies vía `@supabase/ssr`.
- **Descartadas:**
  - *Auth.js (NextAuth)*: más flexible, pero agrega configuración, adaptador de base y tablas propias que no necesitamos. Complejidad sin beneficio para este caso.
  - *Auth propio*: reinventar hashing, sesiones y recuperación de contraseña es un riesgo de seguridad injustificado.
- **Consecuencias:** ya incluido en el stack y gratis; trae invitaciones y recuperación de contraseña. Quedamos atados a Supabase para auth (aceptable: la base también está ahí).

## ADR-02 · Prisma como acceso a datos + RLS "cerrado"
- **Contexto:** el stack fija Prisma + Supabase. Supabase expone automáticamente una API REST por tabla, accesible con la clave pública.
- **Decisión:** toda lectura/escritura pasa por Prisma desde el servidor. RLS **activado sin políticas** en todas las tablas (vía migración), lo que deja la API pública sin acceso. La autorización vive en nuestro código.
- **Descartadas:**
  - *Usar el cliente de Supabase + políticas RLS como autorización*: potente, pero duplica la lógica (políticas SQL + código) y Prisma no las respeta de todos modos.
  - *Dejar RLS apagado*: error grave; cualquier persona con la clave pública podría leer datos personales.
- **Consecuencias:** un único lugar de autorización (`permissions.ts`), testeable. Se usa la conexión con *pooler* para la app (serverless) y la directa para migraciones.

## ADR-03 · Resend para emails
- **Contexto:** hay que avisar a la veterinaria de cada solicitud.
- **Decisión:** Resend + plantillas con React Email.
- **Descartadas:** *Brevo* (más envíos gratis, pero integración menos directa); *SMTP de Gmail* (frágil, límites poco claros, mala práctica en producción).
- **Consecuencias:** experiencia previa del desarrollador (menos riesgo, más velocidad); plan gratis sobra para el volumen. **Sin dominio propio solo se puede enviar a la casilla del dueño de la cuenta** → el email de confirmación al cliente queda para cuando haya dominio.

## ADR-04 · Rate limit con Upstash Redis, *fail-open*
- **Contexto:** hay que limitar envíos por IP en un entorno serverless (sin memoria compartida entre ejecuciones).
- **Decisión:** `@upstash/ratelimit` con ventana deslizante (3/10 min y 10/día por IP hasheada). Si Upstash no responde, se deja pasar y se registra.
- **Descartadas:** *tabla en Postgres* (un servicio menos, pero más escrituras a la base, limpieza manual y peor rendimiento); *memoria del proceso* (no funciona en serverless: cada ejecución puede ser otra instancia).
- **Consecuencias:** una cuenta más (ya existente). Los registros vencen solos (TTL), lo que también ayuda a la privacidad. Fail-open: priorizamos no perder clientes; Turnstile + honeypot cubren mientras tanto.

## ADR-05 · Antispam con Cloudflare Turnstile + honeypot
- **Contexto:** formulario público sin login → objetivo obvio de bots.
- **Decisión:** honeypot (gratis, invisible) + Turnstile (verificado en servidor). Turnstile *fail-closed*.
- **Descartadas:** *reCAPTCHA* (más fricción para el usuario, implicancias de privacidad, más peso); *solo honeypot* (insuficiente contra bots modernos).
- **Consecuencias:** casi sin fricción para humanos; se agrega `challenges.cloudflare.com` a la CSP.

## ADR-06 · Renderizado estático + revalidación bajo demanda
- **Contexto:** Lighthouse ≥ 90 en mobile, pero precios y horarios se editan desde el panel.
- **Decisión:** páginas públicas estáticas; cada acción del panel que cambia contenido público revalida las rutas afectadas.
- **Descartadas:** *renderizado dinámico en cada visita* (más lento, más consultas, riesgo con la base pausada); *revalidación por tiempo* (los cambios tardarían en verse, Laura pensaría que "no anda").
- **Consecuencias:** velocidad de sitio estático con contenido editable. Hay que acordarse de revalidar → centralizado en cada `actions.ts`.

## ADR-07 · Modelo multi-sucursal desde el día 1
- **Contexto:** posible segunda sucursal; no hay que implementarla.
- **Decisión:** entidad `Location` y `locationId` en todo lo del negocio; datos de contacto en la base, no en el código.
- **Descartadas:** *una sola sucursal con datos en el código* (más simple hoy, pero obliga a migrar datos y tocar todo el sistema mañana).
- **Consecuencias:** costo marginal hoy (una tabla y una columna por entidad); la segunda sucursal es una fila + UI de selección.

## ADR-08 · CSP diferenciada público vs. admin
- **Contexto:** la CSP más estricta (con *nonce*) obliga a renderizar dinámicamente, lo que choca con ADR-06.
- **Decisión:** `/admin` (ya dinámico) con CSP estricta con nonce; páginas públicas con CSP de base (orígenes restringidos, `frame-ancestors 'none'`, `object-src 'none'`).
- **Descartadas:** *nonce en todo* (pierde lo estático y el puntaje de rendimiento); *sin CSP* (pierde una capa contra XSS).
- **Consecuencias:** la parte con datos personales tiene la protección máxima; lo público, que no tiene datos sensibles, prioriza rendimiento.

## ADR-09 · Roles en tabla propia + mapa central de permisos
- **Contexto:** 3 roles (ADMIN, RECEPTION, VET) con accesos distintos; principio de mínimo privilegio.
- **Decisión:** `StaffUser.role` en nuestra base; función `can(user, action, resource?)` en `src/lib/auth/permissions.ts` como **única** fuente de verdad.
- **Descartadas:** *roles en metadatos de Supabase Auth* (menos visible, más difícil de auditar y editar desde el panel); *chequeos `if (role === ...)` dispersos* (imposible de mantener y fácil de olvidar uno).
- **Consecuencias:** agregar un rol o permiso = editar un archivo y sus tests.

## ADR-10 · Código en inglés; interfaz y documentación en español
- **Contexto:** equipo hispanohablante; librerías y ecosistema en inglés.
- **Decisión:** identificadores (variables, tablas, funciones) en inglés; textos de la UI, URLs públicas (`/servicios`, `/turnos`) y documentación en español.
- **Descartadas:** *todo en español* (mezcla inevitable con APIs en inglés: `prisma.solicitud.findMany`); *todo en inglés* (URLs en inglés perjudican el SEO local).
- **Consecuencias:** glosario español↔inglés en [glosario.md](glosario.md).

## ADR-11 · Server Actions para mutaciones
- **Contexto:** formulario público y acciones del panel.
- **Decisión:** Server Actions (funciones del servidor llamadas desde formularios), cada una con el pipeline sesión → permiso → validación → operación → auditoría → revalidación.
- **Descartadas:** *API Routes REST* (más código repetido; no hay clientes externos que necesiten una API).
- **Consecuencias:** menos código y mejores formularios (funcionan sin JS). Hay que recordar que **son endpoints públicos** → checklist de seguridad.

## ADR-12 · Cron diario de Vercel para mantenimiento
- **Contexto:** Supabase gratis se pausa con 7 días sin actividad; la ley pide limitar la retención.
- **Decisión:** un único cron diario (`/api/cron/maintenance`, protegido con `CRON_SECRET`) que borra datos vencidos y, de paso, mantiene la base activa.
- **Descartadas:** *GitHub Actions programado* (otra pieza fuera de la app); *no hacer nada* (sitio caído sin aviso).
- **Consecuencias:** el plan Hobby de Vercel permite crons diarios: suficiente.

---

## Decisiones pendientes

| Tema | Opciones | Se decide en |
|---|---|---|
| Backups | GitHub Action semanal con `pg_dump` cifrado vs. procedimiento manual documentado | Etapa 8 |
| Agentes de verificación propios | Crear un subagente "revisor de specs" solo si `/code-review` y `/security-review` no alcanzan | Tras la etapa 4 |
