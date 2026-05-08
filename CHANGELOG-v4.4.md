# Changelog v4.3 → v4.4 (Frontend & MCP Edition)

Sesión de actualización: 8 de mayo de 2026

Origen: lecciones extraídas del módulo 10 de AI4Devs (frontend best practices 2026 + design-to-code con AI). Convergencia del stack en 2026 (React 19 + Next 15/16 + Tailwind v4 + shadcn/ui), consolidación de MCP (donado al Linux Foundation en diciembre 2025), entrada en vigor de la European Accessibility Act (junio 2025) y datos sobre vulnerabilidades en código generado por IA (~40-45% sin revisión).

---

## Resumen del cambio

El sistema era **backend-first**: tenía protocolos sólidos para tests, BD y trazabilidad de decisiones, pero el frontend era un punto ciego. v4.4 lo convierte en ciudadano de primera clase, paralelo a cómo v4.2 hizo lo mismo con base de datos. La filosofía es idéntica: **detectar señales, configurar entorno automáticamente, registrar ADRs, auditar antes de tocar**.

Adicionalmente, v4.4 reconoce explícitamente los MCP servers como capacidades que extienden lo que el agente puede hacer (Figma Dev Mode MCP, shadcn MCP, Chrome DevTools MCP, Playwright MCP), y añade `AGENTS.md` como estándar oficial de instrucciones para agentes IA.

---

## Cambios

### 1. Skill nueva: `prompts/skills/frontend-skill.md`

**Estado:** archivo nuevo (19,4 KB).

Manual de la IA cuando el proyecto tiene frontend. Cubre dos escenarios:

- **Proyecto NUEVO**: stack canónico 2026 (Next.js 15/16 + TypeScript estricto + Tailwind v4 + shadcn/ui + Biome + Zod), estructura de carpetas, TypeScript estricto desde día 1, componentes <150 líneas, accesibilidad WCAG 2.2 AA como mínimo legal, Core Web Vitals como Definition of Done, seguridad frontend (cero secretos en `NEXT_PUBLIC_*`, no `dangerouslySetInnerHTML` sin DOMPurify, validación cliente Y servidor con Zod, headers CSP/HSTS), MCP servers recomendados por escenario, vibe coding (cuándo SÍ y cuándo NO).
- **Proyecto EXISTENTE**: 4 auditorías (stack y mantenibilidad, seguridad, rendimiento CWV, accesibilidad WCAG 2.2 AA) con outputs concretos.

### 2. Skill nueva: `prompts/skills/visual-testing-skill.md`

**Estado:** archivo nuevo (12,9 KB).

Complementaria a `tests-skill.md`. Cubre los 3 niveles de testing frontend:

- **Nivel 2 (componente)**: Vitest + React Testing Library + vitest-axe. Cada componente con UI no trivial debe tener al menos un test `axe(container)` sin violaciones.
- **Nivel 3 (E2E + visual)**: Playwright + @axe-core/playwright + opcional pixelmatch + pngjs para visual diff. Loop automatizable: agente genera código → screenshot → compara con baseline → ajusta.

Incluye setup paso a paso con código real, convenciones de carpetas, scripts `package.json`, integración con el harness Architect-Brain y antipatrones a evitar.

### 3. Workflow nuevo: `.windsurf/workflows/revisar-frontend.md`

**Estado:** archivo nuevo (13 KB).

Slash command `/revisar-frontend` simétrico a `/revisar-bd`. Audita el frontend en 4 categorías sin modificar nada:

1. **Stack y mantenibilidad** — TS estricto, `any` extendido, componentes gigantes, mezcla de UI libs, dependencias obsoletas, linter ausente, tests ausentes, código IA sin revisar.
2. **Seguridad** — secretos en `NEXT_PUBLIC_*`, `dangerouslySetInnerHTML` sin sanitizar, validación solo en cliente, headers ausentes (CSP, HSTS, X-Frame-Options, Referrer-Policy), CORS abierto, `npm audit`.
3. **Rendimiento (CWV)** — LCP/INP/CLS reales con CrUX si hay despliegue, análisis estático si no (imágenes sin optimizar, bundle pesado, code-splitting, fonts).
4. **Accesibilidad WCAG 2.2 AA** — alt, labels, contraste, teclado, landmarks, headings, focus visible, `prefers-reduced-motion`. Honestidad obligatoria: tooling automatizado detecta ~30% de issues; el reporte documenta TODOs de validación manual con NVDA/VoiceOver.

Genera `docs/frontend-audit.md` (informe ejecutivo) y comparte hallazgos de seguridad con `docs/investigacion_seguridad.md` (compartido con auditoría de BD si existe). Crea tickets en `roadmap.md` por cada hallazgo crítico/alto.

Filosofía: **la IA detecta, el humano decide, el humano repara**.

### 4. Modificado: `prompts/architect-brain.md`

**Estado:** versión actualizada de v4.3 a v4.4 + dos protocolos nuevos.

Añadidos:

- **`On(frontend_setup)`** — se ejecuta automáticamente tras `On(testing_setup)` (y `On(database_setup)` si aplica) cuando el proyecto incluye frontend. Detecta o propone el stack canónico 2026, inicializa `tsconfig.json` estricto, linter con reglas a11y, configura headers de seguridad, pregunta sobre flujo diseño-a-código y propone MCP servers, genera `docs/frontend-strategy.md`, registra ADRs, añade tickets al `roadmap.md`.
- **`On(frontend_audit)`** — se ejecuta cuando el usuario invoca `/revisar-frontend` o automáticamente al elegir `/regularizar` opción [4] sobre proyecto con frontend.

Modificados:

- `On(project_start)` — paso 5 nuevo: dispara `On(frontend_setup)` automáticamente si M incluye frontend.
- `On(testing_setup)` — paso 5 ampliado: si hay frontend con T 2 o 3, instala RTL + vitest-axe + Playwright + @axe-core/playwright y carga `visual-testing-skill.md` como guía.
- `On(task_complete)` — checklist incluye `frontend-strategy.md` y "tests de a11y (si toca componente UI)".
- `On(sync_check)` — escanea `NEXT_PUBLIC_*` / `VITE_*` / `PUBLIC_*` con sufijos sospechosos (`*_KEY`, `*_TOKEN`, `*_SECRET`).
- `On(ticket_close)` — verifica tests de a11y si el ticket toca componente UI.
- Sección "Restricciones y Calidad" añade reglas frontend (cero `any`, WCAG 2.2 AA mínimo, CWV como DoD, validación cliente Y servidor con Zod, cero secretos en variables públicas).

### 5. Modificado: `.windsurfrules`

**Estado:** bloque FRONTEND nuevo + interrupción de seguridad ampliada + 3 reglas de oro nuevas + nueva fila en tabla de comandos.

- **Pregunta 6 nueva en interrupción de seguridad**: "¿Esta petición toca el frontend (componentes, páginas, estilos, a11y)? Si es sí, ¿debo actualizar `docs/frontend-strategy.md`?"
- **CASO A paso 6 nuevo**: si M implica frontend, ejecutar `On(frontend_setup)`.
- **CASO B opción [4] ampliada**: si hay frontend, ejecutar también `On(frontend_audit)`.
- **Nueva fila en tabla de comandos**: `/revisar-frontend` con disparadores ("revisa el frontend", "audita la UI", "¿mi frontend cumple a11y/CWV?").
- **Bloque FRONTEND nuevo**: stack canónico 2026, cero `any`, componentes <150 líneas, WCAG 2.2 AA como mínimo, CWV como DoD, validación cliente+servidor con Zod, cero secretos en variables públicas, no `dangerouslySetInnerHTML` sin DOMPurify, headers de seguridad, MCP servers por escenario, vibe coding restringido a prototipos.
- **Reglas de oro 11, 12, 13** nuevas: Frontend primero medir luego optimizar; MCP servers como capacidades; Código IA bajo revisión.
- DECISIONES amplía qué se considera no trivial: framework frontend, librería de componentes, decisiones a11y/CWV, MCP servers activados.

### 6. Nuevo: `AGENTS.md`

**Estado:** archivo nuevo (5,4 KB) — antes solo existía en la versión Cursor; ahora también en Windsurf.

Sigue el estándar `AGENTS.md` donado al Linux Foundation en diciembre 2025, soportado nativamente por Cursor, Windsurf, Copilot, Codex, Zed, Warp, Aider, Devin y otros. Permite portabilidad del proyecto entre IDEs.

Contiene: lista de 12 comandos, 6 skills auxiliares con sus rutas, stack canónico 2026, MCP servers recomendados, 10 reglas de oro, descripción del SSOT.

### 7. Modificado: `inicio-chat.txt`

**Estado:** tabla de 12 comandos (antes 11), sección detallada nueva para `/revisar-frontend`, 13 reglas de oro (antes 10), estructura de archivos actualizada con las dos skills nuevas y `frontend-strategy.md`, protocolo de defensa amplía a `/revisar-frontend`.

### 8. Modificado: `README.md`

**Estado:** versión 4.2 → 4.4 (saltó v4.3 que no actualizó este archivo).

- Tabla de 12 comandos en lugar de 10.
- Sección "Qué hace el sistema" pasa de 7 pilares a 8 (nuevo: Setup automático de frontend).
- Auditorías de calidad pasan de 3 a 4 niveles (nuevo: `/revisar-frontend`).
- Sección nueva "Stack canónico para frontend nuevo (2026)" con tabla del stack recomendado.
- Sección nueva "MCP servers recomendados" con tabla.
- Estructura de archivos actualizada (12 workflows, 6 skills, AGENTS.md, frontend-strategy.md).
- Reglas de oro pasan de 10 a 13.
- Sección "Sobre este sistema" actualizada con v4.3 y v4.4.

---

## Migración

Para integrar v4.4 en tu repo local:

1. Reemplaza la carpeta entera `junior-doc-gen-tests/` con la del repo actualizado.
2. O bien, añade/reemplaza solo estos archivos individualmente:
   - `prompts/skills/frontend-skill.md` (**nuevo**)
   - `prompts/skills/visual-testing-skill.md` (**nuevo**)
   - `.windsurf/workflows/revisar-frontend.md` (**nuevo**)
   - `AGENTS.md` (**nuevo en Windsurf**)
   - `prompts/architect-brain.md` (modificado)
   - `.windsurfrules` (modificado)
   - `inicio-chat.txt` (modificado)
   - `README.md` (modificado)

No hay cambios destructivos. Los workflows existentes (`/inicio`, `/regularizar`, `/revisar-bd`, etc.) siguen funcionando igual. El sistema es retrocompatible.

**Para proyectos en curso bajo v4.3**: los protocolos nuevos no se disparan retroactivamente. Si quieres aplicarlos a un proyecto frontend ya empezado, ejecuta `/revisar-frontend` para auditarlo, y `On(frontend_setup)` se ejecutará si re-disparas `/regularizar` con opción [4].

---

## Verificación post-migración

1. Abre Windsurf en un proyecto cualquiera con esta estructura.
2. En Cascade escribe `/` — deberían aparecer **12 comandos** en el desplegable, incluido `/revisar-frontend`.
3. Pide a Cascade: "Lee `@prompts/skills/frontend-skill.md` y dime cuál es el stack canónico 2026."
   - Si responde "Next.js 15/16 + TypeScript estricto + Tailwind v4 + shadcn/ui + Biome + Zod" → integrado correctamente.
4. Pide: "Si arranco un proyecto Next.js, ¿qué protocolo se dispara después del testing_setup?"
   - Debería responder: `On(frontend_setup)` con stack canónico, headers de seguridad, MCP servers recomendados y `frontend-strategy.md`.
5. Pide: "Si te paso un frontend Next.js para auditar, ¿qué categorías cubres?"
   - Debería responder: stack y mantenibilidad, seguridad, rendimiento (CWV), accesibilidad (WCAG 2.2 AA).

---

## Lo que NO entró en v4.4 (decisiones explícitas)

- **No se añadió `F` a BMADT como sexta dimensión**. El frontend es una decisión de M (Mecanismo), no una dimensión paralela. `On(frontend_setup)` se dispara automáticamente cuando M incluye frontend, igual que `On(database_setup)` se dispara cuando D incluye datos persistidos. Mantener BMADT con 5 dimensiones simplifica la entrevista inicial.
- **No se integró vibe coding (v0/Lovable/Bolt) como flujo principal**. Filosofía opuesta al sistema (velocidad inicial vs trazabilidad). Quedan documentados como prototipado desechable que debe pasar por revisión en Cursor/Windsurf antes de producción.
- **No se integró Figma Make ni Claude Design**. El primero no es production-ready (LogRocket, abril 2026); el segundo está en research preview. Reentrarán en futuras versiones cuando maduren.
- **AGENTS.md no se convirtió aún en SSOT principal de instrucciones**. `.windsurfrules` sigue siendo la fuente operativa para Cascade. El refactor a "AGENTS.md como SSOT, .windsurfrules como puntero delgado" se reserva para una hipotética v4.5 si demuestra valor real durante el uso de v4.4.


---

## Adicion post-v4.4: soporte para Claude Code

**Estado:** archivo nuevo `CLAUDE.md` en la raiz del proyecto (5 KB).

Permite usar el mismo sistema Architect-Brain v4.4 desde **Claude Code** (la CLI de Anthropic). El archivo es delgado y apunta a las fuentes de verdad existentes:

- `.windsurfrules` — reglas globales operativas.
- `prompts/architect-brain.md` — protocolos `On(...)` completos.
- `.windsurf/workflows/*.md` — slash commands.
- `prompts/skills/*.md` — skills auxiliares.

NO duplica contenido. Cualquier cambio en `.windsurfrules` o en los workflows se propaga automaticamente a Claude Code sin tocar `CLAUDE.md`.

**Cero impacto en Windsurf:** Windsurf sigue leyendo `.windsurfrules` y solo eso. Claude Code lee `CLAUDE.md` y solo eso. Cada IDE tiene su archivo de instrucciones, pero ambos consumen el mismo cerebro.

Razon de no haberlo hecho fuente unica (AGENTS.md como SSOT con `.windsurfrules` y `CLAUDE.md` como punteros delgados): para el caso de uso real (un solo desarrollador con Windsurf principal y Claude Code ocasional), el coste del refactor no compensa el ahorro futuro. v4.4.1 es la solucion pragmatica: un archivo extra de 5 KB que da soporte completo a un IDE adicional con cero coste de mantenimiento.
