# Changelog v4.2 → v4.3

Sesión de actualización: 1 de mayo de 2026

Origen: lecciones extraídas de la práctica del módulo 9 (AI4Devs) — endpoints LTI con Windsurf/Cascade.

---

## Cambios

### 1. Nuevo skill: `prompts/skills/prompt-skill.md`

**Estado:** archivo nuevo.

Manual operativo para el **usuario** sobre cómo redactar prompts profesionales siguiendo el patrón **RACEO** (Role + Objective + Context + Constraints + Expected Output). Incluye:

- Por qué cada bloque del patrón.
- Plantilla base copy-paste.
- 3 ejemplos de prompts completos (onboarding, planificación, implementación paso a paso).
- 5 antipatrones del usuario.
- Cuándo NO usar RACEO (tareas triviales).

Origen: el patrón observado en las PRs destacadas del módulo 8 (Arnau, Alberto, Fran).

### 2. Workflow nuevo: `.windsurf/workflows/onboarding.md`

**Estado:** archivo nuevo.

Comando `/onboarding` para entrar a un codebase ajeno sin crear docs ni adoptar el proyecto a largo plazo. Diferencia explícita con `/regularizar`:

- `/regularizar` → adopción del proyecto, blueprints, ADRs, plan de cobertura.
- `/onboarding` → mapa puntual en 6 secciones: stack, capas, traza de un endpoint de referencia, modelo de datos, tests, convenciones. Cierra con TODOs verificables por el usuario.

Detecta automáticamente antipatrones existentes (controllers huérfanos, rutas que bypasean capas) y los reporta — esto condiciona la planificación posterior.

### 3. Modificado: `prompts/skills/tests-skill.md`

**Estado:** sección de mock de Prisma reemplazada.

El ejemplo anterior usaba `mockImplementation(() => ({...}))` que **falla con código de producción Active Record** (cuando cada modelo de dominio instancia su propio `PrismaClient`). El ejemplo nuevo usa `jest.fn(() => mPrisma)` que garantiza que todas las invocaciones a `new PrismaClient()` devuelven la misma instancia, y por tanto el mock alcanza al prisma interno del modelo.

Se añade la regla: **ante la duda, usar el Patrón A** (con `mockReturnValue` / `jest.fn(() => obj)`). Funciona tanto con repositorios separados como con Active Record.

Razón del cambio: durante la implementación del módulo 9 se detectó que el patrón anterior no permitía mockear Prisma cuando `Position.ts` instanciaba su propio cliente (Active Record).

### 4. Modificado: `prompts/architect-brain.md`

**Estado:** versión actualizada de v4.2 a v4.3 + nuevo protocolo.

Añadido `On(implementation_phase)` justo antes de `On(task_complete)`. Este protocolo se activa cuando un ticket pasa de planificación a implementación con plan previo. Exige:

- Anunciar el plan de pasos al usuario antes de tocar nada.
- Detenerse **tras cada paso** (1 archivo = 1 paso) y esperar confirmación explícita "ok, sigue".
- Marcador visible al final de cada paso: `Paso K/N completado. Esperando "ok, sigue".`
- Si encuentra un hueco no previsto, **parar y preguntar**, no improvisar.
- Paso final siempre es `tests + ejecución de la suite` con output completo.

Aplicabilidad: si el plan implica más de 2 archivos, aplica obligatoriamente. Bugfixes triviales no aplican.

### 5. Modificado: `.windsurfrules`

**Estado:** dos filas nuevas en la tabla de comandos del usuario + dos reglas de oro nuevas.

- Disparador para `/onboarding` ("dame un mapa rapido", "no conozco este codigo").
- Disparador para `On(implementation_phase)` ("implementa el plan", "vamos al codigo").
- Regla de oro **Implementacion paso a paso**: si el plan toca >2 archivos, aplica el protocolo.
- Regla de oro **Prompts profesionales**: el usuario consulta `prompt-skill.md` para tareas complejas.

### 6. Modificado: `README.md` e `inicio-chat.txt`

**Estado:** tablas de comandos actualizadas con `/onboarding` (11 comandos en total, antes 10).

### 7. Modificado: `docs/architecture.md`

**Estado:** reescrito como meta-documentación del scaffolding v4.3.

Antes era un documento mixto (mitad meta-doc, mitad plantilla). Ahora describe explícitamente la arquitectura del propio sistema — las 4 capas (Intent / Implementation / Memory / Lifecycle), los 9 protocolos centrales (incluyendo `On(implementation_phase)` nuevo), las garantías del sistema, y la distinción entre **skills estáticas** (las que vienen) y **skills dinámicas** (las que crea Cascade durante el proyecto).

Añade el estado `[ONBOARDING]` y formaliza `[IMPLEMENTING]` en el ciclo de vida.

### 8. Modificado: `docs/testing-strategy.md`

**Estado:** sección "Patrón de mock para BD" ampliada con dos snippets de referencia (Prisma+Jest con el patrón Active-Record-friendly, y PDO+PHPUnit). El placeholder original del proyecto se mantiene; los snippets aparecen como referencia bajo el placeholder.

Razón: que los proyectos que usen el sistema tengan ya el patrón correcto a la vista cuando rellenen su propia testing-strategy.

### 9. Modificado: `docs/decisions-log.md`

**Estado:** sección nueva "Decisiones de comportamiento ad-hoc" justo antes de la plantilla ADR-000.

Documenta el patrón aprendido: cuando una feature compleja tiene preguntas semánticas no cubiertas por el enunciado, se etiquetan como C1/C2/C3 durante la planificación, se confirman con el usuario antes de implementar y se registran como ADR. Esto evita el clásico "porque la IA lo decidió así" en code reviews.

---

## Migración

Para integrar v4.3 en tu repo local:

1. Reemplaza la carpeta entera `junior-doc-gen-tests/` con la del ZIP.
2. O bien, reemplaza solo estos archivos individualmente:
   - `prompts/skills/prompt-skill.md` (nuevo)
   - `.windsurf/workflows/onboarding.md` (nuevo)
   - `prompts/skills/tests-skill.md` (modificado)
   - `prompts/architect-brain.md` (modificado)
   - `.windsurfrules` (modificado)
   - `README.md` (modificado)
   - `inicio-chat.txt` (modificado)

No hay cambios destructivos. Los workflows existentes (`/inicio`, `/regularizar`, `/cuestionar`, etc.) siguen funcionando igual. El sistema es retrocompatible.

---

## Verificación post-migración

Para confirmar que todo está bien:

1. Abre Windsurf en un proyecto cualquiera con esta estructura.
2. En Cascade escribe `/` — deberían aparecer **11 comandos** en el desplegable, incluido `/onboarding`.
3. Pide a Cascade: "Lee `@prompts/skills/prompt-skill.md` y dime qué cubre."
   - Si te resume RACEO + ejemplos → integrado correctamente.
4. Pide: "Si te paso un plan de implementación de 5 archivos, ¿qué protocolo activas?"
   - Debería responder: `On(implementation_phase)` con paradas obligatorias entre pasos.
