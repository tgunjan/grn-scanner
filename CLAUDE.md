# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server on localhost:3000
npm run build        # Production build
npm start            # Start production server
```

No test runner or linter is configured.

## Environment Setup

Copy `.env.example` to `.env.local` and set:
```
ANTHROPIC_API_KEY=sk-ant-...
```

Get an API key from https://console.anthropic.com/

## Architecture

**Single-page Next.js 14 App Router PWA** with no client-side routing.

### Data Flow
```
Mobile Camera → app/page.js (base64 encode) → POST /api/extract → Claude Vision API → Structured JSON → Editable form → GRN creation
```

### Key Files

- **`app/page.js`** — The entire frontend. A single `"use client"` React component implementing a 5-step state machine: `capture → processing → review → po_match → confirm`. Contains all UI, state, and logic. No separate components.
- **`app/api/extract/route.js`** — Next.js API route that calls Claude Vision (`claude-sonnet-4-20250514`) with a detailed extraction prompt. Receives a base64 image, returns extracted fields with confidence scores. The API key stays server-side only.
- **`app/layout.js`** — Root layout with PWA meta tags, Google Fonts (DM Sans + JetBrains Mono), and manifest link.
- **`public/manifest.json`** — PWA manifest (standalone display, portrait orientation, dark theme).

### State Machine (app/page.js)

The `step` state drives which screen renders:
1. **`capture`** — File upload / camera input
2. **`processing`** — Shows scanning animation while API call is in flight
3. **`review`** — Editable form with extracted fields + confidence badges + PO auto-match display
4. **`po_match`** — Scrollable list of all POs for manual selection
5. **`confirm`** — GRN success screen with summary, PO progress bar, mock WhatsApp notification

### Mock Data

`MOCK_POS` in `app/page.js` is hardcoded demo data (4 POs for Nikhil Group construction sites). The production path is to replace this with a StrategicERP API call.

`FIELD_CONFIG` defines the 13 extracted document fields in two groups: challan fields (dc_number, dc_date, supplier_name, material_description, quantity, quantity_unit, vehicle_number, lr_number, driver_name, delivery_site) and weight fields (gross_weight, tare_weight, net_weight).

### Claude API Integration

- Model: `claude-sonnet-4-20250514`
- Handles: printed/handwritten challans, mixed languages (English, Hindi, Marathi), smudged/folded documents
- Returns: per-field confidence scores (`high`/`medium`/`low`), document type, language detected, overall quality

### Styling

Custom CSS variables in `app/globals.css` (dark theme, `--bg-primary: #0F172A`, `--accent: #14B8A6` teal). No Tailwind. Component styles are inline JS objects (the `s` variable in `app/page.js`). Mobile-first, max-width 480px layout.

### GRN Number Format

`NCPL/RPPDGRN{YYYY}{MM}{3-digit-random}` — generated client-side on confirmation.
