# 复刻 Ionic Framework 文档站样式方案

## 概要

将本项目（cancloude，Docusaurus 3.10.2）的排版与配色复刻为 [ionicframework.com/docs](https://ionicframework.com/docs/intro/cdn) 的风格。

**关键发现**：Ionic 官方文档本身就是用 Docusaurus + Infima 构建的（实测其页面包含 `--ifm-*` 变量与 `docItemContainer_*` 类名）。因此复刻无需任何依赖或 swizzle，**只需重写一个文件**：`cancloude/src/css/custom.css`（Infima 变量覆盖 + 针对 `.markdown` 的排版 CSS）。

以下所有数值均为对 Ionic 官网实测（browser computed style）所得，非估算：

| 项目 | Ionic Light | Ionic Dark |
|---|---|---|
| 页面背景 | `#ffffff` | `#03060b` |
| H1 | 48px / 700 / `#03060b`，margin-bottom 25px | 白色 |
| H2 | 32px / 700 / `#03060b`，margin 55px 0 12px，无下边框 | 白色 |
| H3 | 24px / 700 / `#03060b`，margin 16px 0 12px | 白色 |
| 正文 p | 20px / line-height 2.0 / `#35404e` / mb 20px | 20px / 2.0 / `#ced6e0` |
| 主色（链接/按钮） | `#3578e5`（完整 Infima 蓝色阶） | 同蓝色（暗色不变） |
| 代码块 pre | 背景 `#f6f8fc`，圆角 8px，padding 16px | 背景 `#161d25` |
| 行内代码 | 背景 `#f6f8fc`，padding 4px，圆角 6.4px，95% | 同背景 |
| 等宽字体 | `"SFMono-Regular","Roboto Mono",Consolas,"Liberation Mono",Menlo,Courier,monospace` | 同 |
| 正文字体 | `-apple-system,BlinkMacSystemFont,Inter,Helvetica,Arial,sans-serif`（纯系统字体，**无 Webfont**） | 同 |
| 内容区宽度 | 672px（`docItemContainer` 的 max-width） | 同 |
| TOC 右侧目录 | 12.8px / `#445b78` | `#eef1f3` |
| Navbar | 高 60px（默认值一致）、白底 | `#03060b` |

## 现状分析

- 项目结构：`classic` preset + 双文档插件（`docs` / `docs2`，docs2 为中文内容）+ blog；主题入口 `theme.customCss: './src/css/custom.css'`。
- 现有 [custom.css](file:///Users/xinxiucan/blog/blog-design-docusaurus/cancloude/src/css/custom.css) 仅有脚手架默认内容：绿色主色（`#2e8555` 系）、`--ifm-code-font-size: 95%`、代码行高亮背景。
- Ionic 与默认 Docusaurus 的核心差异只有 4 点：① 大号文章排版（h1 48 / h2 32 / h3 24 / 正文 20px、行高 2.0）；② 蓝→已是默认色阶但本项目被改成了绿色，需改回；③ 内容列宽 672px；④ 暗色模式的深蓝黑背景（`#03060b`）。侧边栏、navbar 结构与默认值基本一致，无需改动。

## 修改方案

**唯一改动文件**：`cancloude/src/css/custom.css` —— 整体重写为以下内容（决策完整，可直接落地）：

```css
/**
 * 复刻 ionicframework.com/docs 的排版与配色
 * 所有数值来自对 Ionic 官网 light/dark 两模式的实测 computed style
 */

:root {
  /* ===== 字体：Ionic 使用系统字体栈、不加载 Webfont；追加中文字体保证中文渲染 ===== */
  --ifm-font-family-base: -apple-system, BlinkMacSystemFont, Inter, "Segoe UI",
    Roboto, "Helvetica Neue", Helvetica, Arial, "PingFang SC",
    "Hiragino Sans GB", "Microsoft YaHei", "Noto Sans SC", sans-serif,
    "Apple Color Emoji", "Segoe UI Emoji";
  --ifm-font-family-monospace: "SF Mono", SFMono-Regular, "Roboto Mono", Consolas,
    "Liberation Mono", Menlo, Courier, monospace;

  /* ===== 主色：Ionic 蓝（实测完整色阶） ===== */
  --ifm-color-primary: #3578e5;
  --ifm-color-primary-dark: #306cce;
  --ifm-color-primary-darker: #2d66c3;
  --ifm-color-primary-darkest: #2554a0;
  --ifm-color-primary-light: #538ce9;
  --ifm-color-primary-lighter: #72a1ed;
  --ifm-color-primary-lightest: #9abcf2;

  /* ===== 文本色（实测） ===== */
  --ifm-color-content: #222d3a;

  /* ===== 代码块（实测：背景 #F6F8FC、圆角 8px） ===== */
  --ifm-code-background: #f6f8fc;
  --ifm-code-font-size: 95%;
  --ifm-pre-border-radius: 8px;

  /* ===== 右侧目录（实测 12.8px / #445B78，字号走默认 0.8rem） ===== */
  --ifm-toc-link-color: #445b78;

  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.1);
}

[data-theme='dark'] {
  /* Ionic 暗色实测值 */
  --ifm-background-color: #03060b;
  --ifm-background-surface-color: #03060b;
  --ifm-color-content: #e3e3e3;
  --ifm-color-content-secondary: #eef1f3;
  --ifm-code-background: #161d25;
  --ifm-toc-link-color: #eef1f3;
  --ifm-hover-overlay: rgba(255, 255, 255, 0.05);
  --docusaurus-highlighted-code-line-bg: rgba(0, 0, 0, 0.3);
}

/* ===== 文档内容列宽：实测 672px ===== */
/* docItemContainer 为构建时哈希类名，用属性包含选择器（Ionic 官网同款做法），
   docs 与 docs2 两个插件实例同时生效 */
[class*='docItemContainer'] {
  max-width: 672px;
}

/* ===== 文章排版（.markdown 同时作用于 docs/docs2/blog 正文，与 Ionic 一致；
   navbar、sidebar、footer 等框架 UI 不受影响，因为 Ionic 也是只在正文区放大字号） ===== */
.markdown h1 {
  font-size: 3rem;        /* 48px */
  font-weight: 700;
  color: #03060b;
  margin: 0 0 25px;
}
.markdown h2 {
  font-size: 2rem;        /* 32px */
  font-weight: 700;
  color: #03060b;
  margin: 55px 0 12px;
}
.markdown h3 {
  font-size: 1.5rem;      /* 24px */
  font-weight: 700;
  color: #03060b;
  margin: 16px 0 12px;
}
.markdown h4 {
  font-size: 1.25rem;    /* 20px（按 Ionic 字阶 48/32/24/20 推定，页面无 h4 实测样本） */
  font-weight: 700;
  color: #03060b;
  margin: 16px 0 12px;
}

.markdown p {
  font-size: 1.25rem;    /* 20px */
  line-height: 2;        /* 40px */
  color: #35404e;
  margin-bottom: 20px;
}
.markdown li {
  font-size: 1.25rem;
  line-height: 2;
}

/* 暗色下的标题与正文颜色（实测） */
[data-theme='dark'] .markdown h1,
[data-theme='dark'] .markdown h2,
[data-theme='dark'] .markdown h3,
[data-theme='dark'] .markdown h4 {
  color: #ffffff;
}
[data-theme='dark'] .markdown p {
  color: #ced6e0;
}
```

### 实现要点

1. **不改全局 `--ifm-h1-font-size` 等标题变量**——实测发现 Ionic 官网的这些变量仍是默认值（2rem/1.5rem/1.25rem），48px/32px 大字号是**只作用于正文区**的自定义 CSS。用 `.markdown h1` 选择器复刻同样效果，避免 navbar、footer、首页 hero 的标题被意外放大。
2. `[class*='docItemContainer']`：Docusaurus 构建时给该容器加哈希后缀（如 `docItemContainer_Djhp`），属性包含选择器是社区通用且 Ionic 官网实测采用的做法。
3. 中文支持：Ionic 原字体栈只含西文字体，本站内容为中文，故在其后追加 `PingFang SC / Hiragino Sans GB / Microsoft YaHei / Noto Sans SC`，Mac/Windows 上均可正确渲染。
4. navbar 高度（60px）、TOC 字号（0.8rem）、代码块 padding（1rem）、链接交互（默认无下划线、hover 下划线）均与 Docusaurus/Infima 默认一致，无需改动。
5. 不加载任何 Webfont（Ionic 官网同样只用系统字体），零网络开销。

## 假设与决策

- **应用范围**：全站生效（docs、docs2、blog 正文 + 全站配色/字体）。`.markdown` 类对两个 docs 插件和 blog 通用，这正是"复刻站点风格"的自然边界；框架结构（侧边栏、导航布局）保持 Docusaurus 原样，仅复刻视觉风格。
- **主色**：从当前绿色换成 Ionic 蓝 `#3578e5`（复刻的核心就是视觉风格，绿色是脚手架残留默认值）。
- **h4 字号**：Ionic 页面无 h4 实测样本，按其字阶规律（48/32/24/20）取 20px。
- **blockquote / 表格 / admonition**：沿用 Docusaurus 默认（Ionic 的对应变量也与默认一致），仅在暗色下跟随新配色。
- **不改动** `docusaurus.config.js`、不新增依赖、不 swizzle 组件。

## 验证步骤

1. `cd cancloude && npm install`（若 node_modules 缺失）→ `npm start`。
2. 打开 `/docs/intro` 与 `/docs2/guide/getting-started`，逐项核对：
   - H1 = 48px/粗体/近黑色 `#03060B`；H2 = 32px；正文 = 20px、行高 40px、色 `#35404E`；
   - 正文容器 max-width = 672px；
   - 代码块背景 `#F6F8FC`、圆角 8px；行内代码背景 `#F6F8FC`、圆角 6.4px；
   - 链接为蓝色 `#3578E5`；右侧 TOC 链接 12.8px、`#445B78`。
3. 切换暗色模式核对：背景 `#03060B`、标题纯白、正文 `#CED6E0`、代码块 `#161D25`、navbar `#03060B`。
4. 回归检查首页、blog 列表/文章页无排版异常（标题未被放大、footer 列标题正常）。
5. `npm run build` 通过。
