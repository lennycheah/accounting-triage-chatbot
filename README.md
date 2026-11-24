# Accounting Request Triage Chatbot

A Databricks App that replaces the `accounting@compass.com` email inbox for ticket submission. Built on the [Databricks Agent Chat Template](https://github.com/databricks/app-templates/tree/main/e2e-chatbot-app-next).

## Purpose

This chatbot helps Transaction Management (TM) staff determine whether their issue:
1. **Can be self-resolved by TM** (payment status, running balance, deal data corrections)
2. **Needs an Accounting ticket** (commission corrections, revenue reclassifications, bill credits)
3. **Should be routed to another team** (AP, 1099, W9, Equity, Escrow)

## Architecture

- **Frontend**: NextJS + React chat UI (from Databricks `e2e-chatbot-app-next` template)
- **Backend**: Databricks Agent serving endpoint connected to the Accounting Request Triage Genie Agent
- **Data Sources**:
  - `data_analytics.discovery.pbi_accounting_model_vw` — deal-level financial transactions
  - `main.silver_zendesk.tickets` — Zendesk ticket history
  - `main.silver_zendesk.groups` — Zendesk group routing
  - `main.silver_zendesk.tickets_custom_fields` — ticket categorization
  - `main.silver_zendesk.ticket_fields_custom_field_options` — field option labels

## Genie Agent

- **Space ID**: `01f1e8237756151a902a2e8f79a004b9`
- **Name**: Accounting Request Triage Agent

## Market Routing Matrix

| Market | Primary Contact | Reviewer |
|---|---|---|
| Tennessee, Colorado, Florida, Hawaii | Pablo Ruiz | Leonard Cheah |
| DC, Seattle, Minnesota, NY Region | Evelyn Caro | Alan Sola / Leonard Cheah |
| Texas, Los Angeles | Vanessa Yates | Alan Sola |
| Indiana, Pennsylvania, Wisconsin, NJ, MO | Manuel Millan | Eric Yu |
| Michigan, DC (WFP), Chicago, MA/NE | Ania Blaszkiewicz | Alan Sola |
| Atlanta, Carolinas, San Francisco, San Diego | Kim Hurley | Alan Sola / Leonard Cheah |
| Arizona, Louisiana | Eric Yu | Leonard Cheah |

## Setup

### 1. Copy the Chat UI Template

```bash
git clone https://github.com/bigcitylen/accounting-triage-chatbot.git
cd accounting-triage-chatbot

# Clone the official Databricks template (temporary)
git clone https://github.com/databricks/app-templates /tmp/app-templates

# Copy the chat UI source code (preserving our custom config files)
cp -rn /tmp/app-templates/e2e-chatbot-app-next/client .
cp -rn /tmp/app-templates/e2e-chatbot-app-next/server .
cp -rn /tmp/app-templates/e2e-chatbot-app-next/packages .
cp -rn /tmp/app-templates/e2e-chatbot-app-next/scripts .
cp -rn /tmp/app-templates/e2e-chatbot-app-next/tests .
cp /tmp/app-templates/e2e-chatbot-app-next/package.json .
cp /tmp/app-templates/e2e-chatbot-app-next/package-lock.json .
cp /tmp/app-templates/e2e-chatbot-app-next/tsconfig.json .
cp /tmp/app-templates/e2e-chatbot-app-next/biome.jsonc .
cp /tmp/app-templates/e2e-chatbot-app-next/drizzle.config.ts .
cp /tmp/app-templates/e2e-chatbot-app-next/playwright.config.ts .
cp /tmp/app-templates/e2e-chatbot-app-next/knip.json .
cp /tmp/app-templates/e2e-chatbot-app-next/manifest.yaml .

rm -rf /tmp/app-templates
```

### 2. Configure Authentication

```bash
export DATABRICKS_CONFIG_PROFILE='accounting-triage'
databricks auth login --profile "$DATABRICKS_CONFIG_PROFILE"
```

### 3. Configure the App

```bash
cp .env.example .env
# Edit .env — set DATABRICKS_SERVING_ENDPOINT to your deployed agent endpoint name
# Edit databricks.yml — uncomment and set the default serving_endpoint_name
```

### 4. Install and Run Locally

```bash
npm install
npm run dev
```

The app starts on http://localhost:3000

### 5. Deploy to Databricks

```bash
databricks bundle validate
databricks bundle deploy
databricks bundle run accounting_triage_chatbot
```

### 6. Share with Users

Grant TM staff permission to view the app, then share the app URL.
This replaces the `accounting@compass.com` email for ticket submission.

## Triage Decision Tree

### TM Can Self-Resolve (No ticket needed)
- Payment status inquiries
- Agent running balance lookups
- Deal data entry corrections
- Commission rate questions matching ICA/contract
- Split questions matching deal system entry
- Deposit status checks

### Route to Accounting (Create ticket)
- Commission/split corrections requiring journal entry or bill credit
- Revenue reclassifications between subsidiaries or GL accounts
- Refunds needing processing
- Bill credits or reversals
- 1099 corrections
- Escrow discrepancies requiring GL adjustment

### Route to Other Teams
- Vendor invoices → Accounts Payable
- Agent invoicing/billing → Agent Invoicing and Incentives
- W9 submissions → Inbound W9
- Tax/1099 questions → 1099 group
- Equity questions → Equity
- Escrow processing → Escrow
