# 图床 · Image / Video / Audio Host

一个零依赖、单文件的上传图床 Web 应用：把图片、视频、音频直接上传到 GitHub 仓库，并返回可长期使用的直链与 CDN 加速链接。

- 无需后端服务器：页面直接调用 GitHub API 完成上传
- 自动分类：图片 → `images/`，视频 → `videos/`，音频 → `audios/`
- 输出多种链接：GitHub Raw 直链、jsDelivr CDN 加速、GitHub Pages 链接，以及 Markdown / HTML 嵌入代码
- 支持拖拽批量上传、复制到剪贴板、本地历史记录

## 快速开始

### 1. 申请 GitHub Token（只需一次）

1. 打开 [GitHub Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens)
2. 点 **Generate new token (classic)**（或创建 fine-grained token）
3. 勾选 **`repo`** 权限（fine-grained token 则选择本仓库并授予 Contents: Read and write）
4. 设置过期时间，生成后复制 Token（形如 `ghp_xxxxxxxxxxxx`）

> 安全提示：Token 只保存在当前浏览器的 localStorage 中，不会写入代码仓库。请勿把 Token 提交到任何公开仓库。

### 2. 打开图床页面

- **方式 A（推荐，在线版）**：部署到 GitHub Pages 后访问
  `https://<你的用户名>.github.io/imgbed/`（见下方「部署到 GitHub Pages」）
- **方式 B（离线版）**：直接下载 `index.html`，用浏览器双击打开即可使用

### 3. 上传

1. 在「配置」面板填入 Token、Owner（GitHub 用户名）、仓库名（默认 `imgbed`）、分支（默认 `main`）
2. 点击「保存配置」，可点「校验 Token」确认可用
3. 拖拽或点击选择图片 / 视频 / 音频文件，上传完成后自动列出所有链接，点「复制」即可

## 链接格式

| 类型 | 格式 |
| --- | --- |
| GitHub Raw 直链 | `https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<path>` |
| jsDelivr CDN | `https://cdn.jsdelivr.net/gh/<owner>/<repo>@<branch>/<path>` |
| GitHub Pages | `https://<owner>.github.io/<repo>/<path>` |
| Markdown | `![文件名](<url>)` |
| 视频嵌入 | `<video src="<url>" controls preload="metadata"></video>` |
| 音频嵌入 | `<audio src="<url>" controls></audio>` |

jsDelivr 是 CDN 加速，国内访问快；Raw 直链适合下载。图片建议用任一链接均可；视频/音频建议用 jsDelivr 或 Pages 链接（Raw 链接可能触发下载而非播放）。

## 部署到 GitHub Pages

仓库已包含 `.github/workflows/pages.yml`。首次需启用 Pages：

1. 打开仓库 **Settings → Pages**
2. 在 **Build and deployment** 的 Source 中选择 **GitHub Actions**
3. 保存后工作流会自动运行，约 1 分钟部署完成

之后每次推送代码都会自动重新部署。

## 注意事项

- **单文件上限 100 MB**：GitHub API 的硬限制；超过 50 MB 会提示。超大视频建议先压缩或分片。
- 仓库必须为 **公开**，否则 Raw / CDN 链接无法匿名访问（本仓库已设为公开）。
- 上传会创建 commit，会占用仓库容量；不建议上传涉密或侵权内容。
- jsDelivr 首次访问新文件可能需要几秒到几分钟的缓存时间；若想强制刷新，可访问 `https://purge.jsdelivr.net/gh/<owner>/<repo>@<branch>/<path>`。

## 技术说明

- 纯原生 HTML / CSS / JavaScript，无任何外部依赖（不引第三方库、不调第三方接口）
- 上传走 GitHub Contents API（`PUT /repos/{owner}/{repo}/contents/{path}`，base64 编码）
- 文件名自动生成：`时间戳_随机数.扩展名`（可选保留原始文件名），避免重名冲突与非 ASCII 路径的 CDN 兼容问题

## 目录结构

```
imgbed/
├── index.html                # 图床应用（单文件，全部逻辑内联）
├── README.md                 # 本文档
└── .github/workflows/pages.yml  # GitHub Pages 自动部署
```

---

License: MIT
