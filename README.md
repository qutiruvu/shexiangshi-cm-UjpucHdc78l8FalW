> [【从UnityURP开始探索游戏渲染】](https://github.com)**专栏-直达**

# 发展历史

## ‌**传统可编程管线语义体系**‌早期Unity采用HLSL/CG混合语法，定义了基础语义系统：

* 顶点着色器输入语义：`POSITION`、`NORMAL`、`TEXCOORD0`等用于接收模型数据‌
* 输出语义：`SV_POSITION`传递裁剪空间坐标，`COLOR0`等自定义语义用于插值数据传递‌
* 片元着色器输出语义：`SV_Target`控制颜色缓冲区写入‌

## ‌**URP架构的语义重构**‌2021年推出的URP通过SRP体系重构了语义系统：

* 引入`UNITY_MATRIX_MVP`等宏替代传统矩阵传递方式，优化实例化渲染效率‌
* 标准化顶点着色器输入结构体，强制要求`positionOS`（模型空间坐标）作为基础输入‌
* 新增语义支持屏幕空间坐标转换，如`COMPUTE_SCREEN_POS`宏实现`positionCS`计算‌

## ‌**现代URP语义特性**‌当前URP 12.x版本语义系统呈现以下特征：

* 保留传统语义兼容性（如`TEXCOORD`系列）‌
* 强化系统语义（`SV_Position`用于深度计算，`SV_RenderTargetArrayIndex`支持多目标渲染）‌
* 通过`#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/Common.hlsl"`预定义语义宏，实现跨平台兼容‌

该演进过程体现了从灵活语义定义到标准化渲染接口的转变，同时保持了与旧版管线的兼容性。开发者可通过URP

# 顶点着色器输入数据由应用阶段（如Mesh Renderer）提供，常用语义如下：

| ‌**语义**‌ | ‌**数据类型**‌ | ‌**描述**‌ |
| --- | --- | --- |
| `POSITION` | `float4` | 模型空间顶点坐标‌ |
| `NORMAL` | `float3` | 模型空间法线向量‌ |
| `TANGENT` | `float4` | 模型空间切线向量（`.w`分量存储副切线方向标志）‌ |
| `TEXCOORDn` | `float2/float4` | 顶点纹理坐标（`n=0-7`，如`TEXCOORD0`表示第一组UV）‌ |
| `COLOR` | `fixed4/float4` | 顶点颜色‌ |

## ‌示例代码‌：

```
|  |  |
| --- | --- |
|  | hlsl |
|  | struct appdata { |
|  | float4 vertex : POSITION; |
|  | float3 normal : NORMAL; |
|  | float2 uv : TEXCOORD0; |
|  | }; |
```

# **🔄 ‌顶点着色器输出/片元着色器输入语义‌**

## 顶点着色器向片元着色器传递数据时需定义插值语义：

| ‌**语义**‌ | ‌**数据类型**‌ | ‌**描述**‌ |
| --- | --- | --- |
| `SV_POSITION` | `float4` | ‌**必需语义**‌，输出裁剪空间顶点坐标‌ |
| `TEXCOORDn` | `任意` | 自定义插值数据（如UV、计算中间值），`n=0-7`‌ |
| `COLORn` | `fixed4/float4` | 插值颜色（`n=0-1`）‌ |

## ‌注意事项‌：

* `SV_POSITION` 必须存在于顶点着色器输出结构体中‌；
* `TEXCOORDn` 可自由传递任意数据（如光照向量、自定义参数）‌。

# **🎨 ‌片元着色器输出语义‌**

## 片元着色器最终输出至渲染目标：

| ‌**语义**‌ | ‌**数据类型**‌ | ‌**描述**‌ |
| --- | --- | --- |
| `SV_Target` | `fixed4/float4` | 输出颜色到帧缓存（默认渲染目标）‌ |
| `SV_Targetn` | `任意` | 多渲染目标（MRT）输出（`n=0-7`）‌ |

## ‌示例代码‌：

```
|  |  |
| --- | --- |
|  | hlsl |
|  | struct v2f { |
|  | float4 pos : SV_POSITION; |
|  | float2 uv : TEXCOORD0; |
|  | }; |
|  | fixed4 frag(v2f i) : SV_Target { |
|  | return tex2D(_MainTex, i.uv); |
|  | } |
```

# **️ ‌关键规则与注意事项‌**

## ‌**语义冲突规避**‌：

* 输入/输出结构体中的 `TEXCOORDn` 含义不同：顶点输入时由Unity自动填充纹理坐标，输出时可自定义用途‌；
* 避免在相同结构体重复使用同一 `TEXCOORDn` 索引‌。

## ‌**精度控制**‌：

* 优先用 `fixed`（11位）处理颜色，`half`（16位）处理中等范围数值，减少GPU开销‌。

## ‌**跨管线兼容**‌：

* URP中 `POSITION` 与 `SV_POSITION` 需严格区分：前者为输入模型坐标，后者为输出裁剪坐标‌；
* 内置管线 `POSITION` 在片元输入中已被弃用，需替换为 `SV_POSITION`

---

> [【从UnityURP开始探索游戏渲染】](https://github.com):[蓝猫加速器官网](https://lanmaovqn.com/)**专栏-直达**

（欢迎*点赞留言*探讨，更多人加入进来能更加完善这个探索的过程，🙏）
