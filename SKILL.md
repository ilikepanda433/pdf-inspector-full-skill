---
name: pdf-inspector-skill
description: 使用完整的 Firecrawl pdf-inspector 项目对 PDF 进行类型检测、结构化文本提取、Markdown 转换、表格与多栏版面分析、指定页面处理，并识别需要 OCR 的页面。默认用中文回答。
---

# PDF Inspector Skill（完整版）

本技能基于仓库内的完整 `pdf-inspector` 源码。它的主要目标是：先判断 PDF 的类型和可提取性，再选择合适的解析方式，把结果整理为适合模型继续阅读、总结、引用和分析的 Markdown 或 JSON。

## 什么时候使用

当用户上传 PDF，或要求读取、提取、总结、检索、引用、分析 PDF 内容时，优先使用本技能。特别适合：

- 原生文本 PDF 的快速提取；
- 扫描型、图片型或混合型 PDF 的类型检测；
- 论文、报告、财务文档、法律文档等结构化内容；
- 多栏排版、表格、标题、列表、代码块、链接等结构的保留；
- 只处理指定页；
- 需要页码标记、JSON、TextItem 或布局分析的任务。

默认用中文向用户解释结果；除非用户明确要求其他语言。

## 核心工具

仓库提供两个原生 CLI：

- `pdf2md`：PDF → Markdown / JSON / TextItem JSON。
- `detect-pdf`：判断 PDF 类型，可进行布局分析。

PDF 类型包括：

- `TextBased`
- `Scanned`
- `ImageBased`
- `Mixed`

检测结果可包含需要 OCR 的页面。`pdf-inspector` 本身不负责 OCR；若发现需要 OCR 的页面，应使用当前环境可用的 OCR/视觉工具继续处理这些页面，并将结果与原生文本页合并。

## 优先调用方式

### 1. 如果系统已安装 CLI

直接使用：

```bash
pdf2md "document.pdf"
detect-pdf "document.pdf"
```

### 2. 如果 CLI 未全局安装，但当前工作目录是本仓库

使用 Cargo 从源码运行：

```bash
cargo run --bin pdf2md -- "document.pdf"
cargo run --bin detect-pdf -- "document.pdf"
```

在需要更高性能时可使用 `--release`：

```bash
cargo run --release --bin pdf2md -- "document.pdf"
cargo run --release --bin detect-pdf -- "document.pdf"
```

若 Rust/Cargo 不可用，不要假装已执行成功；明确告诉用户当前环境缺少可执行依赖，并改用当前环境已有的 PDF/视觉能力。

## 常用操作

### 普通 Markdown 提取

```bash
pdf2md "document.pdf"
```

或源码运行：

```bash
cargo run --bin pdf2md -- "document.pdf"
```

### JSON 输出

```bash
pdf2md "document.pdf" --json
```

### TextItem JSON

用于需要位置、字体或更细粒度文本项目数据的任务：

```bash
pdf2md "document.pdf" --items-json
```

### 只输出原始 Markdown

```bash
pdf2md "document.pdf" --raw
```

### 紧凑输出

适合长文档和节省 token：

```bash
pdf2md "document.pdf" --compact
```

### 插入分页标记

当用户需要保留页码、做逐页引用或定位时：

```bash
pdf2md "document.pdf" --pages
```

### 指定页面

```bash
pdf2md "document.pdf" --select-pages 1,3,5-10
```

页码为 1-based。

### 只检测 PDF 类型

```bash
detect-pdf "document.pdf"
```

JSON：

```bash
detect-pdf "document.pdf" --json
```

### 检测并分析布局

用于表格、多栏版面等结构分析：

```bash
detect-pdf "document.pdf" --analyze --json
```

## 推荐工作流

1. 如果任务只涉及少数页，优先使用 `--select-pages`，避免处理整份文档。
2. 对未知 PDF，先运行 `detect-pdf --json`。
3. 如果是 `TextBased`，直接用 `pdf2md` 提取。
4. 如果是 `Mixed`，提取可读文本，同时根据检测结果识别需要 OCR 的页面。
5. 如果是 `Scanned` 或 `ImageBased`，不要把空的 Markdown 当作成功结果；改用当前环境的 OCR/视觉能力。
6. 长 PDF 默认优先考虑 `--compact`；用户需要页码定位时使用 `--pages`。
7. 用户要求表格、版面或阅读顺序分析时，可使用 `detect-pdf --analyze --json`，并结合 Markdown 输出。
8. 最终回答应面向用户任务，不要把大段调试日志原样倾倒给用户。

## 结构与能力说明

本仓库的核心实现包括：

- PDF 类型检测；
- 字体和字符编码处理；
- ToUnicode / CID 解码；
- CJK、RTL、连字等文本处理；
- 内容流解析；
- 多栏布局与阅读顺序检测；
- 矩形、线框和启发式表格检测；
- Markdown 标题、列表、代码、caption、链接等结构转换；
- Tagged PDF 结构角色支持；
- 扫描型/混合型文档识别与 OCR 路由提示。

不要声称 `pdf-inspector` 自身已经执行 OCR。它的职责是检测、提取和路由；OCR 需要由其他可用工具完成。

## 回答风格

- 默认中文。
- 优先直接给用户需要的结论、摘要或提取结果。
- 如果检测到扫描页，说明哪些页面需要 OCR。
- 如果工具执行失败，明确说明是缺少 Cargo、CLI、文件权限、路径或其他运行时问题，不要伪造结果。
