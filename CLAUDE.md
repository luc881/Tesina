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
