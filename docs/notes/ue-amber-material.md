# 不透明琥珀材质（Amber / Resin）

目标：`Blend Mode = Opaque`，不进半透队列、无排序问题，但保留琥珀的透光感、内部流纹与外部抛光高光。实现为 `Default Lit` + `Clear Coat` + 程序化内部纹理与假次表面（走 Emissive）。

## UE5：嵌入式单节点版

与上一版的区别：

1. **不需要配置任何 Inputs**：世界坐标、法线、相机向量、主光方向都在节点内从 `Parameters.*` / `ResolvedView.*` 读取。
2. **参数内嵌**：顶部常量区直接改数字。
3. **单节点一体化**：流纹 / 气泡 / SSS / BaseColor / Roughness 一次算完，通过 Additional Outputs 引出。

材质设置：Surface / Opaque / Default Lit；需要真 SSS 可换 Subsurface，Blend Mode 仍保持 Opaque。UE5 若开启 Substrate 需改用 `Substrate Slab` 或关闭 Substrate。

Custom 节点：`Output Type = CMOT Float3`，Inputs 为空，Additional Outputs 依次为：

| # | Output Name | Type | # | Output Name | Type |
| --- | --- | --- | --- | --- | --- |
| 1 | `SwirlR` | Float1 | 5 | `Speck` | Float1 |
| 2 | `SwirlG` | Float1 | 6 | `SSS` | Float3 |
| 3 | `SwirlB` | Float1 | 7 | `Rough` | Float1 |
| 4 | `Bubble` | Float1 | | | |

!!! tip "命名"
    Output Name 必须与代码里赋值的变量名完全一致，区分大小写。需要暴露给实例的参数（如辉光强度）在节点外乘 `ScalarParameter`。

### 算法要点

- **流纹**：`Q` 做对数螺旋旋转，再用三通道 FBM 域扭曲，`veins = pow(1 - abs(n1-0.5)*2, 6)`。
- **气泡 / 杂质**：3D 网格哈希 + 单元内随机圆心，气泡约 20% 格子命中；杂质由 `h` 阈值控制密度。
- **假 SSS**：`back = pow(saturate(dot(V, -HnBack)), SSS_POWER)`，叠加 wrap 光照与边缘透光，乘光色与透射衰减色。
- **颜色**：`C_DEEP → C_BRIGHT → C_DARK → C_BUBBLE` 依次 lerp；Roughness 由流纹细节与气泡调制。

## UE4：四 Custom 节点版

UE4 的 Custom 节点**没有 Additional Outputs**，改为「4 个 Custom 节点 + 通道打包 + `ComponentMask` 拆分」。

| 节点 | Output Type | 作用 | 推荐输入 |
| --- | --- | --- | --- |
| `AmberPattern` | Float3 | 流纹 R=大结构 / G=细节 / B=流纹 | Scale=6.0, Swirl=1.6, Warp=0.9 |
| `AmberInclusion` | Float2 | R=气泡 / G=杂质 | Scale=14.0, Density=0.35 |
| `AmberBase` | Float3 | 合成 BaseColor | 上两步输出 |
| `AmberSSS` | Float3 | 假次表面 → Emissive | 法线 / 光向 / 厚度 |

```text
WorldPos - CameraVector*Depth  ─┬─ AmberPattern(F3) → Mask R/G/B
                                └─ AmberInclusion(F2) → Mask R/G
                                     ↓
                              AmberBase(F3) → BaseColor
                              AmberSSS(F3) × GlowTint → Emissive
```

!!! warning "UE4 / fxc SM5 限制"
    - Custom 节点内**不能定义函数**（报 nested functions / undeclared identifier），改用 `#define` 宏 + 循环内联。
    - 添加 Input 后默认是 Float1，向量引脚要在右侧下拉改成 Float3，否则类型不匹配。
    - 视差下沉 `P = WorldPos - CameraVector * Depth` 用 `Subtract` 节点在外部完成。

## 关联

- [工作信息总览](work-overview.md)
- [UE 材质开发](ue-material.md)：Custom 节点通用注意事项
