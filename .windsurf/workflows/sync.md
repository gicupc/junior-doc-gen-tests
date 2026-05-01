---
description: Verificar alineacion entre codigo y documentacion - detecta drift
---

# /sync — Revisa sincronizacion

Ejecuta el protocolo `On(sync_check)` definido en `@prompts/architect-brain.md`.

Pasos:

1. Escanea el estado real del proyecto:
   - Archivos fuente y estructura de carpetas.
   - Dependencias en `package.json` / `composer.json`.
   - Tests existentes y su estado (passing/failing/skipped).
   - Commits recientes si hay git.
   - Si hay BD: schema actual y migraciones aplicadas.

2. Compara con los docs vigentes en `/docs/`:
   - `architecture.md` — ¿refleja la estructura real del codigo?
   - `user-stories.md` — ¿todas las funcionalidades del codigo estan documentadas?
   - `roadmap.md` — ¿los tickets `[x]` tienen codigo asociado?
   - `testing-strategy.md` — ¿coincide el scope real con el declarado?
   - `database-strategy.md` (si existe) — ¿las tablas reales coinciden con las documentadas?
   - `decisions-log.md` — ¿hay decisiones tomadas que no estan registradas?
   - `blueprints.md` — ¿los patrones actuales coinciden con los documentados?

3. Reporta las divergencias detectadas en un formato claro:
   ```
   🔴 Criticas: [divergencias que bloquean trabajo futuro]
   🟡 Moderadas: [drift que conviene arreglar pronto]
   🟢 Menores: [drift cosmetico o informativo]
   ```

4. NO arregles nada automaticamente. Al final, pregunta al usuario: "¿Por cual de estos drifts quieres empezar?"
