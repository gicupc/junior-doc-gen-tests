---
description: Arrancar proyecto nuevo desde cero - entrevista BMADT y setup completo
---

# /inicio — Proyecto nuevo

Voy a arrancar un proyecto nuevo siguiendo el protocolo Architect-Brain.

Ejecuta estas acciones en orden:

1. Carga `@prompts/architect-brain.md`.
2. Ejecuta el protocolo `On(project_start)`:
   - Lanza la entrevista BMADT (Beneficio, Mecanismo, Alcance, Datos, Testing).
   - Haz las 5 preguntas UNA A UNA, no todas de golpe. Espera a que responda cada una antes de pasar a la siguiente.
3. Al cerrar la entrevista:
   - Genera los docs en `/docs/`: prd.md, architecture.md, user-stories.md, roadmap.md, testing-strategy.md.
   - Registra las respuestas BMADT como ADRs iniciales en `docs/decisions-log.md`.
4. Ejecuta `On(testing_setup)` ANTES del primer codigo funcional:
   - Instala framework de tests segun el stack elegido en M.
   - Crea configuracion minima.
   - Verifica con smoke test en verde y borralo.
   - Marca como ticket inicial `[x]` en roadmap.md.
5. Muestra un resumen final y pregunta al usuario si aprueba antes de empezar con el primer ticket funcional.

No escribas codigo funcional hasta que el usuario apruebe los docs y el entorno de tests este listo.
