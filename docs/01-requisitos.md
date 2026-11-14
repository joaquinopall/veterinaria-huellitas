# 01 · Requisitos

> **Qué es este documento:** la lista de lo que el sistema TIENE que hacer y cómo se tiene que comportar.
> Cada requisito tiene un **ID** (ej. `RF-07`) para poder referenciarlo desde el código, los tests y los commits.
> Si algo no está acá, **no se construye** sin antes agregarlo y justificarlo.

---

## 1. Contexto del cliente

- **Cliente:** Veterinaria Huellitas (ficticia), Villa Urquiza, CABA, Argentina.
- **Dueña:** Laura (no técnica, presupuesto bajo).
- **Equipo:** 2 veterinarios + 1 recepcionista.
- **Hoy:** WhatsApp + Instagram.
- **Dirección de demo:** Av. Olazábal 4850, C1431 CABA (calle y altura reales, negocio ficticio).
- **Horario de demo:** Lun a Vie 9–13 y 16–20 · Sáb 9–13 · Dom cerrado.
- **Futuro posible:** segunda sucursal.

> ⚠️ Como es un negocio ficticio en una dirección real, el sitio muestra en el pie un aviso
> "Sitio de demostración — Veterinaria Huellitas es un negocio ficticio" y usa un teléfono de ejemplo.

---

## 2. Requisitos funcionales (RF)

### Sitio público

| ID | Requisito | Criterio de aceptación (cómo sé que está hecho) |
|---|---|---|
| RF-01 | **Home** con propuesta de valor, servicios destacados y llamado a la acción "Pedir turno". | Se ve el CTA sin hacer scroll en un celular; los servicios destacados salen de la base de datos (`featured = true`). |
| RF-02 | **Servicios** con nombre, descripción y precio orientativo. | Lista los servicios activos ordenados; precio con formato `$ 12.000` o "Consultar" si no tiene; aviso "precios orientativos". |
| RF-03 | **Nosotros** con el equipo. | Muestra foto, nombre y rol de cada integrante. |
| RF-04 | **Contacto** con dirección, mapa, horarios y botón a WhatsApp. | Horarios salen de la base de datos; el mapa carga solo al interactuar o de forma diferida; el botón abre WhatsApp con un mensaje precargado. |
| RF-05 | **Formulario de solicitud de turno** con: nombre del dueño, teléfono, email, nombre de la mascota, especie, servicio, fecha preferida, franja horaria, comentario (opcional) y aceptación de la política de privacidad. | Todos los campos se validan en el navegador **y** en el servidor. |
| RF-06 | Al enviar: el cliente ve una **confirmación** y la veterinaria recibe un **email**. | La solicitud se guarda SIEMPRE primero en la base; si el email falla, la solicitud igual existe y se registra el error. |
| RF-07 | Solo se pueden elegir **fechas válidas**: desde hoy hasta 60 días, días en que la sucursal abre y franjas que existen ese día. | Validado en servidor con la zona horaria `America/Argentina/Buenos_Aires`. |
| RF-08 | **Política de privacidad** mínima de ejemplo (Ley 25.326). | Página `/privacidad` enlazada desde el formulario y el pie. |

### Panel de administración

| ID | Requisito | Criterio de aceptación |
|---|---|---|
| RF-10 | **Login** solo para personal autorizado. No hay registro público. | Los usuarios se crean por invitación de un admin. |
| RF-11 | **Lista de solicitudes** filtrable por estado y por rango de fechas. | Filtros reflejados en la URL (se pueden compartir/recargar). Paginada. |
| RF-12 | **Cambiar el estado** de una solicitud. | Solo transiciones válidas (ver [03-modelo-de-datos.md](03-modelo-de-datos.md#3-máquina-de-estados)). Cada cambio queda en el historial. |
| RF-13 | **Asignar veterinario** a una solicitud. | Recepción y admin pueden asignar/reasignar. |
| RF-14 | **Nota interna** en una solicitud. | El veterinario asignado, recepción y admin pueden escribirla; nunca se muestra al público. |
| RF-15 | **Historial** de cada solicitud (quién hizo qué y cuándo). | Visible en el detalle de la solicitud. |
| RF-16 | **Editar servicios y precios** (opcional del enunciado — incluido). | Crear, editar, ordenar, destacar y desactivar. Los cambios se ven en el sitio público en segundos. |
| RF-17 | **Editar horarios** y **días especiales** (feriados/cierres). | Impacta en Contacto, en los datos para Google y en las fechas válidas del formulario. |
| RF-18 | **Gestión de usuarios**: invitar, cambiar rol, desactivar. | Solo admin. Un admin no puede desactivarse a sí mismo. |

### Roles y permisos (resumen — detalle en [04-seguridad.md](04-seguridad.md#matriz-de-permisos))

| Rol | Puede |
|---|---|
| **ADMIN** (Laura) | Todo. |
| **RECEPTION** (recepcionista) | Ver todas las solicitudes, cambiar estado, asignar veterinario, escribir notas. |
| **VET** (veterinarios) | Ver **solo** las solicitudes asignadas a él/ella y escribir notas internas. |

---

## 3. Requisitos no funcionales (RNF)

| ID | Categoría | Requisito medible |
|---|---|---|
| RNF-01 | Seguridad | Formulario protegido contra spam/abuso con 4 capas (honeypot, Turnstile, rate limit, validación en servidor). |
| RNF-02 | Seguridad | Datos personales solo accesibles por usuarios autenticados y con permiso; API pública de Supabase bloqueada (RLS activo sin políticas). |
| RNF-03 | Rendimiento | Lighthouse **mobile ≥ 90** en Performance, Accessibility, Best Practices y SEO en la home. |
| RNF-04 | SEO local | Datos estructurados `VeterinaryCare`, metadatos por página, sitemap, robots, NAP (nombre-dirección-teléfono) idéntico en todo el sitio. |
| RNF-05 | Accesibilidad | WCAG 2.2 AA en lo razonable: navegable 100 % con teclado, usable con lector de pantalla, contraste suficiente. |
| RNF-06 | Responsive | Diseño *mobile-first*; usable desde 360 px de ancho. |
| RNF-07 | Mantenibilidad | TypeScript estricto, estructura por funcionalidad, documentación de decisiones, tests de lo crítico, CI en cada push. |
| RNF-08 | Escalabilidad | Modelo de datos multi-sucursal desde el día 1 (todo cuelga de `Location`). |
| RNF-09 | Costo | Todo en planes gratuitos. Único costo opcional: dominio `.com.ar`. |
| RNF-10 | Disponibilidad | La base no se pausa por inactividad (cron diario); monitoreo de caída externo. |
| RNF-11 | Privacidad | Mínimo de datos necesarios; consentimiento explícito; retención limitada (ver [04-seguridad.md](04-seguridad.md#5-privacidad-y-retención-ley-25326)). |

---

## 4. Criterios implícitos detectados (no estaban en el enunciado)

Estos son los "criterios que un profesional consideraría igual". Cada uno está cubierto en algún documento:

1. Bloqueo de la API pública de Supabase (trampa Prisma + RLS) → [04-seguridad](04-seguridad.md).
2. Ley 25.326: política de privacidad, consentimiento, retención → [04-seguridad](04-seguridad.md).
3. Supabase gratis se pausa tras 7 días sin actividad → cron diario ([02-arquitectura](02-arquitectura.md)).
4. Backups propios (el plan gratis no da copias descargables) → [07-plan](07-plan-de-trabajo.md), etapa 8.
5. El email no es la fuente de verdad; la base sí → RF-06.
6. Sin dominio propio, Resend solo envía a la casilla del dueño de la cuenta → limitación documentada.
7. Entornos separados: desarrollo y producción con bases distintas.
8. Zona horaria y validación de fechas → RF-07.
9. Un *Server Action* es un endpoint público: se verifica sesión y permiso **dentro** de cada uno.
10. Auditoría de cambios → RF-15.
11. Las cuentas (Vercel, Supabase, dominio) deberían quedar a nombre del cliente → README, sección "Entrega al cliente".
12. Manual de uso para la recepcionista → etapa 9.
13. Aviso de sitio ficticio por usar dirección real.
14. "Solicitud" ≠ "turno confirmado": el texto de la UI nunca promete un turno.
15. Gestión de la ficha de **Google Business Profile** (no es código, pero es la palanca #1 de SEO local) → README.

---

## 5. Fuera de alcance (Fase 2 — se cotizaría aparte)

| Idea | Por qué queda afuera |
|---|---|
| Ingresos / facturación | El sistema no registra cobros; precio orientativo ≠ precio cobrado. Es otro módulo. |
| Historia clínica de mascotas | Otro sistema, datos más sensibles, otro nivel de responsabilidad. |
| Mensajería interna | Lo resuelven en persona/WhatsApp; la nota interna (RF-14) cubre lo mínimo. |
| Agenda en tiempo real con cupos | El enunciado pide *solicitudes*, no reservas. Implicaría gestión de disponibilidad por profesional. |
| Email de confirmación al cliente | Requiere dominio verificado. Se activa al comprar el dominio. |
| Recordatorios por WhatsApp/SMS | Servicios pagos. |
| Segunda sucursal (UI) | El modelo la soporta; la UI para elegir sucursal se agrega cuando exista. |
| Edición de "Nosotros" desde el panel | No pedido; el equipo cambia muy poco. Vive en un archivo de contenido. |
