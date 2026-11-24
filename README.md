# DMars 的知识库

基于 [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) 搭建的个人知识库站点。

在线地址：<https://DMars.github.io/MarkDown/>

## 本地预览

```bash
pip install -r requirements.txt
mkdocs serve
```

浏览器打开 <http://127.0.0.1:8000>，修改 Markdown 文件会实时刷新。

## 写作

- 所有文章放在 `docs/` 目录下，使用 Markdown 编写。
- 新增文章后，在 `mkdocs.yml` 的 `nav` 中登记路径，才会出现在侧边栏。
- 图片等资源放 `docs/assets/`，用相对路径 `![说明](assets/xxx.png)` 引用。

## 发布

提交到 `main` 分支即可，GitHub Actions 会自动构建并部署到 GitHub Pages：

```bash
git add .
git commit -m "更新知识库"
git push origin main
```

首次使用请在仓库 **Settings → Pages → Build and deployment** 中，将 Source 设为 **GitHub Actions**。

## 目录结构

```
.
├── mkdocs.yml              # 站点配置：主题、导航、插件
├── requirements.txt        # Python 依赖
├── docs/                   # 所有文章（Markdown）
│   ├── index.md            # 首页
│   ├── notes/              # 技术笔记
│   ├── tools/              # 工具链
│   ├── about.md            # 关于
│   └── assets/             # 图片、自定义样式
└── .github/workflows/
    └── deploy.yml          # 自动部署脚本
```
