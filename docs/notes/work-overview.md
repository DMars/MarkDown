# 工作信息总览

对 CodeBuddy 工作区内全部技术资料的分类整理与索引。共梳理 **17 个有效文件**，覆盖 UE 材质 / 着色器、DCC 烘焙工具与编辑器自动化脚本三条主线。

## 分类导航

| 分类 | 内容 | 明细入口 | 文件数 |
| --- | --- | --- | --- |
| WPO 外扩与平滑法线 | 不破面原理、三种方案、Blender / Houdini / 3ds Max / UE4.18 烘焙落地 | [WPO 外扩与平滑法线](ue-wpo-inflate.md) · [平滑法线烘焙工具链](../tools/dcc-bake-toolchain.md) | 6 |
| 琥珀材质 | UE5 嵌入式单节点版与 UE4 四 Custom 节点版 | [不透明琥珀材质](ue-amber-material.md) | 2 |
| UV 遮罩与三阶化 | UV 单通道遮罩、灰阶三阶化、三阶颜色、UV 缩放工具 | [UV 遮罩与三阶化着色](ue-uv-mask.md) | 6 |
| 轮播自发光 | UV 驱动的黄色图标轮播发光材质（全节点版 / Custom 版） | [UV 驱动轮播自发光](ue-carousel-emissive.md) | 2 |
| 螺旋 WPO | 模型空间螺旋形变 + 随机强度场 Custom 节点 | [螺旋 WPO 与随机强度](ue-wpo-spiral.md) | 1 |

## 时间线

目录名即会话日期，反映资料产出顺序：

| 时间 | 产出 | 归属分类 |
| --- | --- | --- |
| 2026-10-10 | 琥珀材质 UE5 / UE4 两份实现文档；WPO 螺旋法线节点 | 琥珀材质、螺旋 WPO |
| 2026-10-13 | UV 通道遮罩 HLSL 与材质生成脚本、灰阶三阶化、三阶颜色、UV 缩放工具 | UV 遮罩与三阶化 |
| 2026-10-16 | WPO 外扩完整方案文档、Custom HLSL、Blender / Houdini 烘焙脚本 | WPO 外扩 |
| 2026-11-09 ~ 11-10 | UE4.18 烘焙脚本与 3ds Max 烘焙工具（含实测补充与验证记录） | WPO 外扩 / 烘焙工具链 |
| 2026-11-17 | 黄色轮播自发光材质生成器（全节点版 + Custom 版） | 轮播自发光 |

## 文件清单

### WPO 外扩与平滑法线

| 文件 | 作用 | 关键结论 |
| --- | --- | --- |
| `20261016000610/README.md` | 完整方案说明 | 三条路线对比：全软边副本 / 烘焙平滑法线（推荐）/ 纯运行时；烘焙脚本只写额外通道，不改原法线 |
| `20261016000610/UE/WPO_Inflate_CustomNode.hlsl` | Custom 节点 HLSL | 四种数据源（烘焙 UV / 顶点色 / 物体空间 / 原法线）+ 凹角自动收窄 `CURVE_FADE` |
| `20261016000610/Tools/bake_smooth_normals_blender.py` | Blender 烘焙 | 按位置合并求平均，存 VCOLOR 或 UV1/UV2；支持 TS / OS 坐标系 |
| `20261016000610/Tools/bake_smooth_normals_houdini.vex` | Houdini 烘焙 | Point Wrangle + Vertex Wrangle 两段，利用共享 point 天然得到平滑法线 |
| `20261109154545/bake_smooth_normal_uv.py` | UE4.18 编辑器脚本 | 三层策略：探测写入接口 / 导出 JSON+CSV / 导出 FRawMesh C++ 片段；实测确认无 UV 写入接口 |
| `20261109154545/BakeSmoothNormalToUV34.ms` | 3ds Max 烘焙工具 | 角度加权面法线；预补偿 UE FBX 的 V 翻转；Scale / 镜像方向问题实测记录 |

### 琥珀材质

| 文件 | 作用 | 关键结论 |
| --- | --- | --- |
| `20261010150943/AmberMaterial_UE/Amber_Opaque_UE.md` | UE5 实现 | 单 Custom 节点、Inputs 为空、参数内嵌、Additional Outputs 输出 7 个引脚 |
| `20261010150943/AmberMaterial_UE/Amber_Opaque_UE4.md` | UE4 实现 | 无 Additional Outputs，改用 4 个 Custom 节点 + Float3/Float2 通道打包 + ComponentMask 拆分；Custom 内不能定义函数 |

### UV 遮罩与三阶化

| 文件 | 作用 | 关键结论 |
| --- | --- | --- |
| `20261013215859/UVChannelMask_UE4.hlsl` | UV 单通道遮罩 | A/B 两种宽度语义；13 个 Input；移动端用 `MF_yaTime` 替代 Time 节点 |
| `20261013215859/Make_UVChannelMask_Material.py` | 材质一键生成 | 自动建材质、接 13 个引脚与 ScalarParameter；已落地版本勿重复执行覆盖 |
| `20261013215859/GrayPosterize3_UE4.hlsl` | 灰阶三阶化 | 灰度切三档分别装入 RGB；硬遮罩时三通道互斥且和为 1 |
| `20261013215859/ThreeToneColor.hlsl` | Shader Graph 三阶颜色 | String 模式 Custom Function；`$precision` 占位、工具函数内联 |
| `20261013215859/ThreeToneColor_UE4.hlsl` | UE4 三阶颜色 | 同一算法的 UE4 版；`return` 类型必须与 Output Type 严格一致 |
| `20261013215859/UV_Scale_Tool.ms` | 3ds Max UV 缩放 | 4 种缩放中心、自动识别 Unwrap UVW / 基础贴图通道 |

### 轮播自发光与螺旋 WPO

| 文件 | 作用 | 关键结论 |
| --- | --- | --- |
| `20261117102550/create_yellow_carousel_material.py` | 全节点版生成器 | 相位由 UV 驱动，`MF_yaTime` 提供时间，可调 UVAxis / PhaseStep / EdgeSoft 等 |
| `20261117102550/create_yellow_carousel_material_custom.py` | Custom/HLSL 版 | 从世界空间相位改为 UV 相位；MaterialFunction 必须通过节点调用，不能在 HLSL 内调用 |
| `20261010150946/WPO_SpiralNormal.hlsl` | 螺旋形变节点 | 局部空间螺旋 + 随机强度场；输出需经 TransformVector(Local→World) 再接 WPO |

## 共性经验

1. **UE4 Custom 节点限制**：只能单输出并 `return`；不能定义函数；Input 必须全部接线，否则可能触发 X3004。
2. **移动端工程时间源**：统一使用 `/Game/Arts/Common/MatFunctions/MF_yaTime`，不接引擎 Time 节点。
3. **数据通道的坑**：FBX 导出会做 `V = 1 - V`，顶点色可能走 sRGB；导入时不要 Recompute Normals。
4. **WPO 只影响渲染**：不改物理碰撞；Nanite 网格忽略 WPO；动态位移要输出 Velocity 防拖影。
5. **批量改资产前先备份**：UE4.18 无可靠 UV 写入接口，手写 FBX 重导会破坏资产。
