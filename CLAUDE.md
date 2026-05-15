# Project Instructions for Claude Code

Este proyecto usa el sistema **Architect-Brain v4.5** para garantizar calidad en codigo, documentacion, tests (unit + integracion + E2E + BDD + IA-asistido), base de datos y frontend.

> **Nota:** Claude Code lee este archivo (`CLAUDE.md`); Windsurf lee `.windsurfrules`; Cursor lee `.cursor/rules/architect-brain.mdc`. Este archivo apunta a las mismas fuentes de verdad que esos otros, para que el sistema funcione identico en los tres entornos. NO duplica contenido.

---

## Fuentes de verdad — leer ANTES de cualquier accion

Cuando inicies sesion en este proyecto, lee en este orden:

1. **`.windsurfrules`** — reglas globales operativas del sistema (mismo contenido que aplicas tu, Claude Code, en cada turno).
2. **`prompts/architect-brain.md`** — protocolos completos `On(...)`: project_start, testing_setup, database_setup, frontend_setup, database_audit, frontend_audit, task_complete, sync_check, ticket_close, implementation_phase, resume_project, revisit_decision.
3. **`AGENTS.md`** — resumen rapido y stack canonico.
4. **`/docs/`** — SSOT del proyecto concreto en el que estas: `prd.md`, `architecture.md`, `blueprints.md`, `user-stories.md`, `roadmap.md`, `testing-strategy.md`, `database-strategy.md` (si hay BD), `frontend-strategy.md` (si hay frontend), `decisions-log.md`.

---

## Comandos disponibles

Aunque Claude Code soporta `.claude/commands/` para slash commands nativos, este sistema vive en `.windsurf/workflows/` (compartido con Windsurf) y se invocan en lenguaje natural o con `/`. Cuando el usuario diga uno de estos, lee el workflow correspondiente y ejecuta su protocolo:

- `/inicio` → `.windsurf/workflows/inicio.md`
- `/onboarding` → `.windsurf/workflows/onboarding.md`
- `/regularizar` → `.windsurf/workflows/regularizar.md`
- `/hoy` → `.windsurf/workflows/hoy.md`
- `/nueva` → `.windsurf/workflows/nueva.md`
- `/reparar` → `.windsurf/workflows/reparar.md`
- `/decidir` → `.windsurf/workflows/decidir.md`
- `/sync` → `.windsurf/workflows/sync.md`
- `/cerrar` → `.windsurf/workflows/cerrar.md`
- `/cuestionar` → `.windsurf/workflows/cuestionar.md`
- `/revisar-bd` → `.windsurf/workflows/revisar-bd.md`
- `/revisar-frontend` → `.windsurf/workflows/revisar-frontend.md`
- `/bdd` → `.windsurf/workflows/bdd.md` *(nuevo en v4.5)*

Skills auxiliares (cargar con `@` o automaticamente segun contexto):

- `prompts/skills/tests-skill.md` — convenciones de tests unitarios por stack.
- `prompts/skills/integration-testing-skill.md` — Supertest, MSW, Testcontainers para integration tests *(nuevo en v4.5)*.
- `prompts/skills/visual-testing-skill.md` — tests UI con axe + Playwright + visual diff + alternativas SaaS.
- `prompts/skills/bdd-skill.md` — BDD con Gherkin, playwright-bdd, Three Amigos, anti-patrones IA *(nuevo en v4.5)*.
- `prompts/skills/ai-testing-skill.md` — Playwright MCP/CLI, prompting, trazabilidad de prompts, GDPR + EU AI Act *(nuevo en v4.5)*.
- `prompts/skills/legacy-testing-skill.md` — characterization tests para legacy.
- `prompts/skills/database-skill.md` — diseno y auditoria de BD.
- `prompts/skills/frontend-skill.md` — diseno y auditoria de frontend.
- `prompts/skills/prompt-skill.md` — manual del usuario para redactar prompts profesionales (RACEO).

---

## Reglas de oro (resumen operativo)

Estas son las que aplicas tu, Claude Code, en cada turno. Para el detalle, lee `.windsurfrules`:

1. **SSOT**: los docs de `/docs/` ganan sobre tu memoria del proyecto.
2. **Antidrift**: ninguna tarea se cierra sin el checklist `On(task_complete)` visible.
3. **Tests como contrato**: ticket cerrado solo cuando los tests asociados estan en verde.
4. **Decisiones trazables**: si no esta en `decisions-log.md`, no existe como decision oficial.
5. **Legacy primero congelar, luego cambiar**: en codigo legacy, characterization tests antes de modificar.
6. **BD primero auditar, luego cambiar**: ejecutar `/revisar-bd` antes de cambios destructivos en BD.
7. **Frontend primero medir, luego optimizar**: ejecutar `/revisar-frontend` antes de optimizaciones a ojo. WCAG 2.2 AA y CWV son criterios objetivos.
8. **Implementacion paso a paso**: si el plan toca >2 archivos, aplicar `On(implementation_phase)` con paradas obligatorias entre archivos.
9. **Cero `any` en frontend**, cero secretos en `NEXT_PUBLIC_*`/`VITE_*`/`PUBLIC_*`, validacion cliente Y servidor con Zod.
10. **Codigo IA bajo revision**: outputs de v0/Lovable/Bolt/Figma Make pasan por revision humana antes de produccion.
11. **BDD sin Three Amigos es disfraz tecnico**: adoptar BDD requiere conversacion previa con stakeholders no tecnicos.
12. **Tests con IA trazables**: cada test generado por LLM lleva su `*.prompt.md` versionado y pasa code review humano antes de mergear.
13. **AI literacy obligatoria + GDPR**: equipos que usan IA en testing deben tener formacion en uso responsable; PII no se envia a LLMs externos sin DPA valido.

---

## Interrupcion de seguridad (obligatoria antes de tocar codigo)

Antes de modificar, anadir o reparar codigo, pregunta:

```
He recibido tu peticion. Segun el protocolo de calidad:
1. ¿Actualizo docs/user-stories.md y docs/roadmap.md primero?
2. ¿Hay nuevas reglas para docs/blueprints.md?
3. ¿Los tests asociados estan planificados en docs/testing-strategy.md?
4. ¿Esta peticion implica una decision no trivial que debo registrar en docs/decisions-log.md?
5. ¿Esta peticion toca la base de datos? Si es si, ¿debo actualizar docs/database-strategy.md?
6. ¿Esta peticion toca el frontend (componentes, paginas, estilos, a11y)? Si es si, ¿debo actualizar docs/frontend-strategy.md?
7. ¿Esta peticion requiere un escenario BDD nuevo o modificado? Si es si, ¿ya tuvimos la conversacion Three Amigos?
8. ¿Voy a usar IA para generar tests? Si es si, recordare la politica: *.prompt.md junto al test + code review humano.

No tocare el codigo hasta que confirmes la sincronizacion documental.
```

---

## Cierre de tarea (obligatorio)

Al terminar CUALQUIER tarea de codigo, muestra el bloque `On(task_complete)` definido en `prompts/architect-brain.md`. Sin ese bloque visible, la tarea NO esta cerrada.
