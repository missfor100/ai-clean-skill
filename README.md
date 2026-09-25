# ai-clean

生成物自动去痕流水线：清理 AI 生成文件上的隐式标识、可见水印与 Office/PDF 工具指纹。

## 清理内容

| 类别 | 目标 | 处理方式 |
|------|------|----------|
| 图片隐式标识 | TC260 AIGC XMP / C2PA / EXIF / iTXt | 优先调用系统 `exiftool`；否则纯 Python 剥离（PNG 按块过滤，JPEG 重编码） |
| 图片可见水印 | 右下角「AI生成」类半透明角标 | 定位深色角标盒并用周边纹理覆盖填充（`--no-watermark` 可跳过） |
| Office 指纹 | `python-docx` / `python-pptx` / `openpyxl` 等 | 改写核心属性 + 重写包内 XML |
| PDF 指纹 | `ReportLab` 等生成器信息 | 用 pypdf 重写元数据 |

支持格式：`png` `jpg` `webp` `tiff` `docx` `xlsx` `pptx` `pdf`。

## 使用

```powershell
python auto_clean.py <文件或目录> [选项]

# 选项
#   --out DIR        输出到指定目录（默认原地处理）
#   --author 名字     写入 Office/PDF 的作者属性（默认「用户」）
#   --no-watermark    只清元数据，不动可见水印
#   --keep-backup     保留 .bak 备份
```

依赖：`Pillow`、`python-docx`、`openpyxl`、`python-pptx`、`pypdf`。

## 作为 MiMo skill 使用

把本仓库目录放入 MiMo Desktop 的 skills 目录（见「设置 → 插件」中的 skill 路径），生成图片/文档后 agent 会自动调用 `auto_clean.py` 并核验结果。skill 定义见 [SKILL.md](SKILL.md)，可用环境变量 `AI_CLEAN_SCRIPT` 覆盖脚本路径。

## 核验

处理完成后扫描结果文件，以下关键词命中数必须为 0：

`AIGC` · `tc260` · `ContentProducer` · `python-docx` · `python-pptx` · `openpyxl` · `Steve Canny` · `ReportLab`

脚本会打印 `[meta-stripped|exiftool|ooxml-cleaned|pdf-cleaned|watermark-covered]` 标签便于核对。

## 合规提醒

- 本工具面向**本地整理、私用文件**的元数据清洁。
- **对外发布** AI 生成内容时，请遵守《人工智能生成合成内容标识办法》等规定，不得以去标识方式冒充人工创作。
- 不要对无权修改的第三方作品去水印。
