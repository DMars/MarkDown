# UV 遮罩与三阶化着色

汇总一组围绕「单通道坐标 → 可控遮罩 / 分阶输出」的 UE4 方案，以及配套 DCC 工具。

## UV 单通道可控宽度遮罩

取 UV 的 U 或 V，重映射到 0~1，在轴上生成宽度可控、可扫动的遮罩带，可用于扫描线、扫光、局部高亮、进度条与溶解。

两种「宽度」语义：

| 方案 | 含义 | 典型用途 |
| --- | --- | --- |
| A | 亮带本身的宽度，停在 `MaskCenter` 并可扫动 | 扫描线、扫光、局部高亮 |
| B | 从 0 向 1 填充时边缘羽化的宽度 | 进度条、揭示 / 擦除、溶解出现 |

宽度按占 0~1 坐标的比例计（0.2 = UV 长度的 20%），与贴图分辨率和模型大小无关。

Custom 节点：`Output Type = CMOT Float1`，13 个 Input：

`UV`(Vector2), `Axis`, `CoordMin`, `CoordMax`, `CustomCoord`, `UseCustomCoord`, `Invert`, `Repeat`, `MaskCenter`, `MaskWidth`, `EdgeSoftness`, `HardMask`, `Sweep`。

核心逻辑：

```hlsl
float raw = lerp(UV.x, UV.y, step(0.5, Axis));
float c = saturate((raw - CoordMin) / max(CoordMax - CoordMin, 0.001));
c = lerp(c, frac(c * max(Repeat,1)), step(1.0001, Repeat));
float center = lerp(MaskCenter, lerp(-halfW, 1+halfW, saturate(MaskCenter)), step(0.5, Sweep));
float d = (c - (center - halfW)) / MaskWidth;
float soft = smoothstep(0, s, d) * (1 - smoothstep(1-s, 1, d));
float mask = saturate(lerp(soft, step(0,d)*(1-step(1,d)), step(0.5, HardMask)));
return mask;
```

!!! note "项目适配"
    移动端工程不接 Time 节点，扫动相位统一用 `/Game/Arts/Common/MatFunctions/MF_yaTime`（`Result/InPeriod`）。`Make_UVChannelMask_Material.py` 可一键建材质并接齐 13 个引脚；目标材质已由项目专用脚本落地，**不要再执行通用脚本覆盖**。

## 灰阶三阶化（Posterize 3）

把灰阶图按灰度切成三档，分别装进 RGB 三通道：`< 阈值0 → R(50)`、`阈值0~阈值1 → G(128)`、`> 阈值1 → B(220)`。硬遮罩时三通道互斥，相加恒为 1，后续 `ComponentMask` 取单通道即可当 Alpha / Opacity / StaticSwitch 条件。

`Output Type = CMOT Float3`，输入含 `GrayTexture`(Texture2D)、`UV`、`InMin/InMax`、`Invert`、`Level0/1/2`、`阈值` 等。灰阶偏灰时先用 `InMin=0.2 / InMax=0.8` 拉开对比再分阶。

## 三阶颜色控制

按 NdotL 把表面分为暗部 / 中间调 / 亮部三阶，可自定义光方向、过渡区间与三阶颜色；输出 `Out`、`Alpha`(原始 NdotL)、`BandMask`(三阶遮罩)。

| 版本 | 载体 | 差异 |
| --- | --- | --- |
| `ThreeToneColor.hlsl` | Shader Graph Custom Function（String 模式） | 用 `$precision` 占位；老 URP 不支持就全局替换为 `float` |
| `ThreeToneColor_UE4.hlsl` | UE4 Material Custom | 不支持 `$precision`；可附加世界空间渐变推进 `GradientPhase` |

!!! warning "常见报错"
    `return` 的类型必须与 Details → Output Type 严格一致（Float3 返回 `float3`，想带 NdotL 就选 Float4 返回 `float4(col, Alpha)`）。UE 不会自动补通道，日志里的 `vec3/half4` 是交叉编译后的叫法。

## DCC 配套：UV 缩放工具

`UV_Scale_Tool.ms`（3ds Max 2022）支持 U/V 独立或统一缩放、4 种缩放中心（UV 原点 / 0-1 平铺中心 / UV 包围盒中心 / 自定义），自动识别 Unwrap UVW 修改器或基础几何贴图通道，也可只缩放 UV 编辑器中选中的点。
