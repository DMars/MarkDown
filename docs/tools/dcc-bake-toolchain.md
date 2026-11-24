# 平滑法线烘焙工具链

为 [WPO 外扩](../notes/ue-wpo-inflate.md)准备平滑法线数据的四种 DCC / 引擎路径。共同目标：**按 3D 位置合并法线，让同一位置的所有顶点副本拿到同一方向**，外扩后不裂。

## 总览

| 环境 | 脚本 | 输出 | 特点 |
| --- | --- | --- | --- |
| Blender | `bake_smooth_normals_blender.py` | 顶点色或 UV1/UV2 | 配置简单，面积加权，支持跨物体合并 |
| Houdini | `bake_smooth_normals_houdini.vex` | `uv2` / `uv3` 或 Cd | Point + Vertex Wrangle 两段 |
| 3ds Max | `BakeSmoothNormalToUV34.ms` | map channel 3 / 4 | 角度加权、FBX V 翻转预补偿、含实测校验 |
| UE4.18 | `bake_smooth_normal_uv.py` | JSON+CSV / FRawMesh C++ 片段 / 解码材质 | 无 UV 写入 API 时的落地方案 |

## Blender

Blender → Scripting → 打开脚本 → 改 `CONFIG` → Run Script。烘焙前 `Ctrl+A` 应用缩放（尤其非等比），且不要处于 Edit Mode。

关键配置：

| 参数 | 建议 |
| --- | --- |
| `STORE_MODE` | 有 3 套以上 UV 余量用 `"UV"`（float32）；否则 `"VCOLOR"`（8bit，方向误差约 0.2°） |
| `FRAME` | 骨骼网格 / 要跟随缩放转用 `"TS"`；静态网格不想依赖切线用 `"OS"` |
| `MERGE_SCOPE` | 盔甲 / 配件贴合处不想被外扩分开用 `"GLOBAL"`（需先等比 Apply Scale） |
| `POS_DECIMALS` | 默认 5，模型单位很大调到 3 |

脚本只写额外通道，不改原有法线，因此本体打光不受影响。

## Houdini

1. Triangulate → PolyFrame（Style = Texture UV）生成 `v@tangentu` / `v@tangentv`。
2. Point Wrangle：面积加权平均法线写入 `@sn`，**保持未归一化**（长度 = 一致度）。
3. Vertex Wrangle：按 UV 手性修正副切线，转切线空间写 `uv2=(x,y)`、`uv3=(z,0)`。
4. FBX ROP 导出，`uv2/uv3` 会作为额外 UV 集带出。

Houdini 的 point 天然共享，点法线就是合并后的平均法线 —— 硬边不会裂。

## 3ds Max

`BakeSmoothNormalToUV34.ms` 默认用**角度加权**面法线（尺度稳定），而非面积加权。实测：立方体沿 X 拉伸 3 倍后，面积加权角法线从 `(-0.577,-0.577,-0.577)` 漂移到 `(-0.229,-0.688,-0.688)`，角度加权保持不变。

通道映射：`max ch1→UE UV0`，`ch2→UV1`，`ch3→UV2(3U=Nx/Ny)`，`ch4→UV3(4U=Nz/marker)`。必须保留 ch1 与 ch2，否则 FBX 导出可能压缩通道导致 3/4 位移。

!!! warning "UE FBX 的 V 翻转"
    UE 导入时会做 `V = 1 - V`（只翻 V、不 clamp）。脚本默认预补偿写 `V' = 1 - V`。在 UE 里检查 4UV.v 应约为 1.0；若为 0，说明补偿方向反了。

方向规则：Max 中先 Reset XForm（清除非等比 / 负缩放）；UE 中保持 Actor 等比缩放；材质里只旋转解码法线（把 LocalToWorld 三列归一化去掉缩放），不能直接用 `mul(n, L2W)`。

还内置 Auto / Manual 两种模式：Manual 可把别处已处理好的法线（来自 map channel 的 UVW 等）直接编码进 3U/4U，不重新计算。

## UE4.18 Python（受限工程）

见 [UE4.18 编辑器 Python 自动化](ue4-python-automation.md)。
