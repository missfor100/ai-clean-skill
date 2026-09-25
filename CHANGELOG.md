# 更新日志

本项目的版本变更记录。格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

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

[1.0.0]: https://github.com/missfor100/ai-clean-skill/releases/tag/v1.0.0
