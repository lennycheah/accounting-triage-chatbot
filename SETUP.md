# Setup Guide: Accounting Triage Chatbot

This guide walks you through deploying the Accounting Triage Chatbot as a Databricks App.

## Prerequisites

1. **Databricks CLI** installed and up to date (`brew install databricks` on macOS)
2. **Node.js 22+** installed
3. **NPM** installed
4. Access to the Databricks workspace containing the Accounting Triage Genie Agent

## Step 1: Clone and Copy the Chat UI Template

This repo contains the Databricks App configuration files. You need to copy the actual
chat UI source code from the official Databricks template:

```bash
# Clone this repo
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

# Clean up
rm -rf /tmp/app-templates
```

## Step 2: Deploy the Genie Agent as a Serving Endpoint

The Genie Agent (space_id: `01f1e8237756151a902a2e8f79a004b9`) needs to be accessible
via a serving endpoint. Options:

- **Databricks UI**: Navigate to the Genie Agent > Deploy as serving endpoint
- **Databricks SDK/CLI**: Deploy programmatically

Note the serving endpoint name — you'll need it in Step 3.

## Step 3: Configure

```bash
# Set up Databricks authentication
export DATABRICKS_CONFIG_PROFILE='accounting-triage'
databricks auth login --profile "$DATABRICKS_CONFIG_PROFILE"

# Create your .env file
cp .env.example .env
# Edit .env and set DATABRICKS_SERVING_ENDPOINT to your endpoint name

# Edit databricks.yml — uncomment and set the default serving_endpoint_name
```

## Step 4: Install Dependencies

```bash
npm install
```

## Step 5: Run Locally

```bash
npm run dev
```

The app starts on http://localhost:3000 (frontend) and http://localhost:3001 (backend)

## Step 6: Deploy to Databricks

```bash
databricks bundle validate
databricks bundle deploy
databricks bundle run accounting_triage_chatbot
```

The app URL will be displayed after deployment.

## Step 7: Share with Users

Grant TM staff permission to view the app in Databricks, then share the app URL.
This replaces the `accounting@compass.com` email for ticket submission.

## Optional: Enable Persistent Chat History

To save conversations across sessions:

1. Uncomment the database sections in `databricks.yml`
2. Run `databricks bundle deploy` (first deploy takes a few minutes for provisioning)
3. Run `npm run db:migrate`

Or use the quickstart script: `./scripts/quickstart.sh`

## Optional: Enable User Feedback

To allow thumbs up/down on responses:

1. Find your experiment ID: `npx tsx scripts/get-experiment-id.ts --endpoint <endpoint-name>`
2. Uncomment the experiment section in `databricks.yml` and set the ID
3. Uncomment `MLFLOW_EXPERIMENT_ID` in `app.yaml`
4. Redeploy: `databricks bundle deploy`
