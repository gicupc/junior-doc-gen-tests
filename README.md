# junior-doc-gen-tests

Sistema de scaffolding para [Windsurf](https://windsurf.com) que convierte a Cascade en un arquitecto senior disciplinado. Mantiene **documentación**, **código**, **tests**, **decisiones**, **base de datos**, **frontend** (accesibilidad + Core Web Vitals + seguridad) y **cadena de suministro** (npm + GitHub Actions + OIDC publishing) sincronizados durante todo el ciclo de vida del proyecto, evitando el clásico *documentation drift*.

---

## Qué resuelve

Cuando trabajas con una IA generando código día tras día, aparecen ocho problemas silenciosos:

1. **Los docs se quedan atrás del código** — nadie los actualiza porque no duele inmediatamente.
2. **Las decisiones técnicas se olvidan** — ¿por qué elegimos esta librería? Nadie se acuerda tres meses después.
3. **Los tests son un "luego" eterno** — se añaden al final, si sobra tiempo, con calidad baja.
4. **El código legacy se toca sin red de seguridad** — cambiar primero y rezar después es receta para bugs nuevos.
5. **Cobertura ≠ calidad** — tests que pasan no garantizan detectar bugs reales.
6. **Las bases de datos crecen sin auditoría** — índices faltantes, datos personales sin proteger, queries lentas que nadie detecta.
7. **El frontend acumula deuda invisible** — accesibilidad rota, Core Web Vitals en rojo, secretos expuestos en variables públicas, código IA sin revisar.
8. **Las dependencias npm pueden estar comprometidas en cualquier momento** — ataques tipo Shai-Hulud o el TanStack del 11 mayo 2026 (84 versiones maliciosas en 42 paquetes con provenance SLSA válido) demuestran que `npm install` sin defensas es ruleta rusa.

Este sistema impone protocolos que obligan a la IA a resolver los ocho problemas por diseño.

---

## Cómo se usa en 30 segundos

1. Copia esta estructura (`.windsurf/`, `.windsurfrules`, `AGENTS.md`, `docs/`, `prompts/`, `inicio-chat.txt`) en la raíz de tu proyecto.
2. Abre Windsurf apuntando a la raíz del proyecto.
3. En Cascade, escribe `/` y te aparecerá un menú con los comandos disponibles.
4. Elige el comando que toque según la situación.

---

## Slash commands disponibles (14)

| Comando             | Cuándo usarlo                                                                 |
|---------------------|-------------------------------------------------------------------------------|
| `/inicio`           | Proyecto vacío. Entrevista BMADT + docs + setup tests + setup BD si aplica + setup FRONTEND si aplica + setup SUPPLY CHAIN si usa npm. |
| `/onboarding`       | Codebase ajeno (ejercicio bootcamp, PR de un compañero, debug urgente). Mapa rápido en 6 secciones, sin generar docs. |
| `/regularizar`      | Proyecto con código pero sin docs. Menú [1][2][3][4]. La opción [4] incluye auditoría de BD, frontend y supply chain. |
| `/hoy`              | "¿Qué hacemos hoy?" Briefing con sugerencia concreta para el día.             |
| `/nueva`            | Añadir funcionalidad. Activa la interrupción de seguridad antes de tocar código. |
| `/reparar`          | Corregir un bug. Si no está cubierto, primero escribe el test que lo reproduce. |
| `/decidir`          | Cambiar una decisión pasada. Propaga cambios sin perder histórico.            |
| `/sync`             | Detectar drift entre código y documentación. Solo reporta.                    |
| `/cerrar`           | Forzar checklist de sincronización si sospechas que se saltó.                 |
| `/cuestionar`       | Auditar la calidad real de los tests con mutation thinking.                   |
| `/revisar-bd`       | Auditar la BD: modelo, seguridad, rendimiento, calidad de datos.              |
| `/revisar-frontend` | Auditar el frontend: stack, seguridad, Core Web Vitals, accesibilidad WCAG 2.2 AA. |
| `/bdd`              | Adoptar BDD: verifica aplicabilidad, instala playwright-bdd o @cucumber/cucumber, primer Three Amigos asistido, genera primer `.feature` validado por humano. *(nuevo en v4.5)* |
| `/revisar-supply-chain` | Auditar la cadena de suministro: dependencias, lockfile, scripts, GitHub Actions, publish path. Cross-referencia con incidentes públicos (TanStack, Shai-Hulud, axios, Trivy). *(nuevo en v4.6)* |

---

## Qué hace el sistema (los 10 pilares)

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

### 9. Adopción controlada de BDD *(nuevo en v4.5)*

Cuando el proyecto tiene stakeholders no técnicos que validan criterios de aceptación, `/bdd` guía la adopción de Behavior-Driven Development con cuatro garantías:

- **Verificación previa**: si no hay disposición a Three Amigos, recomienda NO adoptar BDD y registra la decisión como ADR.
- **Stack 2026 elegido por regla**: playwright-bdd para JS/TS con UI, @cucumber/cucumber para APIs sin UI, Reqnroll para .NET (SpecFlow discontinuado en diciembre 2024), Karate DSL para JVM.
- **Three Amigos asistida**: Example Mapping de Matt Wynne (Reglas → Ejemplos → Preguntas). Si quedan >3 preguntas sin resolver, la historia NO está lista para desarrollarse.
- **Validación humana obligatoria** del `.feature` antes de generar step definitions. Anti-patrones de Gherkin generado por IA documentados en `bdd-skill.md`.

La skill `ai-testing-skill.md` cubre adicionalmente el flujo IA-asistido transversal: Playwright MCP/CLI, prompting efectivo, trazabilidad de prompts en CI, riesgos (tokens, GDPR, EU AI Act).

### 10. Defensa de cadena de suministro en 4 capas *(nuevo en v4.6)*

Cuando el proyecto usa npm registry (Node, JS/TS, frontend con npm), el sistema activa automáticamente `On(supply_chain_setup)` con defensas en cuatro capas:

1. **Dependency resolution** — `minimumReleaseAge` (cooldown de 1+ día), `blockExoticSubdeps`, `trustPolicy: no-downgrade`. Lo que habría bloqueado todos los ataques recientes (Shai-Hulud, axios, TanStack), detectados y retirados del registro en horas.
2. **Install-time execution** — `allowBuilds` (deny by default) o `ignore-scripts` para impedir que `postinstall` arbitrarios se ejecuten.
3. **CI execution (GitHub Actions hardening)** — NUNCA `pull_request_target` con checkout del fork (el patrón "Pwn Request", vector del TanStack). Acciones pineadas a SHA, no a tag mutable. Permisos mínimos por job.
4. **Publish path (OIDC trusted publishing)** — si el proyecto publica al registro, OIDC pineado a `workflow + branch` (no solo a workflow), branch protection en main con block force pushes, 2FA obligatorio.

Adicionalmente, `/revisar-supply-chain` audita proyectos existentes cross-referenciando el lockfile con bases públicas de paquetes comprometidos (TanStack GHSA-g7cv-rxg3-hmpx, Shai-Hulud, axios, Trivy) y reportando hallazgos por severidad. La skill `supply-chain-skill.md` incluye el **protocolo de respuesta a incidente con orden crítico**: NO revocar el token de GitHub antes de limpiar el daemon de persistencia, porque algunos payloads ejecutan `rm -rf ~/` al detectar la revocación.

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
├── CLAUDE.md                          # Soporte Claude Code
├── .windsurf/
│   └── workflows/                     # 14 slash commands
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
│       ├── revisar-frontend.md        # Auditoría de frontend
│       ├── bdd.md                     # Adopción BDD (v4.5)
│       └── revisar-supply-chain.md    # Auditoría supply chain (nuevo v4.6)
├── docs/                              # SSOT - Single Source of Truth
│   ├── prd.md
│   ├── architecture.md
│   ├── blueprints.md
│   ├── user-stories.md
│   ├── roadmap.md
│   ├── testing-strategy.md
│   ├── database-strategy.md           # Si hay BD
│   ├── frontend-strategy.md           # Si hay frontend
│   ├── supply-chain-strategy.md       # Si usa npm registry (nuevo v4.6)
│   └── decisions-log.md               # ADRs
├── prompts/
│   ├── architect-brain.md             # Cerebro: protocolos On(...)
│   └── skills/                        # 10 skills
│       ├── tests-skill.md
│       ├── integration-testing-skill.md  # v4.5
│       ├── visual-testing-skill.md
│       ├── bdd-skill.md                  # v4.5
│       ├── ai-testing-skill.md           # v4.5
│       ├── supply-chain-skill.md         # nuevo v4.6
│       ├── legacy-testing-skill.md
│       ├── database-skill.md
│       ├── frontend-skill.md
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
14. **BDD sin Three Amigos es disfraz técnico** *(v4.5)* — adoptar BDD requiere conversación previa con stakeholders no técnicos. Sin conversación, los `.feature` files no aportan valor real.
15. **Tests con IA trazables** *(v4.5)* — cada test generado por LLM lleva su `*.prompt.md` versionado y pasa code review humano explícito antes de mergear.
16. **AI literacy obligatoria** *(v4.5)* — desde el 2 de febrero de 2025 equipos que usan IA deben formar a su personal en uso responsable. Aplica al testing con IA.
17. **GDPR en testing con IA** *(v4.5)* — NO enviar PII a LLMs externos sin DPA válido. Para datos sensibles, LLMs locales o redacción previa.
18. **Supply chain en 4 capas** *(nuevo en v4.6)* — `minimumReleaseAge` (cooldown 1+ día) + `allowBuilds` (deny by default) + GitHub Actions hardening + OIDC pinning a workflow+branch. Provenance SLSA válido NO es garantía (lo demostró el ataque TanStack del 11 mayo 2026 con 84 versiones maliciosas y provenance SLSA Build Level 3 válido).
19. **Lifecycle scripts denegados por defecto** *(nuevo en v4.6)* — `allowBuilds` (pnpm) o `ignore-scripts` (npm) reducen el vector clásico de worms npm (postinstall que roba credenciales).
20. **Respuesta a incidente con orden crítico** *(nuevo en v4.6)* — si se detecta paquete comprometido instalado, NO revocar token de GitHub antes de limpiar daemon de persistencia. Algunos payloads (Mini Shai-Hulud, TanStack) ejecutan `rm -rf ~/` al detectar revocación.

---

## Sobre este sistema

Construido iterativamente a partir de [`junior-doc-gen`](https://github.com/gicupc/junior-doc-gen).

Evolución:

- **v4.1**: testing integrado, ADRs inmutables, antidrift por checklist, slash commands, characterization tests, mutation thinking.
- **v4.2**: base de datos como ciudadano de primera clase — `On(database_setup)`, `On(database_audit)`, `/revisar-bd`, `database-skill.md`, `database-strategy.md`.
- **v4.3**: `/onboarding` para codebases ajenos, `On(implementation_phase)` con paradas obligatorias entre archivos, patrón Active-Record-friendly para mocks de Prisma, `prompt-skill.md` con patrón RACEO.
- **v4.4** — **Frontend & MCP Edition**: añade frontend como ciudadano de primera clase. `On(frontend_setup)` automático para proyectos nuevos con stack canónico 2026 (Next.js + TS estricto + Tailwind v4 + shadcn + Biome + Zod). `On(frontend_audit)` y `/revisar-frontend` para proyectos existentes con auditoría de stack/seguridad/CWV/a11y. Skills nuevas `frontend-skill.md` y `visual-testing-skill.md`. Doc nuevo `frontend-strategy.md` en SSOT. `AGENTS.md` añadido como estándar oficial (Linux Foundation, dic 2025). Reglas de oro 11, 12 y 13 sobre frontend, MCP servers y código IA bajo revisión.
- **v4.5** — **Testing 2026 Edition (BDD + Integration + AI-Assisted)**: la suite de testing pasa de 3 capas (unit + legacy + visual/E2E) a 6 niveles documentados. Skills nuevas: `integration-testing-skill.md` (Supertest + MSW + Testcontainers), `bdd-skill.md` (Gherkin + playwright-bdd + Three Amigos + Example Mapping + anti-patrones IA), `ai-testing-skill.md` (Playwright MCP/CLI, prompting, trazabilidad, GDPR, EU AI Act). Workflow nuevo `/bdd`. Protocolo nuevo `On(bdd_setup)`. `On(testing_setup)` ampliado de 10 a 13 pasos con preguntas opt-in sobre BDD y uso de IA. Pirámide flexible 2026 (Cohn vs Testing Trophy de Kent C. Dodds) explicada en `testing-strategy.md`. Reglas de oro 14-17 sobre BDD, trazabilidad de tests IA, AI literacy (obligatoria desde 2 febrero 2025) y GDPR en testing con IA. Cumplimiento normativo (EU AI Act + GDPR) integrado en `testing-strategy.md` como sección opt-in.
- **v4.6** *(actual)* — **Supply Chain Security Edition**: cadena de suministro como ciudadano de primera clase tras el ataque a TanStack del 11 mayo 2026 (84 versiones maliciosas en 42 paquetes con provenance SLSA Build Level 3 válido). Skill nueva `supply-chain-skill.md` con las 4 capas de defensa (dependency resolution + install-time execution + CI execution + publish path) y protocolo de respuesta a incidente con orden crítico. Workflow nuevo `/revisar-supply-chain` que cross-referencia el lockfile con bases públicas de paquetes comprometidos. Protocolos nuevos `On(supply_chain_setup)` y `On(supply_chain_audit)`. Doc nuevo `supply-chain-strategy.md` en SSOT. `On(testing_setup)` ampliado a 14 pasos con paso 12 que dispara supply chain en proyectos JS/TS. `/regularizar` opción [4] incluye auditoría supply chain. Reglas de oro 18, 19 y 20 sobre defensa en capas, lifecycle scripts denegados por defecto y orden crítico de respuesta a incidentes (NO revocar token de GitHub antes de limpiar persistencia).

---

## Licencia

MIT


---

## Soporte para Claude Code (post-v4.4)

Además de Windsurf, este sistema funciona también en **Claude Code** (la CLI de Anthropic) gracias al archivo `CLAUDE.md` añadido en la raíz del proyecto. Es un fichero delgado (~5 KB) que apunta a las fuentes de verdad existentes (`.windsurfrules`, `prompts/architect-brain.md`, los workflows y skills), por lo que no duplica contenido y no afecta al comportamiento de Windsurf.

Si abres Claude Code en un proyecto que tenga esta estructura instalada, lee `CLAUDE.md` automáticamente y aplica el mismo sistema Architect-Brain v4.6 que aplica Windsurf: mismas reglas de oro, mismos protocolos `On(...)`, mismos slash commands.
