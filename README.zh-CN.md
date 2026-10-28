# Gotenberg 中文说明

[English](README.md) | 简体中文

Gotenberg 是一个基于 Docker 的文档转换 API。应用通过 HTTP 上传文件或提交网页地址，获取 PDF 等转换结果，无需自行编写 Chromium 或 LibreOffice 的调用流程。

本仓库是 [Gotenberg 上游项目](https://github.com/gotenberg/gotenberg)的 fork，可作为 Cocolian 云文档中台的文档转换服务。Cocolian 致力于构建面向中文文档的国产化 LibreOffice 方案和云文档中台，持续改善中文文档与 Microsoft Office Word 的渲染一致性。

渲染一致性是持续优化和验证的目标。此仓库的中文说明不代表上游镜像已包含 Cocolian 的渲染修正，也不代表转换结果已经与 Word 完全一致。实际效果取决于所部署的引擎版本、字体、配置及文档特性。

## 主要功能

| 场景        | 能力                                                          |
| ----------- | ------------------------------------------------------------- |
| 网页与模板  | 使用 Chromium 将 HTML、网页地址和 Markdown 转为 PDF           |
| Office 文档 | 使用 LibreOffice 将文字文档、电子表格、演示文稿等格式转为 PDF |
| PDF 处理    | 合并、拆分、旋转、表单扁平化，以及水印、印章和加密            |
| 文档归档    | 提供 PDF/A、PDF/UA 转换选项，具体参数以接口文档为准           |
| 其他操作    | 网页截图，读取或写入 PDF 元数据及书签                         |

Gotenberg 提供 API，没有在线文档编辑界面。编辑、权限管理、文件存储及业务流程由接入方或文档中台提供。

## 快速开始

需要 Docker 和 curl。在终端启动上游 Gotenberg 8 镜像：

```bash
docker run --rm --name gotenberg \
  -p 127.0.0.1:3000:3000 \
  gotenberg/gotenberg:8
```

该示例使用上游发布的镜像，只将端口映射到本机。服务启动后，在另一个终端检查健康状态：

```bash
curl --fail http://localhost:3000/health
```

### Office 文档转 PDF

将命令中的 `example.docx` 替换为待转换文档的路径：

```bash
curl --fail-with-body \
  --request POST http://localhost:3000/forms/libreoffice/convert \
  --form 'files=@example.docx' \
  --output result.pdf
```

接口通过 `multipart/form-data` 接收文件；成功时返回转换后的 PDF。其他文档格式及转换参数见[上游接口文档](https://gotenberg.dev/docs/convert-with-libreoffice/convert-to-pdf)。

### 网页转 PDF

```bash
curl --fail-with-body \
  --request POST http://localhost:3000/forms/chromium/convert/url \
  --form 'url=https://example.com' \
  --output webpage.pdf
```

### HTML 文件转 PDF

主 HTML 文件须命名为 `index.html`：

```bash
curl --fail-with-body \
  --request POST http://localhost:3000/forms/chromium/convert/html \
  --form 'files=@index.html' \
  --output page.pdf
```

HTML 引用的本地样式、图片等资源也应按接口要求随请求上传。完整参数和返回行为请查阅[官方文档](https://gotenberg.dev/docs/getting-started/introduction)。

## 中文字体与 Word 渲染一致性

中文文档转换需要在转换容器中安装匹配的字体。仅在宿主机安装字体，不会自动让容器内的 LibreOffice 使用它们。

- 按字体授权范围，将需要的字体安装到转换镜像或挂载到容器内的字体目录，并更新字体缓存。
- 核对文档使用的字体名称与实际字体，例如 `仿宋_GB2312`、`楷体_GB2312`。名称相近的替代字体可能有不同的字形和排版度量。
- 固定转换镜像、LibreOffice 版本和字体版本，避免同一文档在不同环境下出现分页变化。
- 用代表性文档对照 Word 导出的 PDF，检查页数、换行、表格、段落间距、图片位置及页眉页脚。

字体匹配是渲染一致性的基础；复杂排版仍可能需要修正底层引擎。转换服务的 API 可用，不等于版式已经通过一致性验收。

## 接入云文档中台

可按以下流程接入业务系统：

1. 业务系统接收文档，校验访问权限并保存原文件。
2. 服务端调用 Gotenberg，提交待转换文件及转换参数。
3. 检查 HTTP 状态和转换结果，将生成的 PDF 保存到业务存储。
4. 将 PDF 用于预览、下载、打印或归档，并记录源文件版本和转换环境。

中文排版和渲染引擎相关工作可参考 [Cocolian Office 公开源码](https://github.com/cocolian/online.mirror)。接入定制引擎时，需要单独构建、配置并验证对应镜像。

## 部署与开发

生产环境应固定镜像版本或摘要，根据文档大小设置 CPU、内存、超时及并发限制。对外服务时配置身份验证和网络访问策略，保护上传的文档及转换结果。

从本仓库构建和开发前，阅读 [AGENTS.md](AGENTS.md)，并使用项目 Makefile：

```bash
git clone https://github.com/cocolian/gotenberg.git
cd gotenberg
make build
make run
```

构建定义见 [build/Dockerfile](build/Dockerfile)，开发与验证命令见 [Makefile](Makefile)。构建所需的工具版本以仓库配置为准。

## 开源、反馈与支持

Gotenberg 采用 [MIT 许可证](LICENSE)，可以免费使用、修改和分发，也可用于商业项目；使用时须遵守许可证并保留相应版权及许可声明。镜像中的第三方软件和自行安装的字体分别遵循其各自许可证。

欢迎接入 Cocolian 文档中台，提交改进建议和渲染问题。欢迎捐款支持，可通过 [Cocolian 公开项目的 GitHub Issue](https://github.com/cocolian/online.mirror/issues) 联系项目维护者。报告问题时请提供脱敏或可公开的复现材料。

上游通用问题可通过 [Gotenberg Issues](https://github.com/gotenberg/gotenberg/issues) 或 [Discussions](https://github.com/gotenberg/gotenberg/discussions) 反馈；支持上游作者可使用其 [GitHub Sponsors](https://github.com/sponsors/gulien) 页面。
