# Ledger — Personal Expense Tracker

A single-file, client-side expense tracker that parses DBS/POSB bank statement PDFs, auto-categorises transactions using Google Gemini AI, and exports beautifully styled `.xlsx` spreadsheets. Designed to be hosted on GitHub Pages with zero backend or build step required.

<img width="608" height="534" alt="Screenshot 2026-06-08 232001" src="https://github.com/user-attachments/assets/544dd2c6-0432-4747-8ba1-b8af7260cfab" />

---

## Features

- **PDF parsing** — drop in any DBS/POSB consolidated statement PDF; transactions are extracted using positional column detection (not fragile regex on plain text)
- **AI categorisation** — uses Google Gemini Flash (free tier) to classify every transaction; falls back to rule-based matching if no API key is provided
- **12 spending categories** — Food Delivery, Dining Out, Transport, Shopping, Entertainment, Gaming, Subscriptions, Health & Fitness, Utilities, Transfers, Income, Other; all editable inline
- **Live dashboard** — KPI cards (total expenses, income, net flow), spending-by-category bar chart, top merchants bar chart, filterable transaction table
- **Styled Excel export** — generates a multi-sheet `.xlsx` with dark header banners, coloured category pills, KPI cards, mini bar indicators, and SUM formulas throughout; opens natively in Google Sheets or Excel
- **Additive / modular workflow** — re-upload a previously exported `.xlsx` as the base, then add a new month's PDF; duplicates are detected and skipped automatically
- **Fully client-side** — no server, no database, no login; everything runs in the browser; your bank data never leaves your device

---

## Live Demo

[https://javier-ganmw.github.io/exptracker]([https://javier-ganmw.github.io/exptracker/])


---

## Getting Started

### Option A — GitHub Pages (recommended)

1. Fork or clone this repository
2. Rename `expense-tracker.html` to `index.html` (or keep the filename and set it as the source)
3. Go to **Settings → Pages → Source**, set branch to `main` and folder to `/ (root)`
4. Your app will be live at `https://<your-username>.github.io/<repo-name>` within a minute

### Option B — Run locally

No build step needed. Just open the file in any modern browser:

```bash
git clone https://github.com/yourusername/ledger.git
cd ledger
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

> **Note:** Some browsers block local file access to other local files. If the PDF parser fails when opening as `file://`, use a simple local server instead:
> ```bash
> python3 -m http.server 8080
> # then open http://localhost:8080
> ```

---

## Setting Up the Gemini API Key

The AI categorisation uses Google Gemini Flash Lite, which has a **free tier** of 1,500 requests per day — more than enough for personal use.

1. Go to [aistudio.google.com](https://aistudio.google.com)
2. Sign in with your Google account
3. Click **Get API Key → Create API key**
4. Copy the key and paste it into the banner at the top of the app

Your key is saved in `localStorage` and never transmitted anywhere except directly to Google's API. It persists across sessions in the same browser.

If you skip this step the app still works — it falls back to rule-based categorisation which covers the most common DBS/POSB merchants out of the box.

---

## Monthly Workflow

The app is designed around a rolling spreadsheet you keep adding months to:

```
Month 1 (April)
  └── Drop April PDF
  └── Review & correct categories
  └── Save Ledger_2026-04-30.xlsx  ← your master file

Month 2 (May)
  └── Step 1: Drop Ledger_2026-04-30.xlsx  ← loads April back in
  └── Step 2: Drop May PDF                 ← merges May on top
  └── Duplicate detection skips any overlap
  └── Save Ledger_2026-05-31.xlsx          ← now contains April + May

Month 3 (June) ... and so on
```

Each saved file is self-contained — it's also your backup. You can always reload any previous file and the full transaction history comes back.

---

## Exported Spreadsheet Structure

| Sheet | Contents |
|---|---|
| **Dashboard** | Hero banner with period, KPI cards, spending-by-category table with % and mini bar, monthly summary table |
| **All Transactions** | Every transaction ever, sorted newest-first, with auto-filter and a TOTAL row |
| **April 2026** | Month-level KPIs, category breakdown, full transaction list — one sheet per month |
| **May 2026** | *(added next month)* |

All amounts are formatted as `SGD #,##0.00`. Totals use `=SUM(...)` formulas so they stay live if you edit data in Google Sheets.

---

## Supported Banks

Currently optimised for **DBS and POSB** consolidated statements (the multi-page PDF you download from digibank).

The parser uses x/y coordinate-based column detection rather than text patterns, so it handles the full statement layout including:

- Debit card transactions
- FAST Payment / Receipt (PayNow)
- Funds transfers (PayLah top-ups)
- GIRO salary credits
- Multi-line merchant descriptions

Support for other banks (OCBC, UOB, Maybank) can be added by adjusting the column boundary constants in `parsePDF()`.

---

## Project Structure

```
ledger/
├── index.html      # The entire app — HTML, CSS, and JS in one file
└── README.md
```

The app intentionally ships as a single file with no dependencies to install. External libraries are loaded from CDNs at runtime:

| Library | Version | Purpose |
|---|---|---|
| [PDF.js](https://mozilla.github.io/pdf.js/) | 3.11.174 | PDF text extraction with positional data |
| [xlsx-js-style](https://github.com/gitbrent/xlsx-js-style) | 1.2.0 | Excel file generation with full cell styling |
| [Google Fonts](https://fonts.google.com/) | — | DM Serif Display, DM Mono, Instrument Sans |

---

## Category Reference

| Key | Label | Typical merchants |
|---|---|---|
| `food` | Food Delivery | FoodPanda, Grab Food, Deliveroo |
| `dining` | Dining Out | Restaurants, cafes, Ya Kun, Guzman Y Gomez |
| `transport` | Transport | Grab rides, Bus/MRT |
| `shopping` | Shopping | Shopee, Decathlon, TikTok Shop |
| `entertainment` | Entertainment | Movies, events |
| `gaming` | Gaming | Steam |
| `subscriptions` | Subscriptions | Spotify, YouTube, Apple, Patreon, Microsoft 365 |
| `health` | Health & Fitness | Gym (ZYM), pharmacy |
| `utilities` | Utilities | Zero1, phone/internet bills |
| `transfers` | Transfers | PayNow, PayLah top-ups |
| `income` | Income | GIRO salary, incoming PayNow, cashback |
| `other` | Other | Everything else |

Categories can be corrected per-transaction in the table before exporting. The correction is reflected immediately in the charts and totals.

---

## Privacy

- All PDF parsing happens in your browser using PDF.js — the file is never uploaded anywhere
- Transaction data is held in memory only; it is not written to `localStorage` or any external storage
- Your Gemini API key is stored in `localStorage` under the key `gemini_key` — it is only ever sent to `generativelanguage.googleapis.com`
- The exported `.xlsx` is generated entirely client-side and downloaded directly to your device

---

## Contributing

Pull requests are welcome. Key areas for improvement:

- **Additional bank support** — OCBC, UOB, Maybank, Standard Chartered statement formats
- **Category rules** — add more merchant name patterns to `ruleCat()` for better zero-AI coverage
- **Multi-currency** — the parser currently reads SGD amounts only; USD/EUR transactions appear as SGD equivalents
- **Charts** — a proper pie/bar chart using D3 or Chart.js on the dashboard sheet

---

## License

MIT — do whatever you want with it.
