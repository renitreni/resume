# Convert Resume HTML to PDF

This guide explains how to export `index.html` to PDF. The resume uses print-specific CSS (`@media print`) and relies on background colors for the header, sidebar, and skill badges — **always enable background graphics** when printing.

**Required files in this folder:**

- `index.html`
- `VISA_RRT.jpg` (profile photo referenced by the HTML)

**Output:** `Renier_Trenuela_Resume.pdf`

---

## Option 1: Browser Print (recommended for manual conversion)

Works in Chrome, Edge, or Safari without installing anything extra.

1. Open the resume in your browser:
   - **macOS:** double-click `index.html`, or drag it into a browser window
   - **Terminal:**
     ```bash
     open index.html
     ```
2. Open the print dialog:
   - **macOS:** `Cmd + P`
   - **Windows / Linux:** `Ctrl + P`
3. Set these options:
   - **Destination:** Save as PDF
   - **Paper size:** A4
   - **Margins:** None (or Minimum)
   - **Background graphics:** **On** (required for header, sidebar, and badges)
   - **Headers and footers:** Off (avoids URL/date in the PDF)
4. Click **Save** and name the file `Renier_Trenuela_Resume.pdf`.

### Chrome / Edge checklist

| Setting              | Value              |
|----------------------|--------------------|
| Destination          | Save as PDF        |
| Pages                | All                |
| Layout               | Portrait           |
| Paper size           | A4                 |
| Margins              | None               |
| Background graphics  | **Enabled**        |
| Headers and footers  | Disabled           |

---

## Option 2: Chrome Headless (command line)

Uses installed Google Chrome on macOS. Background colors may not render unless you use Option 3.

```bash
cd /path/to/resume

"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new \
  --disable-gpu \
  --no-pdf-header-footer \
  --print-to-pdf="$(pwd)/Renier_Trenuela_Resume.pdf" \
  "file://$(pwd)/index.html"
```

**Note:** The CLI `--print-to-pdf` flag does not always preserve background colors. For a pixel-accurate export, use Option 1 or Option 3.

---

## Option 3: Node.js + Puppeteer (automated, best fidelity)

Preserves backgrounds and matches the browser print output. This repo includes `generate-pdf.mjs`.

### One-time setup

```bash
cd /path/to/resume
npm install
```

### Generate PDF

```bash
npm run pdf
```

Or directly:

```bash
node generate-pdf.mjs
```

The script auto-detects Chrome on macOS, Linux, and Windows. To override the path, edit `executablePath` in `generate-pdf.mjs`.

<details>
<summary>Script source (already in repo as <code>generate-pdf.mjs</code>)</summary>

```javascript
import puppeteer from 'puppeteer-core';
import fs from 'fs';
import path from 'path';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const htmlPath = path.join(__dirname, 'index.html');
const pdfPath = path.join(__dirname, 'Renier_Trenuela_Resume.pdf');

const browser = await puppeteer.launch({
  executablePath: '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome',
  headless: true,
});

const page = await browser.newPage();
await page.goto('file://' + htmlPath, { waitUntil: 'networkidle0' });
await page.pdf({
  path: pdfPath,
  format: 'A4',
  printBackground: true,
  margin: { top: '0', right: '0', bottom: '0', left: '0' },
});

await browser.close();
console.log('PDF saved to:', pdfPath);
```

</details>

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Profile photo missing | Keep `VISA_RRT.jpg` in the same folder as `index.html` |
| Header/sidebar are white | Turn on **Background graphics** (Option 1) or use `printBackground: true` (Option 3) |
| URL and date in PDF footer | Disable **Headers and footers** in the print dialog, or use `--no-pdf-header-footer` (Option 2) |
| Layout looks wrong on mobile | Use a desktop browser window; the resume is designed for A4 / desktop width |
| `file://` blocked in browser | Open the file directly from Finder/Explorer, or serve locally: `python3 -m http.server 8080` then visit `http://localhost:8080/index.html` |

---

## Quick reference

```bash
# Open in browser (manual print)
open index.html

# Headless Chrome (fast, backgrounds may vary)
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --no-pdf-header-footer \
  --print-to-pdf="Renier_Trenuela_Resume.pdf" \
  "file://$(pwd)/index.html"

# Puppeteer (best quality)
npm run pdf
```
