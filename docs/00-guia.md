# 00 · Guía del proyecto (para Joaquín)

> Este es **tu** punto de entrada. El README es para quien evalúa o clona el repo; este documento es para que vos sepas dónde está cada cosa y cómo trabajar con Claude Code sin perderte.

---

## Mapa de documentos

| Documento | Respondé con él… |
|---|---|
| [01-requisitos](01-requisitos.md) | ¿Qué tiene que hacer el sistema? ¿Esto está en el alcance? |
| [02-arquitectura](02-arquitectura.md) | ¿Qué piezas hay y dónde va cada archivo? |
| [03-modelo-de-datos](03-modelo-de-datos.md) | ¿Qué se guarda y cómo se relaciona? |
| [04-seguridad](04-seguridad.md) | ¿Contra qué nos defendemos? ¿Quién puede hacer qué? |
| [05-calidad](05-calidad.md) | ¿Cómo cumplimos rendimiento, SEO y accesibilidad? ¿Cuándo algo está "terminado"? |
| [06-decisiones](06-decisiones.md) | ¿Por qué elegimos X y no Y? |
| [07-plan-de-trabajo](07-plan-de-trabajo.md) | ¿En qué etapa estamos? ¿Qué sigue? |
| [glosario](glosario.md) | ¿Qué significa este término? |

## Cómo trabajar cada tarea con Claude Code

1. **Pedí la tarea por su nombre en el plan**: "Hagamos la etapa 2, tarea 'migración RLS'".
2. Claude lee las specs relevantes, te dice **qué va a hacer y por qué** antes de hacerlo, y te consulta si hay una decisión que es tuya.
3. Claude implementa en una rama, corre las verificaciones (lint, tipos, tests, build).
4. Claude te explica en lenguaje simple: **qué hizo, por qué así, qué alternativa descartó y cómo podés comprobarlo vos**.
5. **Vos revisás** (leé el diff, probalo) y recién ahí se hace el commit.
6. Se marca la tarea en el plan.

## Cómo verificar el trabajo (sin depender de la IA)

- **Contra las specs:** cada tarea cita IDs (`RF-07`, `RNF-03`). Abrí el requisito y comprobá su criterio de aceptación.
- **Automático:** el CI de GitHub (✅/❌ en cada PR) corre lint, tipos, tests y build.
- **Revisión asistida:** `/code-review` para bugs y `/security-review` para seguridad, al cerrar las etapas 4, 5, 6 y 8.
- **Manual:** Lighthouse (DevTools de Chrome → Lighthouse → Mobile), teclado (navegá solo con Tab), celular real.

## Si algo cambia

- ¿Querés agregar una funcionalidad? Primero se agrega a `01-requisitos` (o a "Fuera de alcance") **y después** se construye.
- ¿Cambia una decisión? Se agrega un ADR nuevo que reemplaza al anterior (no se borra el viejo: el historial también enseña).

## Preguntas que te conviene poder responder al final (practicá para la evaluación)

1. ¿Por qué la clave pública de Supabase puede estar en el navegador sin riesgo?
2. ¿Qué pasa si Upstash se cae? ¿Y si se cae Resend?
3. ¿Por qué un Server Action necesita verificar permisos aunque `/admin` ya esté protegido?
4. ¿Cómo agregarías la segunda sucursal?
5. ¿Por qué las páginas públicas son estáticas y el panel no?
6. ¿Qué hace Google con el JSON-LD?
7. ¿Qué datos personales guardás, por cuánto tiempo y por qué?
