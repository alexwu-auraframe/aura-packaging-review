---
name: packaging-review
description: >
  Use this skill whenever a reviewer uploads a packaging design PDF or pastes
  a PDF link (outsert, bellyband, user manual, or QSG) and wants it
  cross-checked against the Aura copy doc. Triggers on any mention of
  "review packaging", "check the design", "packaging PDF", "outsert",
  "bellyband", "user manual review", or when a PDF filename matches the Aura
  design naming convention
  (e.g. 251219-Outsert-US-English-Aspen-Carbon-AFM210-MBLK-China-V1.pdf).
  Also triggers when someone asks to generate a packaging review report and
  post in the designated channel. When sharing the final review report on
  Slack, always explicitly call out any "FLAG" items in the message so team
  members can quickly identify and act on discrepancies.
  Always invoke this skill before attempting a packaging review on your own —
  it encodes Aura-specific rules around copy docs, compliance tables, SKU
  verification, and report format that are critical for accurate reviews.
---

# Aura Packaging Review Skill

You are helping an Aura Home team member review a packaging design PDF against
the official copy doc. Your job is to catch discrepancies, verify compliance
marks, and produce a structured Word (.docx) report.

## The Golden Rule

The **design PDF** (uploaded by the reviewer) is what you are CHECKING.
The **copy doc** (from Google Drive) is the SOURCE OF TRUTH.
Never flag something in the copy doc as wrong — only flag when the design
diverges from the copy doc.

---

## Step 1 — Parse the Design Filename

Extract these fields from the PDF filename before doing anything else:

| Field | Example | Notes |
|---|---|---|
| Product series | Aspen | e.g. Aspen, Carmel, Smith |
| Country/market | US | US, APAC, International, CA, EU, UK, MX, AU |
| Language | English | |
| Packaging type | Outsert | Outsert, Bellyband, UserManual, QSG |
| SKU | AFM210-MBLK | Full SKU code from filename |
| Color/variant | Carbon | e.g. Carbon, Sandstone, Clay, Ink |
| COO | China | Country of origin |

**Filename pattern:**
`YYMMDD-[Type]-[Country]-[Language]-[Series]-[Color]-[SKU]-[COO]-V[N].pdf`

Example: `251219-Outsert-US-English-Aspen-Carbon-AFM210-MBLK-China-V1.pdf`

---

## Step 2 — Verify the SKU

Search Google Drive for the **"Product Info"** sheet and look up the SKU from
the filename.

- If found: proceed normally, note the channel (DTC, Retail, etc.)
- If NOT found: **stop and call this out prominently** before proceeding.
  Show the reviewer the closest matches (same series, same color code suffix)
  and ask them to confirm which SKU applies before continuing the review.
  Do not skip this check — an unverified SKU could mean the wrong copy doc
  is being used.

---

## Step 3 — Find the Correct Copy Doc

Search Google Drive for **"Packaging copy doc"** or navigate directly to:
https://drive.google.com/drive/folders/1zG-dvo3wULZOUTH33AjheRWcWbxS-fTq

**For Bellyband / Outsert** — copy docs are in this Google Sheet:
https://docs.google.com/spreadsheets/d/1sW5WH4OAFoxegkoKkIB8k5ZzoRBMfL3MWVulTqNmIFY/edit
Navigate to the tab matching the product series + market:

| Market | Tab name pattern | Covers |
|---|---|---|
| US | `[Series] US SKU [Year]` | United States only |
| APAC | `[Series] APAC SKU [Year]` | Australia + New Zealand |
| International | `[Series] Intl SKU [Year]` | UK, Canada, EU, Mexico |

If the correct tab is ambiguous (e.g. Aspen US has both DTC and Sam's Club
variants), call it out to the reviewer before proceeding.

**For User Manual / QSG** — copy docs are separate Google Docs inside the
"Packaging copy" folder. Find the doc matching the product series + market
using the same tab name pattern above.

**Strikethrough rule:** In the copy doc, any text shown with strikethrough
means it has been deleted and must NOT appear in the final design. If that
text still appears in the design PDF, flag it as FLAG — the strikethrough
is an explicit instruction to remove it.

---

## Step 4 — Extract Design PDF Text

**Important:** Design PDFs linked from Adobe Acrobat are client-rendered and
cannot be read by text extraction tools alone. Use Claude in Chrome to open
the link and read the content page by page visually.

For uploaded PDF files, use `pdftotext` to extract text:

```bash
pdftotext -layout "/path/to/design.pdf" /tmp/design_extracted.txt
cat /tmp/design_extracted.txt
```

Read carefully — compliance marks (FCC ID, model numbers, legal text) are
often in very small print at the bottom or back of panels.

**Zoom note:** When reviewing a bellyband or outsert via a Chrome link, zoom
in enough to make all text clearly legible before extracting or comparing
content. Many sections use small fonts that are easy to misread at default zoom.

---

## Step 5 — Cross-Check Design vs Copy Doc

Compare every text element panel by panel:

**For Outsert / Bellyband**, check each panel separately:
- Outsert (front/back)
- Bellyband Top
- Bellyband Front
- Bellyband Back
- Bellyband Bottom

**For User Manual / QSG**, check each section:
- Cover / title
- Setup steps
- Compliance statements
- Legal / regulatory section

For each difference you find, classify its severity:

| Severity | Meaning | Example |
|---|---|---|
| **FLAG** | Must be confirmed or fixed before print approval | Wrong product or SKU name, missing required compliance text, strikethrough text still present in design, selling point reordered |
| **INFO** | Style/formatting note, likely intentional | Punctuation difference, capitalization style |
| **PASS** | Matches copy doc exactly | — |

---

## Step 6 — Check Compliance Marks

See `references/compliance_table.md` for the full compliance table.

Key lookup rules:
- **B** = required on Bellyband / Outsert
- **U** = required on User Manual / QSG
- **F** = required on Frame
- **A** = required on Adapter
- **E** = required on E-label
- **W** = required on Website
- **S** = required on Sticker

For outsert/bellyband reviews, check every item where the market column shows **B**.
For user manual reviews, check every item where the market column shows **U**.

**Market to compliance column mapping:**
- US SKU → check US column
- APAC SKU → check AU/NZ column
- International → check UK, CA, EU, MX columns as applicable

**Critical distinctions:**
- FCC ID (2AZGI-AFXXX) = US only
- ISED IC ID (28310-AFXXX) = Canada only — do NOT flag IC as missing on US-only designs
- CE mark = EU + UK only
- RCM logo = AU/NZ only

**Known compliance logo appearances:**
- CTI Certified = "Calm Tech™ Certified ✓" badge (black rounded rectangle with checkmark)
- Google Badge = "GET IT ON Google Play" badge
- Apple MFI = "Made for iPhone | iPad" badge
- S/N Brackets = [ ] bracket fields printed near serial number / item part #
- UL 4200A Warning = orange WARNING box with INGESTION HAZARD text (button battery warning)

---

## Step 7 — Generate the Report

Use the Node.js `docx` package to generate the report. The package is installed
in the outputs/report_work directory. See `references/report_builder.md` for
the complete script template.

### Report structure

1. **Header/footer** — product info, page numbers, "CONFIDENTIAL"
2. **Title block** — report title, filename subtitle, metadata table
3. **SKU Alert box** (FLAG color, only if SKU not found in Product Info sheet)
4. **Legend** — CRITICAL / FLAG / INFO / PASS color key
5. **Section 1 — Discrepancies Found** — table of all non-PASS findings
6. **Section 2 — Content Verification** — table of all PASS checks
7. **Section 3 — Summary & Actions** — consolidated action items

### Column layouts

Section 1: Severity | Panel | Field | Copy Doc (Source of Truth) | Design File (What Was Found) | Action
Section 2: Status | Panel | Field | Verified Content
Section 3: Priority | Issue | Owner | Action | Status

### Color scheme

| Severity | Cell background | Text color |
|---|---|---|
| FLAG | #FCE4D6 | #C55A11 |
| INFO | #DEEAF1 | #2E74B5 |
| PASS | #E2EFDA | #107C41 |

### Output filename
`[Series]_[Market]_[Type]_[SKU]_Review.docx`

---

## Common Pitfalls

- **IC ID ≠ FCC ID.** FCC is US-only. IC is Canada-only. Never flag IC as missing from a US design.
- **Small print matters.** Model numbers, FCC IDs, and legal text are often tiny. Always use pdftotext, not visual reads.
- **Strikethrough = deleted.** Strikethrough in the copy doc means the text was removed and must NOT appear in the design. Flag as FLAG if it still appears.
- **Design style choices are INFO, not CRITICAL.** All-caps headings, comma placement, label shortening are usually intentional.
- **Call out ambiguous SKUs early.** If the SKU prefix looks unusual (e.g. AFM vs AF), flag before reviewing.
- **Adobe Acrobat links require Chrome.** pdftotext will not work on Acrobat-hosted PDFs. Use Claude in Chrome to read them visually, page by page.

---

## Source Links (Google Drive / Notion)

**Bellyband & Outsert copy doc (Google Sheets):**
https://docs.google.com/spreadsheets/d/1sW5WH4OAFoxegkoKkIB8k5ZzoRBMfL3MWVulTqNmIFY/edit

**User Manuals & QSG copy docs (Google Drive folder):**
https://drive.google.com/drive/folders/1zG-dvo3wULZOUTH33AjheRWcWbxS-fTq

**Packaging Review Reports (Google Drive — save all generated .docx reports here):**
https://drive.google.com/drive/folders/1d4FbVWIylq6IRTDeyn6cUCmfcnsHXs_S

**Compliance table (Notion):**
https://app.notion.com/p/pushd/Compliance-4371262ddeab4b7986e77d6802fcaba8

## Known Sheet Tabs (Bellyband/Outsert copy doc)

Use this index to find the right tab by product + market:

| Tab name | Product | Market/Channel |
|---|---|---|
| Aspen US SKU 2026 | Aspen | US only |
| Aspen APAC SKU 2026 | Aspen | AU/NZ |
| Aspen Global SKU 2025 / 2026 Nordics | Aspen | EU/UK/CA/MX |
| Aspen US Retail SKU 2025 | Aspen | US Retail |
| Aspen NA DTC 2025 | Aspen | NA DTC (EN/FR/ES) |
| Aspen Brushed Stone US (Sam's) 2026 | Aspen | US Sam's Club |
| Swarovski US SKU 2026 | Astra (Swarovski) | US |
| Parker US SKU 2026 | Parker | US |
| Parker Costco UK/FR 2026 | Parker | UK/FR Costco |
| Parker Costco MX 2026 | Parker | MX Costco |
| Parker Costco CA 2026 | Parker | CA Costco |
| Parker Costco AU 2026 | Parker | AU Costco |
| Parker Global SKUs | Parker | Global |
| Walden US SKU 2026 | Walden | US only |
| Walden APAC SKU 2026 | Walden | AU/NZ |
| Walden Global SKU 2025 / 2026 Nordics | Walden | Global |
| Walden NA DTC 2025 | Walden | NA DTC |
| Walden US Retail SKU 2025 | Walden | US Retail |
| Carver US SKU 2026 | Carver | US only |
| Carver Mat US SKU 2026 | Carver Mat | US only |
| Carver NA DTC 2025 | Carver | NA DTC |
| Carver Global SKU 2025 / Nordics 2026 | Carver | Global |
| Carver APAC SKU 2026 | Carver | AU/NZ |
| Summit Costco US 2026 | Summit (Walden) | US Costco |
| Target 2026 | Carver Stone (Target) | US Target |
| Kohls Stone 2026 | Carver Stone (Kohl's) | US Kohl's |
| Costco Griffin CA 2026 | Griffin Stone | CA Costco |

**If the tab is ambiguous** (e.g. Aspen US has both DTC, Retail, and Sam's variants), stop and ask the reviewer which channel before proceeding.
