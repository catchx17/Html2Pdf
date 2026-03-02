# Html2Pdf

基于 Node.js、Express 与 Puppeteer 的轻量级服务，将网页转换为：
- 全页截图（PNG）
- 打印友好的 PDF

默认监听端口 4000，通过查询参数接受目标网址。

## 功能概述
- 提供两个 HTTP 接口：`/image` 与 `/pdf`
- 自动去除懒加载属性并等待图片资源加载完成
- 忽略页面 CSP，提升资源加载成功率
- 采用无头 Chromium，启动参数包含 `--no-sandbox`

## 快速开始

### 环境要求
- Node.js 18+
- npm（或兼容的包管理器）

### 安装与启动

```bash
npm install
npm start
```

启动后输出：
```
Listening on PORT: 4000
```

### 目录结构
- 应用入口：[index.js](file:///c:/Users/masterke/Downloads/Html2Pdf/index.js)
- 依赖声明：[package.json](file:///c:/Users/masterke/Downloads/Html2Pdf/package.json)
- 容器配置：[Dockerfile](file:///c:/Users/masterke/Downloads/Html2Pdf/Dockerfile)
- 编排示例：[docker-compose.yml](file:///c:/Users/masterke/Downloads/Html2Pdf/docker-compose.yml)
- Vercel 配置：[vercel.json](file:///c:/Users/masterke/Downloads/Html2Pdf/vercel.json)

## API 使用

所有接口均通过查询参数 `website` 指定目标网页 URL。

- 获取全页截图（PNG）
  - GET `/image?website=<URL>`
  - 响应头：`Content-Type: image/png`

- 导出 PDF
  - GET `/pdf?website=<URL>`
  - 响应头：`Content-Type: application/pdf`
  - 默认参数：A4、`printBackground: true`、`scale: 0.7`、页面宽度 1080px

示例（curl）：

```bash
# 导出截图
curl -L "http://localhost:4000/image?website=https://example.com" --output page.png

# 导出 PDF
curl -L "http://localhost:4000/pdf?website=https://example.com" --output page.pdf
```

示例（Node.js）：

```js
import fetch from "node-fetch";

const url = "http://localhost:4000/pdf?website=https://example.com";
const res = await fetch(url);
const buf = await res.arrayBuffer();
await fs.promises.writeFile("page.pdf", Buffer.from(buf));
```

## Docker

使用 Docker 运行（推荐在服务器环境中）：

```bash
# 构建
docker build -t html2pdf:latest .

# 运行
docker run --rm -p 4000:4000 html2pdf:latest
```

使用 Docker Compose：

```bash
docker compose up -d
```

说明：
- 当前服务容器已安装 Puppeteer 所需的系统依赖
- `docker-compose.yml` 还示例性包含 `browserless/chrome`，本服务默认不依赖该容器

## 部署到 Vercel

仓库包含基本的 [vercel.json](file:///c:/Users/masterke/Downloads/Html2Pdf/vercel.json) 配置，适用于将入口 `index.js` 作为 Serverless 函数部署。由于 Puppeteer/Chromium 在无服务器平台上对系统依赖与运行环境要求较高，若遇到启动失败，建议优先使用 Docker 部署。

## 配置说明

主要参数位于 [index.js](file:///c:/Users/masterke/Downloads/Html2Pdf/index.js)：
- 监听地址：`0.0.0.0:4000`
- 浏览器启动：`headless: 'chrome'`，并附加 `--no-sandbox --disable-setuid-sandbox`
- 页面设置：
  - 统一视口宽度为 1080 像素
  - 移除 `img` 与 `iframe` 的 `loading="lazy"`
  - 显式等待所有图片加载完成后再截图/打印

## 常见问题

- 页面资源受 CSP 限制导致加载失败  
  服务已启用 `page.setBypassCSP(true)`，通常可缓解。

- 截图/打印不完整或图片缺失  
  已移除懒加载并等待图片加载；如仍异常，尝试提高等待时间或调整目标站点。

- 生产环境权限受限  
  容器使用 `--no-sandbox` 参数以提升兼容性；在受限主机上建议通过 Docker 运行。

## 许可证

MIT
