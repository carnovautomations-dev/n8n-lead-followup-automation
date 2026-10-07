# Lead Follow-Up Automation for Window & Door Installers (n8n)

> **This repo is v1**, the open-source starting point (Airtable + Slack + email).
> The version I install for clients today (v2) is described below. Its code is not public.

## Production version (v2)

v2 is built around how installers actually work: they live on their phone, not in a CRM.

- **Every lead source in one place**: website forms, trade-show stand forms (Google Forms / Tally) and **voice notes**. The salesperson records a voice message after a visit and the lead is created from it.
- **Telegram as the interface**: each new lead arrives as a card in a Telegram group, with buttons to change its status (contacted, site visit booked, quote sent, on hold, won, lost). One topic per client company: send a voice note or a text in the lead's thread and the AI updates the record.
- **Automatic follow-ups by email** for web and stand leads, which stop as soon as someone changes the status.
- **AI reply reading**: customer replies to the emails are read by AI. STOP blocks further emails, every other reply is forwarded to the right Telegram topic.
- **PostgreSQL** on a self-hosted server, with **NocoDB** as a password-protected table view for each client, and nightly backups.
- **Modular per client**: one shared system with a configuration record per client (which sources, which modules are on), not a copy of the workflow for each one.

Self-hosted n8n on a Hetzner VPS (Docker, Caddy, HTTPS). Secrets live in environment variables, not inside workflows.

Want a demo? [ty@cautomations.com](mailto:ty@cautomations.com)

---

# v1: open-source workflow (this repo)

Every new lead from your website form gets an instant confirmation email, your sales team gets a Slack alert, and if nobody has contacted the lead after 24 and 72 hours, automatic follow-ups go out. No lead falls through the cracks.

## The problem

Small installers get leads from web forms, but replies depend on whoever remembers to check the inbox. A slow or forgotten reply usually means the customer has already called a competitor.

## What it does

1. **Receives** the form submission through a protected webhook (secret header).
2. **Cleans and validates** the data. Invalid emails are rejected with a `422` and never trigger an email.
3. **Scores** the lead from 0 to 100 (phone, email, budget, number of windows, message length). Budgets are parsed the Italian way (`3.500`, `€ 10.000`, `3,5k`). Leads with a score of 70 or more are marked high priority.
4. **Saves** it to Airtable with a unique record ID.
5. **Alerts** the sales team in Slack, with a distinct message for high-priority leads.
6. **Confirms** to the lead by email.
7. **Follows up** after 24 hours and again at 72 hours, but only if the lead's status in Airtable is still `Nuovo`. Set it to `Contattato` after you call and the follow-ups stop.
8. **Alerts you on errors** through a separate error workflow.

The email templates are in Italian (built for the Italian market) and easy to translate.

## Stack

n8n, Airtable, SMTP (any provider), Slack, JavaScript (Code node), webhooks with header authentication.

## Setup

1. **Airtable:** create a base with a table `Leads` and these fields: `Lead` (text), `Stato` (single select: `Nuovo`, `Contattato`), `Priorita` (single select: `Alta`, `Normale`), `Punteggio` (number), `Email`, `Telefono`, `Comune`, `Tipo intervento`, `N. infissi` (number), `Budget` (text), `Budget numero` (number), `Messaggio`, `Fonte`, `Follow-up 1 inviato` (checkbox), `Follow-up 2 inviato` (checkbox).
2. **Credentials in n8n:** Airtable Personal Access Token (scopes `data.records:read`, `data.records:write`, `schema.bases:read`), SMTP, Slack bot token (`chat:write`, `chat:write.public`), and a Header Auth credential with header name `X-Webhook-Secret`.
3. **Import** `lead-followup-infissi-it.json` into n8n and replace the placeholders: `YOUR_AIRTABLE_BASE_ID`, `YOUR_AIRTABLE_TABLE_ID`, `YOUR_SLACK_CHANNEL_ID`, `YOUR_SENDER@YOUR_DOMAIN` and `[Nome Azienda]`. Then select your credentials on each node.
4. **Error alerts:** import `error-alert-slack.json`, publish it, and set it as the Error Workflow in the main workflow's settings.
5. **Activate** the main workflow and test:

```bash
curl -i -X POST https://YOUR-N8N-DOMAIN/webhook/lead-infissi -H "Content-Type: application/json" -H "X-Webhook-Secret: YOUR_SECRET" -d '{"nome":"mario","cognome":"rossi","email":"mario@example.com","telefono":"+39 333 1234567","comune":"Torino","tipo_intervento":"sostituzione infissi","numero_infissi":6,"budget":"€ 8.000","messaggio":"Vorrei sostituire tutti gli infissi della casa"}'
```

A successful call returns `200` and `{"ok":true,...}`, and the lead appears in Airtable and Slack.

## Customizing

The scoring thresholds are in the Code node's `CONFIG` block (`budgetMin`, `serramentiMin`, `hotThreshold`). The same structure works for other industries (for example solar panels) by changing the fields and the email texts.

## Known limits

- Lead status is updated by hand in Airtable. Automating this from a CRM is the natural next step.
- Sending from a shared SMTP account has daily limits. Use a verified sender on your own domain, ideally through a transactional email service.
- Slack and email nodes are set to continue on error so one failed notification does not block the rest, which means those failures do not trigger the error workflow.
- Add a privacy notice to your form. This is not legal advice.

## Need something like this?

I build automations and internal tools for small businesses.

**Antonio Carnovale** · ty@cautomations.com · [cautomations.com](https://cautomations.com)
