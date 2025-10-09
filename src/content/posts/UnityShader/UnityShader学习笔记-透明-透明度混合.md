---
title: UnityShader学习笔记-透明-透明度混合
published: 2025-08-10
description: ''
image: ''
tags: [Unity, Shader]
category: UnityShader
draft: false 
lang: ''
---
透明度混合（Alpha Blending）是一种常见的透明效果，它可以实现物体真正的**半透明效果**，即部分透明、部分不透明。
与透明度测试不同，透明度混合不会直接舍弃片元，而是使用混合函数（Blend）将当前片元的颜色与颜色缓冲中的已有颜色按比例合成。其原理是：**根据当前片元的透明度（Alpha 值），与已经存储在颜色缓冲中的颜色值进行混合，并在混合前先关闭深度写入（ZWrite Off）。**混合的计算方式可以用下式表示：

$$
C_{out} = C_{src} * \alpha_{src} + C_{dst} * (1 - \alpha_{src})
$$

其中：

* 当前片元颜色：$C_{src}$
* 当前片元透明度：$\alpha_{src}$
* 颜色缓冲中的已有颜色：$C_{dst}$

---

## 透明度混合着色器实现

在Unity中，通常使用 `Blend`命令实现来进行透明度混合。在Unity提供的ShaderLab中提供了四种混合命令的语义，如下表所示：

| 语义                                             | 定义                                            |
| ------------------------------------------------ | ----------------------------------------------- |
| Blend Off                                        | 关闭混合                                        |
| Blend SrcFactor DstFactor                        | 开启混合，统一设置指定颜色的源/目标混合因子     |
| Blend SrcFactor DstFactor, SrcFactorA DstFactorA | 开启混合，分别设置并指定颜色的源/目标的混合因子 |
| BlendOp BlendOperation                           | 开启混合，指定混合运算方式（加法、减法等）      |

### 属性

```hlsl
_Color ("Main Tint", Color) = (1, 1, 1, 1)
_MainTex ("主纹理", 2D) = "white" {}
_AlphaScale ("Alpha Scale", Range(0, 1)) = 1
```

在着色器中定义了纹理属性：颜色、主贴图、透明度缩放

* 颜色：用于描述物体自身颜色
* 主纹理：用于描述物体的漫反射颜色的纹理图片，默认是一个纯白的纹理
* **透明度缩放**：整体控制纹理像素的透明度

### SubShader语义中的标签（Tags）设置

```hlsl
Tags 
{
    "Queue"="Transparent"
    "IgnoreProjector"="True"
    "RenderType"="Transparent"
}
```

与透明度测试设置的标签不同，在透明度混合中，我们定义了3个标签：

* `"Queue" = "Transparent"`：放到透明队列中绘制（即在所有不透明物体之后渲染）
* `"IgnoreProjector" = "True"`：不受 Projector 投影影响
* `"RenderType" = "Transparent"`：标记当前Shader渲染出来的物体属于**透明物体类别**

### Pass中透明度混合设置

```hlsl
//关闭深度写入
ZWrite Off
//开启混合模式
Blend SrcAlpha OneMinusSrcAlpha
```

因为透明度混合需要关闭深度写入，因此，透明度混合的Pass中需要添加 `ZWrite Off`，并同时开启Unity的混合模式 `Blend SrcAlpha OneMinusSrcAlpha`，在这里选用了 `OneMinusSrcAlpha`，即最经典的半透明混合公式，也就是取$1 - \alpha_{src}$作为目标混合因子来参与透明度混合。

### 着色器变量

```hlsl
fixed4 _Color;
sampler2D _MainTex;
float4 _MainTex_ST;
fixed _AlphaScale;
```

* 使用 `sampler2D` 表示纹理资源
* 使用 `float4 _MainTex_ST` 表示纹理属性
* 使用 `fixed _AlphaScale` 表示透明度缩放

### 透明度混合

```hlsl
struct a2v
{
    float4 vertex : POSITION;
    float3 normal : NORMAL;
    float4 texcoord : TEXCOORD0;
};

struct v2f
{
    float4 pos : SV_POSITION;
    float3 worldNormal : TEXCOORD0;
    float3 worldPos : TEXCOORD1;
    float2 uv : TEXCOORD2;
};

 v2f vert(a2v v)
{
    v2f o;
    //将顶点从模型空间坐标转成投影空间坐标
    o.pos = UnityObjectToClipPos(v.vertex);
    o.worldNormal = UnityObjectToWorldNormal(v.normal);
    o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
    o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);

    return o;
}

fixed4 frag(v2f i) : SV_Target
{
    fixed3 worldNormal = normalize(i.worldNormal);
    fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

    fixed4 texColor = tex2D(_MainTex, i.uv);
    fixed3 albedo = texColor.rgb * _Color.rgb;
    fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
    fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
    return fixed4(ambient + diffuse, texColor.a * _AlphaScale);
}
```

### 效果

<!-- markdownlint-disable-next-line MD033 -->
<iframe width="100%" height="468" src="/videos/UnityShader/AlphaBlend.mp4" title="透明度混合" frameborder="0" allowfullscreen></iframe>

从视频中可以看到，随着 AlphaScale 数值的变化，物体的透明度会平滑变化：

* Alpha = 1（完全不透明）时，新片元完全覆盖背景
* Alpha = 0（完全透明）时，保留背景颜色，不显示新片元
* 0 < Alpha < 1 时，新片元与背景颜色按比例叠加，呈现出半透明效果

这种平滑混合方式，与透明度测试的效果不同，能更好地表现玻璃、水面等材质的真实质感。

---

## Reference

* 《UnityShader入门精要》
* [图形学|shader|用一篇文章理解半透明渲染、透明度测试和混合、提前深度测试并彻底理清渲染顺序。 - aganztracy的文章 - 知乎](https://zhuanlan.zhihu.com/p/263566318)
* [透明度测试(AlphaTest)和混合(Blend)的基础知识](https://blog.csdn.net/qqrrjj2011/article/details/137672153)
* [Real-Time Rendering 4th → Chapter5 Shading Basics 着色基础 → 5.5 透明度，Alpha，合成](https://www.wolai.com/aaGGQwtM2CcQYz2C3gUdPS)
