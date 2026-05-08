# junior-doc-gen-tests

Sistema de scaffolding para [Windsurf](https://windsurf.com) que convierte a Cascade en un arquitecto senior disciplinado. Mantiene **documentación**, **código**, **tests**, **decisiones**, **base de datos** y **frontend** (accesibilidad + Core Web Vitals + seguridad) sincronizados durante todo el ciclo de vida del proyecto, evitando el clásico *documentation drift*.

---

## Qué resuelve

Cuando trabajas con una IA generando código día tras día, aparecen siete problemas silenciosos:

1. **Los docs se quedan atrás del código** — nadie los actualiza porque no duele inmediatamente.
2. **Las decisiones técnicas se olvidan** — ¿por qué elegimos esta librería? Nadie se acuerda tres meses después.
3. **Los tests son un "luego" eterno** — se añaden al final, si sobra tiempo, con calidad baja.
4. **El código legacy se toca sin red de seguridad** — cambiar primero y rezar después es receta para bugs nuevos.
5. **Cobertura ≠ calidad** — tests que pasan no garantizan detectar bugs reales.
6. **Las bases de datos crecen sin auditoría** — índices faltantes, datos personales sin proteger, queries lentas que nadie detecta.
7. **El frontend acumula deuda invisible** — accesibilidad rota, Core Web Vitals en rojo, secretos expuestos en variables públicas, código IA sin revisar.

Este sistema impone protocolos que obligan a la IA a resolver los siete problemas por diseño.

---

## Cómo se usa en 30 segundos

1. Copia esta estructura (`.windsurf/`, `.windsurfrules`, `AGENTS.md`, `docs/`, `prompts/`, `inicio-chat.txt`) en la raíz de tu proyecto.
2. Abre Windsurf apuntando a la raíz del proyecto.
3. En Cascade, escribe `/` y te aparecerá un menú con los comandos disponibles.
4. Elige el comando que toque según la situación.

---

## Slash commands disponibles (12)

| Comando             | Cuándo usarlo                                                                 |
|---------------------|-------------------------------------------------------------------------------|
| `/inicio`           | Proyecto vacío. Entrevista BMADT + docs + setup tests + setup BD si aplica + setup FRONTEND si aplica. |
| `/onboarding`       | Codebase ajeno (ejercicio bootcamp, PR de un compañero, debug urgente). Mapa rápido en 6 secciones, sin generar docs. |
| `/regularizar`      | Proyecto con código pero sin docs. Menú [1][2][3][4]. La opción [4] incluye auditoría de BD y frontend. |
| `/hoy`              | "¿Qué hacemos hoy?" Briefing con sugerencia concreta para el día.             |
| `/nueva`            | Añadir funcionalidad. Activa la interrupción de seguridad antes de tocar código. |
| `/reparar`          | Corregir un bug. Si no está cubierto, primero escribe el test que lo reproduce. |
| `/decidir`          | Cambiar una decisión pasada. Propaga cambios sin perder histórico.            |
| `/sync`             | Detectar drift entre código y documentación. Solo reporta.                    |
| `/cerrar`           | Forzar checklist de sincronización si sospechas que se saltó.                 |
| `/cuestionar`       | Auditar la calidad real de los tests con mutation thinking.                   |
| `/revisar-bd`       | Auditar la BD: modelo, seguridad, rendimiento, calidad de datos.              |
| `/revisar-frontend` | Auditar el frontend: stack, seguridad, Core Web Vitals, accesibilidad WCAG 2.2 AA. *(nuevo en v4.4)* |

---

## Qué hace el sistema (los 8 pilares)

### 1. Entrevista inicial estructurada (BMADT)

Al arrancar un proyecto nuevo con `/inicio`, la IA te hace 5 preguntas:

- **B (Beneficio):** objetivo y valor del proyecto.
- **M (Mecanismo):** stack técnico e integraciones.
- **A (Alcance):** módulos del MVP.
- **D (Datos):** entidades y flujos críticos.
- **T (Testing):** nivel de cobertura deseado (básico/estándar/exhaustivo).

Con las respuestas genera automáticamente los docs del SSOT en `/docs/` y **no escribe código** hasta que apruebes.

### 2. Setup automático del entorno de tests

Según el stack detectado, instala el framework adecuado con su soporte de linter (PHPUnit + phpstan-phpunit, Jest + @types/jest, Vitest, pytest, etc.). Si hay frontend con nivel T 2 o 3, añade además RTL + vitest-axe + Playwright + @axe-core/playwright.

### 3. Setup automático de base de datos *(v4.2)*

Si la respuesta M o D del BMADT implica BD, el sistema dispara automáticamente `On(database_setup)`: detecta motor + ORM, identifica campos PII, genera `database-strategy.md`, configura auditoría futura, prepara Testcontainers, registra ADRs.

### 4. Setup automático de frontend *(nuevo en v4.4)*

Si la respuesta M implica frontend (React, Vue, Svelte, Next, Nuxt, Astro, etc.), dispara automáticamente `On(frontend_setup)`:

- **Propone el stack canónico 2026**: Next.js 15/16 + TypeScript estricto + Tailwind v4 + shadcn/ui + Biome + Zod. Cualquier desviación se registra como ADR.
- **Pregunta por el flujo diseño-a-código** (Figma con Dev Mode, capturas, sin diseño) y **recomienda los MCP servers** apropiados (Figma Dev Mode MCP, shadcn MCP, Chrome DevTools MCP, Playwright MCP).
- **Configura headers de seguridad** (CSP, HSTS, X-Frame-Options, Referrer-Policy).
- **Establece reglas operativas** que entran en el ADR de stack: cero `any`, componentes <150 líneas, WCAG 2.2 AA como mínimo legal, Core Web Vitals como Definition of Done, validación cliente Y servidor con Zod, cero secretos en `NEXT_PUBLIC_*`/`VITE_*`/`PUBLIC_*`.
- **Genera `frontend-strategy.md`** con todas las decisiones documentadas.
- **Registra ADRs** por cada decisión (stack, a11y, MCP, RUM).

### 5. Trazabilidad de decisiones (ADRs)

Todas las decisiones no triviales se registran en `decisions-log.md` como **Architecture Decision Records** inmutables: si cambias de opinión, no se borra lo anterior — se añade una nueva ADR y la antigua queda marcada como `Superseded por ADR-XXX`.

### 6. Antidrift documental

Al cerrar **cualquier tarea** de código, la IA muestra un checklist visible marcando cada uno de los docs del SSOT (incluido `frontend-strategy.md` cuando aplica) como actualizado o sin cambios con justificación. Una tarea no está cerrada si el checklist no aparece.

### 7. Auditorías de calidad (cuatro niveles)

- **`/sync`** — detecta drift entre código y docs.
- **`/cuestionar`** — audita calidad de los tests con mutation thinking.
- **`/revisar-bd`** *(v4.2)* — audita la BD en 4 categorías (modelo, seguridad, rendimiento, calidad de datos).
- **`/revisar-frontend`** *(nuevo en v4.4)* — audita el frontend en 4 categorías:
  - **Stack y mantenibilidad**: TS estricto, `any`, componentes gigantes, dependencias, mezcla de UI libs.
  - **Seguridad**: secretos en cliente, `dangerouslySetInnerHTML` sin sanitizar, validación solo en cliente, headers (CSP, HSTS), CORS, `npm audit`, código IA sin revisión.
  - **Rendimiento (Core Web Vitals)**: LCP, INP, CLS reales (CrUX) si hay despliegue; análisis estático si no.
  - **Accesibilidad WCAG 2.2 AA**: alt, labels, contraste, teclado, landmarks, headings, focus visible, `prefers-reduced-motion`.

Filosofía común: **la IA detecta, el humano decide, el humano repara**. Las auditorías nunca modifican código ni datos automáticamente.

### 8. Retomar proyectos tras pausas

`/hoy` lee los docs clave (incluido `frontend-strategy.md` si existe), ejecuta un mini health-check y devuelve un briefing de 10 líneas con sugerencia concreta para el día.

---

## Stack canónico para frontend nuevo (2026)

Salvo desviación documentada en ADR:

| Capa | Recomendación | Razón |
|---|---|---|
| Framework | Next.js 15 / 16 (App Router, RSC, Server Actions) | El que mejor genera la IA; ecosistema sólido |
| Lenguaje | TypeScript estricto (`strict`, `noUncheckedIndexedAccess`) | Sin tipos, la IA degrada |
| Estilado | Tailwind v4 (CSS-first, container queries en core) | Convergencia 2026 |
| Componentes | shadcn/ui (Radix + Tailwind, copy-in) | Accesibles por construcción |
| Forms y validación | react-hook-form + Zod (schemas compartidos front/back) | Validación en cliente Y servidor |
| Linter | Biome v2 (todo-en-uno) | ~35× más rápido que ESLint+Prettier |
| Testing | Vitest + RTL + vitest-axe + Playwright + @axe-core/playwright | Cubre los 3 niveles |
| RUM | Vercel Speed Insights / Cloudflare Web Analytics / Sentry | CWV reales, no de laboratorio |

---

## MCP servers recomendados

| MCP server | Cuándo activarlo | Qué da |
|---|---|---|
| **Figma Dev Mode MCP** | Diseño en Figma con Dev Mode | tokens, jerarquía y component mappings estructurados (no captura) |
| **shadcn MCP** | Stack incluye shadcn/ui | búsqueda e instalación de >6.000 bloques |
| **Chrome DevTools MCP** | Auditoría runtime | DOM/CSS/performance/networking (oficial Google) |
| **Playwright MCP** | Validación visual y E2E | accessibility tree determinista (sin modelo de visión) |
| **Postgres MCP** | BD Postgres | inventario y queries auditables (read-only en producción) |

---

## Estructura de archivos

```
/
├── .windsurfrules                     # Reglas globales cargadas por Cascade
├── AGENTS.md                          # Estándar AGENTS.md (Linux Foundation)
├── .windsurf/
│   └── workflows/                     # 12 slash commands
│       ├── inicio.md
│       ├── regularizar.md
│       ├── hoy.md
│       ├── nueva.md
│       ├── reparar.md
│       ├── decidir.md
│       ├── sync.md
│       ├── cerrar.md
│       ├── onboarding.md
│       ├── cuestionar.md              # Mutation thinking
│       ├── revisar-bd.md              # Auditoría de BD
│       └── revisar-frontend.md        # Auditoría de frontend (nuevo v4.4)
├── docs/                              # SSOT - Single Source of Truth
│   ├── prd.md
│   ├── architecture.md
│   ├── blueprints.md
│   ├── user-stories.md
│   ├── roadmap.md
│   ├── testing-strategy.md
│   ├── database-strategy.md           # Si hay BD
│   ├── frontend-strategy.md           # Si hay frontend (nuevo v4.4)
│   └── decisions-log.md               # ADRs
├── prompts/
│   ├── architect-brain.md             # Cerebro: protocolos On(...)
│   └── skills/
│       ├── tests-skill.md
│       ├── legacy-testing-skill.md
│       ├── database-skill.md
│       ├── frontend-skill.md          # Nuevo v4.4
│       ├── visual-testing-skill.md    # Nuevo v4.4
│       └── prompt-skill.md
├── inicio-chat.txt                    # Manual operativo completo
├── LICENSE                            # MIT
└── README.md                          # Este archivo
```

---

## Reglas de oro

1. **Centralización** — toda documentación vive en `/docs/`.
2. **Fuente de verdad** — los docs ganan sobre la memoria auto-generada de Windsurf.
3. **Sincronización** — ticket terminado = `[x]` en roadmap + tests en verde + checklist visible.
4. **No proliferación** — no crear archivos nuevos para cambios individuales; editar los maestros.
5. **Tests como contrato** — un `.skip` durante más de una semana es un bug del roadmap.
6. **Decisiones trazables** — si no está en `decisions-log.md`, no existe como decisión oficial.
7. **Antidrift** — ninguna tarea se cierra sin el checklist de sincronización documental.
8. **Legacy primero congelar, luego cambiar** — en código legacy, nunca modificar sin haber capturado antes el comportamiento actual.
9. **Cobertura ≠ calidad** — un test que pasa no garantiza detectar bugs.
10. **BD primero auditar, luego cambiar** — en proyectos con BD, ejecutar `/revisar-bd` antes de cambios destructivos.
11. **Frontend primero medir, luego optimizar** — WCAG 2.2 AA y CWV son criterios objetivos. Datos reales (CrUX, axe, lector de pantalla) > intuición.
12. **Implementación paso a paso** — si un plan toca >2 archivos, aplicar `On(implementation_phase)` con paradas obligatorias.
13. **Código IA bajo revisión** — outputs de v0/Lovable/Bolt/Figma Make pasan obligatoriamente por revisión humana en Cursor/Windsurf/Claude Code antes de producción.

---

## Sobre este sistema

Construido iterativamente a partir de [`junior-doc-gen`](https://github.com/gicupc/junior-doc-gen).

Evolución:

- **v4.1**: testing integrado, ADRs inmutables, antidrift por checklist, slash commands, characterization tests, mutation thinking.
- **v4.2**: base de datos como ciudadano de primera clase — `On(database_setup)`, `On(database_audit)`, `/revisar-bd`, `database-skill.md`, `database-strategy.md`.
- **v4.3**: `/onboarding` para codebases ajenos, `On(implementation_phase)` con paradas obligatorias entre archivos, patrón Active-Record-friendly para mocks de Prisma, `prompt-skill.md` con patrón RACEO.
- **v4.4** *(actual)* — **Frontend & MCP Edition**: añade frontend como ciudadano de primera clase. `On(frontend_setup)` automático para proyectos nuevos con stack canónico 2026 (Next.js + TS estricto + Tailwind v4 + shadcn + Biome + Zod). `On(frontend_audit)` y `/revisar-frontend` para proyectos existentes con auditoría de stack/seguridad/CWV/a11y. Skills nuevas `frontend-skill.md` y `visual-testing-skill.md`. Doc nuevo `frontend-strategy.md` en SSOT. `AGENTS.md` añadido como estándar oficial (Linux Foundation, dic 2025). Reglas de oro 11, 12 y 13 sobre frontend, MCP servers y código IA bajo revisión.

---

## Licencia

MIT


---

## Soporte para Claude Code (post-v4.4)

Además de Windsurf, este sistema funciona también en **Claude Code** (la CLI de Anthropic) gracias al archivo `CLAUDE.md` añadido en la raíz del proyecto. Es un fichero delgado (~5 KB) que apunta a las fuentes de verdad existentes (`.windsurfrules`, `prompts/architect-brain.md`, los workflows y skills), por lo que no duplica contenido y no afecta al comportamiento de Windsurf.

Si abres Claude Code en un proyecto que tenga esta estructura instalada, lee `CLAUDE.md` automáticamente y aplica el mismo sistema Architect-Brain v4.4 que aplica Windsurf: mismas reglas de oro, mismos protocolos `On(...)`, mismos slash commands.
