---
description: Anadir nueva funcionalidad o modulo con interrupcion de seguridad
---

# /nueva — Nueva funcionalidad

Voy a pedirte anadir una nueva funcionalidad. Sigue el protocolo de Evolucion del sistema.

Pasos:

1. Pregunta al usuario: "¿Que funcionalidad o modulo quieres anadir? Descripcion lo mas concreta posible."

2. Espera la respuesta. Luego aplica la **Interrupcion de Seguridad**:
   - "Antes de tocar codigo, necesito actualizar la documentacion:
     1. ¿Anado la nueva funcionalidad como user story en `user-stories.md` con criterios de aceptacion testeables?
     2. ¿Creo el(los) ticket(s) correspondiente(s) en `roadmap.md`?
     3. ¿Esta funcionalidad requiere una ADR nueva en `decisions-log.md` (eleccion de libreria, patron nuevo, decision de diseno no trivial)?
     4. ¿Hay que actualizar `blueprints.md` con algun patron nuevo?
     5. ¿Actualizo `testing-strategy.md` si cambia el scope de tests?"
   - Pide confirmacion antes de proceder.

3. Tras actualizar los docs, implementa la funcionalidad:
   - Consulta `blueprints.md` para mantener estilo consistente.
   - Consulta `testing-strategy.md` para saber que y como testear.
   - Escribe los tests ANTES o EN PARALELO al codigo (TDD pragmatico).
   - Ejecuta la suite para verificar verde.

4. Al terminar, ejecuta OBLIGATORIAMENTE `On(task_complete)`:
   - Muestra el checklist de sincronizacion documental completo.
   - Marca el ticket como `[x]` en roadmap SOLO si todos los tests pasan.
   - Actualiza el `%` del roadmap.

No cierres la tarea si el checklist no se ha mostrado.
