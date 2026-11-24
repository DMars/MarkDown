# 技术笔记

这里按主题沉淀日常开发中的知识点与踩坑记录。

## 全部笔记

- [工作信息总览](work-overview.md) — 全部工作资料的分类索引、时间线与文件清单
- [HLSL 着色器入门](hlsl-shader.md) — 常用函数、语义与跨平台注意事项
- [UE 材质开发](ue-material.md) — 材质节点、Custom 节点与性能考量
- [WPO 外扩与平滑法线](ue-wpo-inflate.md) — 不破面原理、三种方案与材质接线
- [不透明琥珀材质](ue-amber-material.md) — UE5 嵌入式单节点版与 UE4 四节点版
- [UV 遮罩与三阶化着色](ue-uv-mask.md) — 可控宽度遮罩、灰阶分阶、三阶颜色
- [UV 驱动轮播自发光](ue-carousel-emissive.md) — UV 相位驱动的黄色图标轮播
- [螺旋 WPO 与随机强度](ue-wpo-spiral.md) — 模型空间螺旋形变 Custom 节点

工具与脚本类内容见 [工具链](../tools/index.md)：[平滑法线烘焙工具链](../tools/dcc-bake-toolchain.md)、[UE4.18 编辑器 Python 自动化](../tools/ue4-python-automation.md)。

## 写作模板

新增笔记时可以套用下面的结构，保持风格统一：

```markdown
# 标题

一句话说明这篇笔记解决什么问题。

## 背景

为什么需要这个知识点。

## 正文

核心内容、代码示例、配图。

## 踩坑记录

!!! warning "注意"
    记录容易出错的地方。

## 参考

- 链接 1
- 链接 2
```
