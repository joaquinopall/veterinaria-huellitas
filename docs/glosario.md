# Glosario

> Términos técnicos que aparecen en el proyecto, explicados sin vueltas. Si aparece uno nuevo, se agrega acá.

## Términos del negocio (español ↔ código)

| En la UI / docs | En el código |
|---|---|
| Sucursal | `Location` |
| Servicio | `Service` |
| Franja horaria (mañana / tarde) | `TimeSlot` (`MORNING` / `AFTERNOON`) |
| Horario semanal | `ScheduleSlot` |
| Día especial / feriado | `SpecialDay` |
| Solicitud de turno | `AppointmentRequest` |
| Estado: pendiente / confirmado / cancelado / atendido | `PENDING` / `CONFIRMED` / `CANCELLED` / `ATTENDED` |
| Historial | `RequestEvent` |
| Usuario del panel | `StaffUser` |
| Rol: administradora / recepción / veterinario | `ADMIN` / `RECEPTION` / `VET` |
| Especie: perro / gato / ave / roedor / reptil / otro | `DOG` / `CAT` / `BIRD` / `RODENT` / `REPTILE` / `OTHER` |

## Términos técnicos

| Término | Qué significa |
|---|---|
| **ADR** | *Architecture Decision Record*. Nota corta que registra una decisión técnica y su porqué. |
| **App Router** | Forma de organizar Next.js donde cada carpeta dentro de `src/app` es una URL. |
| **Auditoría** | Registro de quién hizo qué y cuándo. |
| **Autenticación vs. autorización** | Autenticación: "¿quién sos?" (login). Autorización: "¿qué podés hacer?" (permisos). |
| **Borrado lógico** | Marcar algo como inactivo en vez de borrarlo, para no romper lo que depende de eso. |
| **CI** | *Integración continua*. GitHub revisa automáticamente el código en cada subida. |
| **Client Component** | Componente de React que corre en el navegador (lleva `"use client"`). Solo cuando hay interacción. |
| **Cron** | Tarea que se ejecuta sola cada cierto tiempo (ej. una vez por día). |
| **CSP** | *Content Security Policy*. Lista de orígenes desde donde el navegador puede cargar cosas; frena inyecciones de código. |
| **CTA** | *Call to action*. El botón principal que querés que la persona toque ("Pedir turno"). |
| **Deploy** | Publicar una versión del sitio. |
| **E2E** | *End to end*. Test que usa el sitio como una persona, en un navegador real. |
| **Facade** | Mostrar una imagen liviana en lugar de algo pesado (el mapa) hasta que el usuario lo pida. |
| **Fail-open / fail-closed** | Si una protección se cae: *open* = dejar pasar; *closed* = bloquear. |
| **Hash** | Transformar un dato en un código irreversible. Sirve para comparar sin guardar el original. |
| **Honeypot** | Campo trampa invisible que solo completan los bots. |
| **JSON-LD / datos estructurados** | Bloque de información en un formato que Google lee para entender el negocio (dirección, horarios…). |
| **Lighthouse** | Herramienta de Google que puntúa rendimiento, accesibilidad, buenas prácticas y SEO. |
| **Middleware / proxy** | Código que corre antes de cada pedido; acá lo usamos solo para redirigir al login. |
| **Migración** | Archivo que describe un cambio en la estructura de la base. Se aplican en orden y quedan en Git. |
| **Mínimo privilegio** | Cada persona accede solo a lo que necesita para su trabajo. |
| **NAP** | *Name, Address, Phone*. Deben ser idénticos en todos lados para el SEO local. |
| **ORM** | Herramienta (Prisma) que permite usar la base de datos con código en lugar de SQL a mano. |
| **Pooler** | Intermediario que reutiliza conexiones a la base; necesario en serverless, donde se crean muchas instancias. |
| **Rate limit** | Límite de cuántas veces se puede hacer algo en un período. |
| **Revalidación** | Pedirle a Next.js que regenere una página estática porque sus datos cambiaron. |
| **RLS** | *Row Level Security*. Reglas en la base sobre quién puede ver cada fila. |
| **Scope creep** | Cuando el alcance de un proyecto crece sin control. |
| **Seed** | Script que carga datos iniciales de ejemplo. |
| **Server Action** | Función que corre en el servidor y se llama desde un formulario. Es un endpoint público: hay que protegerla. |
| **Server Component** | Componente de React que se arma en el servidor; no manda JavaScript al navegador. |
| **Serverless** | Modelo donde el código corre "a pedido" en instancias efímeras, sin un servidor fijo. |
| **SSR / estático** | Generar la página en cada visita (SSR) vs. generarla una vez y servirla armada (estático). |
| **TTL** | *Time to live*. Tiempo tras el cual un dato se borra solo. |
| **Turnstile** | CAPTCHA de Cloudflare, casi invisible para humanos. |
| **Variables de entorno** | Configuración y secretos que viven fuera del código (`.env.local`, Vercel). |
| **XSS** | Ataque que inyecta código en una página para que se ejecute en el navegador de otra persona. |
| **Zod** | Librería que define la "forma" válida de un dato y la verifica. |
