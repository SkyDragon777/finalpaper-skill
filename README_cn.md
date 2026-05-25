# finalpaper — AI Agent 论文阅读报告

[English](README.md)

为 AI agent 设计的可复用技能：通过 [MinerU Precision API](https://mineru.net) 解析文件夹下所有 PDF 论文，生成包含作者背景、概念解释和内嵌逐图分析的双语阅读指南。每篇 PDF 需 ≤200 页且 ≤200 MB（MinerU API 限制）。

## 本项目做什么

1. **解析 PDF** → MinerU 云端 API 提取文本、公式（LaTeX）、表格（HTML）、图示以及结构化元数据，输出至 `mineru/<paper-slug>/`。
2. **生成阅读指南** → AI agent 产出 `finalpaper.md`（英文）和 `finalpaper_cn.md`（中文），每份指南包含：
   - 附有可验证来源的作者简介
   - 文章概览与关键词定义
   - 深度概念讲解
   - 逐图描述（内嵌图片）
   - 历史背景与学术圈分析
3. **自文档化** → `agent.md` 作为内置指引，供任何后续 agent 无需人工指导即可继续处理更多论文。

## 目录结构

```
.
├── README.md                           # 本文件（英文）
├── README_cn.md                        # 本文件（中文）
├── agent.md                            # 流水线 agent 指令
├── finalpaper.md                       # 最终阅读指南（英文）
├── finalpaper_cn.md                    # 最终阅读指南（中文）
├── *.pdf                               # 源论文
├── .gitignore
└── mineru/
    ├── manifest.json                   # 机器可读的处理状态
    ├── <paper-slug>/                   # 每篇论文一个目录
    │   ├── full.md                     # MinerU Markdown 输出
    │   ├── *_content_list.json         # 结构化内容清单
    │   ├── *_content_list_v2.json      # 按页分组的结构化输出
    │   └── images/                     # 提取的图示
    └── _api_runs/                      # API 响应日志（临时文件）
```

## 快速开始

### 前置条件

- [MinerU API token](https://mineru.net/apiManage/token)。请将 `agent.md` 中的 `<YOUR_MINERU_API_TOKEN_HERE>` 替换为你的 token，或者让 agent 询问你后手动填入。
- Python 3.10+，安装 `requests` 库（或通过 `pip install mineru-open-sdk`）
- Git（用于版本控制）

### 处理新论文

1. 将 PDF 文件放入本目录。
2. 按照 `agent.md` 中描述的 agent 工作流运行：
   - 检查 `mineru/manifest.json`，跳过已处理的论文
   - 将未处理的 PDF 上传至 MinerU Precision API：
     ```
     POST https://mineru.net/api/v4/file-urls/batch
     ```
   - 将每个 PDF 以 PUT 方式上传到返回的签名上传 URL
   - 轮询 `GET /api/v4/extract-results/batch/{batch_id}` 直至完成
   - 将结果 zip 解压至 `mineru/<paper-slug>/`
3. 所有 PDF 解析完成后，生成 `finalpaper.md` 和 `finalpaper_cn.md`。

完整的分步说明及 API token 设置请参见 `agent.md`。

### 使用 MinerU Python SDK

```python
from mineru import MinerU

client = MinerU("your-api-token")
result = client.extract("./paper.pdf", model="vlm", language="en")
result.save_all("./mineru/paper-slug/")
```

## 设计决策

- **双语输出**：英文保证精确性，中文降低阅读门槛。两份文件采用相同结构和图片引用。
- **仅正文图示**：仅收录论文正文中的图示。附录可视化会注明但不嵌入，使指南聚焦核心内容。
- **来源标签**：每条论断均标注 `[FROM CAPTION]`、`[FROM TEXT]` 或 `[INFERRED]`，方便读者追溯信息来源。
- **自包含图片**：图示使用相对路径从 `mineru/<slug>/images/` 引用——无需外部托管。


