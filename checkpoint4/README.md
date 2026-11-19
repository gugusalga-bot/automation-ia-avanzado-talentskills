# Checkpoint 4 · Integraciones con HubSpot, Gmail y Slack

**Proyecto:** TalentSkills · Agente Calificador de Leads
**Hecho por:** Guadalupe Salgado
**N8N utilizado:** n8n 2.38.7 self-hosted (localhost)

## Herramientas conectadas

| Rol | Herramienta | Credencial en n8n | Autenticación | Scopes |
| --- | --- | --- | --- | --- |
| CRM | HubSpot | HubSpot - CRM TalentSkills | Clave de servicio (App Token) | crm.objects.contacts.read y crm.objects.contacts.write |
| Casilla de correo | Gmail | Gmail - Soporte TalentSkills | OAuth2 desde n8n | gmail.readonly y gmail.compose (Custom Scopes) |
| Canal del equipo | Slack | Slack account | Bot Token emitido por la instalación OAuth v2 de la app | chat:write y channels:read |

![Credenciales activas en n8n](capturas/credenciales.png)

**Notas sobre la autenticación IMPORTANTE:**

- **HubSpot:** desde el 23 de junio de 2026 HubSpot no permite crear apps públicas legacy, que eran las que usaba la credencial OAuth2 de n8n. Las apps OAuth nuevas requieren el sistema de proyectos por CLI. Se usó una clave de servicio, la alternativa oficial de HubSpot para llamar a su API, con solo los dos scopes de contactos.
- **Slack:** Slack exige direcciones de retorno HTTPS para OAuth2, y esta instancia de n8n corre en http://localhost. Se usó el Bot Token de la app, que Slack emite mediante su instalación OAuth v2 (Install to Workspace), con los scopes recortados al mínimo.

## Carril de email (se suma al flujo del Manager orquestador)

Link al flujo visual: checkpoint4/Flujo de entregable 4.png 
Email entrante (Gmail Trigger, filtro to:+talentskills) → ¿Es auto-respuesta? (IF) → Agente de Email (AI Agent) → Limpiar y validar payload (Set) → ¿Email válido? → Buscar contacto en HubSpot (Look up) → Actualizar contacto / Crear contacto → Crear borrador (Gmail Create Draft) → Preparar aviso (Set) → Avisar al equipo (Slack)

El carril del chat con memoria en Airtable y los Workers siguen funcionando en el mismo workflow.

## Controles

1. **IF anti auto-reply:** descarta asuntos con Auto-reply, Out of office, Undeliverable y similares, y remitentes no-reply o mailer-daemon. Corta el bucle infinito de respuestas automáticas.
2. **Look up antes del Create:** busca el contacto por email en HubSpot; si existe lo actualiza y si no, lo crea. Evita el Error 409 por duplicados.
3. **Create Draft (Human-in-the-loop):** la respuesta de la IA queda en Borradores dentro del hilo; ningún nodo envía correos.
4. **Set de limpieza y validación:** deja solo email, nombre, asunto, cuerpo recortado y resultado de la IA, y valida el formato del email. Evita el Error 400. Otro Set deja a Slack solo un texto breve, sin HTML ni adjuntos.

## Mínimo privilegio

- Gmail: 2 scopes en lugar de los 6 por defecto, sin acceso total al correo. El filtro del trigger limita el procesamiento a la dirección +talentskills.
- HubSpot: la clave de servicio solo puede leer y escribir contactos; no accede a empresas, negocios, formularios ni tickets.
- Slack: el bot solo tiene chat:write y channels:read, y solo publica en los canales donde fue invitado.
