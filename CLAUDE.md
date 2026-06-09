# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tesina (thesis) for IPN-ESIME Culhuacán, ICE program. Title: *Diseño e implementación de una plataforma cloud de gestión de inventario y ventas con monitoreo ambiental orientada a Farmacia Selene.* Authors: Jorge Yahir Flores Sánchez, Luciano Juárez Sánchez, Alan Yahir Mosco Ramírez.

## Build Commands

**Must use XeLaTeX** (not pdflatex) — the document uses `fontspec` and `polyglossia`.

```bash
# Single pass (no TOC page numbers)
xelatex main.tex

# Full build with correct TOC (run twice)
xelatex main.tex && xelatex main.tex

# Clean auxiliary files
rm -f *.aux *.log *.toc *.out *.lof *.lot
```

Entry point is `main.tex` in the project root.

## Document Structure

```
main.tex              ← entry point (\documentclass{book})
Preambulo.tex         ← all packages + custom macros (loaded via \input)
1_Preliminares/       ← cover page (portada.tex), TOC/indices (indices.tex)
2_Cuerpo/             ← pre-chapter sections + numbered chapters
3_Referencias/        ← bibliography, cibergrafía, glossary, acronyms
4_Anexos/             ← annexes and progress documentation
0_imagenes/           ← images organized in subdirectories by chapter
```

Sections in `2_Cuerpo/` that are not full chapters (Planteamiento, Justificación, Objetivos) use `\chapter*` / `\chapterstar` so they appear in the TOC but have no chapter number. Full chapters use `\chapterpage{N}{Title}` then manually advance the chapter counter.

## Custom Macros (defined in Preambulo.tex)

| Macro | Purpose |
|---|---|
| `\chapterpage{N}{Title}` | Decorative full-page chapter title (not in TOC, no page number) |
| `\chapterstar{Title}` | Unnumbered chapter that appears in TOC |
| `\chapterpageanex{Title}` | Like `\chapterpage` but for annexes |
| `\chapterpageavan{Title}` | Like `\chapterpage` but for progress sections |
| `\startmainmatter` | Resets numbering to Arabic page 1, enables TOC page numbers |
| `\avancesfigureson` | Switches figure labels to `Av.1`, `Av.2`, … |
| `\avancesfiguresoff` | Restores per-chapter figure numbering (`Fig. N.M`) |
| `\secbreak` | Thin horizontal rule separator between subsections |
| `\fila{name}{unit_price}{qty}[display_qty]` | Table row with auto-summing `\TotalGeneral` |

### `\fila` usage

Used inside a 4-column `tabular`/`longtable` for budget tables. Accumulates totals in `\TotalGeneral` (reset to 0 at the top of each table via `\xdef\TotalGeneral{0}`). The optional 4th argument overrides the displayed quantity text (e.g., `"1 pza."`) while still using the numeric `qty` for the calculation.

## References Format

All references use `thebibliography` (not BibTeX). Format is APA 7 manually typed:
- `\bibitem{key}` in `3_Referencias/1_Bibliografia.tex` for academic works
- `\bibitem{key}` in `3_Referencias/2_Cibergrafia.tex` for web/online sources

Section headers are forced with `\renewcommand{\bibname}{...}` before each `thebibliography` environment to produce "Bibliografía" or "Cibergrafía" as the title.

## Figures

- Figures are numbered per-chapter as `Fig. N.M` (e.g., `Fig. 2.1`)
- Images live in `0_imagenes/<SectionName>/filename.ext`
- Always wrap images in `\fbox{}` for a border, use `[H]` float placement
- Caption format: caption text on first line, then `\footnotesize \textbf{Fuente:} ...` below

## Live Deployments

- **API:** https://api.farmaciaselene.com/ — FastAPI on Railway, PostgreSQL on Railway
- **Frontend:** https://app.farmaciaselene.com/ — React on Vercel

## GitHub Repositories

| Repo | URL | Notes |
|---|---|---|
| API | https://github.com/luc881/pharmatrack-api | FastAPI + SQLAlchemy + Alembic + Poetry |
| Frontend | https://github.com/luc881/pharmatrack-frontend | React 19 + Vite 7 + MUI 7 + SWR 2 |
| Sensor | https://github.com/luc881/pharmatrack-sensor | ESP32 C++ (.ino), no OLEDs in final code |

## Tech Stack (confirmed from repos)

- **API:** FastAPI, SQLAlchemy ORM, Alembic migrations, Pydantic v2, JWT (python-jose + bcrypt), SlowAPI rate limiting, Uvicorn, Poetry
- **Frontend:** React 19, Vite 7, MUI 7, React Router 7, SWR 2, Axios (JWT interceptors), Framer Motion
- **Sensor:** ESP32 firmware in C++, AHT10 over I²C (SDA GPIO 8, SCL GPIO 9), WiFiManager, ArduinoJson, Adafruit AHTX0. Reads every 30 s. No OLEDs in deployed code (serial output only).

## Test Suite (API — pytest)

20 test files, one per module. Uses **real PostgreSQL** (not mocks). Each test truncates all tables for isolation. Coverage:
- `test_auth.py` — login, refresh token, logout, token invalidation (13 tests)
- `test_sales.py` — CRUD, sale completion, insufficient stock, batch usage (SaleBatchUsage), FEFO deduction
- `test_product_batch.py` — CRUD, duplicate lot code prevention
- `test_users.py` — CRUD, search/filter, password change, duplicate email
- + 16 more files covering: roles, permissions, branches, products, categories, brands, ingredients, suppliers, purchases, purchase details, refunds, sale details, sale payments, sale batch usage

## Known Discrepancy: OLEDs

Chapter 3 describes 3 OLED SSD1306 screens on the sensor node. The final deployed firmware (`ProyectoTesinaESP32.ino`) has **no OLED code** — it uses Serial output only. Reason: the first 3D enclosure design did not fit all components (Dupont connectors not accounted for), so the enclosure is being redesigned and OLED code is on hold. For Chapter 4, treat the system as complete without OLEDs.

---

## Chapter 4 Context — Development History

### Scope & tone
- Professors are NOT from CS/engineering background — keep language simple, avoid jargon.
- Mirror Chapter 3 style: explain *why* decisions were made, not just *what* was done.
- Do not mention: Pydantic, Poetry, Alembic, SWR, Vite, React Router by name — describe functionally instead.
- Format per friend's suggestion: "Con la propuesta X se obtuvo Y; fue necesario ajustar Z."

### API — key development facts (390 commits, Jul 2025–May 2026, single author: Luciano)

**Timeline:**
- Jul–Aug 2025: initial CRUD routes built rapidly (127 commits)
- Sep–Nov 2025: sales, batches, refunds modules added
- Dec 2025: sale completion + FEFO stock deduction logic (43 commits — most complex phase)
- Feb–Mar 2026: full rebrand possystem→pharmatrack, API versioning under /api/v1, Alembic migrations added
- Apr–May 2026: test suite published, soft-delete, password reset by email, normalization

**Most-iterated files:** products ORM (43 commits), main entry point (48 commits), sales routes (17 commits), permissions (26 commits)

**Key bugs fixed:**
- CORS headers missing on 500 errors → frontend received network error instead of real message; also `allow_origins=["*"]` conflicted with `allow_credentials=True`
- Monetary columns used floating-point (Double) → replaced with Numeric for exact decimal precision
- SQLAlchemy enums stored Python names (DRAFT) instead of DB values (draft) → fixed with `values_callable`
- Soft-deleted records appeared in listings when `deleted_at IS NULL` filter was missing
- UNIQUE constraints on supplier RFC/email blocked soft-delete reuse → moved to application-layer validation
- Railway deploy failed: Python couldn't find module (src/ not in PYTHONPATH) → fixed in Procfile
- Auth: refresh token expiration miscalculated; change-password endpoint didn't verify caller == target user (horizontal privilege escalation fixed)
- Search on enum columns: PostgreSQL ILIKE doesn't work on native enum types → fixed with cast to String
- Password reset email (Resend library): API changed between versions, required 3 fix commits to stabilize
- Tests: trailing slash on helper `_url()` caused all endpoints to return 404 (API has `redirect_slashes=False`)
- Tests: PostgreSQL deadlocks on parallel TRUNCATE → fixed with `session.rollback()` before `session.close()`

**Sale completion / FEFO** (commit 3939264, Dec 16 2025 — 161 lines added, 4 files):
- Created `utils/sales_stock.py` and `sale_batch_usages` table
- FEFO: batches are consumed in expiration-date order
- Full refund system added Dec 29: stock restored with reverse-FEFO, automatic status `partially_refunded`/`refunded`

**Tests (pytest):** Written retroactively and published as one batch on Apr 2, 2026. Uses real PostgreSQL, truncates between tests. CI failed initially (Node 16 deprecated, PostgreSQL not ready in time) — fixed same day.

**Schema evolution:** 12 Alembic migrations after initial design, including: refresh token field, sensor_readings table, soft-delete columns, enum types, Numeric monetary columns, FK indexes, partial unique index on lot_code.

### Frontend — key development facts (112 commits, Mar–May 2026)

**Timeline:**
- Mar 2026: JWT auth flow, sales module, products/batches/categories
- Mar 31: dashboard connected to real API data (replaced mocks), pharmacy metrics
- Apr 1: barcode scanner, sensor monitoring widget, dashboard quick actions
- Apr 2: Vitest + RTL + MSW test setup (11 tests)
- Apr 13–14: full role/permission system restructured; route guards migrated from role-name to granular permissions
- May 3–5: sale form major improvements (inline batch creation, smart payment section, visual scanner mode)

**Most-iterated files:** axios.js (20 commits), nav config (15), product form (13), user form (13), sale form (11)

**Key bugs fixed (51 fix commits total):**
- All API endpoints had trailing slash → 404; fixed by aligning frontend URL calls
- Nav permission logic was inverted (hiding items that should show, showing items that should hide) — fixed twice
- Settings cached in browser caused blank page on existing sessions after config change
- CORS: frontend received network error instead of API error on 500s (fixed on API side)
- Axios: no handling for HTTP 429 (rate limit) → added one retry respecting Retry-After header
- Enums desynchronized between frontend and backend after API changes
- Payment amount field: typing concatenated with initial "0" instead of replacing it
- Layout: attempted to move sidebar to right side, caused multiple CSS side effects on icons/logo, partially reverted
- CI: VITE_SERVER_URL not set in test environment → fixed same day

**Sensor module:** initial version was technical; rewritten as user-facing guide with WiFi setup steps and troubleshooting.

**Dashboard optimization:** 3 parallel API calls replaced with single `/dashboard/stats` endpoint.

**Vercel config:** single rewrite rule `"source": "/(.*)", "destination": "/"` — required for React Router to work on direct URL access or page reload.

### Sensor firmware — key facts (5 commits, May 5–17 2026)

**Only functional bug fixed:** trailing slash in API_URL caused 404/redirect on all POST requests. Fixed in commit 43584c4.

**Hardware incident:** RGB LED on ESP32-C5 shared GPIO 8 with SDA (I²C bus for AHT10). Attempt to control LED by software caused I²C interference. LED was physically desoldered; all LED code removed.

**Stable parameters (never changed):** 30 s reading interval, −10–85 °C and 0–100 % validation ranges, JSON payload `{temperature, humidity, device_id}`, WiFi reconnect logic, 3-minute captive portal timeout.

**Not yet implemented:** OLED screens (enclosure redesign in progress), local data storage, OTA updates.

**Sensor not yet deployed in real pharmacy** — awaiting enclosure completion.

### Barcode reader
Tested with real pharmacy products — works correctly. Only visual/UX corrections were needed (mode indicator, focus handling).

---

## Writing Style Guide (Chapter 4)

1. **Simple language over technical** — replace jargon with functional descriptions. "Functions as a router directing requests" not "layered filter architecture."
2. **Why over what** — every decision gets a concrete simple reason. "We chose X because Y."
3. **Omit names that can't be defended** — if asked "what is X?" and there's no simple answer, don't mention X by name.
4. **Short paragraphs** — no long introductions, get to the point.
5. **End with visible result** — if there's a screenshot or photo, the section ends pointing to it.
6. **No redundancy with visuals** — if the image shows it, the text complements, not describes.
