# CHANGELOG v4.7 — Template Placeholder Detection

**Fecha**: 2026-05-15
**Tipo**: Bugfix + Quality of Life
**Detectado por**: Gicu (GFS) al arrancar el proyecto `quellevoyo`.

---

## Resumen

Se introduce el marcador `<!-- TEMPLATE-PLACEHOLDER -->` como primera linea de todos los archivos en `/docs/` del scaffolding. La regla inmediata y el workflow `/inicio` detectan este marcador para distinguir templates sin rellenar de documentacion real, eliminando la friccion al arrancar proyectos nuevos.

---

## Problema que resuelve

Cuando un usuario copiaba el scaffolding `junior-doc-gen-tests` a un proyecto nuevo y ejecutaba `/inicio`, Cascade detectaba los archivos `.md` en `/docs/` y, al considerar que ya habia documentacion (aunque fuera contenido placeholder tipo `[NOMBRE_DEL_PROYECTO]`), mostraba un prompt defensivo pidiendo confirmacion antes de proceder. Esto generaba friccion en el caso mas comun de uso (proyecto nuevo con scaffolding fresco).

El comportamiento defensivo de Cascade era correcto — prefiere preguntar antes que pisar trabajo previo. El problema estaba en que el sistema no proporcionaba a Cascade una forma inequivoca de saber que los archivos eran templates del scaffolding, no documentacion real.

---

## Cambios

### 1. Marcador `<!-- TEMPLATE-PLACEHOLDER -->` en todos los templates de `/docs/`

Todos los archivos en `/docs/` del scaffolding ahora empiezan con:

```
<!-- TEMPLATE-PLACEHOLDER -->
```

Es un comentario HTML invisible al renderizar Markdown. Indica inequivocamente que el archivo es un template del scaffolding sin rellenar.

Archivos afectados:
- `docs/prd.md`
- `docs/architecture.md`
- `docs/blueprints.md`
- `docs/user-stories.md`
- `docs/roadmap.md`
- `docs/testing-strategy.md`
- `docs/database-strategy.md`
- `docs/decisions-log.md`
- `docs/supply-chain-strategy.md`

### 2. Regla inmediata ampliada en `.windsurfrules`

La regla pasa de cubrir 1 caso a cubrir 3:

- **Antes**: "si falta documentacion → arranca inicio/regularizacion".
- **Ahora**: detecta los 3 casos posibles:
  1. `/docs/` vacio o inexistente → arranca `On(project_start)`.
  2. `/docs/` poblado pero TODOS los `.md` tienen el marcador → trata como proyecto nuevo, arranca `On(project_start)`, sobrescribe templates al rellenar.
  3. `/docs/` poblado y al menos un `.md` NO tiene el marcador → hay trabajo previo real, ofrece menu del CASO B (regularizar / evolucionar / reparar / regularizar+tests+BD+frontend).

### 3. Workflow `/inicio` con paso 1 de deteccion explicita

`/inicio` ahora hace deteccion explicita del estado de `/docs/` antes de arrancar la entrevista BMADT, asegurando autocontencion (no depende de que Cascade aplique correctamente la regla inmediata). El paso 1 implementa la misma logica de los 3 casos.

Ademas, se anade instruccion explicita en el paso 4 para que la IA ELIMINE el marcador `TEMPLATE-PLACEHOLDER` al rellenar cada archivo, dejando el doc como SSOT real del proyecto.

---

## Limitaciones conocidas (planificado para v4.8)

- **`docs/architecture.md`** del scaffolding actualmente contiene meta-documentacion del sistema (no es un template real). Sigue funcionando como template para el sistema actual, pero al rellenarse pierde esa meta-doc. Plan futuro: separar en `docs/SYSTEM-INFO.md` (no editable, fuera del scope del marcador) y `docs/architecture.md` (template real).

---

## Migracion

Proyectos existentes que usen v4.6 **NO requieren migracion**. Esta version es retrocompatible:

- Si los docs del proyecto YA tienen contenido real (sin marcador), el sistema los respeta y aplica el flujo de CASO B (regularizar/evolucionar).
- Si quieres usar el nuevo flujo en un proyecto existente, anade manualmente `<!-- TEMPLATE-PLACEHOLDER -->` como primera linea de los docs que quieras tratar como templates.

---

## Verificacion

Para verificar que la solucion funciona en un proyecto nuevo:

1. Copia el scaffolding v4.7 a una carpeta nueva.
2. Abre Windsurf en esa carpeta.
3. Ejecuta `/inicio` en Cascade.
4. **Esperado**: Cascade arranca la entrevista BMADT directamente, sin prompt defensivo.
5. Al rellenar los docs, Cascade elimina el marcador `TEMPLATE-PLACEHOLDER` de cada archivo.

Si Cascade muestra el prompt defensivo, significa que algun archivo ha perdido el marcador o que la regla no se aplico correctamente. Revisa `/docs/*.md` y `.windsurfrules`.
