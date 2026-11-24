# UV 驱动轮播自发光材质

UE4.18 中让深色金属面板上的黄色图标区域依次亮起的方案。生成目标为 `M_YellowCarouselEmissive`（全节点版）与 `M_YellowCarouselEmissive_Custom`（Custom 版）。

## 原理

相位由 **UV 坐标**驱动，而非世界空间位置：

```text
phase = (UV.<分量> * UVScale) * PhaseStep + PhaseOffset
raw   = 0.5 + 0.5 * sin(phase + Time * CycleSpeed)
pulse = SmoothStep(0.5 - 0.5*k, 0.5 + 0.5*k, raw)   // k = saturate(EdgeSoft)
pulse = lerp(MinGlow, 1.0, pulse)                    // 保底余辉
Emissive = Mask * pulse * Sweep * GlowColor * Intensity
```

N 段轮播时 `PhaseStep = 2π/N`（三段默认 2.0944），物体各自 UV 需错开才能依次亮起。`UVAxis` 决定相位沿 U(0) 还是 V(1) 推进，材质实例里可切换。

可选流光：`frac(U + Time * SweepSpeed)` → `sin` → `pow(..., SweepSharpness)`，由 `EnableSweep` 开关。

## 时间源

项目移动端工程不接引擎 Time 节点，统一使用材质函数：

```text
/Game/Arts/Common/MatFunctions/MF_yaTime.MF_yaTime
```

!!! warning "Custom 版的关键限制"
    MaterialFunction **不能**在 Custom 的 HLSL 里直接调用，必须用 `MaterialFunctionCall` 节点，把其输出接到 Custom 的 `tm` 输入引脚。这正是全节点版与 Custom 版都必须保留节点的原因。

## 两个版本

| 版本 | 实现 | 适用 |
| --- | --- | --- |
| 全节点版 | 纯 MaterialExpression 节点搭建，无 Custom / HLSL | 便于在材质编辑器里逐节点调试 |
| Custom 版 | 15 个输入引脚 + 一段 HLSL | 参数集中、便于批量调相位算法 |

Custom 版输入引脚：`tm`(float)、`uv`(float2)、`msk`(float)、`uvAxis`、`edgeSoft`、`uvScale`、`phaseStep`、`phaseOffset`、`cycleSpeed`、`minGlow`、`glowColor`(float4)、`intensity`、`sweepSpeed`、`sweepSharpness`、`enableSweep`。

## 参数速查

| 参数 | 默认 | 说明 |
| --- | --- | --- |
| `UVAxis` | 0 | 0=U 向，1=V 向，可传中间值过渡 |
| `UVScale` | 1.0 | UV 相位密度 |
| `CycleSpeed` | 1.2 | 轮播速度 |
| `PhaseStep` | 2π/3 | 相位步长 |
| `EdgeSoft` | 0.15 | 0=硬边方波，1=纯正弦软边 |
| `MinGlow` | 0.06 | 最暗时余辉 |
| `GlowColor` | (1, 0.75, 0.12) | 暖黄 |
| `Intensity` | 6.0 | 发光强度 |
| `SweepSpeed` / `SweepSharpness` / `EnableSweep` | 0.5 / 6.0 / 0 | 可选流光 |

图标贴图用 R 通道当黄色遮罩（黄底黑底图即可）；要看到外扩光晕，场景需开启 Bloom（Post Process Volume）。
