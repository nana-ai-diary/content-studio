# 内容工坊 · 独立网页

无需 ChatGPT 登录，管理小红书、抖音和 X 的选题、草稿、图文分页、视频分镜、审核排期、人工发布登记与效果记录。

## 文件

- `index.html`：已打包全部 JavaScript 和 CSS 的完整工作台，无需在线加载第三方资源。
- `guide.html`：无需运行 JavaScript 即可阅读的公开功能说明，适合分享给其他 AI 对话。
- `source-code.zip`：完整 React / TypeScript / Vite 开发源码及原始构建产物。
- `.nojekyll`：直接提供静态文件。

## 访问和数据

无需注册，数据仅保存在当前浏览器，清理站点数据会丢失；请在业务与设置中导出 JSON 备份。本版本不连接原网站的云端数据。

文案、图文发布包和无配音 WebM 视频可在浏览器中导出。AI 使用复制提示词与导入 JSON 的方式；在线 AI、热点自动获取、封面上传、平台自动发布、知识库暂未接入。

## GitHub Pages

Settings → Pages → Deploy from a branch → main → /(root)。

## 继续开发

解压源码包，使用 Node.js 22.13 或更新版本，执行 npm install、npm run check、npm run build。可使用生成的 docs 目录发布，也可将构建 JS/CSS 内联到 index.html 后更新根目录。
