# finalpaper — AI Agent Paper Reading Report

[中文版](README_cn.md)

An opinionated skill for AI agents: parse all PDF papers in a folder via [MinerU Precision API](https://mineru.net), then generate bilingual reading guides with author backgrounds, concept explanations, and embedded figure analysis. Each PDF must be ≤200 pages and ≤200 MB (MinerU API limits).

## What This Project Does

1. **Parse PDFs** → MinerU cloud API extracts text, formulas (LaTeX), tables (HTML), figures, and structured metadata into `mineru/<paper-slug>/`.
2. **Generate reading guides** → The AI agent produces `finalpaper.md` (English) and `finalpaper_cn.md` (Chinese), each containing:
   - Author biographies with verified sources
   - Article overview and keyword definitions
   - In-depth concept explanations
   - Figure-by-figure descriptions with embedded images
   - Historical context and academic circle analysis
3. **Self-documenting** → `agent.md` serves as the onboard instruction for any compatible AI agent to continue processing additional papers without manual guidance.

## Directory Structure

```
.
├── README.md                           # This file (EN)
├── README_cn.md                        # This file (CN)
├── agent.md                            # Agent instructions for the pipeline
├── finalpaper.md                       # Final reading guide (English)
├── finalpaper_cn.md                    # Final reading guide (Chinese)
├── *.pdf                               # Source papers
├── .gitignore
└── mineru/
    ├── manifest.json                   # Machine-readable processing state
    ├── <paper-slug>/                   # One directory per paper
    │   ├── full.md                     # MinerU Markdown output
    │   ├── *_content_list.json         # Structured content inventory
    │   ├── *_content_list_v2.json      # Page-grouped structured output
    │   └── images/                     # Extracted figures
    └── _api_runs/                      # API response logs (ephemeral)
```

## Quick Start

### Prerequisites

- A [MinerU API token](https://mineru.net/apiManage/token). Replace `<YOUR_MINERU_API_TOKEN_HERE>` in `agent.md` with your token, or let the agent ask you for it.
- Python 3.10+ with `requests` library (or `pip install mineru-open-sdk`)
- Git (for version control)

### Processing New Papers

1. Drop PDF files into this directory.
2. Run the agent workflow described in `agent.md`:
   - Check `mineru/manifest.json` to skip already-processed papers
   - Upload unprocessed PDFs to MinerU Precision API:
     ```
     POST https://mineru.net/api/v4/file-urls/batch
     ```
   - PUT each PDF to the returned signed upload URL
   - Poll `GET /api/v4/extract-results/batch/{batch_id}` until done
   - Extract the result zip into `mineru/<paper-slug>/`
3. After all PDFs are parsed, generate `finalpaper.md` and `finalpaper_cn.md`.

See `agent.md` for complete step-by-step instructions and API token setup.

### Using the MinerU Python SDK

```python
from mineru import MinerU

client = MinerU("your-api-token")
result = client.extract("./paper.pdf", model="vlm", language="en")
result.save_all("./mineru/paper-slug/")
```

## Design Decisions

- **Bilingual output**: English for precision, Chinese for accessibility. Both files share identical structure and image references.
- **Body-only figures**: Only figures from the main paper body are included. Appendix visualizations are noted but not embedded, keeping the guide focused on core content.
- **Provenance tags**: Every claim is tagged `[FROM CAPTION]`, `[FROM TEXT]`, or `[INFERRED]` so readers can trace information back to its source.
- **Self-contained images**: Figures are referenced from `mineru/<slug>/images/` using relative paths — no external hosting needed.


