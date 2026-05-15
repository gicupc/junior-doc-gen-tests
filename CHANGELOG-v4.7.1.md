# Changelog v4.7 → v4.7.1 (Windsurf Edition — Template Detection Refined)

Sesión de actualización: 15 de mayo de 2026

Origen: durante el uso real de v4.7 en el proyecto `quellevoyo` (entrega final del bootcamp AI4Devs), se detectó que el fix de v4.7 dependía de recordatorios visibles al usuario para garantizar que el marcador `<!-- TEMPLATE-PLACEHOLDER -->` se eliminara al rellenar cada doc. Si el usuario tenía que saber del marcador, el sistema no era 100% autónomo. Esta release v4.7.1 mueve la responsabilidad enteramente al sistema con un paso de verificación final obligatorio.

---

## Resumen del cambio

v4.7.1 endurece el fix de v4.7 con **tres refuerzos defensivos** que hacen que el sistema garantice por sí mismo la integridad del scaffolding tras rellenar los docs, sin que el usuario tenga que conocer la existencia del marcador `TEMPLATE-PLACEHOLDER`:

1. **Paso de verificación final obligatorio en `inicio.md`** — tras generar los docs, Cascade ejecuta automáticamente una comprobación equivalente a `head -1 docs/*.md | grep -c "TEMPLATE-PLACEHOLDER"`. Si el resultado es distinto de 0, repara los archivos afectados y avisa al usuario en el resumen final con un mensaje del tipo `⚠️ Reparado: marcador residual en docs/X.md eliminado tras la generación.`

2. **Refuerzo en `On(project_start)` del architect-brain** — el sub-paso de generación del SSOT incorpora dos instrucciones explícitas: (a) ELIMINAR la primera línea con marcador al rellenar cada doc, y (b) VERIFICACIÓN POST-GENERACIÓN no opcional. La frase "no opcional" se incluye textualmente para que ningún modelo decida saltarse el paso interpretándolo como sugerencia.

3. **Nuevo bloque "Integridad del scaffolding (anti-template-residual)" en `On(task_complete)` checklist** — el cierre de cualquier tarea que cree o modifique docs del SSOT incluye obligatoriamente el check de que ningún `/docs/*.md` empieza con el marcador. Esto convierte la auditoría en parte rutinaria del cierre de tareas, no solo de `/inicio`.

Ningún cambio rompe v4.7. v4.7.1 es estrictamente aditivo: si el usuario ya tenía v4.7 funcionando, v4.7.1 solo añade defensas internas; los proyectos en curso (como `quellevoyo`) no necesitan re-aplicar nada porque las verificaciones nuevas son inocuas si los docs ya están bien.

---

## Cambios

### 1. Modificado: `.windsurf/workflows/inicio.md`

**Estado:** reescrito (mismo archivo, contenido reforzado).

Cambios estructurales sobre v4.7:

- **Paso 7 nuevo — "VERIFICACION FINAL OBLIGATORIA — antiresiduo de templates"**. Antes del resumen final (que pasa a ser paso 8), Cascade ejecuta una verificación automática: para cada archivo en `/docs/*.md`, comprueba que la primera línea NO sea `<!-- TEMPLATE-PLACEHOLDER -->`. Si encuentra alguno, lo repara y avisa al usuario en el resumen final.
- **Frase clave añadida**: "Esta verificacion NO es opcional. Un solo doc con marcador residual hace que el sistema vuelva a tratar el proyecto como 'template sin rellenar' la proxima vez que alguien ejecute /inicio, perdiendo el trabajo previo."
- **Renumeración**: paso de resumen final pasa de 7 a 8. Los pasos 1-6 mantienen contenido idéntico a v4.7.

### 2. Modificado: `prompts/architect-brain.md`

**Cambios**:

- **Header bump**: `v4.7 (Template Detection Edition + ...)` → `v4.7.1 (Template Detection Refined + ...)`.

- **`On(project_start)` paso 1 ampliado** con dos sub-instrucciones que antes solo estaban en `inicio.md`:
  - "Si los archivos de `/docs/` ya existian con la primera linea `<!-- TEMPLATE-PLACEHOLDER -->` (templates del scaffolding), ELIMINA esa primera linea al rellenarlos. Un archivo con contenido real NO debe conservar el marcador."
  - "**VERIFICACION POST-GENERACION (no opcional)**: tras rellenar todos los docs, comprueba que ningun archivo de `/docs/*.md` empieza con `<!-- TEMPLATE-PLACEHOLDER -->`."
  
  Razón: las instrucciones tenían que existir TAMBIÉN en el architect-brain porque es el protocolo que se aplica cuando el usuario invoca `On(project_start)` directamente (no solo desde `/inicio`), por ejemplo en proyectos donde alguien lanza la entrevista BMADT manualmente.

- **`On(task_complete)` checklist amplía con un bloque nuevo** entre "Documentacion actualizada" y "Tests":
  ```
  Integridad del scaffolding (anti-template-residual):
  - [x] Ningun archivo `/docs/*.md` empieza con `<!-- TEMPLATE-PLACEHOLDER -->` (verificado con `head -1 docs/*.md`). Si esta tarea creo o modifico algun doc del SSOT, esta verificacion es obligatoria.
  ```
  Razón: cualquier tarea que toque docs del SSOT (no solo el setup inicial) puede dejar marcadores residuales si el modelo recrea un doc desde el template. El check en `On(task_complete)` lo previene como rutina.

### 3. Archivo nuevo: `CHANGELOG-v4.7.1.md`

Este archivo.

---

## Migración

### Archivos modificados

- `.windsurf/workflows/inicio.md`
- `prompts/architect-brain.md`

### Archivos nuevos

- `CHANGELOG-v4.7.1.md`

### Sin cambios

- `.windsurfrules` (la regla inmediata de detección de templates ya estaba ampliada en v4.7).
- Templates en `docs/` (marcador `<!-- TEMPLATE-PLACEHOLDER -->` ya estaba en v4.7).
- Resto de protocolos `On(...)`.

### Comportamiento para proyectos existentes

v4.7.1 es estrictamente aditivo sobre v4.7:

- Proyectos en v4.7 con docs bien rellenados (sin marcador residual): no necesitan acción alguna. Las verificaciones nuevas son inocuas.
- Proyectos en v4.7 donde algún doc quedó con marcador residual por bug del modelo: el primer `/cerrar` o `/sync` posterior a la actualización del scaffolding lo detectará y avisará.
- Proyectos en v4.6 o anteriores (sin el fix de v4.7): aplicar primero v4.7 (copiar marcador a templates + `.windsurfrules` ampliado + `inicio.md` con paso 1 de detección) y luego v4.7.1 (los refuerzos descritos aquí). O aplicar todo de golpe como un único bump v4.7.1, que es lo que se ha hecho en la versión paralela de Cursor.

---

## Verificación post-migración

Tras aplicar v4.7.1, comprobar:

1. **Header de `architect-brain.md`**: empieza con `# Architect-Brain v4.7.1 (Template Detection Refined + ...)`.
2. **`inicio.md` paso 7**: contiene la cadena `VERIFICACION FINAL OBLIGATORIA — antiresiduo de templates`.
3. **`On(project_start)` en architect-brain**: contiene la cadena `VERIFICACION POST-GENERACION (no opcional)`.
4. **`On(task_complete)` en architect-brain**: contiene el bloque `Integridad del scaffolding (anti-template-residual)`.
5. **Smoke test funcional**: copiar el scaffolding a una carpeta nueva, ejecutar `/inicio`, completar BMADT con respuestas mínimas, comprobar que el resumen final NO contiene `⚠️ Reparado: marcador residual...` (porque el sistema debería haberlo limpiado durante la generación, no haber tenido que repararlo después). Si aparece `⚠️ Reparado:`, el modelo se olvidó del paso de eliminar el marcador al rellenar y la verificación final lo arregló — esto es ACEPTABLE (la red de seguridad funcionó) pero indica que el refuerzo de `On(project_start)` no fue suficiente.

---

## Paridad con Cursor v4.7.1

Esta release tiene **paridad funcional total** con la versión Cursor (`junior-doc-gen-tests-cursor`) v4.7.1, publicada en el mismo bump. En Cursor, v4.7.1 es más extenso porque combina dos pasos que en Windsurf se publicaron secuencialmente:

- **v4.7 (Windsurf, esta tarde)**: introducción del marcador `<!-- TEMPLATE-PLACEHOLDER -->` en los 9 templates de `docs/`, regla inmediata ampliada en `.windsurfrules`, paso 1 de detección en `inicio.md`.
- **v4.7.1 (Windsurf, este bump)**: refuerzos descritos aquí.

En Cursor todo se publica en un solo bump v4.7.1 (saltando v4.7 intermedio) porque el repo de Cursor estaba en v4.6 y carecía del fix base. El CHANGELOG-v4.7.1.md de Cursor documenta explícitamente esta combinación.

Las únicas diferencias estructurales son las propias de cada motor:

- **Windsurf**: workflow en `.windsurf/workflows/inicio.md` con frontmatter `description:`, reglas globales en `.windsurfrules`, protocolos en `prompts/architect-brain.md`.
- **Cursor**: skill en `.cursor/skills/inicio/SKILL.md` con frontmatter `name:` + `description:` + `disable-model-invocation: true`, reglas globales en `.cursor/rules/architect-brain.mdc`, protocolos en `prompts/architect-brain.md` (igual nombre que Windsurf, contenido equivalente con paths adaptados).

---

## Lo que NO entró en v4.7.1 (intencionado)

- **Separación de `architecture.md` en `SYSTEM-INFO.md` (meta-doc del scaffolding) + `architecture.md` (template real)**. Detectado durante la sesión de quellevoyo: el `docs/architecture.md` del scaffolding contiene meta-documentación del propio sistema Architect-Brain, no es un template puro. Esto puede confundir a Cascade al rellenarlo. Apuntado como ítem para v4.8.
- **Hook git pre-commit que verifica marcadores residuales**. Considerado pero descartado: añade dependencia de git hooks (no todos los usuarios los activan), y la verificación en `On(task_complete)` cubre el caso real. Si en v4.8 hace falta endurecer más, se valora.
- **Renombre del marcador**. `<!-- TEMPLATE-PLACEHOLDER -->` es funcional y descriptivo. No se renombra.
- **Cambio en el formato del marcador a algo no-HTML** (por ejemplo `# TEMPLATE-PLACEHOLDER` como heading). Descartado: como comentario HTML, el marcador es invisible en preview de markdown, lo que es exactamente lo que queremos (no contamina la lectura del usuario si abre el template).

---

## Próxima parada

v4.8 candidatos (sin compromiso):

- Refactor del meta-doc: separar `architecture.md` de SSOT en `SYSTEM-INFO.md` (sistema) + `architecture.md` (template real del proyecto).
- Si el módulo 12 del bootcamp introduce algo nuevo, bump correspondiente.
- Si surge un caso real en `quellevoyo` o `PromptVault` que requiera nuevo skill o workflow.
