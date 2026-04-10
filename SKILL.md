---
name: obsidian-auto-note
description: 依据圈选优先的焦点总结；若无圈选则生成候选摘要并需用户确认。
---

## 目标与原则

- **目标**：将截图或文档高效转为 Obsidian 原子笔记，包含"原文摘录"与"候选摘要"。
- **原则**：
    1.  **保留事实**：保留原文核心事实与结构，不做主观解读。
    2.  **圈选优先**：若输入包含圈选/高亮，摘要将严格聚焦于这些区域。
    3.  **来源可靠**：所有摘要内容必须直接源自原文，仅做压缩与重组。
    4.  **必需确认**：任何自动生成的摘要在被用户确认前，都处于"待定"状态。

## 输入与输出

### 输入参数

- `files` (必填): `string[]` - 待处理的图片或文档路径数组。
- `vault_path` (必填): `string` - Obsidian Vault 的根目录路径。
- `output_dir` (可选): `string` - 笔记产出目录，默认为 Vault 下的 `notes/`。
- `attachments_dir` (可选): `string` - 附件存放目录，默认为 `attachments/`。
- `note_title` (可选): `string` - 手动指定笔记标题。
- `keep_order` (可选): `boolean` - 保持多文件处理顺序，默认为 `true`。
- `enable_summary` (可选): `enum('auto', 'never')` - 控制摘要生成。`auto` (默认) 表示有圈选则聚焦总结，无圈选则通读全文生成候选摘要；`never` 则完全禁用。
- `require_confirmation` (可选): `boolean` - 是否必须用户确认摘要，默认为 `true`。
- `summary_length` (可选): `number` - 摘要的要点数量，建议范围 3–7，默认为 `5`。
- `summary_style` (可选): `enum('bullet', 'paragraph')` - 摘要样式，默认为 `bullet` (项目符号)。
- `highlight_detection` (可选): `boolean` - 是否启用对图片中圈选/高亮区域的检测，默认为 `true`。

### 输出规范

- **文件命名**: 建议 `YYYYMMDD-HHMM-<basename>.md`。
- **产物结构**: 一来源一笔记。笔记文件包含 Frontmatter、附件引用、原文摘录、候选摘要。
- **Frontmatter 字段**:
    - `title`: `string` - 笔记标题。
    - `created`: `string` - 笔记创建时间。
    - `source_type`: `string` - 来源类型 (`image`, `pdf` 等)。
    - `source_path`: `string` - 附件在 Vault 中的相对路径。
    - `summary_status`: `enum('pending', 'confirmed')` - 摘要确认状态，默认为 `pending`。
    - `summary_source`: `enum('highlight', 'none')` - 摘要内容来源。`highlight` 表示源自圈选区域，`none` 表示源自全文。

## 硬约束（受控总结）

对原文进行摘要时，必须在以下边界内进行，确保信息不失真：

- **禁止**:
    - 引入任何外部知识、个人观点或推断。
    - 创造原文不存在的标签、分类或跨页链接。
    - 改写事实、数字、术语、引用或因果关系。
    - 对原文进行纠错、反驳或补充。
    - 合并/拆分段落来创造新的语义逻辑。
    - 清理所谓的"噪音"，如页眉、页脚、版权声明（除非它们明确不属于正文内容）。

- **允许**:
    - 在不改变事实的前提下，对原文的**核心要点**进行压缩和重组。
    - 将一个或多个连续段落的内容，浓缩为 3–7 条关键信息（项目符号列表）。
    - 保留原文的关键术语、数字、专有名词和引用。
    - 维持原文要点之间的基本逻辑顺序。
    - 若有圈选/高亮，摘要内容**必须**主要来自这些区域的文本。
    - 若无圈选，则通读全文生成要点，并明确标记为"候选摘要"。

## 最小流程

1.  **保存原件**: 将原始文件按日期存入 `attachments_dir`，保留原始文件名。
2.  **检测圈选**:
    - 若输入为图片且 `highlight_detection=true`，识别图中的典型标注（如矩形、手绘圆圈、高亮色块）。
    - 若为 PDF，优先使用 PDF Annotation 工具提取高亮和注释区域。
3.  **提取文本**:
    - **圈选优先**: 如果检测到圈选/高亮区域，优先从这些区域通过 OCR 提取文本作为摘要的核心输入。
    - **通读模式**: 若无圈选，则对整个文档/图片进行 OCR 或文本抽取，作为生成全文摘要的输入。
    - **原文保留**: 完整提取原文所有文本，用于"原文摘录"部分。
4.  **生成 Markdown**:
    - 插入包含 `summary_status: pending` 和 `summary_source` 的 YAML frontmatter。
    - 嵌入对原始附件的 Markdown 引用 `![[...]]`。
    - 创建正文，包含两个固定部分：
        - `## 原文摘录`：存放完整的、未经修改的原文文本。
        - `## 候选摘要（待确认）`：存放依据"硬约束"生成的项目符号列表。
5.  **命名与保存**: 按命名规范写入 `output_dir`。

## 最小模板

```markdown
---
title: "笔记标题"
created: "2026-04-10T18:00:00+08:00"
source_type: "image"
source_path: "attachments/2026-04-10/source_screenshot.png"
summary_status: "pending"
summary_source: "highlight" 
---

![[source_screenshot.png]]

# {{note_title}}

## 原文摘录

{{full_original_text}}

## 候选摘要（待确认）

- {{summary_point_1}}
- {{summary_point_2}}
- {{summary_point_3}}
- {{summary_point_4}}
- {{summary_point_5}}

```

## 使用与确认 (龙虾 / Claude Code)

1.  **实现方式**:
    - 注册一个命令，接收上述定义的输入参数。
    - 内部调用 OCR/文档解析服务，并依据"硬约束"和"流程"执行文本提取与受控总结。
    - 将生成的 Markdown 内容写入用户指定的 Obsidian Vault 路径。
    - **核心逻辑**: 在工具的系统提示（System Prompt）中固化"硬约束 + 流程 + 必需确认"的原则，确保 AI 行为一致。

2.  **确认流程**:
    - **生成后触发**: 笔记成功生成后，Skill 应通过消息或弹窗主动提示用户："笔记已生成，请确认候选摘要。"
    - **用户确认操作**: 用户可以通过点击按钮（如"确认摘要"）、回复特定指令或在笔记中手动修改 frontmatter 来完成确认。
    - **状态更新**: 一旦用户确认，Skill 或相关工具应自动更新对应笔记文件的 frontmatter，将 `summary_status` 字段的值从 `pending` 修改为 `confirmed`。
    - **展示优化**: 在 Obsidian 中，可以利用 `Dataview` 等插件，创建一个视图来展示所有 `summary_status: pending` 的笔记，方便用户批量确认。
