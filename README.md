# Ecosistema de Automatización IA — Clasificación de Leads VIP

Sistema autónomo que clasifica leads entrantes como VIP o Estándar usando IA,
con un punto de aprobación humana (HITL) antes de enviar cualquier email.

**Curso:** Automatización IA — Entrega Final
**Stack:** n8n (orquestador) · Airtable (memoria) · Claude Haiku 4.5 (IA) · Slack + Gmail (salida)

## Contenido de este repo

- `diagrama_arquitectura.pdf` — Mapa de arquitectura, estructuras de datos, matriz de
  costos y documentación de seguridad/resiliencia (criterios 1 a 4 de la rúbrica).
- `blueprint_flujo_1_clasificacion.json` — Export real de n8n: Trigger → Validación →
  IA → Escritura en Airtable → Notificación Slack.
- `blueprint_flujo_2_envio.json` — Export real de n8n: Trigger (Aprobado=true) →
  Envío por Gmail → Actualización de registro final.
- `system_prompt_ia.txt` — Prompt completo usado en el nodo de IA.
- `/screenshots` — Evidencia de las 5 corridas de prueba.

## Resumen del flujo

1. Un lead nuevo entra a la tabla "Leads" de Airtable con Estado = "Nuevo".
2. Si falta el mensaje del lead, el flujo corta ahí y notifica por Slack — nunca
   llega a gastar una llamada a la IA con datos incompletos.
3. Si el dato está completo, Claude Haiku clasifica el lead como VIP o Estándar,
   justifica la clasificación y redacta un borrador de email de bienvenida.
4. El resultado se escribe en Airtable con el checkbox "Aprobado" en `false`, y se
   notifica al vendedor por Slack con el link de revisión.
5. **Nada se envía sin aprobación humana.** Un segundo flujo espera a que el
   checkbox "Aprobado" pase a `true` en Airtable.
6. Una vez aprobado, se envía el email de bienvenida por Gmail y el registro
   queda actualizado con Estado = "Enviado" y la fecha de envío.

## Corridas de prueba realizadas

| # | Caso | Resultado |
|---|------|-----------|
| 1 | Lead con presupuesto alto y pedido de contacto senior | Clasificó VIP → aprobado → email enviado |
| 2 | Lead con consulta general, sin señales de urgencia | Clasificó Estándar → aprobado → email enviado |
| 3 | Lead sin mensaje (camino infeliz) | Cortó en la validación de datos, nunca llegó a la IA |
| 4 | Flujo 1 completo de punta a punta | Trigger → IA → Airtable → Slack, verificado |
| 5 | Flujo 2 completo de punta a punta | Trigger (Aprobado) → Gmail → Airtable, verificado |

## Manejo de errores y resiliencia

- Reintento automático (2 intentos) en el nodo de IA y en el nodo de envío por Gmail.
- Los errores persistentes se registran en la tabla "Log de Errores" y disparan un
  aviso a Slack, sin frenar el resto de las filas en cola.
- El trigger filtra por `Estado = "Nuevo"` (primer flujo) y por
  `Aprobado = true AND Estado != "Enviado"` (segundo flujo), evitando bucles
  infinitos por auto-disparo.

## Optimización de costos

- **Claude Haiku 4.5** para la clasificación (tarea corta, no requiere un modelo grande).
- **Batch API + Prompt Caching** recomendado para el reprocesamiento mensual de
  leads históricos (detalle y cálculo de ahorro en el PDF de arquitectura).


