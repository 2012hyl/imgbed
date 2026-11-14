# 图床 · Image / Video / Audio Host

一个零依赖、单文件的上传图床 Web 应用：把文件直接上传到 GitHub 仓库，并返回可长期使用的直链与 CDN 加速链接。

- 无需后端服务器：页面直接调用 GitHub API 完成上传
- 自动分类：图片 → `images/`，视频 → `videos/`，音频 → `audios/`
- 输出多种链接：GitHub Pages、jsDelivr CDN 加速、GitHub Raw 直链，以及 Markdown / HTML 嵌入代码
- 支持拖拽批量上传（并发 3，自动重试限流）、复制到剪贴板、本地历史记录

## 快速开始

### 1. 申请 GitHub Token（只需一次）

建议直接用 fine-grained token，只给这一个仓库：

1. 打开 [GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/tokens)
2. `Repository access` 选 **Only select repositories** → 选中本仓库
3. `Permissions → Repository permissions → Contents` 设为 **Read and write**
4. 设置过期时间，生成后复制 Token

> 不要用 classic token 的 `repo` 全权限：那等于把账号下所有仓库都交出去。

> 安全提示：Token 只保存在当前浏览器的 localStorage 中，不会写入代码仓库。但 `*.github.io` 下所有仓库共用同一个 origin，别的 Pages 页面里的脚本也能读到这个 Token——不要在 github.io 上跑来源不明的 HTML。

### 2. 打开图床页面

- **方式 A（推荐，在线版）**：部署到 GitHub Pages 后访问 `https://<你的用户名>.github.io/imgbed/`
- **方式 B（离线版）**：直接下载 `index.html`，用浏览器双击打开即可使用

### 3. 上传

1. 在「配置」面板填入 Token、Owner、仓库名、分支，**CDN 线路选 `testingcf.jsdelivr.net`**
2. 点击「保存配置」，可点「校验 Token」确认可用
3. 拖拽或点击选择文件，上传完成后自动列出所有链接

## 链接格式

| 类型 | 格式 |
| --- | --- |
| GitHub Pages | `https://<owner>.github.io/<repo>/<path>` |
| jsDelivr CDN | `https://<cdnHost>/gh/<owner>/<repo>@<branch>/<path>` |
| GitHub Raw | `https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<path>` |
| Markdown | `![文件名](<url>)` |
| 视频嵌入 | `<video src="<url>" controls preload="metadata"></video>` |
| 音频嵌入 | `<audio src="<url>" controls></audio>` |

**链接优先级**：Pages > CDN > Raw。生成嵌入代码时会自动选最可能打得开的那条。

## 各线路在国内的实际表现（2026-11 实测）

| 线路 | 结果 |
| --- | --- |
| `2012hyl.github.io`（Pages） | ✅ 首包 ~0.5s，最快 |
| `testingcf.jsdelivr.net` | ✅ 可用，首包 ~1.8s |
| `gcore.jsdelivr.net` | ✅ 可用，首包 ~2.2s |
| `cdn.jsdelivr.net` | ❌ TLS 被重置（ECONNRESET） |
| `fastly.jsdelivr.net` | ❌ TLS 被重置 |
| `raw.githubusercontent.com` | ❌ TLS 被重置 |

所以默认线路选了 `testingcf`，并且默认开启 Pages。Raw 直链保留但默认标注为「国内多数网络打不开」。

## 不要用它托管视频和音频

实测吞吐（同一网络、同类文件）：

- 图片（几十 KB ~ 1 MB）：够用
- 视频 1.84 MB：约 13 秒拉完（0.14 MB/s）
- 音频 11.58 MB：约 65–140 秒（0.08–0.18 MB/s）

这个速度无法在线播放。视频/音频请用对象存储、GitHub Releases 或视频站，本工具只建议放图片。

## 大小限制

- GitHub 单文件硬限制 100 MB，超过 50 MB 开始警告
- 但浏览器要把文件读成 base64 再塞进 JSON，内存峰值约为文件的 1.3–1.5 倍，手机端 20 MB 以上就可能崩
- **实际建议单文件 ≤ 20 MB**；每个文件会产生一个 commit，仓库体积只增不减（删文件也不会释放历史）

## 部署到 GitHub Pages

仓库已包含 `.github/workflows/pages.yml`，在 **Settings → Pages → Source** 选 **GitHub Actions** 即可。

注意：每次上传都是一个 push，会触发一次 Pages 重建。传得多、传得勤时 Actions 次数会涨得很快。

## 技术说明

- 纯原生 HTML / CSS / JavaScript，无任何外部依赖
- 上传走 GitHub Contents API（`PUT /repos/{owner}/{repo}/contents/{path}`，base64 编码）
- 同时最多 3 个上传任务；遇到 403 / 429 / 5xx 按 `Retry-After` 或指数退避重试，最多 3 次
- GitHub 的 PUT 拿不到上传进度，所以界面只显示已经过的秒数（不再画假进度条）
- 文件名自动生成：`时间戳_随机数.扩展名`（可选保留原始文件名）
- 配置存在 `localStorage`（键名 `imgbed_cfg_v2`）；从旧版升级会自动保留 Token / 仓库 / 分支，链接相关选项（CDN 线路、Pages）统一升级为新默认

## 目录结构

```
imgbed/
├── index.html                   # 图床应用（单文件，全部逻辑内联）
├── README.md                    # 本文档
└── .github/workflows/pages.yml  # GitHub Pages 自动部署
```

---

License: MIT
