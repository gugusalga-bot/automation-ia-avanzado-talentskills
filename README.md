# TalentSkills · Agente Calificador de Leads (Proyecto Integrador)

**Curso:** Automation IA Avanzado  
**Autora:** Guadalupe Salgado  
**Entrega actual:** Checkpoint 1: Configuración e Interfaces Agénticas  
**Plataforma:** n8n 2.38.7 (self-hosted)

## Descripción

TalentSkills es una plataforma SaaS B2B (ficticia) que mapea las habilidades de los empleados, detecta brechas de conocimiento y recomienda capacitaciones. Vende por suscripción mensual a empresas, consultoras de RR. HH. y universidades, con tres planes: Básico, Platinum y Gold.

Este flujo es la primera versión del agente del proyecto integrador. Funciona como un **Asistente de Calificación de Leads**: conversa con el prospecto, lo clasifica como Caliente, Tibio o Frío, recomienda un plan y, si corresponde, **decide por su cuenta** registrar el lead en Google Sheets. Cada ejecución deja un log de observabilidad en Slack para supervisión humana.

## Arquitectura del flujo

```
Chat de entrada ──► Agente Calificador de Leads ──► Responder en el chat ──► Armar log ──► Slack (log)
 (Chat Trigger)      (AI Agent · Tools Agent)          (Respond to Chat)        (Set)
                         ┆               ┆
              Google Gemini Chat     Registrar lead en planilla
              (gemini-3.1-flash-lite)     (Google Sheets Tool · append)
```

| Requisito | Implementación |
| --- | --- |
| Disparador | Chat Trigger, con Response Mode en *Using Response Nodes* |
| Cerebro | AI Agent v3.1. En n8n 2.x este nodo funciona siempre como Tools Agent |
| Modelo | Google Gemini Chat Model (`gemini-3.1-flash-lite`) |
| Guardrail | Max Iterations = **8** |
| System Prompt | Modular: Rol → Ámbito → Objetivo → Reglas → Escalamiento, sin lenguaje inclusivo |
| Herramienta | Google Sheets como Tool lateral, con descripción semántica manual de cuándo usarla y cuándo no |
| Flujo no lineal | El agente recibe el mensaje directo del trigger, sin nodos IF ni Switch antes |
| Observabilidad | Return Intermediate Steps activado: Slack recibe el mensaje, las herramientas usadas con su entrada y resultado, y la respuesta final |

## Criterios de calificación

| Clasificación | Condición |
| --- | --- |
| Caliente | Decide la compra y quiere implementar en 3 meses o menos |
| Tibio | Cumple solo una de las dos condiciones |
| Frío | No cumple ninguna |

Plan recomendado según la cantidad de empleados: hasta 30, Básico; de 31 a 100, Platinum; de 101 a 300, Gold; más de 300, A medida.

## Cómo importarlo

1. En n8n: **Create workflow** → menú **…** → **Import from File** → `checkpoint1_guadalupe_salgado.json`.
2. Asignar las credenciales: Google Gemini (PaLM) API, Google Sheets OAuth2 API y Slack API.
3. En *Registrar lead en planilla*, elegir la planilla con estas columnas:
   `fecha_registro, nombre, email, organizacion, cantidad_empleados, necesidad, clasificacion, plan_recomendado`
4. En el nodo de Slack, elegir el canal de logs.

## Prueba de validación

Mensaje enviado desde el chat de n8n:

> Hola, soy Martina López, Gerente de RR. HH. de Logística del Sur, una empresa de 85 empleados. Necesitamos detectar brechas de habilidades en los mandos medios y armar planes de capacitación. Queremos implementarlo en los próximos 2 meses y tenemos un presupuesto de unos U$ 150 por mes. Mi email es martina.lopez@logisticadelsur.com. ¿Qué plan me recomiendan? Me interesa una demo.

Resultado esperado: el agente decide usar la herramienta, registra el lead (plan Platinum, Caliente), responde en el chat y envía el log a Slack, con la ejecución en verde.

<!-- Después de tu prueba, reemplazá el párrafo anterior por lo que pasó realmente. -->

### Evidencias

<!-- Reemplazá estas líneas por tus capturas, por ejemplo: ![Lienzo en verde](capturas/lienzo.png) -->
- Lienzo con la ejecución en verde
- Panel Logs con el uso de la herramienta
- Fila registrada en Google Sheets
- Log recibido en Slack

## Hoja de ruta del proyecto integrador

- [x] Checkpoint 1: agente base y motor de razonamiento
- [ ] M3 · Memoria y contexto
- [ ] M4 · Herramientas e integraciones reales
- [ ] M5 · RAG / base documental
- [ ] M6 · Voz
- [ ] M11 · Proyecto Final Integrador
