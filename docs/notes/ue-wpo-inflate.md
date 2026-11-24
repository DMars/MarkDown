# WPO 外扩与平滑法线（不破面方案）

沿顶点法线把模型整体外扩，用于描边壳体、膨胀与变形过渡。核心矛盾：**硬边 / UV 接缝 / 分离面处同一位置存在多个法线副本，直接外扩必然撕开网格**。

## 三条路线

| 方案 | 做法 | 不裂 | 蒙皮 | 成本 | 适用 |
| --- | --- | --- | --- | --- | --- |
| A 全软边副本 | DCC 中复制模型并全部设软边，直接用 `VertexNormalWS` | 是 | 是 | 多一份网格 | 描边 / 外壳，最快落地 |
| B 烘焙平滑法线（推荐） | 按 3D 位置合并求平均写入 UV / 顶点色，Custom 读取 | 是 | 是 | 需烘焙一次 | 单网格、兼顾打光与外扩 |
| C 纯运行时 | 只用 `VertexNormalWS` | 否（硬边必裂） | 是 | 0 | 快速验证 / 全软边模型 |

烘焙脚本**只写额外通道，不改动原有法线**，模型本体硬边打光完全保留。

## 烘焙数据约定

- 3UV（通道索引 2）存 `N.xy`，4UV（索引 3）存 `N.z, 1.0`；`v=1` 是「已烘焙」标记。
- 顶点色模式：`N * 0.5 + 0.5` 编码；若 FBX 导出选 sRGB，材质里需 `pow(rgb, 2.2) * 2 - 1`。
- 坐标系：骨骼网格 / 需跟随缩放旋转选切线空间 `TS`；静态网格且不想依赖切线选物体空间 `OS`。
- 按位置合并是关键：同一空间位置的所有 wedge 写入同一个方向，外扩后重新对齐。

## Custom 节点接线

1. 新建 Custom，`Output Type = CMOT Float3`。
2. 整段粘贴 `WPO_Inflate_CustomNode.hlsl`，四选一数据源：
   - `SRC_BAKED_UV`（默认，float32 全精度）
   - `SRC_BAKED_VCOLOR`（省 UV，8bit 足够）
   - `SRC_BAKED_OBJSPACE`（不需切线，但不能蒙皮）
   - `SRC_RAW_NORMAL`（仅兜底）
3. 三个 Input 都要接：`Scale`（世界单位 cm）、`CurveFadePower`（凹角收窄，建议 2~4）、`VertexNormalWS`。
4. 输出接 World Position Offset。`CURVE_FADE` 用烘焙向量长度作为一致度，凹角自动收窄防自交。

## 验证与常见坑

```hlsl
// 临时把 N 输出到 BaseColor 检查连续性
BaseColor = N * 0.5 + 0.5;
```

同平面颜色应均匀，硬边两侧应连续；然后把 Scale 拉到 20cm 绕模型检查开裂与自交。

!!! warning "注意事项"
    - **Nanite**：忽略 WPO，使用该材质的网格必须关闭 Nanite。
    - **TSR / TAA 拖影**：外扩量随时间变化要勾 Output Velocity。
    - **阴影 / 距离场**：动态阴影会应用 WPO，Lumen / DF 用原始几何，外扩过大可能漏光。
    - **碰撞**：WPO 只改渲染，命中判定仍是原网格。
    - **LOD / Foliage**：各 LOD 都要烘焙；注意 `World Position Offset Disable Distance`。
    - **方向变换**：用 `normalize(L2W[0..2])` 构造纯旋转矩阵，不能直接 `mul(N, L2W)`，否则非等比缩放会拉歪方向，负缩放会使壳体向内长。

## 关联工具

- [平滑法线烘焙工具链](../tools/dcc-bake-toolchain.md)：Blender / Houdini / 3ds Max / UE4.18 Python 四种实现。
- [工作信息总览](work-overview.md)：全部源文件清单。
