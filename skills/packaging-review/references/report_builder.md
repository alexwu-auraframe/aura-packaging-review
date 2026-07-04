# Report Builder — Complete Working Script

The report is generated with the Node.js `docx` npm package.
Run scripts from a directory where `node_modules/docx` is installed.
In sessions this is typically: `/sessions/.../mnt/outputs/report_work/`

Install if needed: `cd report_work && npm install docx`

---

## Page layout

- **Orientation:** Landscape — width 15840, height 12240 (DXA)
- **Margins:** 900 DXA all sides
- **Usable width:** 13680 DXA (15840 − 1800 − breathing room)
- **Font:** Arial throughout
- **Base font size:** 18 (9pt) for table cells; 20 for metadata; 22 for body; 26 for subtitle; 40 for title

---

## Severity levels & colors

| Level    | Text color | Background |
|----------|-----------|------------|
| CRITICAL | C00000    | FFCCCC     |
| FLAG     | C55A11    | FCE4D6     |
| INFO     | 2E74B5    | DEEAF1     |
| PASS     | 107C41    | E2EFDA     |

---

## Section column widths

**Section 1 — Discrepancies** (total 11640):
`[900, 1400, 1500, 2600, 2600, 2040]`
Headers: Severity / Panel / Field / Copy Doc (Source of Truth) / Design File (What Was Found) / Action

**Section 2 — Passing Checks** (total 13680):
`[900, 1400, 1500, 9880]`
Headers: Status / Panel / Field / Verified Content

**Section 3 — Summary & Actions** (total 13680):
`[1100, 4480, 2200, 2400, 3500]`
Headers: Priority / Issue / Owner / Action / Status

---

## Complete working script

Replace all `[...]` placeholders with values from the current review.

```javascript
const {
  Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
  AlignmentType, BorderStyle, WidthType, ShadingType,
  PageNumber, Header, Footer
} = require('./node_modules/docx');
const fs = require('fs');

// ── Borders ───────────────────────────────────────────────────────────────────
const border    = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const borders   = { top: border, bottom: border, left: border, right: border };
const noBorder  = { style: BorderStyle.NONE, size: 0, color: "FFFFFF" };
const noBorders = { top: noBorder, bottom: noBorder, left: noBorder, right: noBorder };

// ── Helpers ───────────────────────────────────────────────────────────────────
function heading1(text) {
  return new Paragraph({
    children: [new TextRun({ text, bold: true, size: 28, color: "1F3864", font: "Arial" })],
    spacing: { before: 320, after: 160 },
    border: { bottom: { style: BorderStyle.SINGLE, size: 6, color: "1F3864", space: 4 } }
  });
}

function body(text, opts = {}) {
  return new Paragraph({
    children: [new TextRun({ text, size: 22, font: "Arial", ...opts })],
    spacing: { after: 80 }
  });
}

function tableHeader(labels, widths) {
  return new TableRow({
    tableHeader: true,
    children: labels.map((label, i) => new TableCell({
      borders,
      width: { size: widths[i], type: WidthType.DXA },
      shading: { fill: "1F3864", type: ShadingType.CLEAR },
      margins: { top: 80, bottom: 80, left: 120, right: 120 },
      children: [new Paragraph({ children: [new TextRun({ text: label, size: 20, font: "Arial", bold: true, color: "FFFFFF" })] })]
    }))
  });
}

// Section 1: discrepancy row (6 columns)
function discRow(sev, panel, field, copyDoc, design, action) {
  const sevColor = sev==="CRITICAL"?"C00000":sev==="FLAG"?"C55A11":sev==="INFO"?"2E74B5":"107C41";
  const sevBg    = sev==="CRITICAL"?"FFCCCC":sev==="FLAG"?"FCE4D6":sev==="INFO"?"DEEAF1":"E2EFDA";
  return new TableRow({ children: [
    new TableCell({ borders, width:{size:900,type:WidthType.DXA}, shading:{fill:sevBg,type:ShadingType.CLEAR},
      margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({alignment:AlignmentType.CENTER,children:[new TextRun({text:sev,size:18,font:"Arial",bold:true,color:sevColor})]})] }),
    new TableCell({ borders, width:{size:1400,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:panel,size:18,font:"Arial",bold:true})]})] }),
    new TableCell({ borders, width:{size:1500,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:field,size:18,font:"Arial"})]})] }),
    new TableCell({ borders, width:{size:2600,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:copyDoc,size:18,font:"Arial",color:"107C41"})]})] }),
    new TableCell({ borders, width:{size:2600,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:design,size:18,font:"Arial",color:"C00000"})]})] }),
    new TableCell({ borders, width:{size:2040,type:WidthType.DXA}, shading:{fill:"FFF2CC",type:ShadingType.CLEAR},
      margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:action,size:18,font:"Arial",italics:true})]})] }),
  ]});
}

// Section 2: passing row (4 columns — last col spans remaining width)
function passRow(panel, field, content) {
  return new TableRow({ children: [
    new TableCell({ borders, width:{size:900,type:WidthType.DXA}, shading:{fill:"E2EFDA",type:ShadingType.CLEAR},
      margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({alignment:AlignmentType.CENTER,children:[new TextRun({text:"PASS",size:18,font:"Arial",bold:true,color:"107C41"})]})] }),
    new TableCell({ borders, width:{size:1400,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:panel,size:18,font:"Arial",bold:true})]})] }),
    new TableCell({ borders, width:{size:1500,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:field,size:18,font:"Arial"})]})] }),
    new TableCell({ borders, width:{size:9880,type:WidthType.DXA}, margins:{top:80,bottom:80,left:120,right:120},
      children:[new Paragraph({children:[new TextRun({text:content,size:18,font:"Arial",color:"107C41"})]})] }),
  ]});
}

// ── Document ──────────────────────────────────────────────────────────────────
const doc = new Document({
  sections: [{
    properties: {
      page: {
        size: { width: 15840, height: 12240 },   // landscape
        margin: { top: 900, right: 900, bottom: 900, left: 900 }
      }
    },

    // Header: grey confidential bar
    headers: { default: new Header({ children: [new Paragraph({
      children: [new TextRun({
        text: "PACKAGING REVIEW REPORT  |  CONFIDENTIAL  |  [Series] [Type] — [Market] SKU ([Color], [SKU]) — Design vs Copy Doc",
        size: 18, font: "Arial", color: "888888"
      })],
      border: { bottom: { style: BorderStyle.SINGLE, size: 4, color: "CCCCCC", space: 4 } }
    })] }) },

    // Footer: page number
    footers: { default: new Footer({ children: [new Paragraph({
      children: [
        new TextRun({ text: "Aura Home, Inc. — Internal Use Only    |    Page ", size: 18, font: "Arial", color: "888888" }),
        new TextRun({ children: [PageNumber.CURRENT], size: 18, font: "Arial", color: "888888" }),
      ],
      border: { top: { style: BorderStyle.SINGLE, size: 4, color: "CCCCCC", space: 4 } }
    })] }) },

    children: [

      // ── Title block ─────────────────────────────────────────────────────────
      new Paragraph({ children: [new TextRun({ text: "Packaging Review Report — [Type]", bold: true, size: 40, font: "Arial", color: "1F3864" })], spacing: { after: 80 } }),
      new Paragraph({ children: [new TextRun({ text: "[Series] [Market] SKU  |  [Color]  |  [filename]", size: 26, font: "Arial", color: "2E74B5" })], spacing: { after: 80 } }),

      // ── Metadata table (no borders, 2 rows × 6 cols) ────────────────────────
      new Table({
        width: { size: 13680, type: WidthType.DXA },
        columnWidths: [2000, 3200, 2000, 3200, 1200, 2080],
        borders: { top: noBorder, bottom: noBorder, left: noBorder, right: noBorder, insideH: noBorder, insideV: noBorder },
        rows: [
          new TableRow({ children: [
            new TableCell({ borders: noBorders, width:{size:2000,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"Review Date:",bold:true,size:20,font:"Arial",color:"555555"})]})] }),
            new TableCell({ borders: noBorders, width:{size:3200,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"[Date]",size:20,font:"Arial"})]})] }),
            new TableCell({ borders: noBorders, width:{size:2000,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"SKU (from filename):",bold:true,size:20,font:"Arial",color:"555555"})]})] }),
            new TableCell({ borders: noBorders, width:{size:3200,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"[SKU]",size:20,font:"Arial"})]})] }),
            new TableCell({ borders: noBorders, width:{size:1200,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"Model:",bold:true,size:20,font:"Arial",color:"555555"})]})] }),
            new TableCell({ borders: noBorders, width:{size:2080,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"[Model]",size:20,font:"Arial"})]})] }),
          ]}),
          new TableRow({ children: [
            new TableCell({ borders: noBorders, width:{size:2000,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"Copy Doc Tab:",bold:true,size:20,font:"Arial",color:"555555"})]})] }),
            new TableCell({ borders: noBorders, width:{size:3200,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"[Tab name]",size:20,font:"Arial"})]})] }),
            new TableCell({ borders: noBorders, width:{size:2000,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"Country / Language:",bold:true,size:20,font:"Arial",color:"555555"})]})] }),
            new TableCell({ borders: noBorders, width:{size:3200,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"[Country] / [Language]",size:20,font:"Arial"})]})] }),
            new TableCell({ borders: noBorders, width:{size:1200,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"Color:",bold:true,size:20,font:"Arial",color:"555555"})]})] }),
            new TableCell({ borders: noBorders, width:{size:2080,type:WidthType.DXA}, children:[new Paragraph({children:[new TextRun({text:"[Color]",size:20,font:"Arial"})]})] }),
          ]}),
        ]
      }),

      new Paragraph({ children: [], spacing: { after: 120 } }),

      // ── Legend ──────────────────────────────────────────────────────────────
      new Table({
        width: { size: 13680, type: WidthType.DXA },
        columnWidths: [1200, 2200, 1200, 2200, 1200, 2200, 1200, 2280],
        rows: [new TableRow({ children: [
          new TableCell({ borders: noBorders, width:{size:1200,type:WidthType.DXA}, shading:{fill:"FFCCCC",type:ShadingType.CLEAR}, margins:{top:60,bottom:60,left:120,right:120},
            children:[new Paragraph({alignment:AlignmentType.CENTER,children:[new TextRun({text:"CRITICAL",size:18,font:"Arial",bold:true,color:"C00000"})]})] }),
          new TableCell({ borders: noBorders, width:{size:2200,type:WidthType.DXA}, margins:{top:60,bottom:60,left:80,right:80},
            children:[new Paragraph({children:[new TextRun({text:"Must fix before print approval",size:18,font:"Arial"})]})] }),
          new TableCell({ borders: noBorders, width:{size:1200,type:WidthType.DXA}, shading:{fill:"FCE4D6",type:ShadingType.CLEAR}, margins:{top:60,bottom:60,left:120,right:120},
            children:[new Paragraph({alignment:AlignmentType.CENTER,children:[new TextRun({text:"FLAG",size:18,font:"Arial",bold:true,color:"C55A11"})]})] }),
          new TableCell({ borders: noBorders, width:{size:2200,type:WidthType.DXA}, margins:{top:60,bottom:60,left:80,right:80},
            children:[new Paragraph({children:[new TextRun({text:"Needs team confirmation",size:18,font:"Arial"})]})] }),
          new TableCell({ borders: noBorders, width:{size:1200,type:WidthType.DXA}, shading:{fill:"DEEAF1",type:ShadingType.CLEAR}, margins:{top:60,bottom:60,left:120,right:120},
            children:[new Paragraph({alignment:AlignmentType.CENTER,children:[new TextRun({text:"INFO",size:18,font:"Arial",bold:true,color:"2E74B5"})]})] }),
          new TableCell({ borders: noBorders, width:{size:2200,type:WidthType.DXA}, margins:{top:60,bottom:60,left:80,right:80},
            children:[new Paragraph({children:[new TextRun({text:"Style / formatting note",size:18,font:"Arial"})]})] }),
          new TableCell({ borders: noBorders, width:{size:1200,type:WidthType.DXA}, shading:{fill:"E2EFDA",type:ShadingType.CLEAR}, margins:{top:60,bottom:60,left:120,right:120},
            children:[new Paragraph({alignment:AlignmentType.CENTER,children:[new TextRun({text:"PASS",size:18,font:"Arial",bold:true,color:"107C41"})]})] }),
          new TableCell({ borders: noBorders, width:{size:2280,type:WidthType.DXA}, margins:{top:60,bottom:60,left:80,right:80},
            children:[new Paragraph({children:[new TextRun({text:"Matches copy doc",size:18,font:"Arial"})]})] }),
        ]})]
      }),

      new Paragraph({ children: [], spacing: { after: 200 } }),

      // ── Section 1: Discrepancies ─────────────────────────────────────────────
      heading1("Section 1 — Discrepancies Found"),
      new Paragraph({ children: [], spacing: { after: 120 } }),
      new Table({
        width: { size: 13680, type: WidthType.DXA },
        columnWidths: [900, 1400, 1500, 2600, 2600, 2040],
        rows: [
          tableHeader(["Severity","Panel","Field","Copy Doc (Source of Truth)","Design File (What Was Found)","Action"], [900,1400,1500,2600,2600,2040]),
          // discRow("CRITICAL", "Panel", "Field", "copy doc text", "design text", "action"),
          // discRow("FLAG", ...),
          // discRow("INFO", ...),
        ]
      }),

      new Paragraph({ children: [], spacing: { after: 280 } }),

      // ── Section 2: Passing Checks ────────────────────────────────────────────
      heading1("Section 2 — Content Verification (Passing Checks)"),
      new Paragraph({ children: [], spacing: { after: 120 } }),
      new Table({
        width: { size: 13680, type: WidthType.DXA },
        columnWidths: [900, 1400, 1500, 9880],
        rows: [
          tableHeader(["Status","Panel","Field","Verified Content"], [900,1400,1500,9880]),
          // passRow("Panel", "Field", "verified content ✓"),
        ]
      }),

      new Paragraph({ children: [], spacing: { after: 280 } }),

      // ── Section 3: Summary & Actions ────────────────────────────────────────
      heading1("Section 3 — Summary & Actions"),
      new Paragraph({ children: [], spacing: { after: 120 } }),
      new Table({
        width: { size: 13680, type: WidthType.DXA },
        columnWidths: [1100, 4480, 2200, 2400, 3500],
        rows: [
          tableHeader(["Priority","Issue","Owner","Action","Status"], [1100,4480,2200,2400,3500]),
        ]
      }),

      new Paragraph({ children: [], spacing: { after: 200 } }),
      body("This report was generated automatically by the Aura Packaging Review Tool (MVP). All findings should be verified by the responsible PMM/PM and Compliance team before print sign-off.", { color: "888888", italics: true }),
    ]
  }]
});

Packer.toBuffer(doc).then(buffer => {
  fs.writeFileSync("/sessions/.../mnt/outputs/[OutputFilename].docx", buffer);
  console.log("Done");
});
```

---

## Output filename convention

`[Series]_[Market]_[Type]_[SKU]_Review.docx`

Example: `Aspen_US_Outsert_AFM210-MBLK_Review.docx`
