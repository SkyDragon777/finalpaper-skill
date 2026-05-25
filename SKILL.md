---
name: finalpaper
description: >-
  Parse academic PDF papers (≤200 pages, ≤200 MB each) with MinerU Precision API and generate
  bilingual reading guides (finalpaper.md / finalpaper_cn.md) containing author backgrounds with
  verified sources, article overviews, key concept explanations, and embedded figure-by-figure
  analysis. Use this skill whenever the user wants to parse papers, generate paper reading reports,
  create academic paper summaries with figures, build a bilingual paper knowledge base, or asks
  about "论文阅读报告", "论文解析", "MinerU", "finalpaper", "paper reading guide".
---

# finalpaper — AI Agent Paper Reading Report

Parse all PDF papers in this folder with the [MinerU Precision API](https://mineru.net) and generate bilingual reading guides. Compatible with any LLM-powered agent (Codex CLI, Claude Code, Cursor, opencode, etc.).

## Directory Contract

- PDF files stay in this folder.
- Output files `finalpaper.md` and `finalpaper_cn.md` go in this same folder, next to the PDFs.
- All MinerU parsing outputs, zip files, status files, and manifests stay under `mineru/`.
- Do not create parsing outputs beside the PDFs except for `finalpaper.md`, `finalpaper_cn.md`, and this skill file.

```
<project-folder>/
  *.pdf
  finalpaper.md             # final output (English)
  finalpaper_cn.md          # final output (Chinese)
  SKILL.md                  # this skill file
  mineru/
    manifest.json           # machine-readable processed/skip list
    <paper-slug>/
      full.md
      *_content_list.json
      *_content_list_v2.json
      images/
```

## Required API Token

Replace `<YOUR_MINERU_API_TOKEN_HERE>` below with your [MinerU API token](https://mineru.net/apiManage/token). Do not commit it. If the token is missing, ask the user to provide it.

```text
<YOUR_MINERU_API_TOKEN_HERE>
```

## Skip Already-Processed Papers

Always consult `mineru/manifest.json` before uploading. A PDF is already processed when:

1. It appears in `mineru/manifest.json` under `processed_papers` with `status: "done"`
2. The corresponding `full_md` path exists
3. At least one `*_content_list.json` or `*_content_list_v2.json` exists in the output directory

If all three conditions are met, skip the upload. `manifest.json` is the single source of truth.

## Slug Rule

Use stable lowercase slugs for output directories: remove `.pdf`, lowercase, replace spaces and punctuation with `-`, collapse repeated `-`.

- `Attention Is All You Need.pdf` → `attention-is-all-you-need`
- `Learning representations by back-propagating errors.pdf` → `backpropagation-1986`

## Processing Order

1. List root-level `*.pdf` files only (ignore PDFs inside `mineru/`).
2. Read `mineru/manifest.json` and skip all `done` papers.
3. Upload only missing PDFs to MinerU Precision API → see [references/mineru-api.md](references/mineru-api.md).
4. Store all outputs under `mineru/<paper-slug>/`.
5. Update `mineru/manifest.json` after each successful parse.
6. After all PDFs are parsed, read all `mineru/*/full.md` and generate both `finalpaper.md` and `finalpaper_cn.md` → see [references/output-rules.md](references/output-rules.md) and [references/translation-rules.md](references/translation-rules.md).

## Related Files

| File | Open when |
|---|---|
| [references/mineru-api.md](references/mineru-api.md) | You need the exact MinerU Precision API workflow (upload, poll, download) |
| [references/output-rules.md](references/output-rules.md) | You are generating `finalpaper.md` and need figure/table/formula rules |
| [references/translation-rules.md](references/translation-rules.md) | You are generating `finalpaper_cn.md` and need Chinese translation rules |
