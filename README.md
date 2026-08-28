# n8n Lead Follow-Up Automation

Personal project built with **n8n** to automate lead capture, follow-up, internal notifications, and basic lead tracking.

## What it does

The workflow:

* receives a new lead through a webhook
* cleans and normalizes the incoming data
* calculates a simple lead score
* saves the lead to Google Sheets
* sends an instant confirmation email
* notifies the sales team in Slack
* waits 24 hours and checks the lead status
* sends a follow-up if the lead is still marked as `New`
* checks again after 72 hours
* sends a final follow-up and internal reminder if needed

## Workflow

`Webhook → Data Cleaning → Google Sheets → Gmail + Slack → Wait 24h → Status Check → Follow-Up → Wait 72h → Final Check → Follow-Up + Slack Reminder`

## Technologies

* n8n
* JavaScript
* Webhooks
* Gmail
* Google Sheets
* Slack
* Conditional logic
* Data transformation

## Business objective

The goal is to reduce missed leads caused by slow or inconsistent follow-up and create a repeatable process that can be measured and improved over time.

## Technical notes

The Code node is used to:

* clean incoming fields
* normalize names and text
* parse budget values
* create timestamps
* calculate a simple lead score

The workflow also uses conditional checks to stop unnecessary follow-ups when a lead status changes.

## Security

Credentials and internal identifiers have been removed from the public workflow file.

To use the workflow, you need to configure your own:

* Google Sheets credentials
* Gmail credentials
* Slack credentials
* Google Sheet ID
* Slack channel

## Demo

Built as a personal automation project to demonstrate workflow design, API/webhook logic, data handling, and follow-up automation with n8n.
DEMO LOOM:https://www.loom.com/share/840bb2c9bed74a3f997cb12147657eaf
