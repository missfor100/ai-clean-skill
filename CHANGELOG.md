# 更新日志

本项目的版本变更记录。格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [未发布]

### 新增

- **依赖清单（#1）**：新增 `requirements.txt`（运行依赖）与 `requirements-dev.txt`（测试依赖），README 安装段改为 `pip install -r requirements.txt`，声明 Python 3.10+。
- **测试套件与 CI（#2）**：`tests/` 覆盖 PNG/JPEG 元数据剥离、水印定位与覆盖、OOXML/PDF 指纹清理、CLI 副本行为共 14 项测试；新增 GitHub Actions `test` workflow（pytest），与 gitleaks 并列。
- **水印未检出提示（#4）**：未检出角标时输出 `no-watermark-detected` 标签，不再静默跳过；README 增加水印支持矩阵，SKILL 核验说明同步更新。

### 修复

- **OOXML 清洗范围限定（#3）**：`REPLACEMENTS` 与 AIGC 标记清理现在只作用于 `docProps/*.xml` 属性文件，不再改写 `word/document.xml`、`ppt/slides/`、`xl/worksheets/` 等正文——修复正文含「AIGC」「Steve Canny」等词被误改写的问题。
- **JPEG 重编码保留 ICC（#5）**：`strip_jpeg_meta` 与 `save_image` 保留 ICC 色彩配置；README 明确标注内置 JPEG 路径为有损重编码（质量 95）并给出画质敏感场景的建议。

## [1.0.0] - 2026-09-25

首次公开发布。

### 新增

- **图片隐式标识剥离**：PNG 按块丢弃 `tEXt`/`zTXt`/`iTXt`/`eXIf`；JPEG/WebP/TIFF 重编码丢弃 XMP/EXIF/C2PA；系统装有 ExifTool 时自动优先调用 `-all=` 全量剥离。
- **图片可见水印覆盖**：自动定位右下角深色低饱和角标盒，采样周边纹理高斯模糊覆盖，边缘羽化消接缝；不裁切画幅；`--no-watermark` 可跳过。
- **Office 指纹清理**：重写 docx/xlsx/pptx 核心属性（作者、备注等），并遍历包内 XML 按替换表清洗 `python-docx`、`python-pptx`、`openpyxl`、`Steve Canny`、`ReportLab` 等指纹及国标 AIGC 标记。
- **PDF 指纹清理**：用 pypdf 重建文档并重写元数据，`Creator`/`Producer` 置为中性值。
- **命令行**：支持文件/目录批量、`--out` 指定输出目录、`--author` 指定作者、`--keep-backup` 保留备份；处理结果打印 `[meta-stripped|exiftool|ooxml-cleaned|pdf-cleaned|watermark-covered]` 标签。
- **MiMo skill 定义**（`SKILL.md` + `locales/`）：生成图片/文档后自动触发去痕，强制关键词扫描核验，命中归零才算完成。
- **合规说明**：README 与 skill 内均写明本地整理用途与《人工智能生成合成内容标识办法》发布义务。

[未发布]: https://github.com/missfor100/ai-clean-skill/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/missfor100/ai-clean-skill/releases/tag/v1.0.0
