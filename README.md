# Aura Packaging Review

Cross-checks Aura packaging design PDFs against the official copy doc and generates a structured Word (.docx) review report.

## What it does

- Parses the design PDF filename to identify product, SKU, market, and packaging type
- Verifies the SKU against the Product Info sheet in Google Drive
- Finds the correct copy doc (Google Sheet for bellyband/outsert, Google Doc for UM/QSG)
- Compares every text element panel by panel
- Checks compliance marks against the built-in regional compliance table
- Generates a colour-coded .docx report (FLAG / INFO / PASS) and posts it to Slack

## Severity levels

| Level | Meaning |
|-------|---------|
| FLAG  | Must be confirmed or fixed before print approval |
| INFO  | Style/formatting note, likely intentional |
| PASS  | Matches copy doc exactly |

## Required integrations

Connect these MCPs before using the skill:

- **Google Drive** — to read copy docs and save reports
- **Slack** — to post the report and flag items to the review channel

## Usage

Upload or paste a link to the packaging design PDF and say "review packaging" or "check the design". The skill will handle the rest.
