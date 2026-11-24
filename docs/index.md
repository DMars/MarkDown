# 欢迎来到我的知识库

这里是 DMars 的个人知识库，用来沉淀技术笔记、着色器开发经验与工具链实践。基于 **MkDocs Material** 构建，支持全文搜索、深色模式与代码高亮。

!!! tip "怎么用"
    左侧是导航目录，右上角可以切换深浅色主题、全局搜索。**推送到 main 分支后站点自动更新**，无需手动部署。

<div class="grid cards" markdown>

-   :material-notebook-outline:{ .lg .middle } **技术笔记**

    ---

    着色器、渲染与引擎开发的系统性笔记。

    [:octicons-arrow-right-24: 浏览笔记](notes/index.md)

-   :material-tools:{ .lg .middle } **工具链**

    ---

    Python 自动化、版本管理与效率工具实践。

    [:octicons-arrow-right-24: 查看工具](tools/index.md)

-   :material-information-outline:{ .lg .middle } **关于**

    ---

    这个站点是如何搭建与维护的。

    [:octicons-arrow-right-24: 了解更多](about.md)

</div>

## 快速开始

=== "新增一篇文章"

    1. 在 `docs/` 下新建 Markdown 文件，例如 `notes/my-post.md`。
    2. 在 `mkdocs.yml` 的 `nav` 中登记它，才会出现在侧边栏。
    3. `git push` 后站点自动更新。

=== "本地预览"

    ```bash
    pip install -r requirements.txt
    mkdocs serve
    ```

    打开 <http://127.0.0.1:8000>，改动实时刷新。

## 常用写作语法

| 需求 | 写法 | 效果 |
| --- | --- | --- |
| 提示块 | `!!! note` | 彩色提示框 |
| 折叠块 | `??? tip` | 点击展开 |
| 标签页 | `=== "标题"` | 多标签切换 |
| 代码高亮 | ` ```hlsl ` | 带语言高亮的代码块 |
| 任务清单 | `- [ ] 待办` | 可勾选复选框 |

??? example "看一个提示块的实际效果"

    ```hlsl
    float3 ComputeNormal(float3 worldPos)
    {
        float3 dx = ddx(worldPos);
        float3 dy = ddy(worldPos);
        return normalize(cross(dx, dy));
    }
    ```

    这是一段带语法高亮的 HLSL 示例代码。
