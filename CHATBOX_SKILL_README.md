# Chatbox Skill 说明

这是原始 `pdf-inspector` 完整项目加上 `SKILL.md` 的版本。

- 原项目源码没有删减。
- 新增的 `SKILL.md` 用于让支持 GitHub Skills 的 Chatbox 识别此仓库。
- `pdf-inspector` 本身负责 PDF 分类、结构化文本提取和 Markdown 转换，不自带 OCR。
- 若 Chatbox 所在环境不能执行本地 CLI/Cargo，Skill 仍可能被识别，但无法真正运行仓库里的 Rust CLI；此时需要额外的本地工具执行能力（如 MCP/终端工具）。
