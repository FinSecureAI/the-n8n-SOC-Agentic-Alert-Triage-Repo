[README.md](https://github.com/user-attachments/files/23697764/README.md)
# SOC Agentic Alert Triage → Incident (n8n Workflow)

This repository contains an example **n8n** workflow that implements an AI-augmented SOC flow:

**Alert → Enrichment → Triage (LLM) → Routing → Incident (ServiceNow) + Logging**

## Files

- `workflows/soc-alert-triage-incident.json`  
  Import this file directly into n8n (**Settings → Workflows → Import from File**) to create the workflow.

## Workflow Overview

1. **SIEM Alert Inbound (Webhook)**  
   - Receives alerts from your SIEM at `POST /soc/alert-ingest`.

2. **Normalize Alert (Function)**  
   - Maps raw SIEM payload into a common schema (`alert_id`, `user`, `host`, `src_ip`, etc.).

3. **Enrich Asset / Identity / Threat Intel (HTTP Request nodes)**  
   - Looks up host details in CMDB.
   - Looks up user details in HR/IAM.
   - Looks up IP reputation in Threat Intel.

4. **Merge Enrichment (Function)**  
   - Combines alert + context into a single JSON object.

5. **Triage Agent LLM (OpenAI node)**  
   - Calls an LLM to decide:
     - `decision`: `ESCALATE_INCIDENT` | `QUEUE` | `IGNORE`
     - `priority`: `P1`..`P4`
     - `confidence` and `rationale`.

6. **Decision Route (Switch)**  
   - Routes:
     - `ESCALATE_INCIDENT` → Incident path.
     - `QUEUE` → Queue notification.
     - `IGNORE` → Ignored alert logger.

7. **Incident Summary Agent LLM (OpenAI node)**  
   - Generates a structured incident summary for the ticket.

8. **Create Incident ServiceNow (HTTP Request)**  
   - Creates an incident in ServiceNow (replace the URL and auth).

9. **Log Outcome (HTTP Request)**  
   - Sends triage metadata to your logging/metrics backend.

10. **Queue Notification / Ignore Logger**  
    - Simple HTTP-based stubs to notify a chat system or log ignored alerts.

## Before You Use

- Replace placeholder URLs:
  - `https://cmdb.internal/api/assets`
  - `https://hr-api.internal/users`
  - `https://ti.internal/api/ip/...`
  - `https://servicenow.example.com/api/now/table/incident`
  - `https://logging.internal/...`
  - `https://chat.internal/...`
- Configure authentication on HTTP nodes as needed.
- Configure your OpenAI (or compatible) credentials in n8n and adjust the `model` name if required.
- Adjust any expressions or JSON bodies to match your environment.

## Import Steps

1. Open your n8n instance.
2. Go to **Workflows** → **Import from file**.
3. Select `workflows/soc-alert-triage-incident.json` from this repo.
4. Save and activate (once your endpoints and credentials are configured).

You can now point your SIEM to the webhook URL n8n generates for the **SIEM Alert Inbound** node and start testing end-to-end AI-driven alert triage.
