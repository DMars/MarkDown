# HLSL 着色器入门

记录 HLSL 开发中高频使用的知识点。示例均为通用技术内容。

## 常用内置函数

| 函数 | 作用 |
| --- | --- |
| `ddx` / `ddy` | 求相邻像素导数，常用于法线重建 |
| `saturate` | 将值钳制到 `[0, 1]` |
| `lerp` | 线性插值 |
| `mul` | 矩阵乘法，注意向量是行还是列 |

## 语义与插值

顶点着色器到像素着色器之间传递的数据需要匹配语义，例如 `TEXCOORD0`。插值修饰符会影响取值方式：

- `linear`：默认线性插值
- `noperspective`：透视校正关闭
- `flat`：不插值，取provoking vertex 的值

## 导数与法线重建

```hlsl
float3 ReconstructNormal(float3 worldPos)
{
    float3 dp1 = ddx(worldPos);
    float3 dp2 = ddy(worldPos);
    return normalize(cross(dp1, dp2));
}
```

!!! warning "平台差异"
    在移动端（如 Metal / GLES）上，`ddx` / `ddy` 的精度与桌面端可能不同，半精度（`half`）下尤其明显。跨平台着色器建议对导数结果做归一化与容差处理。

## 参考

- Microsoft HLSL 文档
- 各平台图形 API 的着色器最佳实践
