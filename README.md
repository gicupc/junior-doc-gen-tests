# junior-doc-gen-tests

Sistema de scaffolding para [Windsurf](https://windsurf.com) que convierte a Cascade en un arquitecto senior disciplinado. Mantiene **documentación**, **código**, **tests**, **decisiones** y **base de datos** sincronizados durante todo el ciclo de vida del proyecto, evitando el clásico *documentation drift*.

---

## Qué resuelve

Cuando trabajas con una IA generando código día tras día, aparecen seis problemas silenciosos:

1. **Los docs se quedan atrás del código** — nadie los actualiza porque no duele inmediatamente.
2. **Las decisiones técnicas se olvidan** — ¿por qué elegimos esta librería? Nadie se acuerda tres meses después.
3. **Los tests son un "luego" eterno** — se añaden al final, si sobra tiempo, con calidad baja.
4. **El código legacy se toca sin red de seguridad** — cambiar primero y rezar después es receta para bugs nuevos.
5. **Cobertura ≠ calidad** — tests que pasan no garantizan detectar bugs reales.
6. **Las bases de datos crecen sin auditoría** — índices faltantes, datos personales sin proteger, queries lentas que nadie detecta.

Este sistema impone protocolos que obligan a la IA a resolver los seis problemas por diseño.

---

## Cómo se usa en 30 segundos

1. Copia esta estructura (`.windsurf/`, `.windsurfrules`, `docs/`, `prompts/`, `inicio-chat.txt`) en la raíz de tu proyecto.
2. Abre Windsurf apuntando a la raíz del proyecto.
3. En Cascade, escribe `/` y te aparecerá un menú con los comandos disponibles.
4. Elige el comando que toque según la situación (ver tabla abajo).

---

## Slash commands disponibles

| Comando         | Cuándo usarlo                                                                 |
|-----------------|-------------------------------------------------------------------------------|
| `/inicio`       | Proyecto vacío. Entrevista BMADT + docs + setup de tests + setup de BD si aplica. |
| `/onboarding`   | Codebase ajeno (ejercicio de bootcamp, PR de un compañero, debug urgente). Mapa rápido en 6 secciones, sin generar docs. *(nuevo en v4.3)* |
| `/regularizar`  | Proyecto con código pero sin docs. Menú [1][2][3][4]. La opción [4] incluye auditoría de BD. |
| `/hoy`          | "¿Qué hacemos hoy?" Briefing con sugerencia concreta para el día.             |
| `/nueva`        | Añadir funcionalidad. Activa la interrupción de seguridad antes de tocar código. |
| `/reparar`      | Corregir un bug. Si no está cubierto, primero escribe el test que lo reproduce. |
| `/decidir`      | Cambiar una decisión pasada. Propaga cambios sin perder histórico.            |
| `/sync`         | Detectar drift entre código y documentación. Solo reporta.                    |
| `/cerrar`       | Forzar checklist de sincronización si sospechas que se saltó.                 |
| `/cuestionar`   | Auditar la calidad real de los tests con mutation thinking.                   |
| `/revisar-bd`   | Auditar la BD: modelo, seguridad, rendimiento, calidad de datos.              |

---

## Qué hace el sistema (los 7 pilares)

### 1. Entrevista inicial estructurada (BMADT)

Al arrancar un proyecto nuevo con `/inicio`, la IA te hace 5 preguntas:

- **B (Beneficio):** objetivo y valor del proyecto.
- **M (Mecanismo):** stack técnico e integraciones.
- **A (Alcance):** módulos del MVP.
- **D (Datos):** entidades y flujos críticos.
- **T (Testing):** nivel de cobertura deseado (básico/estándar/exhaustivo).

Con las respuestas genera automáticamente los docs del SSOT en `/docs/` y **no escribe código** hasta que apruebes.

### 2. Setup automático del entorno de tests

Según el stack detectado en la respuesta M, instala el framework adecuado con su soporte de linter:

| Stack              | Framework                              | Linter-friendly               |
|--------------------|----------------------------------------|-------------------------------|
| PHP                | PHPUnit                                | `phpstan/phpstan-phpunit`     |
| JS/TS backend      | Jest (+ ts-jest si TypeScript)         | `@types/jest`                 |
| JS frontend        | Jest con `jsdom`                       | `@types/jest`                 |
| React              | Jest + React Testing Library           | `@types/jest`                 |
| Vue                | Vitest + Vue Test Utils                | Tipos nativos                 |
| Python             | pytest                                 | `pytest-mock`                 |

Ejecuta un smoke test para verificar que todo arranca, y solo entonces permite empezar con código funcional.

### 3. Setup automático de base de datos *(nuevo en v4.2)*

Si la respuesta M o D del BMADT implica BD, el sistema dispara automáticamente `On(database_setup)`:

- **Detecta el motor adecuado** (Postgres por defecto, MongoDB para documentos, Redis para caché).
- **Detecta el ORM apropiado** (Prisma para Node/TS, Eloquent para PHP, SQLAlchemy para Python).
- **Identifica campos PII** y decide su tratamiento (hash, cifrado, plano).
- **Genera `database-strategy.md`** con todas las decisiones documentadas.
- **Configura auditoría futura** (ej. `pg_stat_statements` activado desde día 1).
- **Prepara Testcontainers** para tests de integración con BD real.
- **Registra ADRs** por cada decisión.

### 4. Trazabilidad de decisiones (ADRs)

Todas las decisiones no triviales se registran en `decisions-log.md` como **Architecture Decision Records**:

- Contexto que motivó la decisión.
- Decisión tomada.
- Alternativas descartadas y por qué.
- Consecuencias previstas.

Las ADRs son **inmutables**: si cambias de opinión, no se borra lo anterior — se añade una nueva ADR y la antigua queda marcada como `Superseded por ADR-XXX`.

### 5. Antidrift documental

Al cerrar **cualquier tarea** de código, la IA muestra un checklist visible marcando cada uno de los docs del SSOT como:

- Actualizado (con descripción del cambio), o
- Sin cambios (con justificación).

Una tarea no está cerrada si el checklist no aparece. Los docs nunca se quedan atrás porque cada tarea obliga a revisarlos todos.

### 6. Auditorías de calidad

Tres niveles de auditoría disponibles en cualquier momento:

- **`/sync`** — detecta drift entre código y docs.
- **`/cuestionar`** — audita calidad de los tests con mutation thinking.
- **`/revisar-bd`** *(nuevo en v4.2)* — audita la BD en 4 categorías:
  - **Modelo**: tipos, índices, FKs, audit fields, normalización.
  - **Seguridad**: PII en plano, restricciones débiles, RLS, secrets.
  - **Rendimiento**: N+1, queries lentas, índices faltantes.
  - **Calidad de datos**: nulos, duplicados, rangos, integridad referencial.

Filosofía común: **la IA detecta, el humano decide, el humano repara**. Las auditorías nunca modifican código ni datos automáticamente.

### 7. Retomar proyectos tras pausas

`/hoy` lee los docs clave, ejecuta un mini health-check y devuelve un briefing de 10 líneas:

```
📋 Proyecto: [Nombre]
🎯 Objetivo: [1 frase del PRD]
📈 Progreso: [X/Y tickets — Z%]
✅ Último cerrado: [título]
👉 Siguiente: [título]
🧪 Tests: [N passing / M skipped / K failing]
💡 Últimas decisiones: [2-3 ADRs]
⚠️ Warnings: [drift detectado o "ninguno"]
🚀 Sugerencia: [acción concreta para hoy]
```

En 20 segundos tienes el estado del proyecto sin releer seis archivos.

---

## Estructura de archivos

```
/
├── .windsurfrules              # Reglas globales cargadas por Cascade
├── .windsurf/
│   └── workflows/              # 10 slash commands
│       ├── inicio.md
│       ├── regularizar.md
│       ├── hoy.md
│       ├── nueva.md
│       ├── reparar.md
│       ├── decidir.md
│       ├── sync.md
│       ├── cerrar.md
│       ├── cuestionar.md       # Mutation thinking + auditoría de tests
│       └── revisar-bd.md       # Auditoría de BD (4 categorías)
├── docs/                       # SSOT - Single Source of Truth
│   ├── prd.md                  # Qué es el producto y para quién
│   ├── architecture.md         # Cómo está montado técnicamente
│   ├── blueprints.md           # Patrones de código del proyecto
│   ├── user-stories.md         # Funcionalidades con criterios testeables
│   ├── roadmap.md              # Tickets y progreso
│   ├── testing-strategy.md     # Framework, convenciones, cobertura tests
│   ├── database-strategy.md    # Motor, ORM, PII, migraciones, tests BD
│   └── decisions-log.md        # ADRs (registro inmutable de decisiones)
├── prompts/
│   ├── architect-brain.md      # Cerebro del sistema: protocolos On(...)
│   └── skills/
│       ├── tests-skill.md           # Guía de tests por stack
│       ├── legacy-testing-skill.md  # Characterization tests para legacy
│       └── database-skill.md        # Diseño y auditoría de BD
├── inicio-chat.txt             # Manual operativo completo
├── LICENSE                     # MIT
└── README.md                   # Este archivo
```

---

## Flujo típico de uso

**Proyecto nuevo desde cero (con BD):**

1. Copio la estructura a una carpeta vacía.
2. Abro Windsurf allí.
3. Escribo `/inicio` y contesto la entrevista BMADT.
4. La IA detecta que hay BD en el stack, dispara `On(database_setup)`, me pregunta qué campos son PII, genera `database-strategy.md`.
5. Genera el resto de docs, instala tests, crea los primeros tickets.
6. Voy desarrollando con `/nueva` y `/reparar`.
7. Al día siguiente: `/hoy` y me pone al día en 10 líneas.

**Proyecto ya empezado sin docs (con BD existente):**

1. Copio la estructura a la raíz del proyecto existente.
2. `/regularizar` → menú de opciones, elijo `[4] Regularizar + Cubrir con tests + Auditar BD`.
3. La IA escanea el código y la BD, identifica lógica crítica, detecta legacy sospechoso, instala framework de tests, genera tests distinguiendo SPEC y CHARACTERIZATION.
4. **Audita la BD** en 4 categorías y genera `database-audit.md` + `investigacion_seguridad.md`.
5. Crea tickets en el roadmap por cada hallazgo crítico/alto.
6. Yo decido qué reparar primero usando `/reparar` ticket por ticket.

**Auditoría puntual de BD en cualquier momento:**

1. `/revisar-bd` → elige alcance (completa o parcial: solo modelo, solo seguridad, etc.).
2. La IA examina y reporta sin tocar nada.
3. Decides qué arreglar.

---

## Validación

El sistema se probó con un proyecto completo (gestor de tareas personal en PHP + MySQL) usando **solo** los protocolos del sistema, sin intervención manual en la documentación.

Resultado:

- 9 tickets cerrados, 60% del MVP alcanzado.
- 57 tests unitarios en verde, 0 failing → tras `/cuestionar` y hardening: 63 tests con kill-ratio estimado >90%.
- 4 ADRs registradas con alternativas y consecuencias.
- Cero drift documental en todo el desarrollo.
- Detección y corrección autónoma de un papercut de linter (falsos positivos de Intelephense en mocks de PHPUnit), con su correspondiente ADR.

---

## Reglas de oro

1. **Centralización** — toda documentación vive en `/docs/`. Nada de subcarpetas `docs` internas.
2. **Fuente de verdad** — los docs ganan sobre la memoria auto-generada de Windsurf.
3. **Sincronización** — ticket terminado = `[x]` en roadmap + tests en verde + checklist visible.
4. **No proliferación** — no crear archivos nuevos para cambios individuales; editar los maestros.
5. **Tests como contrato** — un `.skip` durante más de una semana es un bug del roadmap.
6. **Decisiones trazables** — si no está en `decisions-log.md`, no existe como decisión oficial.
7. **Antidrift** — ninguna tarea se cierra sin el checklist de sincronización documental.
8. **Legacy primero congelar, luego cambiar** — en código legacy, nunca modificar sin haber capturado antes el comportamiento actual.
9. **Cobertura ≠ calidad** — un test que pasa no garantiza detectar bugs; la calidad se audita con mutation thinking.
10. **BD primero auditar, luego cambiar** — en proyectos con BD, ejecutar `/revisar-bd` antes de cambios destructivos.

---

## Sobre este sistema

Construido iterativamente a partir de [`junior-doc-gen`](https://github.com/gicupc/junior-doc-gen) (versión anterior centrada solo en documentación con protocolo BMAD).

Evolución:

- **v4.1**: añadió testing integrado, ADRs inmutables, antidrift por checklist, slash commands, characterization tests para legacy, mutation thinking.
- **v4.2** *(actual)*: añade base de datos como ciudadano de primera clase — `On(database_setup)` automático para proyectos nuevos, `On(database_audit)` y `/revisar-bd` para proyectos existentes, skill `database-skill.md` cubriendo diseño y auditoría, doc `database-strategy.md` como parte del SSOT.

---

## Licencia

MIT
