---
title: UnityShader学习笔记-透明-透明度测试
published: 2025-08-09
description: ''
image: ''
tags: [Unity, Shader]
category: UnityShader
draft: false 
lang: ''
---
在实现透明效果的多种方法中，透明度测试（Alpha Test）是一种经典且高效的技术，尤其适用于需要硬边透明的场景，例如树叶、栅栏、铁丝网等模型。

透明度测试的原理非常简单，即在片元着色阶段，根据该片元的 **透明度（Alpha 值）** 与一个设定阈值进行比较：

* 若Alpha值**小于**阈值，则认为该片元完全不可见，直接舍弃（discard），不会参与后续的颜色混合或深度写入
* 若Alpha值**大于等于**阈值，则将该片元当作不透明物体进行正常渲染，包括写入颜色缓冲和深度缓冲

由于逻辑简洁，透明度测试在许多实时渲染场景中依然常用。但它同时具备一些显著的优缺点：

* **优点：** 实现简单、渲染性能较高（被舍弃的片元不会再走后续的混合与写入流程）
* **缺点：** 透明边界是硬切割，无法表现半透明的渐变效果（即只能实现完全透明或者完全不透明）

---

## 透明度测试着色器实现

在Unity中，通常会使用 `clip`函数或 `discard`语句实现来进行透明度测试，如：

```hlsl
fixed4 texColor = tex2D(_MainTex, i.uv);

//透明度测试: clip函数
clip(texColor.a - _Cutoff);

//透明度测试: dicard语句
if (texColor.a - _Cutoff < 0.0)
{
    discard;
}
```

### 属性

```hlsl
_Color ("颜色", Color) = (1,1,1,1)
_MainTex ("主贴图", 2D) = "white" {}
_Cutoff ("AlphaCutoff", Range(0,1)) = 0.5
```

在着色器中定义了纹理属性：颜色、主贴图、透明度阈值

* 颜色：用于描述物体自身颜色
* 主纹理：用于描述物体的漫反射颜色的纹理图片，默认是一个纯白的纹理
* **透明度阈值**：用于控制纹理像素的透明度

### SubShader语义中的标签（Tags）设置

```hlsl
Tags
{
    "Queue"="AlphaTest"
    "IgnoreProjector"="True"
    "RenderType"="TransparentCutout"
}
```

不同于之前基础的光照和贴图着色器，在透明度测试着色器中，我们定义了3个标签：

* **Queue**：指定渲染队列，这里指定为AlphaTest，表示物体的透明度测试，即物体的透明度会受主纹理的透明度影响，当物体的透明度小于阈值时，物体将不会被渲染。
* **IgnoreProjector**：指定是否忽略投影器，这里指定为True，表示物体将不会被投影器影响。
* **RenderType**：指定渲染类型，这里指定为TransparentCutout，表示使用**透明裁剪**，即物体的透明度会受主纹理的透明度影响，当物体的透明度小于阈值时，物体将不会被渲染。

### 着色器变量

```hlsl
fixed4 _Color;
sampler2D _MainTex;
float4 _MainTex_ST;
fixed _Cutoff;
```

* 使用 `sampler2D` 表示纹理资源
* 使用 `float4 _MainTex_ST` 表示纹理属性
* 使用 `fixed _Cutoff` 表示透明度阈值

### 透明度测试

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
    //透明度测试
    if (texColor.a - _Cutoff < 0.0)
    {
        discard;
    }

    fixed3 albedo = texColor.rgb * _Color.rgb;
    fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
    fixed3 diffuse = _LightColor0.rgb * albedo * max(0, dot(worldNormal, worldLightDir));
    return fixed4(ambient + diffuse, 1.0);
}
```

### 效果

<!-- markdownlint-disable-next-line MD033 -->
<iframe width="100%" height="468" src="/videos/UnityShader/AlphaTest.mp4" title="透明度测试" frameborder="0" allowfullscreen></iframe>

在视频中，可以看到 AlphaCutoff 数值变化时：

* 透明度**小于**阈值的像素 → 被剔除，呈现完全透明
* 透明度**大于或等于**阈值的像素 → 保留并正常渲染，呈现完全不透明

因此，画面表现为——要么完全透明，要么完全不透明的效果，没有中间的半透明过渡。

---

## Reference

* 《UnityShader入门精要》
* [图形学|shader|用一篇文章理解半透明渲染、透明度测试和混合、提前深度测试并彻底理清渲染顺序。 - aganztracy的文章 - 知乎](https://zhuanlan.zhihu.com/p/263566318)
* [透明度测试(AlphaTest)和混合(Blend)的基础知识](https://blog.csdn.net/qqrrjj2011/article/details/137672153)
