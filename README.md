# Wang Jing — Personal Portfolio

求职用个人主页。单文件静态站，零依赖、零构建步骤 —— 双击 `index.html` 即可打开，也可以直接丢到任意静态托管上。

**定位**：求职作品集（AI/LLM · Fintech · FDE · 技术架构方向）
**语言**：英文 `index.html` + 中文 `index.zh.html`（两版内容保持一致，互加语言切换链接）
**风格**：工程极简 —— 黑白灰、等宽字标签、细分割线、大留白

---

## 文件结构

```
My Portfolio/
├── index.html              ← 英文主页（HTML + CSS + JS 内联）
├── index.zh.html           ← 中文主页（镜像英文版，nav 含「EN」回链）
├── assets/
│   ├── WangJing_CV.pdf     ← 英文主页「CV」按钮下载的文件
│   ├── cv-print.html       ← 英文 CV 的打印源文件（A4 排版，用于重新生成 PDF）
│   ├── WangJing_CV_zh.pdf  ← 中文主页「简历」按钮下载的文件
│   └── cv-print.zh.html    ← 中文 CV 的打印源文件（A4 排版，用于重新生成 PDF）
└── README.md
```

> **中英文同步原则**：改内容时 `index.html` 与 `index.zh.html` 必须同步改，否则两版会打架；同理 `cv-print.html` 与 `cv-print.zh.html`。英文页加「中文」链接、中文页加「EN」链接已实现，无需重复维护。

---

## 本地预览

直接双击 `index.html`，或起一个本地服务：

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
python3 -m http.server 8899
# 打开 http://127.0.0.1:8899
```

---

## 部署前：清理注入属性

如果 `index.html` / `index.zh.html` 曾在编辑器 / 预览面板里打开过，文件会被注入 `data-page-node-id="..."` 这类内部属性（一次可达数百个）。部署前跑一遍清理：

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
python3 -c "
import re, pathlib
for f in ['index.html', 'index.zh.html']:
    p = pathlib.Path(f); s = p.read_text()
    n = len(re.findall(r' data-page-node-id=\"[^\"]*\"', s))
    p.write_text(re.sub(r' data-page-node-id=\"[^\"]*\"', '', s))
    print(f, 'removed:', n)"
```

只删属性，不影响样式与功能。

---

## 当前部署（已上线）

| 项 | 值 |
|---|---|
| 网址 | **https://iosonolatte.github.io/** |
| 仓库 | https://github.com/iosonolatte/iosonolatte.github.io （public） |
| 托管 | GitHub Pages · source = `main` 分支根目录 · push 后自动重新部署 |

`iosonolatte.github.io` 是 GitHub 的**账号级主页仓库**，一个账号只能有一个。推送后 Pages 自动启用，无需手动配置。

### 日常更新流程

改完 `index.html` / `index.zh.html` 或 `assets/cv-print.html`（及中文版 `cv-print.zh.html`）之后：

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"

# 1. 清理预览注入的内部属性（只要文件在 WorkBuddy 里开过预览就必须做）
python3 -c "
import re, pathlib
for f in ['index.html', 'index.zh.html']:
    p = pathlib.Path(f); s = p.read_text()
    n = len(re.findall(r' data-page-node-id=\"[^\"]*\"', s))
    p.write_text(re.sub(r' data-page-node-id=\"[^\"]*\"', '', s))
    print(f, 'removed:', n)"

# 2. 提交并推送 —— Pages 会自动重新构建
git add -A
git commit -m "Update portfolio"
git push
```

约 1 分钟后刷新 https://iosonolatte.github.io/ 即可看到。构建状态：

```bash
gh api repos/iosonolatte/iosonolatte.github.io/pages --jq .status   # building / built
```

---

## 备用部署方式

### Cloudflare Pages

```bash
npx wrangler pages deploy . --project-name wangjing-portfolio
```

### Vercel

```bash
npx vercel --prod
```

> 自定义域名：在托管平台绑定后，记得把 `index.html` 里的 `og:url` 补上（目前未设置，不影响使用）。

---

## 改内容速查

| 想改什么 | 在 `index.html` 里找 |
|---|---|
| 名字 / 中文名 | `<h1>` 里的 `Wang Jing` 与 `<span class="cn">汪 晶</span>` |
| 顶部状态胶囊（绿点那行） | `<div class="status">` |
| 一句话简介 | `<p class="lede">` |
| 求职方向标签 | `<div class="targets">` 里的 `<span class="chip">` |
| About 正文 | `<section id="about">` |
| 工作经历 | `<section id="experience">`，每段一个 `<article class="job">` |
| 项目卡片 | `<section id="projects">`，每张一个 `<article class="card">`（`card-wide` = 整行宽） |
| 技能表 | `<section id="skills">`，每行一个 `<div class="skill-row">` |
| 邮箱 / GitHub | 搜索 `iosonolatte`，全局替换 |
| 配色 | 文件顶部 `:root` 里的 CSS 变量（`--ink`、`--accent`、`--bg` …） |

---

## 重新生成 CV PDF

改完 `assets/cv-print.html`（英文）或 `assets/cv-print.zh.html`（中文）后：

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
# 英文
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --no-pdf-header-footer --no-sandbox \
  --print-to-pdf="$PWD/assets/WangJing_CV.pdf" \
  "file://$PWD/assets/cv-print.html"
# 中文
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --no-pdf-header-footer --no-sandbox \
  --print-to-pdf="$PWD/assets/WangJing_CV_zh.pdf" \
  "file://$PWD/assets/cv-print.zh.html"
```

校验页数与内容完整性：

```bash
/Users/jing/.workbuddy/binaries/python/envs/default/bin/python -c "
from pypdf import PdfReader
for f in ['assets/WangJing_CV.pdf', 'assets/WangJing_CV_zh.pdf']:
    r = PdfReader(f)
    print(f, 'pages:', len(r.pages),
          'chars:', len(''.join(p.extract_text() or '' for p in r.pages)))
"
```

---

## 已实现的功能

- **响应式**：桌面双栏项目网格 → 移动端单栏；导航在窄屏折叠为品牌 + 主题 + CV
- **深浅色主题**：默认跟随系统，右上角可手动切换，选择持久化到 `localStorage`
- **滚动动效**：`IntersectionObserver` 逐段淡入 + 导航当前章节高亮
- **渐进增强**：动画只在 JS 运行时启用；JS 不可用时内容照常完整显示
- **打印样式**：`Cmd+P` 可直接输出干净的黑白版本，无导航无按钮
- **无障碍**：跳转链接、单一 `h1`、`aria-label`、`:focus-visible` 焦点环、`prefers-reduced-motion` 支持
