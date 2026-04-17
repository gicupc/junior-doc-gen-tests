---
description: Tomar control de un proyecto existente - menu de regularizacion
---

# /regularizar — Proyecto existente

Voy a tomar control de un proyecto que ya tiene codigo pero documentacion incompleta o inexistente.

Pasos:

1. Escanea el proyecto actual:
   - Estructura de carpetas.
   - Archivos fuente principales (ignora `node_modules`, `vendor`, `dist`, `build`).
   - `package.json` / `composer.json` para detectar stack y dependencias.
   - Tests existentes si los hay.

2. Presenta al usuario un resumen de lo detectado:
   - Stack tecnico identificado.
   - Modulos principales.
   - Framework de tests (si existe) y su estado.
   - Documentacion existente en `/docs/` y que falta.

3. Ofrece el menu de 4 opciones:
   ```
   Segun lo detectado, puedes:
   [1] Regularizar documentacion — generar blueprints sin tests.
   [2] Evolucionar — anadir funcionalidad nueva (usa /nueva mejor).
   [3] Reparar — corregir un bug puntual (usa /reparar mejor).
   [4] Regularizar + Cubrir con tests — blueprints + setup de tests + tests para logica critica.
   
   ¿Que opcion eliges?
   ```

4. Segun la eleccion:

   **Si elige [1] Regularizar:**
   - Crea `docs/blueprints.md` extrayendo patrones reales del codigo.
   - Crea `docs/architecture.md` documentando la estructura actual.
   - Crea `docs/prd.md` con lo que puedas inferir (preguntale al usuario si no esta claro).
   - Crea `docs/user-stories.md` identificando las funcionalidades existentes.
   - Crea `docs/roadmap.md` con tickets retroactivos marcados `[x]` para lo ya construido.
   - Crea `docs/decisions-log.md` registrando las decisiones tecnicas inferibles (eleccion de stack, patron arquitectonico).

   **Si elige [4] Regularizar + Cubrir con tests:**
   - Todo lo de [1], mas:
   - Carga `@prompts/skills/tests-skill.md` como guia.
   - Identifica "logica critica" (validadores, servicios, reglas de negocio) ignorando fontaneria.
   - Detecta o propone framework de tests segun el stack.
   - Instala dependencias y crea configuracion.
   - Genera tests para la logica critica priorizada.
   - Documenta en `docs/testing-strategy.md` que esta cubierto y que no.
   - Anade tickets al roadmap por cada zona NO cubierta para abordar mas adelante.

5. Al terminar, ejecuta `On(task_complete)` mostrando el checklist completo.

No toques el codigo existente excepto lo estrictamente necesario para instalar/configurar tests.
