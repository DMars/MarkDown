# 关于

## 这个站点

这是一个用 [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) 搭建的静态知识库，托管在 GitHub Pages 上，完全免费。

- **写作**：Markdown 文件放在 `docs/` 目录
- **配置**：主题、导航、插件集中在 `mkdocs.yml`
- **部署**：推送到 `main` 分支后由 GitHub Actions 自动构建发布

## 站点特性

- 全文搜索（右上角或按 `/`）
- 浅色 / 深色主题自动跟随系统并可手动切换
- 代码语法高亮与一键复制
- 表格、提示块、折叠块、标签页、任务清单
- 移动端自适应

## 维护提示

!!! note "新增页面后记得登记导航"
    在 `docs/` 里新建的 Markdown 文件，需要在 `mkdocs.yml` 的 `nav` 里加上路径，才会出现在侧边栏导航中。

本地预览：

```bash
pip install -r requirements.txt
mkdocs serve
```
