# junior-doc-gen-tests

Sistema de scaffolding para [Windsurf](https://windsurf.com) que convierte a Cascade en un arquitecto senior disciplinado. Mantiene **documentación**, **código**, **tests** y **decisiones** sincronizados durante todo el ciclo de vida del proyecto, evitando el clásico *documentation drift*.

---

## Qué resuelve

Cuando trabajas con una IA generando código día tras día, aparecen cinco problemas silenciosos:

1. **Los docs se quedan atrás del código** — nadie los actualiza porque no duele inmediatamente.
2. **Las decisiones técnicas se olvidan** — ¿por qué elegimos esta librería? Nadie se acuerda tres meses después.
3. **Los tests son un "luego" eterno** — se añaden al final, si sobra tiempo, con calidad baja.
4. **El código legacy se toca sin red de seguridad** — cambiar primero y rezar después es receta para bugs nuevos.
5. **Cobertura ≠ calidad** — tests que pasan no garantizan detectar bugs reales.

Este sistema impone protocolos que obligan a la IA a resolver los cinco problemas por diseño.

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
| `/inicio`       | Proyecto vacío. Entrevista BMADT + docs + setup de tests.                     |
| `/regularizar`  | Proyecto con código pero sin docs. Menú [1][2][3][4].                         |
| `/hoy`          | "¿Qué hacemos hoy?" Briefing con sugerencia concreta para el día.             |
| `/nueva`        | Añadir funcionalidad. Activa la interrupción de seguridad antes de tocar código. |
| `/reparar`      | Corregir un bug. Si no está cubierto, primero escribe el test que lo reproduce. |
| `/decidir`      | Cambiar una decisión pasada. Propaga cambios sin perder histórico.            |
| `/sync`         | Detectar drift entre código y documentación. Solo reporta.                    |
| `/cerrar`       | Forzar checklist de sincronización si sospechas que se saltó.                 |
| `/cuestionar`   | Auditar la calidad real de los tests con mutation thinking.                   |

---

## Qué hace el sistema (los 6 pilares)

### 1. Entrevista inicial estructurada (BMADT)

Al arrancar un proyecto nuevo con `/inicio`, la IA te hace 5 preguntas:

- **B (Beneficio):** objetivo y valor del proyecto.
- **M (Mecanismo):** stack técnico e integraciones.
- **A (Alcance):** módulos del MVP.
- **D (Datos):** entidades y flujos críticos.
- **T (Testing):** nivel de cobertura deseado (básico/estándar/exhaustivo).

Con las respuestas genera automáticamente los 7 docs del SSOT en `/docs/` y **no escribe código** hasta que apruebes.

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

### 3. Trazabilidad de decisiones (ADRs)

Todas las decisiones no triviales (stack, librerías, cambios de scope, adopción de mutation testing) se registran en `decisions-log.md` como **Architecture Decision Records**:

- Contexto que motivó la decisión.
- Decisión tomada.
- Alternativas descartadas y por qué.
- Consecuencias previstas.

Las ADRs son **inmutables**: si cambias de opinión, no se borra lo anterior — se añade una nueva ADR y la antigua queda marcada como `Superseded por ADR-XXX`.

### 4. Antidrift documental

Al cerrar **cualquier tarea** de código, la IA muestra un checklist visible marcando cada uno de los 7 docs del SSOT como:

- Actualizado (con descripción del cambio), o
- Sin cambios (con justificación).

Una tarea no está cerrada si el checklist no aparece. Los docs nunca se quedan atrás porque cada tarea obliga a revisarlos todos.

### 5. Testing sobre código legacy (characterization tests)

Cuando se aplica `/regularizar` con opción [4] sobre código que ya existe, la skill `legacy-testing-skill.md` se activa automáticamente si detecta:

- Código sin tests previos con lógica sospechosa (validaciones raras, branches inaccesibles, error handling silencioso).
- Divergencias entre el código y un contrato formal (OpenAPI, JSON Schema, tipos estrictos).
- Indicación explícita del usuario de que el código tiene bugs conocidos.

En ese caso, los tests generados distinguen dos categorías:

- **`// SPEC: ...`** — comportamiento correcto y deseado. No debería cambiar.
- **`// CHARACTERIZATION: ...`** — comportamiento actual, posiblemente incorrecto. Se captura tal cual para poder modificarlo después con evidencia.

La IA propone. El humano decide qué es bug y qué es feature. Nunca se modifica código legacy sin red de seguridad primero.

### 6. Cuestionamiento de calidad (mutation thinking)

El comando `/cuestionar` audita la calidad real de la suite existente sin instalar herramientas:

1. Elige alcance (archivo, módulo, backend entero).
2. La IA propone 5+ mutaciones realistas sobre el código (cambiar operadores, valores, flujo, lógica de negocio).
3. Evalúa si la suite actual detectaría cada una.
4. Lista mutaciones "supervivientes" y propone los tests faltantes prioritarios.
5. Ofrece ir más allá: instalar mutation testing real (Infection, StrykerJS, PIT, mutmut según stack) como decisión opt-in con ADR.

Menciona property-based testing (fast-check, Hypothesis) si detecta invariantes claras, sin implementarlo automáticamente.

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
│   └── workflows/              # 9 slash commands
│       ├── inicio.md
│       ├── regularizar.md
│       ├── hoy.md
│       ├── nueva.md
│       ├── reparar.md
│       ├── decidir.md
│       ├── sync.md
│       ├── cerrar.md
│       └── cuestionar.md
├── docs/                       # SSOT - Single Source of Truth
│   ├── prd.md                  # Qué es el producto y para quién
│   ├── architecture.md         # Cómo está montado técnicamente
│   ├── blueprints.md           # Patrones de código del proyecto
│   ├── user-stories.md         # Funcionalidades con criterios testeables
│   ├── roadmap.md              # Tickets y progreso
│   ├── testing-strategy.md     # Framework, convenciones, cobertura
│   └── decisions-log.md        # ADRs (registro inmutable de decisiones)
├── prompts/
│   ├── architect-brain.md      # Cerebro del sistema: protocolos On(...)
│   └── skills/
│       ├── tests-skill.md           # Guía de tests por stack
│       └── legacy-testing-skill.md  # Characterization tests para legacy
├── inicio-chat.txt             # Manual operativo completo
├── LICENSE                     # MIT
└── README.md                   # Este archivo
```

---

## Flujo típico de uso

**Proyecto nuevo desde cero:**

1. Copio la estructura a una carpeta vacía.
2. Abro Windsurf allí.
3. Escribo `/inicio` y contesto la entrevista BMADT.
4. La IA genera docs, instala tests, crea el primer ticket.
5. Voy desarrollando con `/nueva` y `/reparar`.
6. Al día siguiente: `/hoy` y me pone al día en 10 líneas.

**Proyecto ya empezado sin docs:**

1. Copio la estructura a la raíz del proyecto existente.
2. `/regularizar` → menú de opciones, elijo `[4] Regularizar + Cubrir con tests`.
3. La IA escanea el código, identifica lógica crítica, detecta si hay código legacy sospechoso (y activa `legacy-testing-skill` si procede), instala framework de tests, genera tests distinguiendo SPEC y CHARACTERIZATION, y crea tickets para las zonas pendientes.

**Auditar calidad de tests ya existentes:**

1. `/cuestionar` → elige alcance.
2. La IA propone mutaciones y evalúa cuáles detectaría la suite.
3. Lista tests faltantes prioritarios.
4. Decides si añades solo esos tests o si además instalas mutation testing real.

---

## Validación

El sistema se probó con un proyecto completo (gestor de tareas personal en PHP + MySQL) usando **solo** los protocolos del sistema, sin intervención manual en la documentación.

Resultado:

- 9 tickets cerrados, 60% del MVP alcanzado.
- 57 tests unitarios en verde, 0 failing.
- 3 ADRs registradas con alternativas y consecuencias.
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

---

## Sobre este sistema

Construido iterativamente a partir de [`junior-doc-gen`](https://github.com/gicupc/junior-doc-gen) (versión anterior centrada solo en documentación con protocolo BMAD).

Esta versión añade: testing integrado en el flujo, trazabilidad de decisiones con ADRs, antidrift por checklist obligatorio, slash commands de Windsurf para invocar los protocolos fácilmente, characterization tests sobre legacy, y auditoría de calidad con mutation thinking.

---

## Licencia

MIT
