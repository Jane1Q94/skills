---
name: web-to-pdf
description: 将任意网页转换为 PDF 文件并保存到指定目录。
---

## 目标与原则

- **目标**：将用户提供的 URL 网页转换为 PDF 文件，保存到 Obsidian vault 的附件目录中。
- **依赖**：使用 Google Chrome（macOS 下的 `/Applications/Google Chrome.app`）的 headless 模式生成 PDF。
- **原则**：
    1.  **完整渲染**：等待页面完全加载后再生成 PDF，确保 JavaScript 渲染的内容也被包含。
    2.  **无头运行**：Chrome 以 headless 模式运行，不显示窗口。
    3.  **保留命名**：PDF 文件名与笔记中引用的文件名保持一致。

## 输入与输出

### 输入参数

- `url` (必填): `string` - 要转换的网页 URL。
- `output_dir` (必填): `string` - PDF 文件输出目录（建议为 Obsidian vault 的 `attachments/` 目录）。
- `filename` (可选): `string` - PDF 文件名（含扩展名 `.pdf`），默认为 URL 对应的 slug 或时间戳命名。
- `wait_for` (可选): `number` - 等待页面加载的秒数，默认为 5 秒。

### 输出

- 保存到 `output_dir` 的 PDF 文件路径。
- 返回文件路径供笔记引用。

## 最小流程

1.  **生成 PDF 文件名**：若未指定 `filename`，从 URL 提取 slug 或使用 `YYYYMMDD-HHMM-url-slug.pdf` 格式。
2.  **调用 Chrome headless**：执行 Chrome headless 模式的打印命令，将 URL 输出为 PDF。
    ```bash
    /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
      --headless \
      --disable-gpu \
      --no-sandbox \
      --print-to-pdf="$output_dir/$filename" \
      --print-to-pdf-no-header \
      --run-all-compositor-stages-before-draw \
      --dump-dom \
      "$url" 2>/dev/null || \
    /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
      --headless \
      --disable-gpu \
      --print-to-pdf="$output_dir/$filename" \
      "$url"
    ```
    > 注：建议在执行命令后等待 `wait_for` 秒确保渲染完成。
3.  **确认文件**：检查 PDF 是否成功生成。
4.  **返回路径**：返回 PDF 文件的绝对路径，供笔记引用。

## 引用格式

生成的 PDF 在笔记中以以下格式引用：

```markdown
[📄 PDF](attachments/YYYY-MM-DD/filename.pdf)
```

## 错误处理

- 若 Chrome 命令失败，尝试使用 `--print-to-pdf-no-header` 参数。
- 若仍失败，返回错误信息并提示用户手动使用浏览器"打印为 PDF"功能。
