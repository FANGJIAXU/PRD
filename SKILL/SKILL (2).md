---
name: figma-to-frontend
description: 将 Figma UI/UX 设计稿转换为 HTML 文件。当用户提供 Figma 截图、导出图片、设计规范文档、JSON 设计 token 或 Figma Dev Mode 导出的 HTML 代码，并希望生成 HTML 文件时，必须使用此 Skill。触发关键词包括：Figma 转代码、设计稿还原、UI 切图、组件还原、从设计到代码、design to code、Figma export、Figma Dev Mode、截图转 HTML。即使用户只是上传了界面截图或粘贴了一段 Figma 导出的 HTML 并说"帮我写成 HTML"，也应触发此 Skill。
---

# Figma → HTML Skill

将 Figma 设计稿资料还原为可直接在浏览器打开的 HTML 文件。  
CSS 样式内嵌于 `<style>` 标签，SVG 图标内联写入，所有内容在**单一 HTML 文件**中完整呈现。

---

## 📁 目录结构

```
D:\PROJECT\AIProjectWeb\                    ← 项目根目录
│
├── figma-assets\                           ← 【输入】用户提供的所有 Figma 素材
│   ├── screenshots\                        设计稿截图（必须）
│   ├── tokens\                             Design Token 文件（强烈建议）
│   ├── specs\                              Inspect 标注截图 / 设计规范（建议）
│   ├── devmode-html\                       Figma Dev Mode 导出的 HTML（强烈建议）
│   └── assets\                             图标 / 字体等静态资源（可选）
│       ├── icons\
│       └── fonts\
│
└── output\                                 ← 【输出】生成的 HTML 文件
    └── {页面名}.html                        单个可直接打开的 HTML 文件
```

---

## 📥 输入目录详细说明

所有 Figma 导出素材统一放在：

```
D:\PROJECT\AIProjectWeb\figma-assets\
│
├── screenshots\                ← 设计稿截图（✅ 必须）
│   ├── page-home.png             每个页面单独截图
│   ├── page-login.png
│   └── component-button.png     重要组件的单独截图
│
├── tokens\                     ← Design Token（⭐ 强烈建议）
│   ├── tokens.json               Tokens Studio 插件导出的 JSON
│   └── variables.css             或 Figma Variables 导出的 CSS
│
├── specs\                      ← 标注规范（⭐ 强烈建议）
│   ├── inspect-home.png          Figma Inspect 面板截图（含间距/字号/颜色数值）
│   ├── inspect-login.png
│   └── design-spec.pdf           设计规范文档（如有）
│
├── devmode-html\               ← Figma Dev Mode HTML（⭐ 强烈建议）
│   ├── page-home.html            首页的 Dev Mode HTML
│   ├── page-login.html           登录页的 Dev Mode HTML
│   └── component-button.html     按钮组件的 Dev Mode HTML
│
└── assets\                     ← 静态资源（💡 可选）
    ├── icons\
    │   ├── icon-search.svg
    │   └── icon-user.svg
    └── fonts\
        └── CustomFont.woff2
```

### 各类素材说明

| 素材 | 存放目录 | 格式 | 必须程度 | Figma 导出方法 |
|------|---------|------|---------|--------------|
| 设计稿截图 | `figma-assets\screenshots\` | PNG / JPG | ✅ 必须 | Frame → Export → PNG 2x |
| Design Token | `figma-assets\tokens\` | JSON / CSS | ⭐ 强烈建议 | 插件 Tokens Studio → Export JSON |
| Inspect 标注截图 | `figma-assets\specs\` | PNG / JPG | ⭐ 强烈建议 | Inspect 面板截图（含数值） |
| Dev Mode HTML | `figma-assets\devmode-html\` | .html / 直接粘贴 | ⭐ 强烈建议 | Inspect → Code → HTML → 复制 |
| SVG 图标 | `figma-assets\assets\icons\` | .svg | 💡 可选 | 图层 → Export → SVG |
| 字体文件 | `figma-assets\assets\fonts\` | .woff2 | 💡 可选 | 字体供应商下载 |
| 技术说明 | — | 文字 | ✅ 必须 | 直接在对话中填写 |

**对话中需说明的信息：**
```
页面名称：（如：首页 / 登录页 / 贡献知识页）
字体：Open Sans / 思源黑体 / 系统默认
是否需要响应式：是 / 否（桌面端固定宽度）
需要还原的交互：hover 效果 / 无
```

---

## 🔄 执行流程

### Step 1 — 分析与确认
读取 `figma-assets\` 下的所有素材，输出分析报告，**等用户确认后再生成 HTML**：
- 识别到的页面结构与区块划分
- 从 Dev Mode HTML 提取的精确数值（尺寸、颜色、间距、字体）
- 从截图 / Token 文件提取的设计变量
- 不确定的设计细节（明确列出，不得猜测后直接生成）

---

### Step 2 — Dev Mode HTML 转换规则

Figma Dev Mode 生成的 HTML 是原始参考代码，需按以下规则转换后写入输出 HTML：

| Figma 原始写法 | 转换为 |
|--------------|--------|
| `position: absolute` + `left/top` | Flexbox / Grid 布局 |
| `style="..."` 全内联样式 | `<style>` 标签内的 CSS class |
| 魔法数字 `width: 1920px` | `max-width: 1920px; width: 100%` |
| 颜色 `#7DB368` 写死 | CSS 变量 `var(--color-primary)` |
| `font-size: 18px` 写死 | CSS 变量 `var(--font-size-base)` |
| `gap: 20px` 写死 | CSS 变量 `var(--spacing-lg)` |
| 全部 `<div>` 无语义 | `<nav>` `<header>` `<main>` `<section>` `<button>` 等 |

---

### Step 3 — 生成单一 HTML 文件

按以下结构生成完整 HTML，所有内容在一个文件内：

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>页面名称</title>
  <!-- 外部字体（Google Fonts 或 @font-face） -->
  <link href="https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;600;700&display=swap" rel="stylesheet">
  <style>
    /* ── Design Tokens ── */
    :root {
      --color-primary: #7DB368;
      --color-text-main: #111827;
      --color-text-muted: #6B7280;
      --color-bg-page: #F9F9F9;
      --color-bg-white: #FFFFFF;

      --font-family-main: 'Open Sans', sans-serif;
      --font-size-sm: 14px;
      --font-size-base: 18px;
      --font-size-lg: 20px;
      --font-weight-regular: 400;
      --font-weight-semi: 600;
      --font-weight-bold: 700;

      --spacing-xs: 4px;
      --spacing-sm: 8px;
      --spacing-md: 16px;
      --spacing-lg: 20px;
      --spacing-xl: 24px;

      --radius-sm: 4px;
      --radius-md: 8px;
      --radius-full: 9999px;
    }

    /* ── Reset ── */
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: var(--font-family-main); background: var(--color-bg-page); color: var(--color-text-main); }

    /* ── Layout ── */
    .layout { display: flex; min-height: 100vh; }
    .sidebar { ... }
    .main { ... }

    /* ── Components ── */
    .navbar { ... }
    .step-indicator { ... }
    .btn { ... }
    .btn:hover { ... }
    .form-input { ... }
    .tag { ... }
  </style>
</head>
<body>
  <!-- 页面结构（语义化 HTML） -->
  <div class="layout">
    <nav class="sidebar">...</nav>
    <main class="main">
      <header class="navbar">...</header>
      <section class="content">...</section>
    </main>
  </div>

  <!-- SVG 图标内联（来自 figma-assets\assets\icons\） -->
  <!-- <svg>...</svg> -->
</body>
</html>
```

---

## 📤 输出文件说明

**输出路径：** `D:\PROJECT\AIProjectWeb\output\{页面名}.html`

```
D:\PROJECT\AIProjectWeb\
└── output\
    ├── page-home.html           首页
    ├── page-login.html          登录页
    └── page-contribute.html     贡献知识页
```

| 项目 | 说明 |
|------|------|
| 文件数量 | 每个页面对应一个独立 HTML 文件 |
| CSS 位置 | 全部内嵌在 `<style>` 标签内，无外部 CSS 文件 |
| 图标处理 | SVG 内联写入 HTML，无需外部图片依赖 |
| 字体处理 | Google Fonts 用 `<link>` 引入；私有字体用 `@font-face` 内嵌 |
| 打开方式 | 双击 .html 文件即可在浏览器直接预览 |
| 图片资源 | 设计稿中的插图 / 照片生成占位块，注释标明图片路径 |

---

## 🗺️ 输入 / 输出路径速查表

### 输入

| 素材 | 存放路径 | 必须程度 |
|------|---------|---------|
| 设计稿截图 | `D:\PROJECT\AIProjectWeb\figma-assets\screenshots\` | ✅ 必须 |
| Dev Mode HTML | `D:\PROJECT\AIProjectWeb\figma-assets\devmode-html\` 或直接粘贴 | ⭐ 强烈建议 |
| Design Token JSON | `D:\PROJECT\AIProjectWeb\figma-assets\tokens\tokens.json` | ⭐ 强烈建议 |
| Inspect 标注截图 | `D:\PROJECT\AIProjectWeb\figma-assets\specs\` | ⭐ 建议 |
| SVG 图标 | `D:\PROJECT\AIProjectWeb\figma-assets\assets\icons\` | 💡 可选 |
| 字体文件 | `D:\PROJECT\AIProjectWeb\figma-assets\assets\fonts\` | 💡 可选 |
| 页面说明 | 直接在对话中提供 | ✅ 必须 |

### 输出

| 产物 | 输出路径 |
|------|---------|
| HTML 页面文件 | `D:\PROJECT\AIProjectWeb\output\{页面名}.html` |

---

## ✅ 还原质量标准

1. **布局结构** — 绝对定位全部转为 Flex / Grid，保持与截图视觉一致
2. **颜色体系** — 从 Dev Mode HTML 提取精确 hex 值，统一用 CSS 变量管理
3. **字体层级** — 从 Text Styles 提取 family / size / weight / line-height
4. **间距系统** — 从 Dev Mode HTML 的 gap / padding / margin 归纳为 CSS 变量
5. **细节样式** — 圆角、阴影、边框、透明度与设计稿一致
6. **交互状态** — hover、focus、active、disabled 用纯 CSS 实现
7. **语义化** — `<nav>` `<header>` `<main>` `<button>` 替换无意义 `<div>`
8. **可直接打开** — 双击 HTML 文件即可在浏览器完整预览，无需启动服务

---

## ❓ 边界情况处理

| 情况 | 处理方式 |
|------|---------|
| Dev Mode HTML 大量 `position: absolute` | 全部重构为 Flex / Grid，以截图为视觉参考 |
| 截图与 Dev Mode HTML 视觉不一致 | 以截图视觉为准，Dev Mode HTML 仅作数值参考 |
| 无 Token 文件，仅有截图 | 视觉提取颜色/间距，输出前标注需用户确认的数值 |
| 私有字体无法获取 | 用 `@font-face` 占位，注释标明字体文件路径 |
| 设计稿有动效 | 用 CSS transition / animation 实现基础动效 |
| 图片 / 插图资源缺失 | 生成带尺寸的占位色块，注释说明对应图片路径 |
| 多页面，不知优先级 | 询问用户，默认从核心页面（首页）开始 |
