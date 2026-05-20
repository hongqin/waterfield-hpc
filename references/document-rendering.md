# Document Rendering: DOCX and PDF Generation

Generate NIH-formatted DOCX and PDF documents from markdown source files on Waterfield. Both tools use Python libraries already available in the `evo2` conda environment.

## Prerequisites

```bash
source ~/miniforge3/etc/profile.d/conda.sh
conda activate evo2
pip install python-docx lxml weasyprint markdown-it-py
```

## Generating DOCX (Word) Files

The R01 project uses a custom renderer (`app/render.py`) that converts markdown to NIH R01-style DOCX with automatic figure insertion.

```bash
cd ~/2026-06-05-R01-viralGPT

# Render all docs/*.md → app/*.docx
python3 app/render.py

# Render a single file
python3 app/render.py docs/research_strategy_approach.md
```

**How it works:**
- Uses `python-docx` + `lxml` to build Word documents
- Applies NIH formatting: Arial 11pt, 0.5" margins, single-spaced
- Supports headings, bold/italic, bullet/numbered lists, pipe tables
- Figures referenced in the text (e.g., `(Figure 9)`) are automatically pulled from `preliminary_figures/` using the `FIGURE_REGISTRY` dictionary in `render.py`
- Output goes to `app/` directory

**Adding new figures:** Edit the `FIGURE_REGISTRY` dict in `app/render.py`:
```python
"Figure N": (
    "figN_name.png",
    "Figure N. Caption text describing the figure."
),
```

## Converting DOCX to PDF

The project includes `app/render_pdf.py`, which renders markdown directly to PDF via `weasyprint` (HTML intermediary).

```bash
cd ~/2026-06-05-R01-viralGPT

# Default: render research_strategy_approach.md → app/research_strategy_approach.pdf
python3 app/render_pdf.py

# Render any markdown file
python3 app/render_pdf.py docs/significance.md
```

**How it works:**
- Parses markdown with `markdown-it-py`
- Applies NIH-style CSS (Arial 11pt, 0.5" margins, letter size)
- Embeds available preliminary figures as base64 inline images
- Produces a single PDF suitable for internal routing / review
- Adds a "DRAFT for internal routing" header

**No LibreOffice needed.** The PDF is generated entirely in Python — no `libreoffice --headless` or external tools required. This matters on Waterfield where LibreOffice is not installed.

## Workflow: Markdown → DOCX + PDF

Typical workflow for generating both formats:

```bash
cd ~/2026-06-05-R01-viralGPT

# 1. Edit the markdown source
#    e.g., docs/research_strategy_approach.md

# 2. Generate DOCX for NIH submission
python3 app/render.py docs/research_strategy_approach.md

# 3. Generate PDF for internal review / routing
python3 app/render_pdf.py docs/research_strategy_approach.md

# Output files:
#   app/research_strategy_approach.docx  (for NIH upload)
#   app/research_strategy_approach.pdf   (for internal routing)
```

## Troubleshooting

| Problem | Solution |
|---|---|
| `ModuleNotFoundError: weasyprint` | `pip install weasyprint markdown-it-py` |
| `ModuleNotFoundError: python-docx` | `pip install python-docx lxml` |
| Figure missing in output | Check that the PNG exists in `preliminary_figures/` and is registered in `FIGURE_REGISTRY` |
| PDF text is wrong font | WeasyPrint uses system fonts; Arial/Helvetica should be available. Falls back to sans-serif. |
| Large PDF file size | Figures are base64-embedded; reduce source PNG resolution or DPI if needed |
