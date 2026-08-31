# GitHub Pages 部署教程

本文档介绍如何将本项目的 Docusaurus 站点（位于 `cancloude/` 子目录）部署到 GitHub Pages。

- GitHub 用户名：`LouisCan`
- 仓库名：`blog-design-docusaurus`
- 部署后站点地址：`https://LouisCan.github.io/blog-design-docusaurus/`
- 部署方式：GitHub Actions 自动部署（每次 push 到 `master` 分支自动触发）

---

## 一、整体流程

```
本地修改代码 → push 到 GitHub → Actions 自动构建 cancloude/ → 部署到 GitHub Pages
```

共需要完成 4 步：

1. 修改 `docusaurus.config.js`（GitHub Pages 专属配置）✅ 已完成
2. 添加 GitHub Actions workflow ✅ 已完成
3. 在 GitHub 创建仓库并推送代码
4. 在仓库设置中启用 GitHub Pages（Source 选 GitHub Actions）

---

## 二、配置说明（已完成，供参考）

### 1. `cancloude/docusaurus.config.js`

GitHub Pages 项目页面的访问路径是 `https://<用户名>.github.io/<仓库名>/`，因此需要以下配置：

```js
const config = {
  // 站点完整域名
  url: 'https://LouisCan.github.io',

  // 站点部署的子路径，必须是 /<仓库名>/
  baseUrl: '/blog-design-docusaurus/',

  // GitHub Pages 部署配置
  organizationName: 'LouisCan',          // GitHub 用户名
  projectName: 'blog-design-docusaurus', // 仓库名
  // ...
};
```

> ⚠️ `baseUrl` 是最容易出错的一项。如果忘记设置或设置错误，部署后会出现白屏、样式丢失、404 等问题。

### 2. `.github/workflows/deploy.yml`

由于站点在 `cancloude/` 子目录（而不是仓库根目录），workflow 中所有构建步骤都需要加 `working-directory: cancloude`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
          cache-dependency-path: cancloude/package-lock.json

      - name: Install dependencies
        working-directory: cancloude
        run: npm ci

      - name: Build website
        working-directory: cancloude
        run: npm run build

      - uses: actions/upload-pages-artifact@v3
        with:
          path: cancloude/build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## 三、创建 GitHub 仓库并推送代码

### 1. 创建仓库

访问 [github.com/new](https://github.com/new)，创建名为 `blog-design-docusaurus` 的仓库（不要勾选添加 README / .gitignore，本地已有代码）。

### 2. 添加远程并推送

```bash
cd blog-design-docusaurus

# 添加 GitHub 远程（origin）
git remote add origin https://github.com/LouisCan/blog-design-docusaurus.git

# 提交部署配置
git add cancloude/docusaurus.config.js .github/workflows/deploy.yml
git commit -m "chore: 配置 GitHub Pages 部署"

# 推送
git push -u origin master
```

---

## 四、启用 GitHub Pages

1. 打开 GitHub 仓库页面 → **Settings**（设置）
2. 左侧菜单选择 **Pages**
3. **Source** 选择 **GitHub Actions**（不要选 "Deploy from a branch"）

设置完成后，推送会触发 Actions 自动构建部署。可在仓库的 **Actions** 标签页查看构建进度。

---

## 五、验证部署

1. 在 **Actions** 页面确认 `Deploy to GitHub Pages` 工作流显示绿色 ✅
2. 访问站点：`https://LouisCan.github.io/blog-design-docusaurus/`

> 首次部署可能需要几分钟。如果显示 404，等待 1–2 分钟后刷新重试。

---

## 六、日常更新

之后每次更新内容，只需正常提交推送即可自动重新部署：

```bash
git add .
git commit -m "update: 更新博客内容"
git push
```

---

## 七、本地验证构建（可选）

推送前可以先在本地验证能否构建成功：

```bash
cd cancloude
npm run build   # 构建，产物在 cancloude/build
npm run serve   # 本地预览构建结果（注意路径带 /blog-design-docusaurus/ 前缀）
```

---

## 八、常见问题

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| 页面白屏 / 样式丢失 | `baseUrl` 未设置为 `/<仓库名>/` | 检查 `docusaurus.config.js` 中的 `baseUrl` |
| 站点 404 | Pages 未启用或部署未完成 | 检查 Settings → Pages 的 Source 是否为 GitHub Actions；查看 Actions 是否构建成功 |
| Actions 构建失败：npm ci 报错 | 依赖锁文件与代码不同步 | 本地执行 `npm install` 后把 `package-lock.json` 一起提交 |
| 图片 / 资源加载失败 | 引用了绝对路径 | 资源引用使用相对路径，或通过 `@site/static/...` 引入 |
| 部署后路径变化导致本地 `npm run serve` 访问根路径 404 | 站点部署在子路径 | 访问 `http://localhost:3000/blog-design-docusaurus/` |

---

## 附：目录结构对照

```
blog-design-docusaurus/          # 仓库根目录
├── .github/
│   └── workflows/
│       └── deploy.yml           # GitHub Actions 部署流程
└── cancloude/                   # Docusaurus 站点目录
    ├── docusaurus.config.js     # 站点配置（url / baseUrl）
    ├── docs/                    # 文档内容
    ├── blog/                    # 博客内容
    ├── src/                     # 自定义组件与样式
    ├── static/                  # 静态资源
    └── build/                   # 构建产物（不提交到 git）
```
