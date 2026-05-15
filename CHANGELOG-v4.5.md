# Changelog v4.4 → v4.5 (Testing 2026 Edition)

Sesión de actualización: 15 de mayo de 2026

Origen: lecciones extraídas del módulo 11 de AI4Devs — Testing 2026 (Integration & E2E + BDD + Testing asistido por IA). Convergencia 2024-2026 alrededor de Playwright como motor cross-browser, consolidación de Playwright MCP (Microsoft, marzo 2025) y Playwright CLI como herramientas oficiales para agentes IA, discontinuación de SpecFlow (diciembre 2024), entrada en vigor del calendario EU AI Act (2 febrero 2025 AI literacy, 2 agosto 2025 GPAI, 2 agosto 2026 sistemas de alto riesgo) y consolidación de la distinción operativa entre IA tradicional (ML clásico) y LLMs aplicada a testing.

---

## Resumen del cambio

La suite de testing del scaffolding pasa de **3 capas** (unit + legacy + visual/E2E) a **6 niveles documentados**, con dos transversales nuevas:

- **Capa intermedia** (integración) que tenía nivel cubierto solo de pasada en `tests-skill.md`.
- **Capa de aceptación con BDD** (Gherkin + playwright-bdd + Three Amigos) que no existía.
- **Transversal IA aplicada al testing** (Playwright MCP/CLI, prompting, trazabilidad, anti-patrones, GDPR, EU AI Act) que estaba mencionada solo en líneas sueltas.

La filosofía es la misma de versiones anteriores: **detectar señales, configurar entorno automáticamente, registrar ADRs, validación humana en los puntos críticos**. No se rompe nada existente; lo nuevo es opt-in vía preguntas explícitas durante `On(testing_setup)`.

---

## Cambios

### 1. Skill nueva: `prompts/skills/bdd-skill.md`

**Estado:** archivo nuevo (~17 KB).

Cubre Behavior-Driven Development end-to-end:

- **Principio fundamental**: BDD es conversación que produce ejemplos, no Gherkin ceremonial. Sin Three Amigos, los `.feature` files no aportan valor real.
- **TDD vs BDD** con tabla de cuándo gana cada uno.
- **Sintaxis Gherkin** con reglas no negociables (un solo `When`, lenguaje del dominio, `Background` solo para precondiciones comunes, `Scenario Outline` cuando varían solo los datos).
- **Herramientas 2026** con regla de elección por stack: playwright-bdd para JS/TS con UI, `@cucumber/cucumber` (con scope) para APIs sin UI, `@badeball/cypress-cucumber-preprocessor` para Cypress legacy, Reqnroll para .NET (SpecFlow discontinuado diciembre 2024), Karate DSL para JVM. Advierte explícitamente sobre paquetes abandonados (`cucumber` sin scope desde 2021).
- **Setup completo con playwright-bdd** (config, estructura `features/`, step definitions con fixtures, scripts `package.json`).
- **Three Amigos + Example Mapping de Matt Wynne** (tarjetas amarilla/azul/verde/roja) + validación INVEST.
- **BDD asistido por IA**: prompt RACEO efectivo para generar `.feature` desde user-story, prompt para generar step definitions, y la lista crítica de **7 anti-patrones** de Gherkin generado por IA (escenarios imperativos de UI, demasiado técnicos, múltiples When/Then, falta de Examples, lenguaje inconsistente, escenarios fantasma, pérdida del lenguaje ubicuo).
- **Living documentation** con herramientas 2026 (Cucumber Reports Service, Allure Report, HTML reporter Playwright) y aviso explícito de discontinuados (Pickles, SpecFlow+ LivingDoc).
- **Antipatrones operativos**, regla de oro final.

### 2. Skill nueva: `prompts/skills/integration-testing-skill.md`

**Estado:** archivo nuevo (~14 KB).

Cubre la capa intermedia de la pirámide:

- **Principio fundamental**: los unit tests con mocks mienten cómodamente; los E2E son caros; los de integración están en el punto óptimo de la curva coste/valor.
- **Qué SÍ es y qué NO es** integración (diferenciado de unit y E2E con ejemplos concretos).
- **Herramientas por stack**: Supertest + Vitest/Jest + Testcontainers + MSW para Node; PHPUnit + cliente HTTP real para PHP; pytest + TestClient + Testcontainers Python para Python; MockMvc + Testcontainers para JVM. Pact para contract testing cross-stack.
- **4 patrones con código completo**:
  - Patrón 1: Supertest + Testcontainers + Prisma (helpers reutilizables `startTestDb`/`stopTestDb`).
  - Patrón 2: MSW para mocks de red reutilizables entre tests de integración y E2E (mismos handlers).
  - Patrón 3: PHPUnit + cliente HTTP real contra app levantada en modo test.
  - Patrón 4: Pact (mención breve para microservicios).
- **Tabla de cuándo escalar unit → integration → E2E**.
- **Pirámide flexible 2026**: Cohn vs Testing Trophy de Kent C. Dodds; documentar la elección como ADR.
- Convenciones de carpetas, scripts `package.json`, antipatrones (BD mockeada en tests "integration", reiniciar container entre tests, etc.).

### 3. Skill nueva: `prompts/skills/ai-testing-skill.md`

**Estado:** archivo nuevo (~22 KB).

Skill transversal aplicable a todas las capas cuando se introduce IA en el flujo:

- **Principio fundamental**: la IA es la pluma, no el autor; sin entendimiento del dominio, los tests son test theater.
- **IA tradicional (ML clásico) vs IA generativa (LLMs)** con tabla de cuándo gana cada una:
  - ML clásico: self-healing de selectores (Healenium), predicción de flakiness, test impact analysis, generación determinista de unit tests (Diffblue Cover), anomaly detection.
  - LLMs: generación desde user-story, debugging conversacional, self-healing semántico, Gherkin desde requisitos.
- **Playwright MCP** (Microsoft, marzo 2025, `npx @playwright/mcp@latest`): instalación, configuración en Cursor/Claude Desktop/Claude Code, capacidades, flujo típico.
- **Playwright CLI** (2026, para agentes con filesystem): comparación de tokens (~27k vs ~114k del MCP), comandos típicos, recomendación oficial de Microsoft sobre cuándo usar cada uno.
- **5 patrones de prompting efectivos**: Given/When/Then antes que pseudocódigo, queries accesibles obligatorias, verificación de estabilidad (3 ejecuciones), POM en dos pasos, restricciones de dominio para BDD asistido.
- **8 anti-patrones de tests generados por IA** (test theater, snapshots gigantes, selectores frágiles, acoplados a implementación, datos irreales, setup duplicado, `expect` huérfanos, sesgos del training data).
- **Self-healing por categorías** (selector-level con Healenium = determinista, visual con Applitools/Percy = parcial, semántico con LLMs = baja determinación) + advertencia sobre cuándo el self-healing es peligroso (botón crítico que desaparece).
- **Plataformas SaaS por categoría** (unit gen, E2E free tier, visual regression, self-healing enterprise, observabilidad, test data) con ejes de decisión + confusiones comunes (Meticulous NO genera unit tests; TestCafe retirado de recomendaciones).
- **Trazabilidad de prompts en CI**: patrón `*.prompt.md` junto al test, gates en CI (lint, flakiness, coverage de prompts, code review obligatorio).
- **Riesgos**: coste en tokens (10-100 USD/run), no-determinismo, GDPR/RGPD (DPA, LLMs locales, redactar antes de enviar), **EU AI Act** con calendario completo (2 feb 2025 AI literacy + prohibiciones, 2 ago 2025 GPAI, 2 ago 2026 alto riesgo; multas hasta 35M€ o 7%), sesgos, casos donde la IA NO funciona bien (performance, fuzzing, contract testing, property-based, lógica regulada).
- **Checklist obligatorio antes de mergear** tests generados por IA, antipatrones, regla de oro final.

### 4. Workflow nuevo: `.windsurf/workflows/bdd.md`

**Estado:** archivo nuevo (~8 KB).

Slash command `/bdd` para adoptar BDD con verificación previa de aplicabilidad. 10 pasos del protocolo:

1. Verificar aplicabilidad (stakeholders no técnicos + disposición Three Amigos). Si no se cumple, NO procede.
2. Detectar stack y elegir herramienta según regla 2026.
3. Instalación y configuración (playwright-bdd, @cucumber/cucumber, etc.).
4. Smoke test BDD que pasa, luego se borra.
5. (Opcional) Primera sesión Three Amigos asistida con Example Mapping.
6. Generar primer `.feature`.
7. **Validación humana obligatoria del `.feature`** antes de generar step definitions.
8. Generar step definitions reutilizando los existentes.
9. Documentar ADR + tickets en roadmap.
10. Cierre con `On(task_complete)`.

### 5. Modificado: `prompts/skills/tests-skill.md`

**Estado:** referencias a skills hermanas + nueva sección "Cuándo escalar a integration o E2E" + mención a Vitest 2026.

- Cabecera nueva con referencias a `integration-testing-skill.md`, `visual-testing-skill.md`, `bdd-skill.md`, `legacy-testing-skill.md`, `ai-testing-skill.md`.
- Sección "JavaScript/TypeScript + Jest" ahora "JS/TS + Jest o Vitest" con recomendación 2026 (Vitest preferido para proyectos NUEVOS sobre Vite/Next 15+/React 19, mantener Jest si ya está, React Native sigue con Jest) y ejemplo de `vi.mock`.
- Sección nueva "Cuándo escalar a integration o E2E" con tabla decisional + reglas operativas (¿necesita navegador? E2E; ¿mockea dependencias? unit; ¿usa BD/MSW reales? integration).

### 6. Modificado: `prompts/skills/visual-testing-skill.md`

**Estado:** cabecera ampliada + referencia cruzada a `ai-testing-skill.md` + nueva sección de alternativas SaaS.

- Cabecera referencia a `tests-skill`, `integration-testing-skill`, `bdd-skill` y carga adicional de `ai-testing-skill` cuando se usen agentes IA para tests E2E.
- "Soporta Playwright MCP server..." ampliado a "Playwright MCP y Playwright CLI" con apuntador a `ai-testing-skill.md` para setup y prompting.
- Sección nueva "Alternativas SaaS para visual regression" con tabla (Chromatic / Percy / Applitools / Argos CI / Lost Pixel) + recomendación por escenario + distinción entre visual regression con ML clásico vs visual AI semántico.

### 7. Modificado: `docs/testing-strategy.md`

**Estado:** plantilla SSOT ampliada con 4 secciones nuevas + corrección de typo.

- **Sección nueva al inicio "Pirámide de tests del proyecto"** con tabla de 5 niveles (unit / integración / E2E + visual + a11y / BDD opcional / legacy condicional) y nota sobre pirámide flexible 2026 (Cohn vs Testing Trophy) + checkboxes.
- **Sección nueva "BDD (opcional)"** con plantilla rellenable: herramienta elegida, estructura de carpetas, política de adopción, tags semánticos.
- **Sección nueva "Testing con IA"** con plantilla: agentes en uso, trazabilidad de prompts, gates en CI.
- **Sección nueva "Cumplimiento normativo (si aplica)"**: GDPR/RGPD + EU AI Act (AI literacy, alto riesgo Annex III, DPA con proveedor LLM).
- Typo corregido: "frigiles" → "frágiles".

### 8. Modificado: `prompts/architect-brain.md`

**Estado:** versión v4.4 → v4.5 + protocolo `On(testing_setup)` ampliado + protocolo `On(bdd_setup)` nuevo + checklist `On(task_complete)` ampliado + Restricciones y Calidad ampliadas.

- Cabecera v4.5 (Testing 2026 Edition).
- `On(testing_setup)` pasa de **10 a 13 pasos**:
  - Paso 6 nuevo: si hay APIs/BD y nivel T 2 o 3, instala Supertest + Testcontainers + MSW y carga `integration-testing-skill.md`.
  - Paso 10 nuevo: pregunta opt-in sobre adopción de BDD (si stakeholders no técnicos + disposición Three Amigos).
  - Paso 11 nuevo: pregunta opt-in sobre uso de IA en testing, carga `ai-testing-skill.md`, registra ADR con proveedor LLM y región.
- **Protocolo nuevo `On(bdd_setup)`** insertado antes de `On(database_setup)` con 11 puntos resumen (apunta a `.windsurf/workflows/bdd.md` para el detalle).
- `On(task_complete)` añade dos líneas al checklist: "Features BDD añadidas/modificadas" y "Tests generados por IA con `*.prompt.md` asociado".
- "Restricciones y Calidad" añade 4 reglas nuevas (BDD sin Three Amigos = disfraz, un solo `When` por escenario, tests IA con `*.prompt.md` + code review, GDPR + EU AI Act).

### 9. Modificado: `.windsurfrules`

**Estado:** interrupción de seguridad ampliada + nueva fila en tabla de comandos + bloques TESTING/BDD/IA reescritos + 4 reglas de oro nuevas.

- **Interrupción de seguridad**: 2 preguntas nuevas (pregunta 7 sobre BDD/Three Amigos, pregunta 8 sobre uso de IA en tests).
- **Tabla de comandos del usuario**: nueva fila para `/bdd` ("adoptemos BDD", "vamos a hacer Gherkin", "setup playwright-bdd", "three amigos").
- **Bloque TESTING**: lista explícita de los 5 niveles de la pirámide con sus skills, líneas nuevas para `integration-testing-skill.md` y `ai-testing-skill.md`.
- **Bloque BDD nuevo** (opcional, solo si aplica): regla de Three Amigos previa, regla de elección 2026 por stack, reglas no negociables de Gherkin, carga de `bdd-skill.md`.
- **Bloque TESTING CON IA nuevo** (si aplica): `*.prompt.md` obligatorio, code review humano, checklist de calidad, distinción ML clásico vs LLM, GDPR, EU AI Act + AI literacy, carga de `ai-testing-skill.md`.
- **Reglas de oro 14-17 nuevas**: BDD sin Three Amigos = disfraz técnico, tests con IA trazables, AI literacy obligatoria, GDPR en testing con IA.
- Total: pasa de **13 a 17 reglas de oro**.

### 10. Modificado: `CLAUDE.md`

**Estado:** versión v4.4 → v4.5 + listado de skills + workflows + reglas de oro actualizados.

- Cabecera v4.5 con descripción ampliada ("tests unit + integración + E2E + BDD + IA-asistido").
- Tabla de comandos pasa de 12 a 13 (añade `/bdd`).
- Lista de skills auxiliares pasa de 6 a 9: añade `integration-testing-skill.md`, `bdd-skill.md`, `ai-testing-skill.md`. Reordenadas para que las testing-related queden juntas.
- Reglas de oro pasan de 10 a 13: añade BDD sin Three Amigos, tests con IA trazables, AI literacy + GDPR.
- Interrupción de seguridad: 2 preguntas nuevas (BDD, uso de IA en tests).

### 11. Modificado: `AGENTS.md`

**Estado:** versión v4.4 → v4.5 + listado de comandos + skills + reglas de oro actualizados.

- Cabecera v4.5.
- Comandos: añade `/bdd`.
- Skills: añade `integration-testing-skill.md`, `bdd-skill.md`, `ai-testing-skill.md`. Reordenadas.
- Reglas de oro pasan de 10 a 13.

### 12. Modificado: `README.md`

**Estado:** versión v4.4 → v4.5 + tabla de comandos + pilares + estructura + reglas de oro actualizados.

- Tabla de comandos pasa de 12 a 13 (añade `/bdd`).
- "Qué hace el sistema" pasa de **8 a 9 pilares** (nuevo pilar 9: "Adopción controlada de BDD" con las cuatro garantías: verificación previa, stack 2026 por regla, Three Amigos asistida, validación humana obligatoria).
- Estructura de archivos: 13 workflows (antes 12) con `bdd.md`, 9 skills (antes 6) con las tres nuevas. Añade `CLAUDE.md` que faltaba en el árbol.
- Reglas de oro pasan de **13 a 17**: nuevas 14 (BDD sin Three Amigos), 15 (tests con IA trazables), 16 (AI literacy obligatoria), 17 (GDPR en testing con IA).
- "Sobre este sistema" añade entrada v4.5.
- "Soporte para Claude Code" actualizado a v4.5.

### 13. Modificado: `inicio-chat.txt`

**Estado:** versión v4.4 → v4.5 + tabla de comandos + sección detallada + reglas de oro + estructura + protocolo de defensa actualizados.

- Cabecera v4.5.
- Tabla de 13 comandos (añade `/bdd`) + ejemplo de uso.
- Sección detallada nueva `/bdd` con los 10 pasos del protocolo.
- Reglas de oro pasan de 13 a 17 (añade 14-17).
- Estructura de archivos actualizada con `CLAUDE.md`, 13 workflows (incluye `bdd.md`) y 9 skills (incluye las tres nuevas).
- Protocolo de defensa amplía menciones a BDD sin Three Amigos y tests con IA sin trazabilidad.

---

## Migración

Para integrar v4.5 en tu repo local:

1. Reemplaza la carpeta entera `junior-doc-gen-tests/` con la del repo actualizado.
2. O bien, añade/reemplaza solo estos archivos individualmente:

   **Nuevos:**
   - `prompts/skills/bdd-skill.md`
   - `prompts/skills/integration-testing-skill.md`
   - `prompts/skills/ai-testing-skill.md`
   - `.windsurf/workflows/bdd.md`
   - `CHANGELOG-v4.5.md` (este archivo)

   **Modificados:**
   - `prompts/skills/tests-skill.md`
   - `prompts/skills/visual-testing-skill.md`
   - `prompts/architect-brain.md`
   - `.windsurfrules`
   - `docs/testing-strategy.md`
   - `CLAUDE.md`
   - `AGENTS.md`
   - `README.md`
   - `inicio-chat.txt`

No hay cambios destructivos. Los workflows existentes (`/inicio`, `/regularizar`, `/cuestionar`, `/revisar-bd`, `/revisar-frontend`, etc.) siguen funcionando igual. El sistema es retrocompatible.

**Para proyectos en curso bajo v4.4**: los protocolos nuevos son opt-in. `On(testing_setup)` ahora pregunta por BDD e IA testing en sus pasos 10 y 11, pero los proyectos viejos no reciben las preguntas retroactivamente. Para adoptar BDD en un proyecto existente, invocar `/bdd` manualmente.

---

## Verificación post-migración

1. Abre Windsurf en un proyecto cualquiera con esta estructura.
2. En Cascade escribe `/` — deberían aparecer **13 comandos** en el desplegable, incluido `/bdd`.
3. Pide a Cascade: "Lee `@prompts/skills/bdd-skill.md` y dime qué herramienta BDD recomienda para JS/TS con UI."
   - Si responde "playwright-bdd" → integrado correctamente.
4. Pide: "Lee `@prompts/skills/ai-testing-skill.md` y dime cuántos tokens consume aproximadamente Playwright MCP vs Playwright CLI para una tarea típica."
   - Debería responder: "Playwright MCP ~114k tokens vs Playwright CLI ~27k tokens, recomendación oficial Microsoft 2026 de usar CLI cuando el agente tiene filesystem."
5. Pide: "Si arranco un proyecto Node + Postgres con frontend React, ¿qué dependencias de testing me instalas en `On(testing_setup)`?"
   - Debería responder: Vitest + RTL + vitest-axe + Playwright + @axe-core/playwright + Supertest + Testcontainers + MSW. Y debería preguntar opt-in por BDD y uso de IA.

---

## Lo que NO entró en v4.5 (decisiones explícitas)

- **No se añadió un protocolo `On(ai_testing_setup)` separado**. La adopción de IA en testing se gestiona como pregunta opt-in dentro de `On(testing_setup)` (paso 11) + carga de `ai-testing-skill.md`. Crear un protocolo aparte habría duplicado lógica sin ganar separación real.
- **No se añadió workflow `/test-ia`**. El flujo IA-asistido es transversal a las skills existentes (`tests-skill`, `integration-testing-skill`, `visual-testing-skill`, `bdd-skill`); convertirlo en un workflow propio habría fragmentado la documentación. La skill `ai-testing-skill.md` actúa como referencia cargable a demanda.
- **No se cubrió MSW de forma exhaustiva como skill propia**. MSW vive dentro de `integration-testing-skill.md` con patrón completo (handlers compartidos entre integration y E2E). Si en el futuro MSW gana suficiente complejidad propia (handlers tipados, devtools, integración con Storybook), se podría extraer.
- **No se cubrió contract testing (Pact) más allá de mención breve**. Es relevante para microservicios pero no para la mayoría de proyectos del scaffolding. Si un proyecto lo requiere, se carga manualmente o se genera skill dinámica.
- **No se añadió mutation testing como nuevo nivel de la pirámide**. Ya está cubierto por `/cuestionar` (mutation thinking) + opt-in a herramientas reales (Infection, StrykerJS, PIT, mutmut). No es una capa, es una técnica de auditoría sobre las capas existentes.
- **No se rompió la compatibilidad con Cucumber.js puro**. Aunque playwright-bdd se recomienda para JS/TS con UI, `bdd-skill.md` cubre también `@cucumber/cucumber` para APIs sin UI y proyectos legacy. La elección sigue siendo del usuario.
- **No se integraron herramientas SaaS de pago como dependencia obligatoria**. Octomind, Mabl, Applitools, Functionize, etc. aparecen en `ai-testing-skill.md` como referencia con ejes de decisión, no como recomendación por defecto. El stack canónico sigue siendo Vitest + Playwright + Testcontainers + MSW, todo open source.
