# QRCC Data Center

QRCC Data Center adalah aplikasi dashboard internal untuk mengelola workflow bulanan FQMS/F-COST, termasuk PPM, sales, defect/non-defect, repair action, validasi data, preview report, export PDF/Excel, dan backup database.

Sumber kebenaran utama produk ada di [prd.md](prd.md). Dokumen tambahan di [.doc/](.doc/) berfungsi sebagai discovery notes, arsip PRD panjang, dan supporting plan yang mengikuti keputusan di [prd.md](prd.md).

## Tech Stack

- Nuxt 4 full-stack monolith
- Vue 3 + TypeScript strict
- Nuxt UI 4
- SQLite + Drizzle ORM
- Zod untuk validasi input server-side
- ExcelJS untuk export Excel berbasis template
- Playwright untuk export PDF dari HTML/CSS print layout

## Setup

Install dependencies:

```bash
npm install
```

## Development Server

Start development server:

```bash
npm run dev
```

## Production

Build for production:

```bash
npm run build
```

Preview production build locally:

```bash
npm run preview
```

## Project Docs

- [prd.md](prd.md) — PRD utama dan single source of truth.
- [.doc/prd-long.md](.doc/prd-long.md) — versi PRD panjang untuk konteks tambahan.
- [.doc/phase-0-discovery.md](.doc/phase-0-discovery.md) — hasil discovery template, FQMS, dan F-COST.
- [.doc/system-design-mvp-plan.md](.doc/system-design-mvp-plan.md) — supporting plan arsitektur dan fase MVP.
