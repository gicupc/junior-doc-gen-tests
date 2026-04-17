# junior-doc-gen-tests

Sistema de scaffolding para [Windsurf](https://windsurf.com) que convierte a Cascade en un arquitecto senior disciplinado. Mantiene **documentación**, **código**, **tests** y **decisiones** sincronizados durante todo el ciclo de vida del proyecto, evitando el clásico *documentation drift*.

---

## Qué resuelve

Cuando trabajas con una IA generando código día tras día, aparecen cuatro problemas silenciosos:

1. **Los docs se quedan atrás del código** — nadie los actualiza porque no duele inmediatamente.
2. **Las decisiones técnicas se olvidan** — ¿por qué elegimos esta librería? Nadie se acuerda tres meses después.
3. **Los tests son un "luego" eterno** — se añaden al final, si sobra tiempo, con calidad baja.
4. **Retomar un proyecto tras una pausa cuesta horas** — hay que volver a cargarse el contexto completo.

Este sistema impone protocolos que obligan a la IA a resolver los cuatro problemas por diseño.

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

---

## Qué hace el sistema (los 5 pilares)

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

Todas las decisiones no triviales (stack, librerías, cambios de scope) se registran en `decisions-log.md` como **Architecture Decision Records**:

- Contexto que motivó la decisión.
- Decisión tomada.
- Alternativas descartadas y por qué.
- Consecuencias previstas.

Las ADRs son **inmutables**: si cambias de opinión, no se borra lo anterior — se añade una nueva ADR y la antigua queda marcada como `Superseded por ADR-XXX`. Trazabilidad completa del porqué de cada decisión a lo largo del tiempo.

### 4. Antidrift documental

Al cerrar **cualquier tarea** de código, la IA debe mostrar un checklist visible marcando cada uno de los 7 docs del SSOT como:

- Actualizado (con descripción del cambio), o
- Sin cambios (con justificación).

Una tarea no está cerrada si el checklist no aparece. Esto resuelve el drift por diseño: los docs nunca se quedan atrás porque cada tarea obliga a revisarlos todos.

### 5. Retomar proyectos tras pausas

`/hoy` lee los docs clave, ejecuta un mini health-check (tests en verde, tickets con commits, drift básico) y devuelve un briefing de 10 líneas:

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
│   └── workflows/              # 8 slash commands
│       ├── inicio.md
│       ├── regularizar.md
│       ├── hoy.md
│       ├── nueva.md
│       ├── reparar.md
│       ├── decidir.md
│       ├── sync.md
│       └── cerrar.md
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
│       └── tests-skill.md      # Guía de generación de tests por stack
├── inicio-chat.txt             # Manual operativo completo
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
3. La IA escanea el código, identifica lógica crítica, instala framework de tests, genera tests para esa lógica, documenta qué cubrió y qué no, y crea tickets para las zonas pendientes.

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

---

## Sobre este sistema

Construido iterativamente a partir de [`junior-doc-gen`](https://github.com/gicupc/junior-doc-gen) (versión anterior centrada solo en documentación con protocolo BMAD).

Esta versión añade: testing integrado en el flujo, trazabilidad de decisiones con ADRs, antidrift por checklist obligatorio, y slash commands de Windsurf para invocar los protocolos fácilmente.

---

## Licencia

MIT
