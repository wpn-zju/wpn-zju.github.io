---
layout: post
title: "Margin Highlight"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - C#
  - Unity
---

最近几个月在慢慢学习Shader编程，只看书本上的代码不太记得住，打算找一些网上的例子动手实现一下加深印象。

这是一个边缘高亮的效果，原理非常简单，设x是视点到物体表面要渲染的某点的连线，y是该点的法线（需要在同一坐标系中），判断x与y的夹角（利用点乘符号判断），如果x与y垂直或接近垂直，就可以判定该点位于边界上。具体实现在Fragment Shader中进行。(Line 69-74)

```
Shader "TestShader/Margin" {
	Properties{
		_Color("Color Tint", Color) = (1, 1, 1, 1)
		_MarginColor("Margin Color", Color) = (1, 1, 1, 1)
		_MainTex("Main Tex", 2D) = "white" {}
		_AlphaScale("Alpha Scale", Range(0, 1)) = 1
		_Para1("Para 1", Range(0, 0.5)) = 0.3
	}
	SubShader{
		Tags {"Queue" = "Transparent" "IgnoreProjector" = "True" "RenderType" = "Transparent"}

		// Extra pass that renders to depth buffer only
		Pass {
			ZWrite On
			// "ColorMask RGB | A | 0 |"
			// "ColorMask 0" means this pass DO NOT write color buffer
			ColorMask 0
		}
		Pass {
			Tags { "LightMode" = "ForwardBase" }

			ZWrite Off
			Blend SrcAlpha OneMinusSrcAlpha

			CGPROGRAM

			#pragma vertex vert
			#pragma fragment frag

			#include "Lighting.cginc"

			fixed4 _Color;
			fixed4 _MarginColor;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			fixed _AlphaScale;
			fixed _Para1;

			struct a2v {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
				float4 texcoord : TEXCOORD0;
			};

			struct v2f {
				float4 pos : SV_POSITION;
				float3 worldNormal : TEXCOORD0;
				float3 worldPos : TEXCOORD1;
				float2 uv : TEXCOORD2;
			};

			v2f vert(a2v v) {
				v2f o;
				o.pos = UnityObjectToClipPos(v.vertex);
				o.worldNormal = UnityObjectToWorldNormal(v.normal);
				o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
				o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
				return o;
			}

			fixed4 frag(v2f i) : SV_Target {
				fixed3 worldNormal = normalize(i.worldNormal);
				fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
				fixed4 texColor = tex2D(_MainTex, i.uv);
				fixed3 albedo = texColor.rgb * _Color.rgb;
				fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
				fixed3 diffuse = _LightColor0.rgb * albedo;

				// 计算表面法线与视线夹角
				float val = dot(normalize(UnityWorldSpaceViewDir(i.worldPos)), worldNormal);
				if (val < _Para1)
					return fixed4(_MarginColor.rgb, _AlphaScale);
				else
					return fixed4(ambient + diffuse, _AlphaScale);
			}

			ENDCG
		}
	}
	FallBack "Transparent/VertexLit"
}
```

![未开启](/res/notes/MarginHighlight0.png)

![开启](/res/notes/MarginHighlight1.png)

---