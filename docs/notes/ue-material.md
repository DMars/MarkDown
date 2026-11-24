# UE 材质开发

记录材质系统开发中的通用思路，示例为公开技术概念。

## 材质节点分类

- **输入节点**：Texture Sample、Constant、Vector Parameter
- **数学节点**：Add、Multiply、Lerp、Clamp
- **流程控制**：If、For、Function（Material Function）
- **输出节点**：Base Color、Roughness、Normal、Emissive

## Custom 节点

当内置节点无法满足需求时，可以用 Custom 节点注入 HLSL 代码片段。

!!! tip "可读性优先"
    Custom 节点里只放必要的计算逻辑，复杂算法建议封装成 Material Function，并写好注释与输入说明。

```hlsl
// 示例：返回两个颜色的加权混合
float3 BlendColor(float3 a, float3 b, float t)
{
    return lerp(a, b, saturate(t));
}
return BlendColor(ColorA, ColorB, Weight);
```

## 性能考量

- 减少采样器数量，复用纹理采样结果
- 避免分支发散，尽量用 `step` / `lerp` 替代 `if`
- 半精度与全精度混用会带来额外转换开销
- 移动端关注指令数与带宽，必要时用贴图烘焙预计算

??? example "一个优化前后对比的思路"

    优化前：逐像素计算复杂光照公式。
    优化后：把静态部分烘焙到贴图，运行时只做一次查表与混合。

## 参考

- 引擎官方材质文档
- 平台渲染性能优化指南
