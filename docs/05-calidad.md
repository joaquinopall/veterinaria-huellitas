# 05 · Calidad: rendimiento, SEO, accesibilidad, mantenibilidad y tests

> **Qué es este documento:** cómo cumplimos los requisitos no funcionales que no son seguridad, y cómo lo **verificamos**.

---

## 1. Rendimiento (RNF-03 · Lighthouse mobile ≥ 90)

Lighthouse simula un celular mediano con internet lenta. Lo que más lo afecta y qué hacemos:

| Factor | Medida |
|---|---|
| Tiempo de servidor | Páginas públicas **estáticas** + revalidación bajo demanda. |
| Imágenes | Siempre `next/image` (tamaño correcto, formato moderno, carga diferida). La imagen principal de la home con `priority`. |
| Fuentes | `next/font` (se descargan con el sitio, sin saltos de texto). Máximo 1 familia, 2 pesos. |
| JavaScript | Server Components por defecto; `"use client"` solo donde hay interacción (formulario, filtros). |
| Mapa de Google | **Nunca en la home.** En Contacto: imagen estática + botón "Ver mapa interactivo" que recién ahí carga el iframe (*facade*). |
| Turnstile | Solo se carga en `/turnos`. |
| Terceros | Nada de analytics pesados. Si se necesita, Vercel Analytics (liviano). |

**Verificación:** Lighthouse mobile sobre la URL de producción al cierre de cada etapa que toque lo público; puntajes anotados en el README.

---

## 2. SEO local (RNF-04)

Lo que Google necesita para entender que es **un negocio físico en Villa Urquiza**:

1. **Datos estructurados JSON-LD** tipo `VeterinaryCare` en todas las páginas públicas: nombre, dirección (`PostalAddress`), coordenadas (`GeoCoordinates`), teléfono, horarios (`OpeningHoursSpecification`, generados desde la base), URL, `areaServed: "Villa Urquiza"`, `priceRange`.
2. **Metadatos por página** (`title`, `description`, Open Graph) con el barrio en los textos importantes: "Veterinaria en Villa Urquiza | Huellitas".
3. **Un `<h1>` por página**, jerarquía de títulos correcta.
4. **NAP consistente**: nombre, dirección y teléfono salen de un único lugar (`Location`) → idénticos en la web, JSON-LD y Google Business Profile.
5. **`sitemap.xml` y `robots.txt`** generados por Next.js; `/admin` excluido.
6. **URL canónica** y `lang="es-AR"`.
7. **Fuera del código (se documenta para Laura):** crear y verificar la ficha de **Google Business Profile** (la palanca #1), pedir reseñas, cargar el sitio en Instagram.

**Verificación:** Lighthouse SEO + [Rich Results Test](https://search.google.com/test/rich-results) de Google sobre producción.

---

## 3. Accesibilidad (RNF-05)

| Área | Medida |
|---|---|
| Teclado | Todo se puede usar con Tab/Shift+Tab/Enter/Espacio/Esc; foco visible siempre; **"Saltar al contenido"** como primer elemento. |
| Lector de pantalla | HTML semántico (`header`, `nav`, `main`, `footer`), `alt` en imágenes, `aria-label` en botones con solo ícono. |
| Formulario | Cada campo con `<label>`; errores asociados con `aria-describedby`; al enviar con errores, el foco va al primer error y un resumen se anuncia (`role="alert"`). Campos obligatorios marcados en texto, no solo con color. |
| Contraste | ≥ 4.5:1 en texto. Paleta verificada. |
| Movimiento | Respetar `prefers-reduced-motion`. |
| Componentes | shadcn/ui está construido sobre Radix, que ya resuelve teclado y ARIA en menús, diálogos y selects. |

**Verificación:** Lighthouse Accessibility + `@axe-core/playwright` en tests E2E + prueba manual con teclado y con NVDA (lector de pantalla gratuito de Windows) en el formulario.

---

## 4. Responsive (RNF-06)

- *Mobile-first*: los estilos base son para celular; `md:`/`lg:` agregan para pantallas grandes.
- Objetivos táctiles ≥ 44×44 px.
- El panel también debe ser usable en celular (la recepcionista puede revisarlo desde el teléfono): la tabla de solicitudes se convierte en tarjetas en pantallas chicas.
- Probado en 360 px, 768 px y 1280 px.

---

## 5. Mantenibilidad (RNF-07)

| Práctica | Para qué |
|---|---|
| TypeScript `strict`, sin `any` | El editor detecta errores antes de ejecutar. |
| Zod en todos los bordes (formularios, env, params) | Un único esquema = validación + tipos. |
| Estructura por funcionalidad (`src/features/`) | Para cambiar "servicios" se mira una sola carpeta. |
| Lógica de negocio pura y testeable (`status.ts`, `permissions.ts`, `schedule/availability.ts`) | Las reglas importantes no dependen de React ni de la base. |
| ESLint + Prettier | Estilo uniforme sin discutirlo. |
| Conventional Commits | Historial legible: `feat(turnos): ...`, `fix(admin): ...`. |
| ADRs ([06-decisiones.md](06-decisiones.md)) | El "por qué" de cada decisión no se pierde. |
| CI en GitHub Actions | Nada roto llega a `main`. |

---

## 6. Estrategia de tests

No testeamos todo: testeamos **lo que más duele si se rompe**.

| Nivel | Herramienta | Qué cubre | Prioridad |
|---|---|---|---|
| Unit | Vitest | `canTransition` (máquina de estados) · `can()` (permisos, incl. VET solo asignadas) · validación de fecha/franja según horarios y días especiales · esquema Zod del formulario · normalización de teléfono | **Alta** |
| Integración | Vitest + mocks | Server Action del formulario: honeypot descarta, Turnstile inválido rechaza, rate limit rechaza, éxito guarda y llama email, falla de email no pierde la solicitud | **Alta** |
| E2E | Playwright | Enviar solicitud de punta a punta · login y cambio de estado · VET no ve solicitudes ajenas · navegación con teclado + axe | Media |

**CI (`.github/workflows/ci.yml`)** en cada push y PR: `lint` → `typecheck` → `test` → `build`.

---

## 7. Definición de "Terminado" (Definition of Done)

Una tarea está terminada cuando:

1. Cumple los criterios de aceptación de sus requisitos (`RF-xx`/`RNF-xx`).
2. `npm run lint`, `npm run typecheck`, `npm test` y `npm run build` pasan.
3. Si toca un Server Action: cumple el [checklist de seguridad](04-seguridad.md#7-checklist-de-seguridad-por-server-action).
4. Si toca UI pública: probada con teclado y en 360 px.
5. Si tomó una decisión nueva: está en [06-decisiones.md](06-decisiones.md).
6. Tiene una explicación en lenguaje simple de qué se hizo y por qué.
