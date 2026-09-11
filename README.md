# Wang Jing — Personal Portfolio

求职用个人主页。单文件静态站，零依赖、零构建步骤 —— 双击 `index.html` 即可打开，也可以直接丢到任意静态托管上。

**定位**：求职作品集（AI/LLM · Fintech · FDE · 技术架构方向）
**语言**：英文（与 `WangJing_CV` 英文版内容保持一致）
**风格**：工程极简 —— 黑白灰、等宽字标签、细分割线、大留白

---

## 文件结构

```
My Portfolio/
├── index.html              ← 主页全部内容（HTML + CSS + JS 内联）
├── assets/
│   ├── WangJing_CV.pdf     ← 顶部「CV」按钮下载的文件
│   └── cv-print.html       ← CV 的打印源文件（A4 排版，用于重新生成 PDF）
└── README.md
```

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

如果 `index.html` 曾在编辑器 / 预览面板里打开过，文件会被注入 `data-page-node-id="..."` 这类内部属性（一次可达数百个）。部署前跑一遍清理：

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
python3 -c "
import re, pathlib
p = pathlib.Path('index.html'); s = p.read_text()
print('removed:', len(re.findall(r' data-page-node-id=\"[^\"]*\"', s)))
p.write_text(re.sub(r' data-page-node-id=\"[^\"]*\"', '', s))"
```

只删属性，不影响样式与功能。

---

## 部署（三选一）

### Cloudflare Pages（推荐，你已经在用 Workers）

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
npx wrangler pages deploy . --project-name wangjing-portfolio
```

### GitHub Pages

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
git init && git add -A && git commit -m "Add personal portfolio"
git branch -M main
git remote add origin git@github.com:iosonolatte/portfolio.git
git push -u origin main
# 然后在仓库 Settings → Pages 选择 main 分支根目录
```

### Vercel

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
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

改完 `assets/cv-print.html` 后：

```bash
cd "/Users/jing/WorkBuddy/My Portfolio"
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --no-pdf-header-footer --no-sandbox \
  --print-to-pdf="$PWD/assets/WangJing_CV.pdf" \
  "file://$PWD/assets/cv-print.html"
```

校验页数与内容完整性：

```bash
/Users/jing/.workbuddy/binaries/python/envs/default/bin/python -c "
from pypdf import PdfReader
r = PdfReader('assets/WangJing_CV.pdf')
print('pages:', len(r.pages))
print('chars:', len(''.join(p.extract_text() or '' for p in r.pages)))
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
