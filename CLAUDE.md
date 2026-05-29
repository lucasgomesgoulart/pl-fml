# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file expense tracking web app ([index.html](index.html)) for tracking monthly income and expenses, synchronized with Google Sheets. Written in Brazilian Portuguese, no build process.

## Running the App

Open `gastos-pai.html` directly in any browser — no server, build step, or dependencies required.

## Architecture

The entire application lives in a single HTML file with inline CSS and JavaScript. No external dependencies beyond Google Fonts (loaded from CDN).

### Google Sheets Integration

All data persists to a Google Sheets spreadsheet via the Sheets API v4:
- **Sheet ID** and **API Key** are hardcoded in the `<script>` block near the top
- Sheet name: `dados`
- Reads all rows with `lerPlanilha()`, writes individual cells with `escreverCelula()`

### Data Model

Rows in the sheet map to transaction entries. The in-memory state is:

```javascript
dados = {
  "jan26": { saidas: [{id, data, desc, valor}], entradas: [{id, data, desc, valor}] },
  "fev26": { ... }
}
```

The `id` field is the row index in the sheet (1-based). Month keys are 3-letter Portuguese abbreviation + 2-digit year (e.g. `"mai26"`).

### Key Functions

| Function | Purpose |
|---|---|
| `lerPlanilha()` | Fetches all rows from Google Sheets and rebuilds `dados` |
| `renderizar()` | Re-renders summary cards and transaction tables for `mesAtual` |
| `adicionarTransacao()` | Opens the modal for adding a new entry |
| `salvarTransacao()` | Writes a new/edited row to the sheet |
| `deletarLinha(id)` | Clears a row in the sheet (soft delete) |
| `atualizarResumo()` | Recalculates totals and updates the 3 summary cards |

### Auth

Login uses SHA-256 hashing against a hardcoded hash. Authenticated state is kept in `sessionStorage`.

### UI Flow

1. Login screen → password verified client-side
2. `lerPlanilha()` fetches all sheet data on load
3. Month/year selector updates `mesAtual` and calls `renderizar()`
4. Add/edit/delete actions call the Sheets API then reload data and re-render
