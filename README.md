# ai-clean

生成物自动去痕流水线：清理 AI 生成文件上的**隐式标识**、**可见水印**与 **Office/PDF 工具指纹**，让产出文件落盘即干净。

- 支持格式：`png` `jpg` `webp` `tiff` `docx` `xlsx` `pptx` `pdf`
- 单文件或整目录批量，原地处理或输出到指定目录
- 纯本地运行，不上传任何文件；不引入重量级模型依赖
- 既是独立命令行脚本，也可作为 MiMo Desktop 的 agent skill 自动接管

## 目录

- [快速开始](#快速开始)
- [命令行参数](#命令行参数)
- [工作原理](#工作原理)
- [作为 MiMo skill 使用](#作为-mimo-skill-使用)
- [处理结果核验](#处理结果核验)
- [补强工具](#补强工具)
- [合规提醒](#合规提醒)
- [更新日志](#更新日志)

## 快速开始

```powershell
# 依赖（任选其一安装方式）
pip install Pillow python-docx openpyxl python-pptx pypdf

# 单文件原地去痕（默认连右下角水印一起盖掉）
python auto_clean.py "照片.png"

# 整目录批量
python auto_clean.py ".\输出目录"

# 只清元数据，不动可见水印
python auto_clean.py "截图.jpg" --no-watermark

# 输出到别处，并指定写入文档的作者名
python auto_clean.py ".\docs" --out ".\cleaned" --author 张三

# 原地处理但保留 .bak 备份
python auto_clean.py "报告.docx" --keep-backup
```

系统若安装了 [ExifTool](https://exiftool.org)，图片会自动优先走 ExifTool 全量剥离，效果更好；没有则回退到内置纯 Python 实现。

## 命令行参数

| 参数 | 默认 | 说明 |
|------|------|------|
| `path`（位置参数） | — | 要处理的文件或目录（目录递归处理其中所有支持格式） |
| `--out DIR` | 原地 | 输出目录；不指定时直接覆盖原文件 |
| `--author 名字` | `用户` | 写入 Office/PDF 核心属性的作者名 |
| `--no-watermark` | 关 | 只清元数据，跳过可见水印覆盖 |
| `--keep-backup` | 关 | 处理前保留 `.bak` 备份 |

处理结果以标签形式打印，例如 `[meta-stripped+watermark-covered] 路径`：

| 标签 | 含义 |
|------|------|
| `meta-stripped` | 纯 Python 剥离图片元数据 |
| `exiftool` | 由 ExifTool 完成全量剥离 |
| `ooxml-cleaned` | Office 文件已清指纹 |
| `pdf-cleaned` | PDF 已重写元数据 |
| `watermark-covered` | 右下角水印已覆盖 |

## 工作原理

按文件类型分四条流水线，依次执行：

### 1. 图片隐式标识（XMP / C2PA / EXIF / AIGC）

AI 生成图片的隐式标识一般藏在容器的元数据段里，处理策略按格式分：

- **PNG**：手工解析文件块（chunk），保留像素与必要结构，直接丢弃所有文本/元数据块 —— `tEXt`、`zTXt`、`iTXt`（含 XMP 与国标 AIGC 标识）、`eXIf`。
- **JPEG**：用 Pillow 解码后仅重编码 RGB 像素（质量 95），APP1 里的 XMP / EXIF / C2PA 段随重编码自然丢弃。
- **WebP / TIFF**：同样重编码丢弃元数据。
- **有 ExifTool 时**：直接 `exiftool -all=` 全量剥离，优先于以上路径。

### 2. 图片可见水印（右下角角标）

不裁切画幅，用周边纹理「盖」掉角标：

1. **定位**：只扫描右下角区域（约宽 28% × 高 12%），找偏暗（RGB < 90）且低饱和的连续像素簇 —— 对应半透明深色圆角水印盒；像素太少或盒子太小视为噪声，不处理。
2. **兜底**：定位失败时使用固定右下角区域。
3. **覆盖**：从角标左侧/上方采样一块相近大小的纹理，缩放对齐后贴上，高斯模糊降噪；再对贴补边缘做一圈羽化模糊消除接缝。

### 3. Office 指纹（docx / xlsx / pptx）

两步走：

1. **核心属性**：用 python-docx / openpyxl / python-pptx 把 `author`、`lastModifiedBy` 改为指定作者，清空 `comments` / `subject` / `category` 等字段。
2. **包内 XML 重写**：Office 文件本质是 zip，遍历其中所有 `.xml`，按替换表清洗工具指纹，例如：
   - `python-docx` / `python-pptx` / `Steve Canny` → 移除或替换为中性值
   - `openpyxl` → `Microsoft Excel`
   - `ReportLab PDF Library` → `Microsoft Word`
   - `Microsoft Macintosh Word/PowerPoint/Excel` → 对应 Windows 版字符串
   - 正则删除国标 `{"AIGC": ...}` 标记 JSON

### 4. PDF 指纹

用 pypdf 逐页重建文档，重写元数据字典：`Creator` / `Producer` 改为 `Microsoft Word`，`Author` 取 `--author` 参数，清空 `Title` / `Subject` / `Keywords`，从而抹掉 ReportLab 等生成器指纹。

## 作为 MiMo skill 使用

把本仓库目录放入 MiMo Desktop 的 skills 目录（路径见「设置 → 插件」），即可让 agent 在每次生成图片/文档后**自动**调用去痕并核验：

1. skill 定义（[SKILL.md](SKILL.md)）描述触发时机：`image_gen`/`image_edit` 出图后、python-docx/openpyxl/python-pptx/reportlab 写出文件后、或用户说「去痕 / 去掉 AI 标记」。
2. agent 按顺序解析脚本路径：环境变量 `AI_CLEAN_SCRIPT` → skill 目录下的 `auto_clean.py`。
3. 处理完成后 agent 必须执行关键词扫描核验（见下节），命中归零才算完成。

> MiMo Desktop 中可用 `$env:MIMO_PYTHON` 运行脚本，自动使用内置 Python 运行时。

## 处理结果核验

无论手动还是 skill 调用，处理完都建议对结果文件再扫一遍关键词，**命中为 0 才算完成**：

```
AIGC · tc260 · ContentProducer
python-docx · python-pptx · openpyxl · Steve Canny · ReportLab
```

图片另外可肉眼确认右下角是否还有「AI生成」角标。

## 补强工具

| 需求 | 方案 |
|------|------|
| 更强可见水印擦除（多平台角标） | [OpenNoMark](https://github.com/NanmiCoder/OpenNoMark) |
| ExifTool 全量剥元数据 | [xxd-strip-ai-meta](https://github.com/nevertoday/xxd-strip-ai-meta)，或直接安装 ExifTool（本工具会自动检测） |
| 纯 Python 看/删图元数据 | [img-meta](https://github.com/themostjomo/img-meta) |

## 合规提醒

- 本工具面向**本地整理、私用文件**的元数据清洁。
- **对外发布** AI 生成内容时，请遵守《人工智能生成合成内容标识办法》等规定，**不得以去标识方式冒充人工创作**。
- 不要对无权修改的第三方作品去水印。

## 更新日志

见 [CHANGELOG.md](CHANGELOG.md)。
