# 螺旋 WPO 与随机强度

`WPO_SpiralNormal.hlsl`：基于顶点法线的螺旋波纹 + 空间连续随机强度场，全部在**模型 / 局部空间**计算，Custom 节点 `Output Type = CMOT Float3`。

## 输入引脚（顺序与名字必须一致）

| 输入 | 类型 | 说明 / 默认 |
| --- | --- | --- |
| `LocalPos` | float3 | AbsoluteWorldPosition → TransformPosition(World→Local) |
| `LocalN` | float3 | VertexNormalWS → TransformVector(World→Local) → Normalize |
| `Axis` | float3 | 局部螺旋轴，默认 (0,0,1) |
| `Pivot` | float3 | 螺旋中心，默认 (0,0,0) |
| `Time` | float | Time 节点 |
| `Speed` | float | 相位推进速度，默认 2.0 |
| `Arms` | float | 螺旋臂数，整数最易闭合，默认 2.0 |
| `HeightFreq` | float | 沿轴波纹频率（每局部单位），默认 0.5 |
| `Amplitude` | float | 沿法线起伏幅度（局部单位），默认 10.0 |
| `Twist` | float | 切向扭转强度，默认 0.15 |
| `Mask` | float | 逐顶点遮罩 0~1，不接给 1 |
| `Seed` | float | 随机种子，默认 0 |
| `RandomScale` | float | 随机块大小倒数，默认 0.05 |
| `RandomAmount` | float | 强度随机幅度 0~1，默认 0.6 |
| `RandomSpeed` | float | 随机场流动速度，默认 0.3 |
| `InstanceRandom` | float | PerInstanceRandom；非实例化给 0 |
| `ObjRandom` | float | 每物体整体强度随机，默认 0 |

## 算法要点

- 把相对 Pivot 的向量 `d` 分解为沿轴高度 `h` 与径向 `rad`；构造垂直于轴的正交基 `U, V`。
- 等相位线 `phase = ang * Arms + h * HeightFreq - Time * Speed`，`wave = sin(phase)`。
- 法线方向起伏 `nOff = LocalN * wave * Amplitude`。
- 切向扭转 `tOff = tangent * wave * Twist * r`，乘 `r` 使角位移均匀、**轴心不撕裂**。
- 随机强度用局部空间的 3D value noise（格点哈希 + smoothstep 三线性插值），必须是空间连续的低频块；逐顶点独立随机会把三角面撕碎。

## 接线

Custom 输出的是**局部空间**位移，必须再接 `TransformVector(Local Space → World Space)` 才能接 World Position Offset。

!!! warning "注意事项"
    - 每个 Input 都要在 Details → Inputs 中建立并**全部接线**，未连线的输入在部分版本会被跳过声明，触发 `X3004: undeclared identifier`。
    - `Speed` 是 `RandomSpeed` 的子串；若出现相关怪错，把 `Speed` 改名为 `PhaseSpeed`（代码与 Inputs 同步改）规避。
