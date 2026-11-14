# 04 · Seguridad y privacidad

> **Qué es este documento:** contra qué nos defendemos (amenazas) y cómo (controles).
> Laura pidió "que no la hackeen": esto es lo que esa frase significa en concreto.

---

## 1. Modelo de amenazas (qué puede salir mal)

| # | Amenaza | Ejemplo real | Controles |
|---|---|---|---|
| A1 | **Spam de bots** en el formulario | Un bot manda 5.000 solicitudes falsas por noche | Honeypot · Turnstile · rate limit · validación servidor |
| A2 | **Abuso humano** | Alguien envía 50 solicitudes a mano por broma | Rate limit por IP (3 cada 10 min, 10 por día) |
| A3 | **Lectura de datos personales** vía API de Supabase | Alguien copia la clave pública del sitio y consulta la tabla de solicitudes | RLS activo sin políticas; la clave pública no da acceso a nada |
| A4 | **Acceso al panel sin permiso** | Un ex-empleado intenta entrar | Login obligatorio · usuarios activos/inactivos · sin registro público |
| A5 | **Escalada de privilegios** | Un veterinario intenta ver solicitudes de otro, o cambiar precios | Permisos verificados en el servidor en cada acción, no solo ocultando botones |
| A6 | **Llamar Server Actions directo** | Alguien inspecciona la red y repite el pedido sin sesión | Cada acción verifica sesión + rol + datos por sí misma |
| A7 | **Inyección** (SQL / HTML / email) | Comentario con `<script>` o `'; DROP TABLE` | Prisma parametriza consultas · React escapa HTML · plantillas de email escapan · Zod limita formato y longitud |
| A8 | **Fuerza bruta al login** | Probar contraseñas | Rate limit de Supabase Auth · contraseñas fuertes · mensajes de error genéricos |
| A9 | **Filtración de secretos** | Subir `.env` a GitHub | `.gitignore` · `.env.example` sin valores · variables en Vercel · `server-only` · escaneo de secretos de GitHub activado |
| A10 | **Clickjacking / XSS** | Meter el panel en un iframe ajeno | Cabeceras de seguridad (CSP, `frame-ancestors 'none'`, etc.) |
| A11 | **Dependencias vulnerables** | Una librería con un fallo conocido | Dependabot · `npm audit` en CI |
| A12 | **Pérdida de datos** | Borrado accidental o falla del proveedor | Borrado lógico de servicios · backups semanales cifrados (etapa 8) |
| A13 | **Caída silenciosa** | Supabase pausa la base y nadie se entera | Cron diario + UptimeRobot |

---

## 2. Defensa del formulario público (en capas)

Ninguna capa es perfecta sola; juntas hacen que el abuso no valga la pena.

| Capa | Qué hace | Si falla… |
|---|---|---|
| 1. **Honeypot** | Campo oculto (para humanos y lectores de pantalla) que solo un bot completa. Si viene lleno: respondemos "ok" pero no guardamos nada (el bot no se entera de que lo detectamos). | Bots sofisticados lo saltean → sigue la capa 2 |
| 2. **Turnstile** | Cloudflare verifica que es un navegador real. Token verificado **en el servidor**. | Si Cloudflare no responde: rechazamos con mensaje "reintentá" (*fail-closed*: sin esta capa el formulario queda muy expuesto). |
| 3. **Rate limit** (Upstash) | Máx. 3 envíos cada 10 min y 10 por día por IP. La IP se guarda **hasheada** (no en claro) y se borra sola al vencer. | Si Upstash no responde: **dejamos pasar** (*fail-open*) y lo registramos. Turnstile + honeypot siguen protegiendo y Laura no pierde clientes. ([ADR-04](06-decisiones.md#adr-04--rate-limit-con-upstash-redis-fail-open)) |
| 4. **Validación Zod en servidor** | Tipos, longitudes, formato de email/teléfono, fecha y franja válidas, servicio activo existente. | — (es la última barrera y no depende de terceros) |

---

## 3. Autenticación y autorización

**Autenticación** = "¿quién sos?" → **Supabase Auth** (email + contraseña).
- Sin registro público: el admin invita desde el panel (Supabase envía el link).
- La sesión se guarda en cookies `HttpOnly` (JavaScript del navegador no puede leerlas) manejadas por `@supabase/ssr`.
- En el servidor la identidad se valida contra Supabase (`getUser()`/`getClaims()`), **nunca** confiando solo en la cookie sin verificar.

**Autorización** = "¿qué podés hacer?" → **nuestro código**, con un mapa central: `src/lib/auth/permissions.ts`.

### Matriz de permisos

| Permiso | ADMIN | RECEPTION | VET |
|---|:-:|:-:|:-:|
| `request:read` (todas) | ✅ | ✅ | — |
| `request:read` (asignadas a mí) | ✅ | ✅ | ✅ |
| `request:changeStatus` | ✅ | ✅ | — |
| `request:assignVet` | ✅ | ✅ | — |
| `request:writeNote` | ✅ | ✅ | ✅ (solo asignadas) |
| `service:manage` | ✅ | — | — |
| `schedule:manage` | ✅ | — | — |
| `staff:manage` | ✅ | — | — |

Reglas:
- Ocultar un botón es **UX**, no seguridad. La verificación real siempre está en el servidor.
- Un VET que pide una solicitud no asignada recibe **404** (no 403), para no revelar que existe.
- Un admin no puede quitarse su propio rol ni desactivarse (evita quedarse sin administradores).

---

## 4. Cabeceras de seguridad

Configuradas en `next.config.ts`:

| Cabecera | Para qué |
|---|---|
| `Content-Security-Policy` | Lista blanca de orígenes de scripts, estilos, frames (Turnstile, Google Maps). `frame-ancestors 'none'`, `object-src 'none'`, `base-uri 'self'`. |
| `Strict-Transport-Security` | Obliga HTTPS. |
| `X-Content-Type-Options: nosniff` | El navegador no "adivina" tipos de archivo. |
| `Referrer-Policy: strict-origin-when-cross-origin` | No filtra URLs completas a otros sitios. |
| `Permissions-Policy` | Desactiva cámara, micrófono, geolocalización. |

> Compromiso documentado: las páginas públicas son estáticas, y una CSP con *nonce* (la más estricta) obliga a generarlas en cada pedido, lo que baja el rendimiento. Usamos CSP estricta en `/admin` y una CSP de base en lo público ([ADR-08](06-decisiones.md#adr-08--csp-diferenciada-público-vs-admin)).

---

## 5. Privacidad y retención (Ley 25.326)

- **Minimización:** solo pedimos lo necesario para coordinar el turno.
- **Consentimiento:** casilla obligatoria "Acepto la política de privacidad" con enlace; se guarda `privacyAcceptedAt`.
- **Finalidad:** los datos se usan solo para gestionar la solicitud. Nada de marketing.
- **Retención:** solicitudes en estado final se **borran a los 12 meses** (cron diario). Los IP hasheados en Redis vencen solos en ≤ 24 h.
- **Derechos ARCO** (acceso, rectificación, cancelación/supresión): la política indica a qué email escribir; el admin puede borrar una solicitud a pedido.
- **Logs:** nunca se loguean datos personales completos (ni email, ni teléfono). Se loguea el id de la solicitud.
- **Registro AAIP:** no se realiza por ser un proyecto de portfolio; con un cliente real correspondería evaluarlo (ver limitaciones en el README).

---

## 6. Gestión de secretos

| Secreto | Dónde vive | Visible en navegador |
|---|---|---|
| `DATABASE_URL`, `DIRECT_URL` | `.env.local` / Vercel | ❌ |
| `SUPABASE_SECRET_KEY` (solo para invitar usuarios) | `.env.local` / Vercel | ❌ |
| `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` | ídem | ✅ (diseñado para ser público; RLS lo vuelve inofensivo) |
| `RESEND_API_KEY`, `TURNSTILE_SECRET_KEY`, `UPSTASH_*`, `CRON_SECRET`, `IP_HASH_SALT` | ídem | ❌ |
| `NEXT_PUBLIC_TURNSTILE_SITE_KEY` | ídem | ✅ (es pública por diseño) |

Regla de oro: **solo** las variables con prefijo `NEXT_PUBLIC_` llegan al navegador. Un secreto nunca lleva ese prefijo. `src/lib/env.ts` valida todas al arrancar: si falta una, la app no arranca (mejor fallar en el deploy que en la cara del cliente).

Las **credenciales de prueba** del panel se comparten por chat con el evaluador; **nunca** en el repo.

---

## 7. Checklist de seguridad por Server Action

Antes de dar por terminada cualquier acción del servidor:

- [ ] Empieza con `import "server-only"` (o está en un archivo `"use server"` que lo garantiza).
- [ ] Verifica sesión y que el `StaffUser` esté activo (excepto la acción pública del formulario).
- [ ] Verifica el permiso con `can(...)`, incluyendo propiedad del recurso (VET → solo asignadas).
- [ ] Valida la entrada con Zod (nunca confía en datos del cliente, ni siquiera en ids).
- [ ] Registra en `RequestEvent` si modifica una solicitud.
- [ ] Devuelve errores genéricos al usuario; el detalle va al log sin datos personales.
- [ ] Tiene test del caso "sin permiso".
