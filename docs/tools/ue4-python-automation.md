# UE4.18 编辑器 Python 自动化

记录在一套 **UE4.18.1「Content + 预编译 DLL」检出工程**中做材质 / 网格自动化的可行路径与实测约束。该工程没有 `Survive/Source` 与引擎 Source，无法编译 C++ Commandlet。

## 实测环境结论（2026-11-10）

在运行中的编辑器里做完整反射扫描：Python 3.12，`dir(unreal)` 共 15142 项、171 个 `*Library` 类。

- **没有任何 UV 写入接口**，只有 `set_vertex_color_static_mesh`（顶点色）和 `reimport_mesh_lod` / `import_asset_tasks`（FBX 重导）。
- `KismetProceduralMeshLibrary.get_section_from_static_mesh` 只能读网格源数据；`FMeshDescription` / `FStaticMeshOperations` 是 4.22+ API，4.18 不存在，源数据走 `FRawMesh`。
- 手写 ASCII FBX 重导**会破坏资产**：顶点被重新焊接 524→410、Y 轴翻转、UV0 错乱。因此脚本不自动重导，只产出中间数据。

!!! warning "操作纪律"
    批量改资产有破坏性：先 p4 备份、只在测试目录验证；首次运行保持 `APPLY_WRITE=False` 做 dry-run。

## 平滑法线烘焙脚本的三层策略

`bake_smooth_normal_uv.py` 在任何环境下都有确定产出：

| 层 | 策略 | 产出 |
| --- | --- | --- |
| A | 运行时探测 `dir(unreal)`，若自研插件 `UVToolEditor` / `VertexMeshColor` 暴露写入接口则自动调用 | 写回 3UV/4UV（需 `APPLY_WRITE=True`） |
| B | 无论 A 是否成功，导出逐 wedge 数据 | `Saved/SmoothNormalBake/data/` 下 JSON + CSV，可喂给 DCC / C++ 扩展 |
| C | 导出一份 UE4.18 `FRawMesh` C++ 写入实现 | `SmoothNormalBake_RawMesh.cpp.txt`，最可靠无损写回路径 |

解码材质始终生成，不受 `APPLY_WRITE` 影响：`M_SmoothNormal_Decode`（Emissive 显示法线）与 `M_SmoothNormal_Outline`（Inverted Hull 外壳）。

编码约定：`3UV=(Nx,Ny)`、`4UV=(Nz,1.0)`；材质解码时只取 `LocalToWorld` 的旋转部分，避免非等比缩放拉歪方向。骨骼网格的 WPO 在蒙皮前局部空间生效，不要做变换。

运行方式（编辑器 Output Log）：

```text
py "d:/PUBGM_Main/trunk/Survive/Content/Python/bake_smooth_normal_uv.py"
```

只重建材质不碰模型：`b.create_materials()`。

## 材质生成脚本

`Make_UVChannelMask_Material.py` 通过 `MaterialEditingLibrary` 创建 `.uasset`（二进制资产无法用文本直接生成）：

- Custom 的 Code / Output Type / 13 个 Input 全部设好，UV 接 `TextureCoordinate`，其余 12 个接带默认值的 `ScalarParameter`。
- 输出接 Emissive Color，材质设 Unlit，避免 Tonemapper 改数值。
- 同时把 HLSL 源码写到同一内容目录，便于查阅。
- 可重复运行：资产已存在时先清空旧表达式再重建。

!!! note "项目适配提示"
    本项目是 UE4.18 移动端工程，扫动时间统一用材质函数 `MF_yaTime`（`/Game/Arts/Common/MatFunctions/`），不接引擎原生 Time 节点。目标材质 `M_UVChannelMask` 已由项目专用脚本完成落地（13 引脚全接 + yaTime 扫动开关），**通用脚本勿再执行覆盖**。

## 相关生成器

- `create_yellow_carousel_material.py` / `_custom.py`：黄色轮播自发光材质，见 [UV 驱动轮播自发光](../notes/ue-carousel-emissive.md)。

## 运行方式汇总

| 方式 | 命令 |
| --- | --- |
| 编辑器菜单 | Tools / File → Execute Python Script… 选择文件 |
| Output Log（Python 模式） | `exec(open(r'路径', encoding='utf-8').read())` |
| 命令行 | `UE4Editor-Cmd.exe <Project>.uproject -ExecutePythonScript="路径" -unattended` |

前提是启用 **Python Editor Script Plugin**。
