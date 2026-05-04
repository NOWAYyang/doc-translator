# Changes v1

## 修复内容

### 1. PDF 导出 HTTP 500

**根因 1 — `multi_cell` x 位置未重置**
fpdf2 v2.8.x 中 `multi_cell` 完成后 `x` 停留在右边距，下一个 `multi_cell(0, ...)` 可用宽度为 0，抛出 `Not enough horizontal space to render a single character`。

- 修复：每个 `multi_cell` 前调用 `pdf.set_x(pdf.l_margin)`

**根因 2 — `bytearray` 与 Starlette Response 不兼容**
`pdf.output()` 返回 `bytearray`，但 Starlette 的 `Response.render()` 只接受 `bytes | memoryview`，传入 `bytearray` 导致 `AttributeError`。

- 修复：改为 `bytes(pdf.output())`

**根因 3 — 字体参数错误**
- `uni=True` 已从 fpdf2 v2.5.1 起废弃
- `index=0` 参数在 fpdf2 2.8.7 中不被 `add_font()` 支持（TTC 字体无需索引参数即可自动使用）
- 字体搜索路径覆盖不足（缺少 macOS STHeiti / Hiragino Sans GB / Songti，Windows msyhbd / simfang，Linux NotoSansSC / DroidSansFallbackFull）

- 修复：清理 `add_font` 参数，扩展 `_CHINESE_FONT_CANDIDATES` 列表

### 2. 恢复翻译 HTTP 500

**根因 — Worker 函数对已翻译块返回 None**
`_sse_generate` 中 `worker()` 对 `idx in skip_set` 使用 bare `return`（返回 `None`），主循环 `for t in done_set` 尝试把 `None` 解包为 5 元组 `(idx, text, elapsed, attempts, err)`，导致 `cannot unpack non-iterable NoneType object`。

- 修复：不再创建已跳过块的任务，用 `if i not in skip_set` 过滤 `work_tasks`

**相关问题 — 循环引用**
`resume_translate` 中 `params = progress.get("params", {})` 获取的是原 dict 引用；`params["resume_data"] = progress` 后形成 `params↔progress↔params` 循环引用，后续 `json.dump` 序列化时抛出 `Circular reference detected`。

- 修复：改为浅拷贝 `dict(progress.get("params", {}))`
- 另：`_save_progress` 保存时过滤掉运行时字段 `resume_data`

**命名不一致**
`saved params key` 为 `source_language`/`target_language`（来自 `StreamTranslateRequest` 模型），但 resume 代码读取 `source_lang`/`target_lang`，导致 fallback 生效但语言参数实际被忽略。

- 修复：对齐为 `source_language`/`target_language`

### 3. 导出端点错误处理增强

TXT / DOCX / PDF 三个导出端点均添加 `try/except`，异常时返回中文错误信息（JSON 格式 500 响应），避免抛出未处理异常导致通用 Internal Server Error。

---

## 影响范围

| 文件 | 改动 |
|------|------|
| `main.py` | PDF 导出（x 位置、bytearray、字体）、恢复翻译（worker 跳过、循环引用、命名）、导出错误处理、字体搜索路径扩展 |
