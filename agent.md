# Skill: finalpaper — AI Agent Paper Reading Report

Parse all PDF papers (≤200 pages, ≤200 MB each) in this folder with the [MinerU Precision API](https://mineru.net) and generate bilingual reading guides. Compatible with any LLM-powered agent (Codex CLI, Claude Code, Cursor, opencode, etc.).

> **You are an AI agent executing this skill.** Follow the instructions below to parse PDFs and generate `finalpaper.md` / `finalpaper_cn.md`.

## Directory Contract

- PDF files stay in this folder.
- The final output files `finalpaper.md` and `finalpaper_cn.md` must be generated in this same folder, next to the PDFs.
- All MinerU parsing outputs, zip files, status files, manifests, and intermediate artifacts must stay under `mineru/`.
- Do not create parsing outputs beside the PDFs except for `finalpaper.md`, `finalpaper_cn.md`, and this skill file (`agent.md`).

Recommended layout:

```text
<your-project-folder>/
  *.pdf
  finalpaper.md             # final output (English)
  finalpaper_cn.md          # final output (Chinese)
  agent.md                  # this skill file
  mineru/
    manifest.json           # machine-readable processed/skip list
    <paper-slug>/
      full.md
      *_content_list.json
      *_content_list_v2.json
      *_model.json
      layout.json
      images/
    _api_runs/
      ...                   # API response logs, downloaded zips, temporary run artifacts
```

## MinerU API Token

Use the official MinerU Precision API token below for this local workflow only. Do not print it in logs, do not commit it, and do not send it to services other than `https://mineru.net` / MinerU upload URLs.

```text
<YOUR_MINERU_API_TOKEN_HERE>
```

Get your free token at: https://mineru.net/apiManage/token

## Already Processed Papers — Do Not Re-upload

These PDFs have already been processed by MinerU. If their `full.md` exists, skip upload and reuse the parsed outputs.

| Source PDF | Output directory | Main Markdown | Status | Notes |
|---|---|---|---|---|
| `Attention Is All You Need.pdf` | `mineru/attention-is-all-you-need/` | `mineru/attention-is-all-you-need/full.md` | done | Good parse; title/authors/abstract/figures present. |
| `Learning representations by back-propagating errors.pdf` | `mineru/backpropagation-1986/` | `mineru/backpropagation-1986/full.md` | done | Good enough parse; OCR enabled; front noise before the title is acceptable. |

Also read `mineru/manifest.json`; it is the machine-readable skip list. A PDF is considered already processed when:

1. It appears in `mineru/manifest.json` under `processed_papers` with `status: "done"`, and
2. The corresponding `full_md` path exists, and
3. At least one `*_content_list.json` or `*_content_list_v2.json` exists in the same output directory.

## Official MinerU Precision API Workflow

Use the official cloud API, not local `magic-pdf` CLI and not the lightweight Agent API.

### 1. Request signed upload URLs

Endpoint:

```text
POST https://mineru.net/api/v4/file-urls/batch
```

Headers:

```text
Authorization: Bearer <MINERU_API_TOKEN>
Content-Type: application/json
Accept: */*
```

Request body for English academic PDFs:

```json
{
  "files": [
    {
      "name": "example.pdf",
      "data_id": "example-slug",
      "is_ocr": false
    }
  ],
  "model_version": "vlm",
  "language": "en",
  "enable_formula": true,
  "enable_table": true
}
```

Notes:

- Use `model_version: "vlm"` for academic papers.
- Use `language: "en"` for these English papers.
- Use `is_ocr: true` for scanned or old PDFs. It was used successfully for `Learning representations by back-propagating errors.pdf`.
- A single request supports up to 50 files. For this folder size, batches of 10-20 are safer.

### 2. Upload each local file

The response contains `data.batch_id` and `data.file_urls`.

Upload each PDF with:

```text
PUT <file_urls[i]>
```

Body: raw PDF bytes. Do not add a custom `Content-Type` header.

After upload, MinerU automatically starts parsing. Do not call a separate submit endpoint.

### 3. Poll batch results

Endpoint:

```text
GET https://mineru.net/api/v4/extract-results/batch/{batch_id}
```

States to handle:

- `waiting-file`
- `pending`
- `running`
- `converting`
- `done`
- `failed`

When `state == "done"`, read `full_zip_url`.
When `state == "failed"`, record `err_msg` and do not silently retry forever.

### 4. Download and unpack results

Download `full_zip_url` into `mineru/_api_runs/<run-id>/` or directly archive it under `mineru/`.
Extract its contents into:

```text
mineru/<paper-slug>/
```

Keep at least:

- `full.md`
- `*_content_list.json`
- `*_content_list_v2.json`
- `*_model.json`
- `layout.json`
- `images/`

Then update `mineru/manifest.json` so future agents skip the processed PDF.

## Slug Rule

Use stable lowercase slugs for output directories:

- remove `.pdf`
- lowercase
- replace spaces and punctuation with `-`
- collapse repeated `-`
- keep historically meaningful short names when obvious

Examples:

- `Attention Is All You Need.pdf` -> `attention-is-all-you-need`
- `Learning representations by back-propagating errors.pdf` -> `backpropagation-1986`

## finalpaper.md Generation Requirements

After all PDFs have MinerU outputs, generate exactly one final `finalpaper.md` in this folder.

For each paper, include:

1. Authors, affiliations, and major-author background/circle. Use external sources when needed; mark unverified claims.
2. Article overview, keywords, and key concepts that are necessary to understand the paper.
3. Detailed figure-by-figure explanations.
   - The agent does not need to read image pixels.
   - Use `full.md`, figure captions, nearby paragraphs, and cross-references such as "Figure 1".
   - Clearly distinguish `[FROM CAPTION]`, `[FROM TEXT]`, and `[INFERRED]`.
   - If the Markdown does not support a claim, say so instead of inventing.
   - **CRITICAL — Embed the actual figure images**: For each figure, embed the extracted image from `mineru/<paper-slug>/images/<hash>.jpg` using relative paths from the root folder. Example: `![Figure 1](mineru/attention-is-all-you-need/images/d018247de...jpg)`. The images are already extracted by MinerU — the reader needs to see them alongside the textual explanations.
4. **Formula formatting**: Use proper LaTeX math delimiters exactly as MinerU outputs them in `full.md`:
   - Display (block) formulas: `$$ ... $$`
   - Inline formulas: `$ ... $`
   - Do NOT convert formulas to plain Unicode/ASCII text. Keep the original LaTeX from `full.md` so they render correctly in Markdown viewers with MathJax/KaTeX support.
5. **Appendix content filtering**: Only include figures and tables that appear in the main body of the paper (i.e., before the References / Acknowledgements section). Do NOT include appendix-only figures/tables unless they are truly essential to understanding the paper's core contribution. When appendix content is excluded, add a brief note in the output listing what was omitted and where the extracted images can be found (e.g., `mineru/<paper-slug>/images/`). Example: the Transformer paper has 5 figures but Figures 3–5 are attention visualization appendix figures — only Figures 1–2 (architecture and attention mechanism) belong in the main body.
6. **Bilingual output**: Alongside the English `finalpaper.md`, generate a Chinese version `finalpaper_cn.md` in the same directory. Translation rules:
   - Translate all explanatory text, analysis paragraphs, and concept explanations to natural Chinese (简体中文).
   - Keep UNCHANGED: paper titles, author names, institution names, figure captions (original English), ALL mathematical formulas, image markdown syntax, table markdown, citation markers, provenance tags like `[FROM CAPTION]`, URLs.
   - For technical terms on first occurrence, use the form "自注意力（Self-Attention）" — Chinese term first, English in parentheses.
   - The Processing Metadata section at the bottom can be fully in Chinese.
   - Maintain EXACT same section structure, image paths, and formula blocks. Only the narrative text between these elements changes language.

The front noise in some MinerU Markdown is acceptable. Do not discard an otherwise good parse only because text appears before the actual title; title detection can be handled during summary generation.

## Recommended Processing Order

When you execute this skill, follow this order:

1. List root-level `*.pdf` files only. Ignore PDFs inside `mineru/` because those are MinerU output copies.
2. Read `mineru/manifest.json`.
3. Skip all `done` papers whose `full.md` exists.
4. Upload only missing/unprocessed root-level PDFs to MinerU Precision API.
5. Store all outputs under `mineru/<paper-slug>/`.
6. Update `mineru/manifest.json` after each successful parse.
7. Only after every PDF is parsed, read all `mineru/*/full.md` files and generate both root-level `finalpaper.md` (English) and `finalpaper_cn.md` (Chinese).
