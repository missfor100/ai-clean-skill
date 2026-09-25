---
name: ai-clean
description: 自动消除 AI 生成文件上的生成标记（图片 AIGC/C2PA 元数据、右下角「AI生成」水印、Office/PDF 工具指纹）。凡是刚生成或改写完图片、docx/xlsx/pptx/pdf，或用户说「去痕 / 去掉 AI 标记 / 清元数据 / 自动消除印记」时使用。
---

# ai-clean — 生成物自动去痕

把刚产出的文件跑一遍去痕流水线，落盘即干净。

## 何时触发

- `image_gen` / `image_edit` 写完图片后
- 用 python-docx / openpyxl / python-pptx / reportlab 写出 Office/PDF 后
- 用户要求「去 AI 标记 / 去痕 / 清掉水印元数据」

## 标准动作

脚本位置（按顺序解析）:

1. 环境变量 `AI_CLEAN_SCRIPT` 指定的路径（若已设置）
2. 本 skill 目录下的 `auto_clean.py`

单文件 / 批量:

```powershell
$py = $env:MIMO_PYTHON; if (-not $py) { $py = "python" }
$script = $env:AI_CLEAN_SCRIPT; if (-not $script) { $script = "<本 skill 目录>\auto_clean.py" }

# 默认输出副本去痕（生成 路径\文件.cleaned.png，原图不动）
& $py $script "路径\文件.png"

# 原地覆盖（旧行为，有丢失原始数据风险）
& $py $script "路径\文件.png" --in-place

# 整目录（默认输出到同级 <目录名>.cleaned\）
& $py $script "路径\目录"

# 只清元数据、不动可见水印
& $py $script "文件.png" --no-watermark

# 输出到别处 + 指定作者
& $py $script ".\docs" --out ".\cleaned" --author 张三
```

覆盖格式: `png/jpg/webp/tiff`、`docx/xlsx/pptx`、`pdf`。

依赖: `Pillow`、`python-docx`、`openpyxl`、`python-pptx`、`pypdf`（系统若装有 `exiftool` 会自动优先调用）。

## 做完必须核验

对结果文件再扫一遍关键词，命中为 0 才算完成:

- `AIGC` / `tc260` / `ContentProducer`
- `python-docx` / `python-pptx` / `openpyxl` / `Steve Canny` / `ReportLab`
- 可选：肉眼看右下角是否还有「AI生成」

## 补强（可选，非必须）

| 需求 | 方案 |
|------|------|
| 更强可见水印擦除（多平台角标） | [OpenNoMark](https://github.com/NanmiCoder/OpenNoMark) `opennomark in.png -o out/` |
| ExifTool 全量剥元数据 | [xxd-strip-ai-meta](https://github.com/nevertoday/xxd-strip-ai-meta)，或本工具检测到 `exiftool` 时自动调用 |
| 纯 Python 看/删图元数据 | [img-meta](https://github.com/themostjomo/img-meta) |

## 合规提醒（必须遵守）

- 本工具面向**本地整理、私用文件**的元数据清洁。
- **对外发布** AI 生成内容时，请遵守《人工智能生成合成内容标识办法》等规定，不得以去标识方式冒充人工创作。
- 不要对无权修改的第三方作品去水印。

## 精简原则

- 默认输出 `.cleaned` 副本；需要原地覆盖时显式加 `--in-place`（可配 `--keep-backup`）
- 不引入重量级模型依赖；检测到 ExifTool 就用，没有就走纯 Python
- 处理后打印 `[meta-stripped|exiftool|ooxml-cleaned|pdf-cleaned|watermark-covered]` 便于核对
