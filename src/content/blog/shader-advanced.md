---
title: Shader开发（下）
---

> 本文整理自图形学 / Unity Shader 学习笔记，对应第 3 章「光照 / 纹理 / 透明 / 渲染路径 / 多光源 / 阴影 / 标准光照着色器 / 高级纹理」的优化整理，承接《Shader 开发（上）》的必备基础（渲染管线、线性代数、ShaderLab 与 CG 语法）。


## 目录

- [第 3 章 Shader 开发知识](#第-3-章-shader-开发知识)
  - [第 1 节 光照](#第-1-节-光照)
    - [光照模型概述](#光照模型概述)
    - [逐顶点光照和逐片元光照](#逐顶点光照和逐片元光照)
    - [兰伯特光照模型](#兰伯特光照模型)
    - [半兰伯特光照模型](#半兰伯特光照模型)
    - [Phong式高光反射模型](#phong式高光反射模型)
    - [Phong光照模型](#phong光照模型)
    - [Blinn Phong式高光反射光照模型](#blinn-phong式高光反射光照模型)
    - [Blinn Phong光照模型](#blinn-phong光照模型)
    - [为什么逐片元比逐顶点平滑](#为什么逐片元比逐顶点平滑)
    - [Unity内置光照计算相关函数](#unity内置光照计算相关函数)
  - [第 2 节 纹理](#第-2-节-纹理)
    - [纹理概念](#纹理概念)
    - [纹理颜色采样](#纹理颜色采样)
    - [纹理基础知识](#纹理基础知识)
    - [纹理相关设置](#纹理相关设置)
    - [纹理结合光照模型](#纹理结合光照模型)
    - [凹凸纹理基本概念](#凹凸纹理基本概念)
    - [高度纹理贴图概述](#高度纹理贴图概述)
    - [法线纹理贴图概述](#法线纹理贴图概述)
    - [两种法线纹理贴图的存储方式](#两种法线纹理贴图的存储方式)
    - [切线空间法线贴图的计算方式](#切线空间法线贴图的计算方式)
    - [渐变纹理基本概念](#渐变纹理基本概念)
    - [渐变纹理基础实现](#渐变纹理基础实现)
    - [渐变纹理综合实现](#渐变纹理综合实现)
    - [遮罩纹理基本概念](#遮罩纹理基本概念)
    - [高光遮罩纹理综合实现](#高光遮罩纹理综合实现)
    - [遮罩纹理中的RGBA值](#遮罩纹理中的rgba值)
  - [第 3 节 透明](#第-3-节-透明)
    - [透明必备知识点—渲染顺序](#透明必备知识点渲染顺序)
    - [透明必备知识点—设置深度写入和渲染队列](#透明必备知识点设置深度写入和渲染队列)
    - [透明必备知识点—设置混合命令](#透明必备知识点设置混合命令)
    - [透明度测试](#透明度测试)
    - [透明度混合](#透明度混合)
    - [开启深度写入的半透明效果](#开启深度写入的半透明效果)
    - [双面渲染的透明效果](#双面渲染的透明效果)
  - [第 4 节 渲染路径](#第-4-节-渲染路径)
  - [第 5 节 多种光源处理](#第-5-节-多种光源处理)
  - [第 6 节 阴影](#第-6-节-阴影)
  - [第 7 节 标准光照着色器](#第-7-节-标准光照着色器)
  - [第 8 节 高级纹理](#第-8-节-高级纹理)

---

# 第 3 章 Shader 开发知识

## 第 1 节 光照


### 光照模型概述

**承上启下**  
我们之前已经完成了 Unity Shader 相关必备基础知识的学习其中重要的知识有

1.渲染管线相关（应用阶段→几何阶段→光栅化阶段）

2.数学基础相关（点、向量、矩阵、坐标系变换）

3.语法基础相关（ShaderLab语法、CG语法）  
通过这些知识的学习我们基本上能够大致了解了应该如何进行Shader开发简而言之：我们可以通过CG等语言结合ShaderLab语法规则来编写Shader文件我们需要在顶点、片元着色器的回调函数中具体实现着色器逻辑在顶点着色器回调函数中我们利用语义获取到模型中的数据将数据处理完毕后又传递给片元着色器回调函数中进行处理还可以结合ShaderLab属性来进行更多的可变处理从而呈现出不同的表现效果那么接下来我们要学习的Shader入门相关知识主要就是学习Shader的一些基本表现效果他们分别是

1.光照效果

2.纹理效果

3.透明效果  
学习了这些知识，我们才能实现最基础的渲染效果

**知识点一 什么是光照模型**  
光照模型指的是用于模拟光照效果的一组数学公式和算法用于确定在3D场景中的模型表面应该如何对光进行反射和散射从而实现视觉上逼真的照明效果说人话：光照模型就是用来计算光照效果的数学公式是业界前辈们探索出来的计算规则

**知识点二 我们将要学习哪些内容**  
在现实生活中，一个物体的颜色是由射进眼睛里的光线决定的当光线照射到物体表面时，一部分被物体表面吸收，一部分被反射。对于透明物体，还有一部分光会穿过透明体，产生透射光只有反射光和透射光才能进入眼镜，从而产生视觉效果（物体呈现出的亮度和颜色）因此，物体表面的光照颜色是由入射光、物体材质、以及材质和光的交互规律共同决定的

1.漫反射光照模型  
漫反射：指光线被粗糙表面无规则地向各个方向反射的现象是一种在各个方向上平均反射光线的模型比如：墙壁、纸张、布料等等物体的表现效果

2.高光反射光照模型  
高光反射：考虑了视角和光源的方向，使物体表面出现高光点比如：金属、陶瓷、塑料等等物体的表现效果

3.Unity内置光照函数  
等等

**总结**  
之后我们将要学习的光照模型就是前辈们探索总结出来的计算公式通过这些光照计算公式的学习我们可以让3D物体表现出受光照影响的效果我们之后学习的大部分Shader相关的知识其实本质上都是一些计算公式、计算规则（算法）Shader的本质其实就是利用各种数学计算呈现出不同的表现效果

### 逐顶点光照和逐片元光照

**知识回顾 光照模型**  
光照模型就是用来计算光照效果的数学公式是业界前辈们探索出来的计算规则

**提出问题**  
具体的光照效果相关的计算应该写在顶点还是片元着色器中呢？本节课学习的必备知识就是用来回答该问题的

**知识点一 逐顶点光照**  
在哪计算：顶点着色器 回调函数中计算方式：逐顶点光照会在每个物体的顶点上进行光照计算这意味着光照计算只在物体的顶点位置上执行而在顶点之间的内部区域使用插值来获得颜色信息优点：逐顶点光照的计算量较小，通常在移动设备上性能较好，适用于移动游戏等要求性能的场景缺点:照明效果可能不够精细，特别是在物体表面上的细节区域，因为颜色插值可能不足以捕捉到细微的照明变化适用场景：逐顶点光照适用于需要在有限资源下获得较好性能的场景，例如移动游戏

**知识点二 逐片元光照**  
在哪计算：片元着色器 回调函数中计算方式：逐片元光照会在每个像素（片元）上进行光照计算这意味着每个像素都会根据其位置、法线、材质等信息独立地进行光照计算优点：逐片元光照提供了更高的精细度，可以捕捉到物体表面上的细微照明变化，提供更逼真的效果缺点:计算量较大，对于像素密集的场景需要更多的计算资源适用场景：逐片元光照通常用于需要高质量照明效果的PC和主机游戏，以及要求视觉逼真度较高的场景

**知识点三 关于逐顶点光照的插值运算**  
我们不需要自己去处理这个插值运算插值运算是由图形硬件（GPU）来执行的GPU负责处理3D图形的渲染，包括顶点插值和像素插值等操作这个过程在图形硬件中被高度优化过，因此在实时渲染中能够快速而高效地执行我们只需要了解插值运算的大概规则即可假设：三角面片的三个顶点A、B、C该三角面片中的任何像素P，首先会计算出它相对于3个顶点的位置权重然后使用这个权重参与到P点的颜色计算中PixelColorP = WeightA ColorA + WeightB ColorB + WeightC * ColorC

**总结**  
光照效果的计算在顶点着色器和片元着色器中都可以做在顶点着色器回调函数中：消耗低，效果差，适合低配设备在片元着色器回调函数中：消耗高，效果好，适合高配设备具体在哪里实现，根据项目实际情况而定我们之后在学习光照模型时每一种光照模型都会讲解两种实现即 逐顶点 和 逐片元 光照


### 兰伯特光照模型

**知识点一 两个颜色相乘**  
两个颜色变量相乘，通常用于计算光照、材质混合、纹理混合等等需求它是一种用于混合颜色的操作计算结果可以理解为两种颜色叠加在一起的效果在颜色相乘中，通常是使用颜色通道的值来执行逐通道的乘法举例：颜色A、B都是 fixed4 类型的变量颜色A 颜色B = (A.r B.r, A.g B.g, A.b B.b, A.a * B.a);那么得到的这个结果就是A、B颜色叠加后的结果

**总结：**  
两个颜色变量相乘，得到的结果表示两个颜色叠加在一起

**知识点二 漫反射基本概念**  
漫反射（Diffuse Reflection）是光线撞击一个物体表面后以各个方向均匀地反射出去的过程在漫反射下，光线以无规律的方式散射，而不像镜面反射那样按照特定的角度反射。  
这种散射导致了物体表面看起来均匀而不闪烁的效果。

**知识点三 兰伯特光照模型的来历和原理**  
兰伯特(Lambert)光照模型，也称为朗伯反射模型是由瑞士数学家约翰·海因里希·朗伯（Johann Heinrich Lambert）于1760年左右首次提出朗伯是18世纪的一个杰出科学家，他在光学和数学领域作出了众多贡献。  
兰伯特光照模型描述了漫反射表面对光线的反射行为，它成为计算机图形学和渲染中重要的基础模型之一原理：兰伯特光照模型的理论是认为漫反射光的强度仅与入射光的方向和反射点处表面法线的夹角的余弦成正比

**知识点四 兰伯特光照模型的公式**  
公式：漫反射光照颜色 = 光源的颜色 材质的漫反射颜色 max（0, 标准化后物体表面法线向量· 标准化后光源方向向量）其中：

1.标准化后物体表面法线向量· 标准化后光源方向向量 得到的结果就是 cosθ

2.max(0,cosθ)的目的是避免负数，对于模型背面的部分，认为找不到光，直接乘以0，变为黑色

**知识点五 如何在Shader中获取公式中的关键信息**

1.光源的颜色  
Lighting.cginc 内置文件中的 \_LightColor0

2.光源的方向  
_WorldSpaceLightPos0 表示光源0在世界坐标系下的位置

3.向量归一化方法  
normalize

4.取最大值方法  
max

5.点乘方法  
dot

6.兰伯特光照模型环境光变量（用于模拟环境光对物体的影响，避免物体阴影部分完全黑暗）  
UNITY_LIGHTMODEL_AMBIENT.rgb

7.将法线从模型空间转换到世界空间  
UnityObjectToWorldNormal

**顶点着色器实现**

**关键步骤**

- 材质漫反射颜色属性声明
- 渲染标签Tags设置 将LightMode光照模式 设置为ForwardBase向前渲染（通常用于不透明物体的基本渲染）
- 引用内置文件UnityCG.cginc和Lighting.cginc
- 结构体声明
- 基于公式实现逻辑
- 注意：为了阴影出不全黑，需要加上兰伯特环境光颜色公共变量

**代码**

```hlsl
Shader "Unlit/Lesson29_Lambert"
{
    Properties
    {
        //材质的漫反射光照颜色
        _MainColor("MainColor", Color) = (1,1,1,1)
    }
    SubShader
    {
        //设置我们的光照模式 ForwardBase这种向前渲染模式 主要是用来处理 不透明物体的 光照渲染的
        Tags { "LightMode"="ForwardBase" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //引用对应的内置文件 
            //主要是为了之后 的 比如内置结构体使用，内置变量使用
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //材质的漫反射颜色
            fixed4 _MainColor;

            //顶点着色器传递给片元着色器的内容
            struct v2f
            {
                //裁剪空间下的顶点坐标信息
                float4 pos:SV_POSITION;
                //对应顶点的漫反射光照颜色
                fixed3 color:COLOR;
            };

            //逐顶点光照 所以相关的漫反射光照颜色的计算 需要写在顶点着色器 回调函数中
            v2f vert (appdata_base v)
            {
                v2f v2fData;
                //把模型空间下的 顶点转换到裁剪空间下
                v2fData.pos = UnityObjectToClipPos(v.vertex);

                //在模型空间下的法线 
                //v.normal
                //获取到 相对于世界坐标系下的 法线信息
                float3 normal = UnityObjectToWorldNormal(v.normal);
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);

                //有了相关的参数 就可以用公式来进行计算了
                //光照颜色
                //_LightColor0
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(normal, lightDir));

                //记录颜色 传递给片元着色器
                //我们加上兰伯特光照模型环境光变量的目的 是希望阴影处不要全黑 不然看起来有一些不自然
                //目的就是为了让表现效果更接近于真实世界 所以需要加上它
                v2fData.color = UNITY_LIGHTMODEL_AMBIENT.rgb + color;

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //把计算好的兰伯特光照的颜色 传递出去就可以了
                return fixed4(i.color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

**片元着色器实现**

**关键步骤**

- 基本和逐顶点一致
- 区别：在顶点着色器中计算顶点和法线，在片元着色器中计算兰伯特光照

**代码**

```hlsl
Shader "Unlit/Lesson30_LambertF"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
    }
    SubShader
    {
        Tags { "LightMode"="ForwardBase" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 normal:NORMAL;
            };
            
            //一定注意 顶点着色器回调函数 主要就是用于处理顶点、法线、切线等数据的坐标转换
            v2f vert (appdata_base v)
            {
               v2f v2fData;
               //转换模型空间下的顶点到裁剪空间中
               v2fData.pos = UnityObjectToClipPos(v.vertex);
               //转换模型空间下的法线到世界空间下
               v2fData.normal = UnityObjectToWorldNormal(v.normal);

               return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //得到光源单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //计算除了兰伯特光照的漫反射颜色
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(i.normal, lightDir));
                //为了让背光的地方不至于是黑色 所以加上自带的漫反射颜色 看起来更加真实
                color = UNITY_LIGHTMODEL_AMBIENT.rgb + color;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

最终效果图远处为顶点处理，近处为片元处理，下面的效果图如无特殊说明皆为如此。罗伯特的球体外貌是比较固定的，因为对于外来变量，它只受到光源的影响。

**总结**  
兰伯特光照模型漫反射光颜色 = 光源的颜色 材质的漫反射颜色 max（0, 标准化后物体表面法线向量· 标准化后光源方向向量）


### 半兰伯特光照模型

**知识点一 单位向量点乘的结果范围**  
根据数学公式我们知道假设单位向量A和单位向量B，它们的点乘结果为A·B = |A| |B| cosθ = cosθcosθ 的范围在 - 1 到 1 之间（夹角为0度时,点乘结果为1;当夹角为180度时,点乘结果为-1）因此单位向量进行点乘时它们的结果是 -1 ~ 1关于兰伯特光照公式漫反射光照颜色 = 光源的颜色 材质的漫反射颜色 max（0, 标准化后物体表面法线向量· 标准化后光源方向向量）

**知识点二 半兰伯特光照模型的来历和原理**  
半兰伯特光照模型是基于 兰伯特光照模型 的基础上进行改进的它没有任何物理依据，只是一个视觉加强技术它出现的主要原因是因为我们在使用兰伯特光照模型时在背光面是全黑的而半兰伯特光照模型可以让背光面也可以有明暗变化半兰伯特光照模型没有特定的发明者它是图形学领域的众多研究人员共同的贡献研究人员们经常相互借鉴和改进现有的模型以更好地模拟真实世界中的光照和材质反射原理和兰伯特光照模型的理论是一样的认为漫反射光的强度仅与入射光的方向和反射点处表面法线的夹角的余弦成正比

**知识点三 半兰伯特光照模型的公式**  
公式：漫反射光照颜色 = 光源的颜色 材质的漫反射颜色 （（标准化后物体表面法线向量· 标准化后光源方向向量）* 0.5 + 0.5）对比：兰伯特光照模型的 后半部分：max（0, 标准化后物体表面法线向量· 标准化后光源方向向量）点乘小于0的部分都会变成0半兰伯特光照模型的 后半部分：（（标准化后物体表面法线向量· 标准化后光源方向向量）* 0.5 + 0.5）点乘小于0的部分都会变成 0 ~ 0.5-1 ~ 1 映射到了 0 ~ 1

**顶点着色器实现**  
介绍半兰伯特光照模型的逐顶点实现和兰伯特一模一样唯一的区别就是公式漫反射光照颜色 = 光源的颜色 材质的漫反射颜色 （（标准化后物体表面法线向量· 标准化后光源方向向量）* 0.5 + 0.5）

**代码**  
仅需要在兰伯特顶点着色器的shader上，修改color变量的点乘相关数学部分即可

```hlsl
fixed3 color = _LightColor0.rgb * _MainColor.rgb * (dot(normal,lightDir) * 0.5 + 0.5);
```

**片元着色器实现**

**代码**  
仅需要在兰伯特片元着色器的shader上，修改color变量的点乘相关数学部分即可

```hlsl
fixed3 color = _LightColor0.rgb * _MainColor.rgb * (dot(i.normal, lightDir) * 0.5 + 0.5);
```

最终效果图由于全局亮度提升，所以暗影部分减少，可以看到的暗处细节更多但是貌似顶点和片元的差别也不是那么明显了。

**总结**  
半兰伯特光照模型 是 基于兰伯特光照模型进行的修改主要目的是让背光面可以有明暗变化漫反射光照颜色 = 光源的颜色 材质的漫反射颜色 （（标准化后物体表面法线向量· 标准化后光源方向向量）* 0.5 + 0.5）


### Phong式高光反射模型

**知识点一 Phong式高光反射光照模型的来历和原理**  
来历:高光反射光照模型没有单一的发明者，因为有多种计算的方式和半兰伯特光照模型一样，是经过很多从业者的研究和发展而演化出来的其中比较关键的几位贡献者为Phong光照模型的提出者：裴祥风（Bui-Tuong Phong，越南裔美国计算机学家）Blinn-Phone光照模型的提出者：吉姆·布林（Jim Blinn，美国计算机科学家）原理:Phong式高光反射光照模型的理论是基于光的反射行为和观察者的位置决定高光反射的表现效果认为高光反射的颜色和 光源的反射光线以及观察者位置方向向量夹角的余弦成正比并且通过对余弦值取n次幂来表示光泽度（或反光度）

**知识点二 Phong式高光反射光照模型的公式**  
公式：高光反射光照颜色 = 光源的颜色 材质高光反射颜色 max（0, 标准化后观察方向向量· 标准化后的反射方向）幂

1.标准化后观察方向向量· 标准化后的反射方向 得到的结果就是 cosθ

2.幂 代表的是光泽度 余弦值取n次幂

**知识点三 如何在Shader中获取公式中的关键信息**

1.观察者的位置（摄像机的位置）  
_WorldSpaceCameraPos

2.相对于法向量的反射向量 方法  
reflect(入射向量,顶点法向量) 返回反射向量



3.指数幂 方法  
pow(底数，指数) 返回计算结果

**顶点着色器实现**

**关键步骤**

- 属性声明（材质高光反射颜色、光泽度）
- 渲染标签Tags设置 将LightMode光照模式 设置为ForwardBase向前渲染（通常用于不透明物体的基本渲染）
- 引用内置文件UnityCG.cginc和Lighting.cginc
- 结构体声明
- 基本公式实现逻辑

**代码**

```hlsl
Shader "Unlit/Lesson35_Specular"
{
    Properties
    {
        //高光反射颜色
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        //光泽度
        _SpecularNum("SpecularNum", Range(0, 20)) = 0.5
    }
    SubShader
    {
        Tags { "LightMode"="ForwardBase" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            struct v2f
            {
                //裁剪空间下的 顶点坐标
                float4 pos:SV_POSITION;
                //颜色信息
                fixed3 color:COLOR;
            };

            //对应属性当中的颜色和光泽度
            fixed4 _SpecularColor;
            float _SpecularNum;

            v2f vert (appdata_base v)
            {
                v2f data;
                //1.将顶点坐标转换到裁剪空间当中
                data.pos = UnityObjectToClipPos(v.vertex);
                //2.计算颜色相关
                //高光反射光照颜色 = 光源的颜色 * 材质高光反射颜色 * max（0, 标准化后观察方向向量· 标准化后的反射方向）幂
                //光源的颜色 _LightColor.rgb
                //材质高光反射颜色 _SpecularColor.rgb
                //幂(光泽度) _SpecularNum

                //1.标准化后观察方向向量
                //将模型空间下的顶点位置 转换到 世界空间下 
                // UNITY_MATRIX_M : 从模型空间下 转到 世界空间下的转换矩阵
                float3 worldPos = mul(UNITY_MATRIX_M, v.vertex);
                //得到的就是视角方向
                float3 viewDir = _WorldSpaceCameraPos.xyz - worldPos;
                //单位化（归一化）
                viewDir = normalize(viewDir);

                //2.标准化后的反射方向
                //得到的光位置的方向向量（世界空间下的）
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //法线在世界空间下的向量
                float3 normal = UnityObjectToWorldNormal(v.normal);
                //反射光线向量
                float3 reflectDir = reflect(-lightDir, normal);

                //高光反射光照颜色 = 光源的颜色 * 材质高光反射颜色 * max（0, 标准化后观察方向向量· 标准化后的反射方向）幂
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(viewDir, reflectDir)), _SpecularNum);

                data.color = color;

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                return fixed4(i.color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

**片元着色器实现**

**关键步骤**

- 基本和逐顶点一致
- 区别：在顶点着色器中计算顶点和法线相关数据，在片元着色器中计算Phong式高光反射光照

**代码**

```hlsl
Shader "Unlit/Lesson36_SpecularF"
{
    Properties
    {
        //高光反射颜色  光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0, 20)) = 1
    }
    SubShader
    {
        Pass
        {
            //如果有多个Pass渲染通道时 一般情况下会把光照模式的 Tags放到对应的Pass中
            //以免影响其他Pass
            Tags { "LightMode"="ForwardBase" }

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            struct v2f
            {
                //裁剪空间下的顶点坐标
                float4 pos:SV_POSITION;
                //世界空间下的 法线信息
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            fixed4 _SpecularColor;
            float _SpecularNum;

            v2f vert (appdata_base v)
            {
                v2f data;
                //1.顶点裁剪空间变换
                data.pos = UnityObjectToClipPos(v.vertex);
                //2.进行法线空间变换
                data.wNormal = UnityObjectToWorldNormal(v.normal);
                //3.顶点转到世界空间
                //data.wPos = mul(UNITY_MATRIX_M, v.vertex).xyz;
                //data.wPos = mul(_Object2World, v.vertex).xyz;
                data.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //1.视角单位向量
                float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos );

                //2.光的反射单位向量
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //光反射光线的单位向量
                float3 reflectDir = reflect(-lightDir, i.wNormal);
                //color = 光源颜色 * 材质高光反射颜色 * pow( max(0, dot(视角单位向量, 光的反射单位向量)), 光泽度 )
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(viewDir, reflectDir)), _SpecularNum );

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

效果图可以看得出来Phong式光照下的物体不再是兰伯特那种固定的外貌，而是会随着摄像机的观察角度变换，因为它的公式里面利用到了摄像机观察方向。

**总结**  
Phong式高光反射高光反射光照颜色 = 光源的颜色 材质高光反射颜色 max（0, 标准化后观察方向向量· 标准化后的反射方向）幂


### Phong光照模型

**知识点一 两个颜色相加**  
我们在学习兰伯特光照模型时了解了两个颜色相乘它的作用是让两个颜色叠加在一起用两个颜色的RGBA值各自相乘得到一个新的颜色而两个颜色相加的作用 同样是让两个颜色叠加在一起它和两个颜色相乘的区别是：相乘:颜色相乘时，最终颜色会往黑色靠拢计算两个颜色混合时一般用颜色相乘因为真实世界中多个颜色混在一起最终会变成黑色相加:颜色相加时，最终颜色会往白色靠拢计算光照反射时一般用颜色相加，因为向白色靠拢能带来 更亮的感觉，复合光的表现

**知识点二 Unity Shader中的环境光**  
我们在学习兰伯特和半兰伯特光照模型时在计算完漫反射光照后，加上了一个环境光变量 UNITY_LIGHTMODEL_AMBIENT那么这个环境光变量其实是可以在Unity中进行设置的Window→Rendering→LightingEnvironment(环境)页签中的Environment Lighting(环境光)这里可以设置环境光来源当是Skybox和Color时，我们可以通过 UNITY_LIGHTMODEL_AMBIENT 获取到对应环境光颜色当是Gradient（渐变）时，通过以下3个成员可以得到对应的环境光unity_AmbientSky（周围的天空环境光）unity_AmbientEquator（周围的赤道环境光）unity_AmbientGround（周围的地面环境光）注意：这些内置变量都包含在 UnityShaderVariables.cginc 中在编译时，会自动包含该文件，可以不用手动包含

**知识点三 Phong光照模型的来历和原理**  
来历：Phong光照模型我们之前提到过它是由裴祥风（Bui-Tuong Phong，越南裔美国计算机学家）在1975年时，提出的一种局部光照经验模型原理：裴祥风认为物体表面反射光线是由三部分组成的环境光 + 漫反射光 + 镜面反射光（高光反射光）

**知识点四 Phong光照模型的公式**  
Phong光照模型公式：物体表面光照颜色 = 环境光颜色 + 漫反射光颜色 + 高光反射光颜色其中：环境光颜色 = UNITY_LIGHTMODEL_AMBIENT(unity_AmbientSky、unity_AmbientEquator、unity_AmbientGround)漫反射光颜色 = 兰伯特光照模型 计算得到的颜色高光反射光颜色 = Phong式高光反射光照模型 计算得到的颜色

**顶点着色器实现**  
实现思路

**公式**  
物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色

**关键步骤:**

- 计算兰伯特光照模型
- 计算Phong式高光反射光照模型
- 叠加所有计算结果

**代码**  
（逐顶点版本涉及上述组合计算；片元版本见下）

**片元着色器实现**

**代码**

```hlsl
Shader "Unlit/Lesson39_PhongF"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        //高光反射颜色  光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0, 20)) = 1
    }
    SubShader
    {
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            //得到兰伯特光照模型计算的颜色 （逐片元）
            fixed3 getLambertFColor(in float3 wNormal)
            {
                //得到光源单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //计算除了兰伯特光照的漫反射颜色
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(wNormal, lightDir));

                return color;
            }

            //得到Phong式高光反射模型计算的颜色（逐片元）
            fixed3 getSpecularColor(in float3 wPos, in float3 wNormal)
            {
                //1.视角单位向量
                float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - wPos );

                //2.光的反射单位向量
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //光反射光线的单位向量
                float3 reflectDir = reflect(-lightDir, wNormal);
                //color = 光源颜色 * 材质高光反射颜色 * pow( max(0, dot(视角单位向量, 光的反射单位向量)), 光泽度 )
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(viewDir, reflectDir)), _SpecularNum );

                return color;
            }

            v2f vert (appdata_base v)
            {
               v2f v2fData;
               //转换模型空间下的顶点到裁剪空间中
               v2fData.pos = UnityObjectToClipPos(v.vertex);
               //转换模型空间下的法线到世界空间下
               v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
               //顶点转到世界空间
               v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

               return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //计算兰伯特光照颜色
                fixed3 lambertColor = getLambertFColor(i.wNormal);
                //计算Phong式高光反射颜色
                fixed3 specularColor = getSpecularColor(i.wPos, i.wNormal);
                //物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色
                fixed3 phongColor = UNITY_LIGHTMODEL_AMBIENT.rgb + lambertColor + specularColor; 

                return fixed4(phongColor.rgb, 1);
            }
            ENDCG
        }
    }
}
```

效果图高光处和漫反射区的颜色可以分开调整，搭配出不同的效果

**总结**  
Phong光照模型公式：物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色


### Blinn Phong式高光反射光照模型

**知识点一 Blinn-Phong式高光反射光照模型的来历和原理**  
来历通过我们之前知识的学习Phong式高光反射光照模型 是 Phong光照模型的一部分因此很容易推测出 Blinn-Phong式高光反射光照模型其实也是Blinn-Phong光照模型的一部分，主要是用于计算高光反射颜色的Blinn-Phone光照模型的提出者：吉姆·布林（Jim Blinn，美国计算机科学家）原理:Blinn-Phong式高光反射光照模型的理论是它是对Phong式高光反射光照模型的改进它不再使用反射向量计算镜面反射，而是使用半角向量来进行计算半角向量为视角方向和灯光方向的角平分线方向认为高光反射的颜色和 顶点法线向量以及半角向量夹角的余弦成正比并且通过对余弦值取n次幂来表示光泽度（或反光度）

**知识点二 Blinn-Phong式高光反射光照模型的公式**  
公式：高光反射光照颜色 = 光源的颜色 材质高光反射颜色 max（0, 标准化后顶点法线方向向量 · 标准化后半角向量方向向量）幂

1.标准化后顶点法线方向向量 · 标准化后半角向量方向向量 得到的结果就是 cosθ

2.半角向量方向向量 = 视角单位向量 + 入射光单位向量

3.幂 代表的是光泽度 余弦值取n次幂

**知识点三 Phong和Blinn-Phong的区别**  
由于两个光照模型中高光反射光照模型的计算方式不一样从而会带来一些表现上的不同

1.高光散射  
Blinn-Phong模型的高光通常会产生相对均匀的高光散射，这会使物体看起来光滑而均匀。Phong模型的高光可能会呈现更为锐利的高光散射，因为它基于观察者和光源之间的夹角。  
这可能导致一些区域看起来特别亮，而另一些区域则非常暗。

2.高光锐度  
Blinn-Phong模型的高光通常具有较广的散射角，因此看起来不那么锐利。Phong模型的高光可能会更加锐利，特别是在观察者和光源夹角较小时，可能表现为小而亮的点。

3.光滑度和表面纹理  
Blinn-Phong模型通常更适合表现光滑的表面，因为它考虑了表面微观凹凸之间的相互作用，使得光照在表面上更加均匀分布。Phong模型有时更适合表现具有粗糙表面纹理的物体，因为它的高光散射可能会使纹理和细节更加突出。

4.镜面高光大小  
Blinn-Phong模型通常产生的镜面高光相对较大，但均匀分布。Phong模型可能会产生较小且锐利的镜面高光。

**顶点着色器实现**

**关键步骤**

1.属性声明（材质高光反射颜色、光泽度）

2.渲染标签Tags设置 将LightMode光照模式 设置为ForwardBase向前渲染（通常用于不透明物体的基本渲染）

3.引用内置文件UnityCG.cginc和Lighting.cginc

4.结构体声明

5.基本公式实现逻辑

**代码**

```hlsl
Shader "Unlit/Lesson41_BlinnPhongSpecular"
{
    Properties
    {
        //高光反射材质颜色 以及 光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0,20)) = 5
    }
    SubShader
    {
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"
           
            struct v2f
            {
                float4 pos:SV_POSITION;
                fixed3 color:COLOR;
            };

            fixed4 _SpecularColor;
            float _SpecularNum;

            v2f vert (appdata_base v)
            {
                v2f data;
                //顶点转到裁剪空间
                data.pos = UnityObjectToClipPos(v.vertex);

                //高光反射光照颜色 = 光源的颜色 * 材质高光反射颜色 * max（0, 标准化后顶点法线方向向量 · 标准化后半角向量方向向量）幂

                //1.标准化后顶点法线方向向量
                float3 wNormal = UnityObjectToWorldNormal(v.normal);

                //2.标准化后半角向量方向向量
                //视角方向 单位向量
                float3 wPos = mul(unity_ObjectToWorld, v.vertex);
                float3 viewDir = normalize( _WorldSpaceCameraPos.xyz - wPos );
                //光线方向 单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //半角方向向量
                float3 halfA = normalize(viewDir + lightDir);

                //高光反射光照颜色 = 光源的颜色 * 材质高光反射颜色 * max（0, 标准化后顶点法线方向向量 · 标准化后半角向量方向向量）幂
                data.color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(wNormal, halfA)) , _SpecularNum);

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                return fixed4(i.color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

**片元着色器实现**

**关键步骤**

- 基本和逐顶点一致
- 区别：在顶点着色器中计算顶点和法线相关数据，在片元着色器中计算Blinn Phong式高光反射光照

**代码**

```hlsl
Shader "Unlit/Lesson42_BlinnPhongSpecularF"
{
    Properties
    {
        //高光反射材质颜色 以及 光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0,20)) = 5
    }
    SubShader
    {
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            struct v2f
            {
                //裁剪空间的位置
                float4 pos:SV_POSITION;
                //基于世界坐标系下的顶点位置
                float3 wPos:TEXCOORD0;
                //基于世界坐标系下的法线
                float3 wNormal:NORMAL;
            };

            fixed4 _SpecularColor;
            float _SpecularNum;

            v2f vert (appdata_base v)
            {
                v2f data;
                //顶点转到裁剪空间
                data.pos = UnityObjectToClipPos(v.vertex);
                //把顶点从模型空间转换到世界空间中 进行矩阵乘法运算
                data.wPos = mul(unity_ObjectToWorld, v.vertex);
                //法线计算
                data.wNormal = UnityObjectToWorldNormal(v.normal);

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //高光反射光照颜色 = 光源的颜色 * 材质高光反射颜色 * max（0, 标准化后顶点法线方向向量 · 标准化后半角向量方向向量）幂

                float3 viewDir = normalize( _WorldSpaceCameraPos.xyz - i.wPos );
                //光线方向 单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //半角方向向量
                float3 halfA = normalize(viewDir + lightDir);

                //高光反射光照颜色 = 光源的颜色 * 材质高光反射颜色 * max（0, 标准化后顶点法线方向向量 · 标准化后半角向量方向向量）幂
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(normalize(i.wNormal), halfA)) , _SpecularNum);

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

效果图Blinn-Phong模型的高光通常具有较广的散射角，因此看起来没有Phong那么锐利，比较柔和。

**总结**  
Blinn-Phong式高光反射高光反射光照颜色 = 光源的颜色 材质高光反射颜色 max（0, 标准化后顶点法线方向向量 · 标准化后半角向量方向向量）幂


### Blinn Phong光照模型

**知识点一 Blinn Phong光照模型的来历和原理**  
来历：Blinn Phong光照模型我们之前提到过它是由吉姆·布林（Jim Blinn，美国计算机科学家）在1977年时，在Phong光照模型基础上进行修改提出的它和Phong一样是一个经验模型，并不符合真实世界中的光照现象它们只是看起来正确原理：Blinn Phong和Phong光照模型一样，认为物体表面反射光线是由三部分组成的环境光 + 漫反射光 + 镜面反射光（高光反射光）

**知识点二 Blinn Phong光照模型的公式**  
Blinn Phong光照模型公式：物体表面光照颜色 = 环境光颜色 + 漫反射光颜色 + 高光反射光颜色其中：环境光颜色 = UNITY_LIGHTMODEL_AMBIENT(unity_AmbientSky、unity_AmbientEquator、unity_AmbientGround)漫反射光颜色 = 兰伯特光照模型 计算得到的颜色高光反射光颜色 = Blinn Phong式高光反射光照模型 计算得到的颜色

**顶点着色器实现**

**关键步骤**

- 计算兰伯特光照模型
- 计算Blinn Phong式高光反射光照模型
- 叠加所有计算结果

**代码**  
（逐顶点版本为上述高光模型的组合；片元版本见下）

**片元着色器实现**

**代码**

```hlsl
Shader "Unlit/Lesson45_BlinnPhongF"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        //高光反射颜色  光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0, 20)) = 1
    }
    SubShader
    {
       
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            //得到兰伯特光照模型计算的颜色 （逐片元）
            fixed3 getLambertFColor(in float3 wNormal)
            {
                //得到光源单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //计算除了兰伯特光照的漫反射颜色
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(wNormal, lightDir));

                return color;
            }

            //得到Blinn Phong式高光反射模型计算的颜色（逐片元）
            fixed3 getSpecularColor(in float3 wPos, in float3 wNormal)
            {
                //1.视角单位向量
                float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - wPos );

                //2.光的反射单位向量
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);

                //半角方向向量
                float3 halfA = normalize(viewDir + lightDir);
                
                //color = 光源颜色 * 材质高光反射颜色 * pow( max(0, dot(视角单位向量, 光的反射单位向量)), 光泽度 )
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(wNormal, halfA)), _SpecularNum );

                return color;
            }

            v2f vert (appdata_base v)
            {
                 v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //计算兰伯特光照颜色
                fixed3 lambertColor = getLambertFColor(i.wNormal);
                //计算BlinnPhong式高光反射颜色
                fixed3 specularColor = getSpecularColor(i.wPos, i.wNormal);
                //物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色
                fixed3 blinnPhongColor = UNITY_LIGHTMODEL_AMBIENT.rgb + lambertColor + specularColor; 

                return fixed4(blinnPhongColor.rgb, 1);
            }
            ENDCG
        }
    }
}
```

效果图拥有比Phong光照模型更平缓更大的高亮，高亮边缘有类似光晕的效果。

**总结**  
Blinn Phong光照模型公式：物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Blinn Phong式高光反射光照模型所得颜色


### 为什么逐片元比逐顶点平滑

**重要知识回顾**  
通过知识点回顾，我们知道在顶点着色器和片元着色器之间还存在其他流程，比如：

1.裁剪

2.屏幕映射

3.三角形设置

4.三角形遍历  
等等也就是说顶点着色器回调函数中返回的数据，还会在这些中间流程中进行处理，再传递给片元着色器。这也是为什么逐片元比逐顶点更加平滑的原因

**为何逐顶点光照比较粗糙**  
如果我们在顶点着色器中进行了颜色相关的计算，我们只是计算了模型顶点位置的颜色信息。比如下图的A、B、C就是三角形面片的三个顶点。而上图的三角形里面，我们又可以看出顶点将片元包裹在其中的画面。  
那问题来了，大部分情况下，顶点的数量是远远少余片元的。那么，当我们只有使用顶点着色器实现光照的颜色时候，那么为什么可以仅仅通过顶点着色器返回的几个顶点的值，就可以处理大部分的片元，它是如何推算出片元的数据呢？  
解决办法就是，这些数据在渲染管线中传递时，会在光栅化阶段的片元着色器之前对三角形围着的片元颜色进行插值运算，再传递给片元着色器。下图的红点就是我们假设的图元，你会发现它的法线和位置明显是和三个顶点不一样的，所以，我们可以通过三个顶点的插值对它的数据进行推断。  
也就是说顶点之间的片元颜色信息，其实都不是使用该位置的相关信息计算的， 而是粗暴的插值计算，计算颜色的平均值，那么自然呈现出来的效果是比较粗糙的。

**为何逐片元光照更加平滑**  
首先我们要明确的是，片元着色器中传入的参数结构虽然和顶点着色器中返回的一样，但是里面的数据是不同的，片元着色器中传入的数据，都是为每一个片元进行专门插值计算后的结果。  
因此我们利用这些数据再次进行光照计算，自然更加的平滑真实。一个是插值，一个是真正的通过我们的算式运算得出的结果也就是说，在片元着色器中进行颜色相关计算时，能够为每一个像素点，基于它的相关数据单独进行颜色计算，而不是直接利用插值颜色，自然最终的表现效果会更加的平滑真实。  
也因为如此，你可以把片元着色器上的另外处理视为“二次绘图”，因为即使你完全不用它对颜色进行再处理而是直接返回，你也会通过插值得到一个顶点信息插值推算而来的图元颜色信息。

**注意点**

1.语义的存在一大用处的也是为了给底层识别以提供合适的插值公式。

2.结构体通常被用来到顶点着色器到片元着色器的数据传输，传输到片元着色器后，里面的值也自然会被插值

3.顶点着色器输出的归一化向量在传递到片段着色器的过程中会进行插值。由于线性插值不会保持向量的长度，这意味着插值后的向量很可能不再是单位向量。在片段着色器中重新归一化这些向量可以确保它们具有正确的单位长度，从而使得光照和其他基于这些向量的计算更加精确。  
所以你可以发现我们上面的几节的片元着色器实现里，涉及到法线对象的时候都会在片元着色器进行归一化。

4.建议将顶点坐标转换信息放到顶点着色器处理，顶点坐标转换信息的插值比起颜色插值通常是比较可靠的，而且这样可以提高性能，毕竟顶点着色器函数的执行次数也是比较少的。  
可以参考的文献UnityShader中从顶点着色器到片元着色器的插值变换_unity shader颜色插值-CSDN博客

**总结**

1.为什么逐顶点光照的渲染表现比较粗糙  
顶点以外的像素点颜色，是进行插值运算的结果，效果不真实

2.为什么逐片元光照的渲染表现更加的平滑  
每一个像素点都利用其相关数据（法线、位置等）进行颜色计算，效果更真实注意：也侧面说明了片元着色器的性能消耗更大，顶点着色器的性能消耗更小


### Unity内置光照计算相关函数

**知识点一 内置光照计算相关函数是什么？**  
我们在实现光照模型时经常会进行一些数学计算比如 坐标、法线相关的转换在UnityCG.cginc中提供了一些常用的函数可以帮助我们快捷的进行数学计算比如我们之前将法线从模型空间转换到世界空间的方法UnityObjectToWorldNormal 等有了这些内置函数，我们就不需要自己去通过数学计算来得到结果了直接调用API即可得到我们想要的结果除了之前我们在课程中用到的一些函数外还有一些其他的内置函数这节课我们主要就是对他们进行了解

**知识点二 常用内置函数**

1. float3 WorldSpaceViewDir(float4 v)     
   传入模型空间下的顶点位置，返回世界空间中从该点到摄像机的观察方向
2. float3 UnityWorldSpaceViewDir(flaot4 v)     
   传入世界空间中的顶点位置，返回世界空间中从该点到摄像机的观察方向
3. float3 ObjSpaceViewDir(float4 v)     
   传入模型空间中的顶点位置，返回模型空间中从该点到摄像机的观察方向
4. float3 WorldSpaceLightDir(float4 v)     
   (仅用于向前渲染中)传入模型空间中的顶点位置，返回世界空间中该点到光源的光照方向（没有归一化）
5. float3 UnityWorldSpaceLightDir(float4 v)     
   (仅用于向前渲染中)传入世界空间中的顶点位置，返回世界空间中从该点到光源的光照方向（没有归一化）
6. float3 ObjSpaceLightDir(float4 v)     
   (仅用于向前渲染中)传入模型空间中的顶点位置，返回模型空间中从该点到光源的光照方向（没有归一化）
7. float3 UnityObjectToWorldNormal(float3 normal)     
   把法线从模型空间转换到世界空间中
8. float3 UnityObjectToWorldDir(float3 dir)     
   把方向矢量从模型空间转换到世界空间中
9. float3 UnityWorldToObjectDir(float3 dir)     
   把方向矢量从世界空间转换到模型空间中

10.float4 UnityObjectToClipPos(float4 v)  
输入模型空间中的顶点位置，返回裁剪空间下的顶点位置

## 第 2 节 纹理

### 纹理概念

**知识点一 纹理是用来干嘛的？**  
模型的骨肉皮骨：模型骨骼肉：三角面片（由模型顶点围成模型的轮廓）皮：纹理图片由此可以看出纹理的主要作用就是使用一张图片来控制模型的外观使用纹理映射技术，将图片和模型联系起来，让模型能呈现出图片中的颜色表现纹理映射：是计算机图形学中的一种技术，它用来将图像（纹理）映射到三维模型的表面，从而赋予模型更加真实和细致的外观这个过程实际上是将二维图像映射到三维空间的过程

**知识点二 如何进行纹理映射？**  
在建模软件中，美术同学会利用纹理展开技术把纹理映射坐标存储在每个顶点上模型表面的顶点都与纹理坐标相关联纹理坐标通常使用二维坐标系统（称为UV坐标），其中U表示水平轴，V表示垂直轴简而言之就是美术通过使用建模软件进行纹理映射之后导出的模型数据中存储了每个顶点对应的纹理坐标（UV坐标）

**知识点三 UV坐标对于Shader开发的意义**  
我们在进行Shader开发时在顶点着色器回调函数传入的数据中我们可以获取到模型中的UV坐标的数据有了UV坐标，只要有正确的纹理贴图（图片）我们就可以用顶点的UV坐标从图片中取出映射的颜色用于渲染模型便可以呈现出“皮”的颜色效果

**知识点四 UV坐标的注意事项**

1.纹理坐标（UV坐标）的横轴和纵轴的取值范围是被归一化过的，在[0,1]的范围中  
主要目的是为了适应不同大小的纹理图片（256x256、512x512、1024x1024等等）

2.只有顶点中记录了UV坐标，因此在数据传递进片元着色器之前，UV坐标会在中间阶段进行插值运算  
也就是每个片元中得到的UV坐标大部分都是插值计算得结果，因此我们可以得到图片中非顶点位置的颜色

**总结**  
纹理就是模型的“皮”它决定了模型的颜色表现在建模时通过纹理映射技术将顶点和纹理图片建立联系在模型数据中记录顶点对应的UV坐标之后我们在进行Shader开发时就可以利用UV坐标从纹理图片中取出对应的颜色给片元“上色”了注意：

1.UV坐标的取值范围在[0~1]之间

2.顶点之间的点（片元），会通过插值运算得到对应的UV坐标  
从而从纹理图片中取出对应位置的颜色


### 纹理颜色采样

**知识点 书写单张纹理颜色采样Shader**  
第一步 完成Shader文件基本结构，让其不报错第二步 纹理属性和CG成员变量声明关键知识点CG中映射ShaderLab中的纹理属性，需要有两个成员变量一个用于映射纹理颜色数据，一个用于映射纹理缩放平移数据ShaderLab中的属性图片属性（2D）用于利用UV坐标提取其中颜色CG中用于映射属性的成员变量

1.sampler2D 用于映射纹理图片

2.float4 用于映射纹理图片的缩放和平移  
固定命名方式 纹理名_ST (S代表scale缩放 T代表translation平移)第三步 用缩放平移参数参与uv值计算

1.如何获取模型中携带的uv信息？  
在顶点着色器中，我们可以利用TEXCOORD语义获取到模型中的纹理坐标信息它是一个float4类型的xy获取到的是纹理坐标的水平和垂直坐标zw获取到的是纹理携带的一些额外信息，例如深度值等

2.如何计算  
固定算法先缩放，后平(偏)移缩放用乘法，平(偏)用加法纹理坐标.xy * 纹理名_ST.xy + 纹理名_ST.wz或者直接用内置宏TRANSFORM_TEX(纹理坐标变量, 纹理变量)该宏在内部会进行相同的计算第四步 在片元着色器中进行纹理颜色采样fixed4 tex2D(sampler2D tex, float2 s)传入纹理图片和uv坐标返回纹理图片中对应位置的颜色值

**实现**

```hlsl
Shader "Unlit/Lesson48"
{
    Properties
    {
        //主纹理
        _MainTex("MainTex", 2D) = ""{}
    }
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"

            //映射对应纹理属性的图片颜色相关数据
            sampler2D _MainTex;
            //映射对应纹理属性的 缩放 平(偏)移数据
            float4 _MainTex_ST; //xy代表缩放 zw代表平移

            v2f_img vert (appdata_base v)
            {
               v2f_img data;
               data.pos = UnityObjectToClipPos(v.vertex);
               //v.texcoord.xy //代表uv坐标
               //v.texcoord.zw //代表一些额外信息
               //先缩放 后平移 这个是一个固定的算法 规则如此
               //如果没有进行缩放和平移 那么 这个计算后 值是不会产生变化的
               //因为缩放默认值是1和1 ，平移默认值是0和0
               data.uv = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
               //这是另一种写法
               //TRANSFORM_TEX(v.texcoord.xy, _MainTex);
               return data;
            }

            fixed4 frag (v2f_img i) : SV_Target
            {
                //这传入的uv 是经过插值运算后的 就是每一个片元都有自己的一个uv坐标
                //这样才能精准的在贴图当中取出颜色
                fixed4 color = tex2D(_MainTex, i.uv);

                return color;
            }
            ENDCG
        }
    }
}
```

**总结**  
纹理颜色采样非常简单

1.顶点着色器中获取到模型中的纹理坐标信息  
进行相关的缩放偏移计算后返回

2.片元着色器得到插值后的uv坐标进行纹理采样  
返回对应的颜色

### 纹理基础知识

Unity核心 - 张先生的小屋 (klned.com)查看此文章的一二节即可

### 纹理相关设置

**知识点一 关于纹理图片导入相关设置**  
在Unity四部曲Unity核心当中，我们详细的讲解过图片导入相关的设置若不清楚的同学学习选修课程即可

**知识点二 重要纹理相关设置回顾**

1.Texture Type(纹理图片类型) 和 Texture Shape(纹理图片类型)  
决定了我们是否能在Shader当中获取正确数据

2.Wrap Mode(循环模式)  
决定了缩放偏移的表现效果Repeat：在区块中重复纹理Clamp： 拉伸纹理的边缘Mirror：在每个整数边界上镜像纹理以创建重复图案Mirror Once：镜像纹理一次，然后将拉伸边缘纹理Per-axis：单独控制如何在U轴和V轴上包裹纹理

3.Filter Mode(过滤模式)  
决定了放大缩小纹理时看到的图片质量Point：纹理在靠近时变为块状Bilinear：纹理在靠近时变得模糊Trilinear：与Bilinear类似，但纹理也在不同的Mip级别之间模糊过滤模式在开启MipMaps根据实际表现选择，可以达到不同的表现效果


### 纹理结合光照模型

**知识点 单张纹理结合BlinnPhong光照模型**  
在计算时，有以下的3点注意点纹理颜色需要和漫反射颜色 进行乘法叠加, 它们两共同影响最终的颜色兰伯特光照模型计算时，漫反射材质颜色使用 1 中的叠加颜色计算最终使用的环境光叠加时，环境光变量UNITY_LIGHTMODEL_AMBIENT需要和 1 中颜色进行乘法叠加，这是为了避免最终的渲染效果偏灰其他的计算步骤同BlinnPhong的逐片元光照实现

**实现**

```hlsl
Shader "Unlit/Lesson50"
{
    Properties
    {
       //主要就是将单张纹理Shader和Blinn-Phong光照模型逐片元Shader进行一个结合
       _MainTex("MainTex", 2D) = ""{}
       //漫反射颜色、高光反射颜色、光泽度
       _MainColor("MainColor", Color) = (1,1,1,1)
       _SpecularColor("SpecularColor", Color) = (1,1,1,1)
       _SpecularNum("SpecularNum", Range(0,20)) = 15
    }
    SubShader
    {
        Pass
        {
            Tags{ "LightMode" = "ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //纹理贴图对应的映射成员
            sampler2D _MainTex;
            float4 _MainTex_ST;
            //漫反射颜色、高光反射颜色、光泽度
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            struct v2f
            {
                //裁剪空间下的顶点坐标
                float4 pos:SV_POSITION;
                //UV坐标
                float2 uv:TEXCOORD0;
                //世界空间下的法线
                float3 wNormal:NORMAL;
                //世界空间下的顶点坐标
                float3 wPos:TEXCOORD1;
            };

            v2f vert (appdata_base v)
            {
               v2f data;
               //把模型空间下的顶点转换到裁剪空间下
               data.pos = UnityObjectToClipPos(v.vertex);
               //uv坐标计算
               data.uv = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
               //世界空间下的法线
               data.wNormal = UnityObjectToWorldNormal(v.normal);
               //世界空间下的顶点坐标
               data.wPos = mul(unity_ObjectToWorld, v.vertex);
               return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //新知识点：纹理颜色需要和漫反射材质颜色叠加（乘法） 共同决定最终的颜色
                fixed3 albedo = tex2D(_MainTex, i.uv).rgb * _MainColor.rgb;
                //光的方向（指向光源方向）
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //兰伯特漫反射颜色 = 光的颜色 * 漫反射材质的颜色 * max(0, dot(世界坐标系下的法线, 光的方向))
                //新知识点：兰伯特光照模型计算时，漫反射材质颜色使用 1 中的叠加颜色计算
                fixed3 lambertColor = _LightColor0.rgb * albedo.rgb * max(0, dot(i.wNormal, lightDir));
                
                // 视角方向
                //float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos);
                float3 viewDir = normalize(UnityWorldSpaceViewDir(i.wPos));
                //半角向量 = 视角方向 + 光的方向
                float3 halfA = normalize(viewDir + lightDir);
                //高光反射的颜色 = 光的颜色 * 高光反射材质的颜色 * pow(max(0, dot(世界坐标系下的法线, 半角向量)), 光泽度)
                fixed3 specularColor = _LightColor0.rgb * _SpecularColor * pow( max(0, dot(i.wNormal, halfA)), _SpecularNum);

                //Blinn-Phong光照颜色 = 环境光颜色 + 兰伯特漫反射颜色 + 高光反射的颜色
                //新知识点：最终使用的环境光叠加时，环境光变量UNITY_LIGHTMODEL_AMBIENT需要和 1 中颜色进行乘法叠加
                //         为了避免最终的渲染效果偏灰
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo + lambertColor + specularColor;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

效果图左边为Standard默认材质，右边为本节课效果图，对高光处进行了红光混合。

### 凹凸纹理基本概念

**凹凸纹理是用来做什么的？**  
纹理除了可以用来进行颜色映射外，另外一种常见的应用就是进行凹凸映射凹凸映射的目的是使用一张纹理来修改模型表面的法线，让我们不需要增加顶点，而让模型看起来有凹凸效果。  
原理：光照的计算都会利用法线参与计算，决定最终的颜色表现效果。那么在计算“假”凹凸面时，使用“真”凹凸面的法线参与计算，呈现出来的效果可以以假乱真凹凸纹理最大的作用就是让模型可以在不添加顶点（不增加面）的情况下让模型看起来同样充满细节（凹凸感），是一种视觉上的“欺骗”技术。  
要进行凹凸映射，目前有两种主流方式：高度纹理贴图法线纹理贴图我们接下来就来详细的了解他们的基本原理

### 高度纹理贴图概述

高度纹理贴图一般简称高度图它存储了模型表面上每个点的高度信息。通常它使用灰度图像，其中不同灰度值表示不同高度。较亮区域通常对应较高的点，较暗的区域对应较低的点。  
它主要用于模拟物体表面的位移。存储规则：图片中的某一个像素点的RGB值是相同的，都表示高度值，A值一般情况下为1。高度值范围一般为0~1，0代表最低，1代表最高优点：可以通过高度图很明确的知道模型表面的凹凸情况缺点：无法在Shader中直接得到模型表面点的法线信息，而是需要通过额外的计算得到，因此会增加性能消耗，而且rgb三位全部只存一个数，利用的效率实在有点低，所以我们几乎很少使用它。  
我们在使用凹凸纹理时，一般都会使用法线纹理贴图

### 法线纹理贴图概述

法线纹理贴图一般简称法线贴图 或 法线纹理它存储了模型表面上每个点的法线方向。存储规则：图片中的RGB值分别存储法线的X、Y、Z分量值，A值可以用于存储其他信息，比如材质光滑度等。  
优点：从法线贴图中取出的数据便是法线信息，可以直接简单处理后就参与光照计算，性能表现更好缺点：我们无法直观的看出模型表面的凹凸情况由于法线纹理贴图是我们之后实现“凹凸感”的主要方式因此我们需要更详细的了解它

**读取分量数据的规则**  
两种法线纹理贴图的存储方式法线纹理贴图读取分量数据的规则由于法线XYZ分量范围在[-1,1]之间而像素RGB分量范围在[0,1]之间因此我们需要做一个映射计算存储图片时：像素分量 = (法线分量 + 1) / 2因此当我们取出像素分量使用时需要进行逆运算读取数据时：法线分量 = 像素分量 * 2 -1


### 两种法线纹理贴图的存储方式

法线纹理贴图中主要存储法线信息，而法线信息其实就是个方向向量，而方向向量就得有相对坐标系因此，法线贴图的存储方式按相对坐标系有两种方式：基于模型空间的法线纹理基于切线空间的法线纹理 （重点）

**基于模型空间的法线纹理**  
模型数据中自带的法线数据，是定义在模型空间中的，因此最直接的存储法线贴图数据的方式就是存储基于模型空间下的法线信息。（注意：模型数据中的法线数据是“真”数据，法线贴图中对的法线数据是“假”数据）如右下角图所示，由于模型空间中每个点存储的法线方向是各式各样的比如法线（0,1,0）映射到像素后(法线分量 + 1) / 2 是 (0.5,1.0.5) 绿色法线（0,-1,0）映射到像素后(法线分量 + 1) / 2 是 (0.5,0,0.5) 紫色因此基于模型空间的法线纹理一般是五颜六色的这种法线纹理贴图数据取出来直接参与Shader计算即可

**基于切线空间的法线纹理**  
虽然基于模型空间的法线纹理贴图看起来很符合计算需求但是在实际开发时，美术给到我们的法线贴图一般都是基于切线空间的每个顶点都有自己的切线空间原点：顶点本身X轴：顶点切线Z轴：法线方向（顶点的原法线）Y轴：X和Z的叉乘结果，也被称为副切线在切线空间下，如果该顶点的法线不变化（不需要“凹凸感”）那么它的坐标是(0,0,1)，因为在切线空间下，Z轴就是原法线方向因此：法线（0,0,1）映射到像素后(法线分量 + 1) / 2 是 (0.5,0.5,1) 浅蓝色这个浅蓝色就是 切线空间下法线贴图 存在大片蓝色的原因因为大部分顶点的法线和模型本身法线是一致的只有凹凸部分的颜色才会有些许差异这种法线纹理贴图数据取出来后需要进行坐标空间转换再参与Shader计算

**为什么要使用切线空间下的法线纹理贴图**  
之所以在实际开发时要使用切线空间下的法线贴图原因有以下几点：

1. 可以用于不同模型 ——如果模型空间下法线，不能用于其他模型
2. 方便处理模型变形 ——同上
3. 可以复用 ——一个砖块，6个面贴图都是一样的，可以只用一张法线贴图即可用于6个面计算
4. 可以压缩 ——可以只存储两个轴的分量
5. 方便制作UV动画 ——UV坐标改变可以实现凹凸移动效果，如果是模型空间下法线贴图表现会有问题

**一些可能的疑问和解答**

1.为什么模型空间的法线纹理图五颜六色，切线空间的法线纹理图颜色基本近蓝？  
模型空间的法线是基于整个模型的全局坐标系来计算的，这意味着法线的方向是相对于模型整体的位置和方向。因此，不同点上的法线可能会显示出很大的差异，尤其是在模型形状复杂或包含多个不同朝向的表面时。这就是为什么在模型空间法线可视化中，你会看到多种颜色，反映了各个方向的法线。  
相比之下，切线空间的法线是在每个顶点的局部坐标系中计算的，这个局部坐标系是通过顶点的切线、副切线和法线定义的。这意味着切线空间中的法线主要表示的是相对于表面局部几何的偏移，而不是整个模型的全局方向。因为这种计算方式更加注重局部表面的细节变化，大多数情况下，这些局部的法线变化相对较小，主要是垂直于表面的微小偏差。因此，切线空间法线贴图中蓝色（代表垂直于表面的法线方向）占主导，而红色和绿色（代表水平方向的偏移）通常较少见，除非存在显著的表面细节变化，则也是可能出现类似模型空间法线纹理图的效果。

2.法线纹理图的值的确定  
模型空间的法线直接取自于凹凸模型上的点的值切线空间法线没有像模型空间法线的值那么好理解，它实际上涉及到两个模型的切换。设一个模型拥有凹凸模型和平滑模型切线纹理图各点的值，是来自于凹凸模型的法线在平滑模型的切线空间坐标系下的值。  
理解了这些，后面我们在解析法线图片的时候就会容易的多。


### 切线空间法线贴图的计算方式

**两种主流计算方式**  
通过上节课的学习我们知道，想要实现凹凸效果，主要就是使用基于切线空间的法线贴图中的法线信息参与到光照计算中。而在计算光照模型时，通常会有两种选择：在切线空间下进行光照计算，需要把光照方向、视角方向变换到切线空间下参与计算在世界空间下进行光照计算，需要把法线方向变换到世界空间下参与计算

**各自的优缺点**  
效率在切线空间中计算，效率更高，因为可以在顶点着色器中就完成对光照、视角方向的矩阵变换，计算量相对较小。( 矩阵变换在顶点着色器中计算)在世界空间中计算，效率较低，由于需要对法线贴图进行采样，所以变换过程必须在片元着色器中实现，我们需要在片元着色器中对法线进行矩阵变换。  
( 矩阵变换在片元着色器中计算)全局效果在切线空间中计算，对全局效果的表现可能会不够准确在处理一些列如镜面反射、环境映射效果时表现效果可能不够准确在世界空间中计算，对全局效果的表现更准确可以更容易的应用于全局效果的计算

**总结**  
因此我们在选择使用哪种计算方式时主要考虑若没有全局效果要求，我们优先使用在切线空间下进行光照计算，因为它效率较高反之，我们选择在世界空间下计算

**在切线空间下计算**  
在切线空间下进行光照计算，需要把光照方向、视角方向变换到切线空间下参与计算关键点：计算模型空间到切线空间的变换矩阵由于是父到子坐标，所以，变换矩阵为坐标系变换矩阵的逆矩阵由于我们主要用变换矩阵来进行矢量的变换而非点的变换，不需要平移，因此可以变为3x3矩阵而x、y、z轴分别为切线空间中顶点的切线、副切线、法线已知切线、法线（从模型数据中可以获取），副切线为切线、法线的叉乘结果而3个轴为相互垂直的单位向量，而我们的变换矩阵正好是这个三个轴构建出来的，因此可以推出构建出来的变换矩阵是正交矩阵，正交矩阵的转置矩阵就是逆矩阵。  
因此该坐标系变换矩阵的转置矩阵它就是模型空间到切线空间的变换矩阵需要用到的核心知识：内置函数：得到模型空间光的方向：ObjSpaceLightDir(模型空间顶点坐标)得到模型空间视角方向：ObjSpaceViewDir(模型空间顶点坐标)得到光方向和视角方向相对于模型空间的数据表达后再与模型空间到切线空间的变换矩阵进行运算即可将他们转换到切线空间下参与后续计算

**在世界空间下计算**  
在世界空间下进行光照计算，需要把法线方向变换到世界空间下参与计算关键点：计算切线空间到世界空间的变换矩阵变换矩阵为子到父的变换由于我们主要用变换矩阵来进行矢量的变换而非点的变换，因此可以变为3x3矩阵而x、y、z轴分别为切线空间中顶点的切线、副切线、法线所以我们只需要得到3个轴相对于世界空间的向量表达，即可得到该变换矩阵需要用到的核心知识：法线从模型空间到世界空间：UnityObjectToWorldNormal(模型空间法线数据)切线从模型空间到世界空间： UnityObjectToWorldDir(模型空间切线数据)世界空间的副切线：用上面计算的结果叉乘即可由这三个向量组成最终的切线空间到世界的空间的变换矩阵即可

**关键知识点补充**  
切线空间下使用法线贴图问题凹凸系数实现里面会碰到一个难题，那就是_BumpScale（凹凸系数）的算法，假如你试图直接拿系数去乘法线，你会发现由于法线本身的长度被总体缩短了，导致法线的所有点乘算法的结果也变小，最终导致光照整体变小！  
解决的思路就是保证法线长度为单位向量的同时，并令系数作用于法线的X，Y上（切线空间法线的xy是扰动的主要标识，因为默认法线是（0，0，1））具体算法如下：

1.只让法线中的xy乘以凹凸系数  
tangentNormal.xy *= \_BumpScale;

2.保证法线为单位向量（让法线不会为0，而是趋近于顶点法线）  
x² + y² + z² = 1z² = 1 - (x² + y²)z = 根号下(1 - (x² + y²))写成untiy shader就是：tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));通过这样的计算，当凹凸系数在0~1之间变化时，会保证法线为单位向量这样就不会影响光照表现了尽量取lfoat4我们目前在v2f结构体中世界坐标顶点位置和变换矩阵使用了float3 和 float3x3 的两个变量来存储但是在很多世界空间下计算 法线贴图的Shader中往往会使用3个 float4 类型的变量来存储它们这样做的目的是因为这种写法在很多情况下可以提高性能，因为它更好地与GPU的硬件架构匹配float4 类型的寄存器是非常高效的因为现代GPU通常会以 4 分量的向量为基本单位进行并行计算float3x3 矩阵相对来说需要更多的寄存器和指令来表示和计算

### 渐变纹理基本概念

**渐变纹理是用来做什么的**  
通过单张纹理和凹凸纹理相关知识的学习，我们知道图片中存储的数据不仅仅可以是颜色数据，还可以是高度、法线数据。理论上来说，图片中存储的数据我们可以自定义规则，我们可以往图片中存储任何满足我们需求的数据用于渲染。  
而渐变纹理就是用于控制漫反射光照结果的一种存储数据的方式它的主要作用是让游戏中的对象具有插画卡通风格下图中的模型就是使用不同渐变纹理呈现出来的效果渐变纹理的使用可以保证物体的轮廓线相比之前使用的传统漫反射光照更加明显而且还能提供多种色调的变化，可以让模型更具卡通感这种卡通感的来源主要是让光源过渡变得比较干净利落。


### 渐变纹理基础实现

**实现思路**  
主要是修改漫反射里面的值，将半兰伯特的后半段直接当作uv坐标（uv相同）去获取漫反射的颜色，然后和漫反射光照混合即可。

**实现**

```hlsl
Shader "Unlit/Lesson53"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        _RampTex("RampTex", 2D) = ""{}
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(8, 256)) = 18
    }
    SubShader
    {
        Pass
        {
            Tags {"LightMode" = "ForwardBase"}
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            fixed4 _MainColor;
            sampler2D _RampTex;
            float4 _RampTex_ST;
            fixed4 _SpecularColor;
            float _SpecularNum;

            struct v2f
            {
                //裁剪空间下顶点坐标
                float4 pos:SV_POSITION;
                //世界空间下顶点坐标
                float3 worldPos:TEXCOORD0;
                //世界空间下法线
                float3 worldNormal:TEXCOORD1;
            };

            v2f vert (appdata_base v)
            {
                v2f data;
                data.pos = UnityObjectToClipPos(v.vertex);
                data.worldPos = mul(unity_ObjectToWorld, v.vertex);
                data.worldNormal = UnityObjectToWorldNormal(v.normal);
                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0);
                
                //漫反射颜色（通过渐变纹理得到的颜色来进行叠加）
                fixed halfLambertNum = dot(normalize(i.worldNormal), lightDir) * 0.5 + 0.5;
                //漫反射颜色 = 光的颜色 * 漫反射颜色 * 渐变纹理中取出的颜色
                fixed3 diffuseColor = _LightColor0.rgb * _MainColor.rgb * tex2D(_RampTex, fixed2(halfLambertNum, halfLambertNum));

                //高光反射颜色
                //视角方向
                float3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                float3 halfDir = normalize(lightDir + viewDir);
                fixed3 specularColor = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0, dot(i.worldNormal, halfDir)), _SpecularNum);

                //Blinn-Phong 公式
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb + diffuseColor + specularColor;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }
}
```

效果图动画风格的体现，主要是阴影过渡变得更硬了。

### 渐变纹理综合实现

概述在上一节的基础上，为shader加入纹理和法线贴图。

### 遮罩纹理基本概念

**遮罩纹理是用来做什么的**  
遮罩纹理通常用于控制或限制某些效果的显示范围。它允许我们可以保护某些区域，使它们免于某些修改。一般情况下，遮罩纹理也会是一张灰度图，其中的RGB值会是相同的我们利用它存储的值参与到光照（指定某些区域受光影响的程度）透明度（指定某些区域透明的程度）特效（指定某些区域出现特效）等等相关的计算中 从而来让指定区域达到我们想要的效果我们以高光遮罩纹理举例上图三个胶囊体的对比就是高光遮罩纹理起到的效果利用高光遮罩纹理我们可以控制模型上的各个区域受到高光影响的强弱

**高光遮罩纹理的基本原理**  
高光遮罩纹理的基本原理是：从纹理中取出对应的遮罩掩码值（颜色的RGB值都可以使用）用该掩码值和遮罩系数(我们自己定义的)相乘得到遮罩值用该掩码值和高光反射计算出来的颜色相乘最终呈现出来的高光反射表现就会受到高光遮罩纹理 和 遮罩系数 的影响从而表现出最终效果

### 高光遮罩纹理综合实现

步骤

- 从纹理中取出对应的遮罩掩码值（颜色的RGB值都可以使用）
- 用该掩码值和遮罩系数(我们自己定义的)相乘得到遮罩值
- 用该掩码值和高光反射计算出来的颜色相乘



### 遮罩纹理中的RGBA值

对于高光遮罩纹理中的RGBA值，是非常浪费的因为我们只使用其中一个值就可以得到我们想要的数据因此对于遮罩纹理来说我们可以合理的利用其中的每一个值来存储我们想要的数据随着以后的学习我们可以在遮罩纹理当中存储更多信息比如：R值代表高光遮罩数据G值代表透明遮罩数据B值代表特效遮罩数据等等甚至可以用 n 张遮挡纹理存储 4xn 个会参与 每个片元渲染计算的值

## 第 3 节 透明


### 透明必备知识点—渲染顺序

**建议回顾**  
在学习本章之前，建议先去回顾下：渲染标签— 渲染队列渲染状态— 深度缓冲渲染状态— 深度测试渲染状态— 混合方式

**深度测试和深度写入带来的好处**  
有了深度测试和深度写入发挥作用让我们不需要关心不透明物体的渲染顺序比如一个物体A 挡住了物体B即使底层逻辑中先渲染A，后渲染B，我们也不用担心B的颜色会把A覆盖因为在进行深度测试时，远处的B的深度值无法通过深度测试因为它的深度会比已经写入深度缓冲中A的深度值大重合处的片元会被丢弃，颜色自然就不会写入，最终重叠处渲染出来的会是A的颜色我们之前写的所有Shader都没有刻意的去设置深度测试(默认小于判断)、深度写入(默认开启) 、混合模式(默认不混合) 、渲染队列(默认几何Geometry队列) 相关内容是因为对于不透明的物体来说，使用默认设置就能够得到正确的渲染效果但目前我们将要学习的透明相关知识，就需要对他们进行改变其中，最最最重要的改变就是：处理透明混合时，我们通常会选择关闭深度写入

**透明混合与深度写入**  
混合物体为什么关闭深度写入假设有两个透明混合对象重叠，一旦靠近摄像机的混合对象先被渲染，会导致更远处的透明对象直接不通过深度测试无法显示。为了在两个透明混合对象重叠的时候，假设它们的渲染顺序相同，那无论是位于前方还是位于后方的对象，在渲染的时候都能得到相对准确的效果。（只是相对准确，后渲染的对象靠近相机是正确结果，但是后渲染的对象远离相机的话混合结果不一定准确）注意混合物体关闭深度写入与非透明物体无关，由于渲染顺序的存在，我们可以保证混合对象都是在非透明物体之后开始渲染的，对于非透明物体，由于它们通常先渲染，并且它们的深度信息会被写入深度缓冲区，因此不受关闭深度写入影响。

**总结**  
再面对透明混合物体和非透明物体同时在场的情况，我们通常会如下处理：先渲染所有不透明物体，并开启它们的深度测试和深度写入。把半透明物体按它们距离摄像机的远近进行排序，然后按照从后往前的顺序渲染这些半透明物体，并开启它们的深度测试，但关闭深度写入。  
但是需要强调的是，还有可能会有其他特殊情况

**特殊情况**  
上文里面我们提到的情况还属于理想的状态,即物体之间的位置老老实实的分清楚前后。但是，实际上，我们还常常会碰到非透明物体不同部位之间互相重叠，有的在前有的在后，难分你我的情况。

**多个独立混合物体重叠**  
解决：拆分网格成多个不重叠的子物体，分开处理渲染尽量建模时就不要让物体有深度交叉（如尽量不要有凹面）。让透明通道更柔和，让异常穿插看起来不那么明显。使用开启深度写入的半透明效果来模拟半透明

**复杂结构的混合物体**  
方法实际上和上面类似，并且最常用的解决方法是最后一种，我们会在后面详细说明


### 透明必备知识点—设置深度写入和渲染队列

**设置深度写入**  
深度写入默认是开启的我们需要通过渲染状态中的ZWrite off 指令主动关闭深度写入当我们把它写在Pass渲染通道中时，它只会影响该Pass若我们把它写在SubShader语句块中，它将影响其中的所有Pass

**设置渲染队列**  
在Unity Shader中我们可以通过渲染标签主动的设置物体的渲染顺序Tags{ "Queue" = "标签值" }

1.Background(背景)(队列号:1000)  
最早被渲染的物体的队列，一般用来渲染天空盒或者背景

2.Geometry(几何)(队列号:2000)  
不透明的几何体通常使用该队列，不设置时的默认队列

3.AlphaTest(透明测试)(队列号:2450)  
需要透明度测试的物体使用的队列

4.Transparent(透明的)(队列号:3000)  
半透明物体的渲染队列，该队列中几何体按照由远到近的顺序进行绘制，所有进行透明混合的几何体都应该使用该队列

5.Overlay(覆盖)(队列号:4000)  
用是放在最后渲染的队列，于叠加渲染的效果

6.自定义队列  
基于Unity预先定义好的这些渲染队列标签来进行加减运算来定义自己的渲染队列 如：Tags{ "Queue" = "Geometry+1" } 代表的队列号就是2001渲染队列一般都定义在SubShader语句块中，影响之后的所有Pass渲染通道使用该Shader(着色器)的物体，就会根据你设置的渲染队列在特定的时间进行渲染在使用渲染队列Queue 时，一般会搭配忽视投影器IgnoreProjector 和渲染类型RenderType 一起使用

**忽视投影器IgnoreProjector 标签**  
投影器（Projector）是Unity中的一种特殊的光源，它用于在场景中投射纹理（Texture）或简单的几何形状（如圆形 或方形）来模拟光照、阴影或其他视觉效果。

**渲染类型RenderType 标签**  
对着色器进行分类，用于着色器替换功能，摄像机上有对应API，可以指定渲染类型替换成别的着色器 Opaque(不透明的)、 Transparent(透明的)、 TransparentCutout(透明切割)、 Background(背景)、 Overlay(覆盖)但是本身并不会对shader渲染造什么影响。


### 透明必备知识点—设置混合命令

**导入**  
在编写Shader时可以通过添加混合方式的渲染状态来控制源颜色和目标颜色如何进行混合计算混合默认是关闭的当我们在使用了Blend混合命令时（除Blend off），Unity内部就会自动的帮助我们开启混合我们在实现透明效果时，就需要设置混合方式这个渲染状态

**混合的基本原理**  
当我们在进行渲染时，当片元通过了深度测试后，会进入到混合流程中。在混合流程中：当前片元的颜色被称为源颜色(当前混合物体颜色)颜色缓冲区中的颜色被称为目标颜色（背景物体颜色）混合就是将源颜色和目标颜色用对应的混合算法进行计算后输出一个新的颜色更新到颜色缓冲区中注意：这些颜色都是RGBA，包含透明通道A

**混合的计算规则**  
我们假设当前片元的颜色被称为源颜色= S (source)颜色缓冲区中的颜色被称为目标颜色= D (destination)混合后的输出颜色= O（out）混合计算的规则就是需要构建两个混合等式

1. 计算RGB通道的混合等式     
   Orgb = 源因子 Srgb + 目标因子 Drgb
2. 计算A通道的混合等式     
   Oa = 源透明因子* Sa + 目标透明因子* Da

**混合因子**  
混合计算的规则就是需要构建两个混合等式

1. Orgb = 源因子 Srgb + 目标因子 Drgb
2. Oa = 源透明因子* Sa + 目标透明因子* Da     
   从渲染状态上体现（如下图）如果我们使用写法二来设置因子由于没有指定透明相关因子，因此，在计算时源透明因子= 源因子目标透明因子= 目标因子即
3. Orgb = 源因子 Srgb + 目标因子 Drgb
4. Oa = 源因子* Sa + 目标因子* Da     
   在Unity当中ShaderLab为我们提供了很多设定好的混合因子我们根据需求直接使用即可

**实战举例：**  
混合操作我们刚才学习的混合计算规则当中，都是使用对应混合因子和源颜色与目标颜色相乘后再相加。其实Unity当中还可以选择其他的计算方式来进行混合计算在ShaderLab当中除了可以使用Blend 混合命令来设定混合因子还提供了一个BlendOp 混合操作命令来设定混合的计算方式它的基本语法是在Unity当中ShaderLab为我们提供了很多设定好的混合操作我们根据需求直接使用即可

**总结**  
在进行渲染状态的混合相关设置中我们的步骤为：进行混合操作的设置（非必须，默认为Add）进行混合方式的设置（主要设置混合因子）常见的混合类型（套路写法）


### 透明度测试

**知识点一 透明测试是用于处理哪种透明需求的？**  
在游戏开发中对象的某些部位完全透明而其他部位完全不透明这种透明需求往往不需要半透明效果相对比较极端，只有看得见和看不见之分比如树叶、草、栅栏等等

**知识点二 透明测试的基本原理**  
基本原理：通过一个阈值来决定哪些像素应该被保留，哪些应该被丢弃具体实现：片元携带的颜色信息中的透明度（A值）不满足条件时（通常是小于某个阈值）该片元就会被舍弃，被舍弃的片元不会在进行任何处理，不会对颜色缓冲区产生任何影响满足条件时（通常是大于等于某个阈值）该片元会按照不透明物体的处理方式来处理阈值判断使用的方法：利用CG中的内置函数：clip(参数)该函数有重载，参数类型可以是 float4 float3 float2 float 等等如果传入的参数任何一个分量是负数就会舍弃当前片元它的内部实现会用到一个 discard 指令，代表剔除该片元 不再参与渲染void clip(float4 x)//模拟该API的内部指令{if(any(x < 0))discard;}

**知识点三 透明测试实现思路**

1.在属性中加一个阈值_Cutoff,取值范围为0~1，用来设定用来判断的阈值。并在CG中添加属性的映射成员

2.将渲染队列设置为AlphaTest，并配合IgnoreProjector和RenderType一起设置

3.在片元着色器中获取了颜色贴图颜色后，就进行阈值判断  
请注意，进行透明度测试的时候，不可以关闭深度写入，否则会直接无法正常显示在Game里面！下文会进行解释

**实现**

```hlsl
Shader "Unlit/colortext"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
          _Cutoff ("Cutoff", Range(0,1)) = 1
    }
    SubShader
    {
            Tags{  "Queue"="AlphaTest" "IgnoreProjector"="True"  }

        Pass
        {
             Tags{  "LightMode" = "ForwardBase"  }
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

    		sampler2D _MainTex;
            float4 _MainTex_ST;
            float _Cutoff;

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

        

            v2f vert (appdata_base v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
               o.uv=v.texcoord;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv);
             clip(col.a-_Cutoff);
       
                return col;
            }
            ENDCG
        }
    }
}
```

效果图注意在我测试上面的代码的时候，我发现一个有趣的问题，我像试一试透明测试对象能不能关闭深度写入，然后我发现：在Scene窗口是正常的，但是在Game无法正常显示目标对象！  
后来我搜索了Unity官方论坛，从一个专业图形学的用户回答里面得到了解答问题出在渲染排序小于2500后未开启深度写入的对象会直接被天空盒覆盖原因如下：

1.在Scene视图中，首先渲染天空盒（Skybox），然后是背景队列，接着是几何体（默认）队列等。一切都按照预期进行。  
然而，在Game视图中，背景队列会首先被渲染，然后是几何体和Alpha测试，最后才是天空盒的渲染。

2.而由于天空盒的顺序在Game里面大于2500，这最终导致了没有开启深度写入，而渲染层级通常会设置为AlphaTest（小于2500）的透明度测试对象就直接被天空盒覆盖了。  
综上所述：注意Scene和Game里面的渲染顺序差别结论：排序为2500内的对象，ZWrite On必须开启才可以在Game里面正常显示。查询地址：ZWrite off works only when reander Quere large than 2500(AlphaTest+50)? - Unity ForumSubShader with "ZWrite Off" visible in Scene View but not in Game Preview - Unity Forum后续发现国内也有人探讨：Unity天空盒渲染顺序及shader中的zwrite的设定_unity shader zwrite off-CSDN博客疑似Unity5.X开始天空盒的层级才被设置成了2500.5，原因等待调查。


### 透明度混合

**知识点一 透明度混合是用来处理哪种需求的？**  
上一节我们学习的透明度测试，并不能用于实现半透明效果它只存在看得见和完全看不见两种状态，一般用来处理镂空效果而这节课要学习的透明度混合，主要就是用来实现半透明效果的

**知识点二 透明度混合的基本原理**  
基本原理：关闭深度写入，开启混合，让片元颜色和颜色缓冲区中颜色进行混合计算具体实现：

1. 采用半透明的混合因子进行混合 Blend SrcAlpha OneMinusSrcAlpha     
   因此目标颜色 = SrcAlpha 源颜色 + (1-SrcAlpha)目标颜色= 源颜色透明度 源颜色 + (1-源颜色透明度) 目标颜色
2. 声明一个0~1区间的_AlphaScale用于控制对象整体透明度

**知识点三 透明度混合实现**

1.在属性中加一个阈值_AlphaScale,取值范围为0~1，用来设定对象整体透明度。并在CG中添加属性的映射成员

2.将渲染队列设置为Transparent，并配合IgnoreProjector和RenderType一起设置

3.关闭深度写入Zwrite off，设置混合因子Blend SrcAlpha OneMinusSrcAlpha

4.在片元着色器中获取了颜色贴图颜色后，修改最后返回颜色的A值为 纹理.a * \_AlphaScale

**实现**

```hlsl
Shader "Unlit/colortext"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
          _AlphaScale ("AlphaScale", Range(0,1)) = 1
    }
    SubShader
    {
            Tags{  "Queue"="Transparent" "IgnoreProjector"="True"  }
            Zwrite off
        Pass
        {
            Blend SrcAlpha OneMinusSrcAlpha
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            sampler2D _MainTex;
            float4 _MainTex_ST;
            float _AlphaScale;
            
            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

          

            v2f vert (appdata_base v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
               o.uv=v.texcoord;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv);
               col.a*=_AlphaScale;
       
                return col;
            }
            ENDCG
        }
    }
}
```

效果图


### 开启深度写入的半透明效果

**知识点一 开启深度写入的半透明效果是用来处理哪种需求的？**  
如上图所示对于本身结构较为复杂的模型使用之前的透明度混合Shader会由于关闭了深度写入会产生错误的渲染效果虽然我们可以通过拆分模型的方式解决部分问题但是对于一些结构复杂的模型，拆分模型的方式会增加工作量因此我们可以采用 开启深度写入的半透明Shader 来优化效果

**知识点二 开启深度写入的半透明效果的基本原理**  
基本原理：使用两个Pass渲染通道来处理渲染逻辑第一个Pass：开启深度写入，不输出颜色目的是让该模型各片元的深度值能写入深度缓冲第二个Pass：进行正常的透明度混合（和上节课一样）这样做的话，当执行第一个Pass时，会执行深度测试，并进行深度写入如果此时该片元没有通过深度测试会直接丢弃，不会再执行第二个Pass对于同一个模型中处于屏幕同一位置的片元们，会进行该位置的深度测试再决定渲染哪个片元如何做到不输出颜色？  
使用 ColorMask 颜色遮罩 渲染状态(命令)它主要用于控制颜色分量是否写入到颜色缓冲区中ColorMask RGBA 表示写入颜色的RGBA通道ColorMask 0 表示不写入颜色ColorMask RB 表示只写入红色和蓝色通道注意：

1.开启深度写入的半透明效果，模型内部之间不会有任何半透明效果（因为模型内部深度较大的片元会被丢弃掉）

2.由于有两个Pass渲染通道，因此它会带来一定的性能开销

**知识点三 实现 开启深度写入的半透明效果**

1.在SubShader中之前的Pass渲染通道前面加一个Pass渲染通道

2.在新加Pass渲染通道中开启深度写入，并且使用 ColorMask 0 颜色遮罩 渲染命令，不输出颜色

**实现**

```hlsl
Shader "Unlit/colortext"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
     
    }
    SubShader
    {
            Tags{  "Queue"="Transparent" "IgnoreProjector"="True"  }
            
        Pass
        {
         Zwrite On
         ColorMask 0
        }
        Pass
        {
            Zwrite off
            Blend SrcAlpha OneMinusSrcAlpha
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            sampler2D _MainTex;
            float4 _MainTex_ST;
       
            
            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

          

            v2f vert (appdata_base v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
               o.uv=v.texcoord;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv);
                return col;
            }
            ENDCG
        }
    }
}
```

效果图


### 双面渲染的透明效果

**知识点一 双面渲染的透明效果用来处理哪种需求的？**  
对于现实世界的半透明物体，我们不仅可以透过它看到其他物体的样子也可以看到这个物体自己的内部结构但是我们之前实现的 透明度测试 和 透明度混合 相关Shader都无法看到模型的内部结构而双面渲染的透明效果Shader就是来解决该问题的让我们不仅可以透过半透明物体看到其他物体的样子还可以看到自己的内部结构

**知识点二 双面渲染的透明效果的基本原理**  
基本原理：默认情况下，Unity会自动剔除物体的背面，而只渲染物体的正面双面渲染的基本原理就是利用我们之前学习过的 Cull 剔除指令来进行指定操作Cull Back 背面剔除Cull Front 正面剔除Cull Off 不剔除不设置的话，默认为背面剔除对于透明度测试Shader他的透明是剔除的点形成的，而被剔除的点是不会深度写入的所以我们无需担心正面透明效果的点会挡住背面的渲染并且由于它无需混合，我们也不需要额外的混合处理因此我们直接 关闭剔除即可对于透明度混合Shader由于它需要进行混合需要使用两个Pass，一个用于渲染背面，一个用于渲染正面两个Pass中除了剔除命令不同 其他代码和之前一致

**知识点三 实现**  
透明度测试

1.复制透明度测试相关Shader代码

2.在Pass中关闭剔除 Cull Off

```hlsl
Shader "Unlit/colortext"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
          _Cutoff ("Cutoff", Range(0,1)) = 1
    }
    SubShader
    {
            Tags{  "Queue"="AlphaTest" "IgnoreProjector"="True"  }
             
        Pass
        {
            Cull off//关闭剔除
            
             Tags{  "LightMode" = "ForwardBase"  }
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

           sampler2D _MainTex;
            float4 _MainTex_ST;
            float _Cutoff;

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

        

            v2f vert (appdata_base v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
               o.uv=v.texcoord;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                fixed4 col = tex2D(_MainTex, i.uv);
             clip(col.a-_Cutoff);
       
                return col;
            }
            ENDCG
        }
    }
}
```

透明度混合

1.复制透明度混合相关Shader代码

2.复制之前的Pass，变成两个一模一样的Pass

3.在第一个Pass中剔除正面 Cull Front，在第二个Pass中剔除背面Cull Back  
相当于一个片元先渲染背面再渲染正面效果图左为双面渲染处理前，右为双面渲染处理后透明度测试（效果图略）透明度混合（背面和背面后面的对象，都是可以正常混合的）


## 第 4 节 渲染路径

渲染路径概述导入我们在之前的Shader入门相关知识的学习中已经学习过光照模型相关知识，我们目前已经可以实现出一些基础的光照表现效果但是需要注意的是，我们之前实现的那些光照相关Shader都不能直接应用到项目开发中，因为它还不够完善。  
比如，之前的光照模型相关知识中，我们在Shader中只考虑了场景中的平行光，如果我们的场景中没有平行光会导致之前的Shader出现错误的表现效果。在实际的游戏开发中，我们场景中的光源肯定是更多、更复杂的！  
一个平行光的处理，完全不能满足我们的需求。因此之前关于光照模型的相关学习，只是为了给我们打下一个基础，让我们能够理解光照处理的底层逻辑是光照模型的计算。要处理更多的光源，我们就需要了解Unity底层是如何处理这些光源的，我们将首先学习渲染路径相关的知识来了解这一点渲染路径是什么渲染路径（Rendering Path）是指在图形渲染过程中，图形引擎按照特定的步骤和顺序来处理场景中的几何、光照、材质等信息，最终生成屏幕上的图像的一种算法或策略。  
它决定了图形引擎如何组织和执行渲染过程，以产生最终的视觉效果。对于我们来说：在Unity中，渲染路径决定了光照如何应用到Unity Shader中，如果要在UnityShader中和光源打交道，我们需要为每个Pass渲染通道匹配对应的渲染路径，这样才能在Shader当中获取到正确的光源数据进行处理。  
总而言之：渲染路径会影响光照处理，从而影响最终的渲染效果（光照、阴影等）。并且存在多种不同的渲染路径为什么会影响：我们可以简单理解，使用不同的渲染路径时，Unity在Shader中准备光源数据的流程是不同的，那么我们在Shader开发时，获取光源数据的方式就会有所不同渲染路径的种类和设置我们可以在Camera组件中的Rendering Path（渲染路径）对其进行修改在内置渲染管线中主要有3种渲染路径，分别是：

1. Forward（前向渲染路径）     
   默认的标准的渲染方式，适用于相对简单的场景和较少的光源
2. Deferred（延迟渲染路径）     
   可以处理较复杂的场景，有大量光源时可以提供更好的性能
3. Legacy Vertex Lit（(遗产)顶点照明渲染路径）     
   较适用于为简单的渲染方式，适用于性能受限的场景。基本已经不会使用注意：当显卡不支持选定的渲染路径时会自动选择一个较低精度的渲染路径比如不支持延迟渲染路径时，前向渲染路径会被采用LightMode标签的作用我们之前在编写Shader时都会使用LightMode（光模式）标签。     
   它的主要作用就是来指明该Pass 匹配的渲染路径是哪种只要匹配正确，我们便可以获取到正确的光源相关数据注意：LightMode标签通常应该与Camera中的Rendering Path匹配用于指定Pass在渲染过程中的哪个阶段如果它们不匹配，可能导致渲染不正确LightMode 标签支持的渲染路径设置选项有其中向前渲染需要写两种，也就是需要两个pass。     
   在Shader开发中如果我们不进行LightMode渲染标签的设置。比如摄像机默认的是前向渲染路径，但是我们没有为Pass设置相关的标签，那么这个Pass会被当作一个顶点照明渲染路径的Pass。     
   这时光源相关的数据就不会被正确的进行赋值，我们计算出来的结果就会出现错误，从而可能呈现出错误的渲染效果。因此我们需要进行正确的LightMode标签设置，从而匹配当前使用的渲染路径前向渲染路径前向渲染路径处理光照的方式前向渲染路径中会将光源分为以下3种处理方式：逐像素处理（需要高等质量处理的光）逐顶点处理（需要中等质量处理的光）球谐函数（SH）处理（需要低等质量处理的光）球谐函数处理光照的方式是将光照场景投影到球谐函数的空间中，通过一组球谐系数来表示光照。     
   内存换性能，细节表现效果差（不需要我们自己书写，Unity底层会帮助我们进行处理）场景当中的各种光源将采用哪种方式处理在前向渲染中，一部分最亮的灯光以逐像素处理，然后4个点光源以逐顶点方式处理，其余的灯光以SH处理一个光源是逐像素、逐顶点还是SH处理主要取决于以下几点：
4. 渲染模式设置为Not Important(不重要)的灯光始终以     
   逐顶点或者SH的方式渲染

2.渲染模式设置为Important(重要)的灯光始终是逐像素渲染

3.最亮的平行光总是逐像素渲染

4.如果逐像素光照的灯光数量少于项目质量设置中的  
Pixel Light Count(像素灯光计数)的数量，那么其余比较亮的灯光将会被逐像素渲染注意：如果灯光渲染模式设置为Auto(自动)，Unity会根据灯光的亮度以及与物体的距离自动判断重要性举例说明：如下图 A~~H是8个具有相同的颜色和强度 并且光源渲染模式都是Auto(自动)的其中最亮的光源以逐像素光照模式渲染（A~~D）然后最多4个光源以逐顶点光照模式渲染（D~~G）最后剩下的光源以SH模式渲染（G~~H）注意：灯光D既是逐像素也是逐顶点处理灯光G既是逐顶点也是SH处理是因为物体移动或灯光移动时，不同渲染模式的灯其中最亮的光源以逐像素光照模式渲染（A~~D）然后最多4个光源以逐顶点光照模式渲染（D~~G）最后剩下的光源以SH模式渲染（G~H）光交界处会出现明显瑕疵，为了避免该问题Unity将不同灯光组之间进行了重叠简而言之：Unity当中有一套划分光源“三六九等”的规则主要通过灯光渲染模式、项目质量设置中的像素灯光计数的数量、光照强度、距离物体距离来综合判定在前向渲染路径中会将光源分成所谓的逐像素、逐顶点、SH三种处理类型有了对光源的“高中低”的身份认知Unity底层就可以将这些光源的数据存储到Shader中对应的内置变量中我们就可以通过这些内置变量获取到对应“身份”的光源数据从而进行差异化的处理前向渲染路径在哪里进行光照计算要进行光照计算，那肯定是在Shader当中的Pass渲染通道中进行计算。  
但是对于前向渲染来说，有两种Pass可以用来进行光照处理：

1.Base Pass（基础渲染通道）  
渲染物体的主要光照通道，用于处理主要的光照效果主要用于计算逐像素的平行光以及所有逐顶点和SH光源可实现的效果：漫反射、高光反射、自发光、阴影、光照纹理等

2.Additional Pass（附加渲染通道）  
渲染物体额外的光照通道，用于处理一些附加的光照效果主要用于计算其他影响物体的逐像素光源每个光源都会执行一次该Pass可实现的效果：描边、轮廓、辉光等对于一个前向渲染路径下的Unity Shader通常会定义一个Base Pass（基础渲染通道）以及一个Additional Pass（附加渲染通道）每次渲染时一个Base Pass仅会执行一次（多个Base Pass情况除外）主要用于渲染环境光或自发光等而一个Additional Pass会根据影响该物体的其他逐像素光源的数量被多次调用每个逐像素光源都会调用一次Additional Pass 由于开启了混合，渲染结果会和之前的光照颜色进行混合注意：

1. Base Pass也可以有多个，比如需要双面渲染的情况
2. Base Pass默认支持阴影， Additional Pass默认不支持     
   可以通过添加#pragma multi_compile_fwdadd_fullshadows编译指令开启阴影
3. 这些Pass当中我们具体处理光照的方式是由我们自己决定的，使用逐顶点光照还是逐像素光照的计算方式     
   都根据我们的具体实现而定，前文提到的逐像素光源只是按照期望处理类型来分的而已前向渲染路径的内置光照变量和函数常用内置光照变量常用内置光照函数顶点照明渲染路径顶点照明渲染路径处理光照的方式顶点照明渲染路径仅仅是前向渲染路径的一个子集所有在顶点照明渲染路径中能实现的效果都可以在前向渲染路径中实现它对硬件配置要求最少、运算性能最高，但是效果是最差的它不支持那些逐像素才能得到的效果，比如阴影、法线纹理、高精度高光反射等它的基本思想就是所有的光都按照逐顶点的方式进行计算的在内置渲染管线中，它只会最多记录8个光源的数据，会根据光源类型、强度、距离等因素来决定Unity中的顶点照明渲染路径只会将光相关的数据填充到那些逐顶点相关的内置光源变量中意味着我们不能像前向渲染路径中那样使用逐像素相关的内置变量注意：顶点照明渲染路径使用场景较少，我们主要做了解顶点照明渲染路径在哪里进行光照计算要进行光照计算，那肯定是在Shader当中的Pass渲染通道中进行计算。     
   顶点照明渲染路径通常在一个Pass当中就可以完成对物体的渲染。在这个Pass当中我们会计算我们关心的所有光源对该物体的影响并且会按照逐顶点的方式一次性对所有光照去进行计算因此它是Unity内置渲染管线当中最快速的渲染路径，并且具有最广泛的硬件支持只是相对来说渲染效果最差顶点照明渲染路径的内置光照变量和函数常用内置光照变量常用内置光照函数延迟渲染路径延迟渲染路径处理光照的方式延迟渲染路径对光照的数量没有任何限制，并且所有灯光都可以采用逐像素渲染。理论上来说，即使场景中有成百上千个实时灯光，依然可以保持比较流畅的渲染帧率。     
   它支持法线纹理、阴影等等效果的处理；但是它不能处理半透明物体，并且不支持真正的抗锯齿。这些会自动使用前向渲染路径。延迟渲染的效率不依赖于场景的复杂度，而是和我们使用的屏幕空间的大小有关。     
   这是因为延迟渲染路径中除了使用颜色缓冲和深度缓冲外，还会利用一个叫做G缓存的额外缓存区它会存储我们关心的表面（通常是离摄像机最近的表面）的其他信息，比如表面的法线、位置、材质属性等等。总之我们需要的信息都存储到缓冲区中，而这些缓冲区可以理解为一张张的2D图片，我们实际上是在这些图像空间中进行处理的。     
   延迟渲染路径在哪里进行光照计算要进行光照计算，那肯定是在Shader当中的Pass渲染通道中进行计算。延迟渲染路径中主要包含两个Pass第一个Pass（对于每个物体，该Pass只会执行一次，通常无需我们自己实现）主要判断哪些片元可见，并且将可见片元的相关信息存储到G缓冲区中比如：表面法线、视角方向、漫反射系数等等数据第二个Pass利用G缓冲区中各个片元的相关信息进行真正的相关计算，最终将颜色写入颜色缓冲区注意：在第二个Pass中计算光照时，默认情况下只能用Unity内置的标准(Standard)光照模型计算顶点照明渲染路径的内置光照变量常用内置光照变量渲染路径对比各渲染路径处理光照的区别各渲染路径对于我们来说的最大区别就是它们对光照的处理不同前向渲染路径有一套划分光源“三六九等”的规则，将光源分成了高中低三种身份主要通过灯光渲染模式、项目质量设置中的像素灯光计数的数量、光照强度、距离物体距离来综合判定顶点照明渲染路径基本思想就是所有的光都按照逐顶点的方式进行计算的在内置渲染管线中，它只会最多记录8个光源的数据只会将光相关的数据填充到那些逐顶点相关的内置光源变量延迟渲染路径对光照的数量没有任何限制,并且所有灯光都可以采用逐像素渲染它不能处理半透明物体，并且不支持真正的抗锯齿各渲染路径Pass处理的区别前向渲染路径Base Pass（基础渲染通道）：主要用于处理影响该物体的一个高质量光源（平行光）、所有中（逐顶点处理）低质量（SH处理）光源等Additional Pass（附加渲染通道）：主要用于处理影响该物体的除平行光以外的其它高质量光源（每个高质量光源都会调用）顶点照明渲染路径在一个Pass当中按照逐顶点的方式一次性对所有光照去进行计算延迟渲染路径第一个Pass（对于每个物体，该Pass只会执行一次，通常无需我们自己实现）主要判断哪些片元可见，并且将可见片元的相关信息存储到G缓冲区中第二个Pass利用G缓冲区中各个片元的相关信息进行真正的相关计算，最终将颜色写入颜色缓冲区各渲染路径的优缺点前向渲染路径优点：适用于相对简单的场景和较少数量的光源，基本可以实现任何渲染效果。设备支持率较高缺点：对于复杂场景和大量光源的情况性能消耗相对较大顶点照明渲染路径优点：相对来说，性能开销较小，适用于资源受限设备极差时的轻量级渲染情景缺点：表现效果较差，光照计算精度较低延迟渲染路径优点：适用于复杂场景和大量光源，能够有效减少光照计算的开销缺点：对于透明物体和一些特殊效果不能直接支持，需要复杂的处理。并且对硬件有一定要求，不是所有设备都支持，在一些性能较差的移动设备上不受支持

**总结**  
在选择渲染路径时，我们应该根据项目的实际情况去进行考虑比如

1.针对的平台

2.场景的复杂度  
等等


## 第 5 节 多种光源处理

多种光源导入学习本节前，建议先去复习：Unity入门 - 张先生的小屋 (klned.com)里面的光源系统，位于第四大节第六小节。Shader开发中常用的光源属性我们之前在讲解光照模型相关知识点时，场景中仅仅只有一个光源，并且光源类型为平行光。  
但是在Unity当中一共支持四种光源类型：平行光（Directional）点光源（Point）聚光灯（Spot）面光源（Area）— 面光源仅在烘焙时有用，因此我们不讨论它不管光源类型到底是什么，我们在Shader开发当中经常会使用到的光源相关属性有：位置、方向、颜色、强度、衰减也就是说我们在Shader中处理光照效果时，经常会用到这些光的属性参与到计算当中对比平行光、点光源、聚光灯平行光充当角色：太阳照射范围：无限制特点：

1. 它不存在固定的位置
2. 它的重要属性只有方向（可以通过Transform的Rotation属性来改变方向）
3. 它到场景中所有点的方向都是一样的
4. 由于它没有位置，因此它没有衰减的概念（光的强度不会随着距离而发生变化）     
   点光源充当角色：灯泡、烛光等照射范围：有限特点：
5. 它的光是由一个点发出的，向四面八方延伸的光
6. 它的范围由参数Range来决定
7. 它的位置由Transform中的Position来决定
8. 它存在衰减，随着物体离点光源距离决定衰减强弱     
   聚光灯充当角色：探照灯、手电筒等照射范围：有限特点：
9. 它的光范围由空间中的一块锥形区域定义
10. 它的范围由参数Range和Spot Angle 共同决定
11. 它的位置由Transform中的Position来决定
12. 它存在衰减，随着物体离聚光灯距离决定衰减强弱。      
    但是它相对点光源衰减计算公式更复杂，因为需要点是否在锥形范围内如何在Shader中判断光源类型导入学习本节前，建议先去复习：C#进阶 - 张先生的小屋 (klned.com)内的预处理器指令

**知识点一 前向渲染路径中我们主要关注处理什么内容？**  
两个Pass

1.Base Pass（基础渲染通道，每个片元只会计算一次）：  
只需要处理一个逐像素平行光源（一般场景中最亮会自动赋值对应变量）其他的中（逐顶点）、低质量（SH）光源Unity会帮助我们处理

2.Additional Pass（附加渲染通道）:  
除了最亮的平行光、其他高质量的光源(可能是平行光、点光源、聚光灯)都会调用一次该Pass进行计算因此我们一般需要在Additional Pass中判断光源类型来分别处理部分逻辑

**知识点二 如何在Shader中判断光源类型**  
Unity中提供了三个重要的宏分别是：DIRECTIONALLIGHT:平行光POINTLIGHT:点光源SPOTLIGHT:聚光灯宏在这里的作用：可以用于在编译时根据条件判断来包含或排除不同的代码块，实现条件编译我们可以使用这些宏配合Unity Shader中的条件编译预处理指令用于在编译时根据一定的条件选择性地包含或排除代码块#if defined(\_DIRECTIONAL_LIGHT)平行光逻辑#elif defined (\_POINT_LIGHT)点光源逻辑#elif defined (\_SPOT_LIGHT)聚光灯逻辑#else其他逻辑#endifUnity底层会根据该条件编译指令生成成多个 Shader Variants（着色器变体）这些 Variants 变体共享相同的核心代码但根据条件编译的选择会包含不同的代码块Shader variants 的基本概念是在编写 shader 时，通过条件编译指令（#if, #elif, #else, #endif）根据不同的配置选项生成多个版本的 shader。  
这些不同版本的 shader 称为 shader variants。光照衰减

**知识点一 光照衰减的基本概念**  
光照衰减通常指的是在渲染过程中考虑光线在空间中传播时的减弱效应比如：任何光源的光照亮度会随着物体离光源的距离增加而迅速衰减一般常见的光照衰减计算方式有

1.线性衰减  
光强度与距离成线性关系。即光照衰减与光源到被照射表面的距离成正比。

2.平方衰减  
光强度与距离的平方成反比。这种模型更符合现实世界中光照的特性，因为光在空间中的传播过程中通常会遵循平方衰减规律。

**知识点二 Unity中的光照衰减**  
Unity中为了提升性能，我们一般不会直接通过数学公式计算光照衰减而是使用一张纹理作为查找表（LUT, lookup table） 在片元着色器中计算逐像素光照的 衰减Unity Shader中有一个内置的纹理类型的变量 \_LightTexture0该纹理中存储了衰减值相关的数据Unity 内部预先就计算好了相关数据 并存入到该纹理中，避免重复计算，提升性能表现其中它的对角线上的纹理颜色值，表明了光源空间中不同位置的点对应的衰减值纹理中的对角线起点 (0,0) 位置，表示和光源重合的点的衰减值终点（1,1) 位置，表示在光源空间中离光源距离最远的点的衰减值一般我们直接从_LightTexture0中进行纹理采样后，利用其中的UNITY_ATTEN_CHANNEL宏来得到衰减值所在的分量tex2D(\_LightTexture0, 对应纹理uv坐标).UNITY_ATTEN_CHANNEL注意:如果光源存在cookie，也就是灯光遮罩那么衰减查找纹理便是 \_LightTextureB0

**知识点三 光源空间变换矩阵**  
Unity Shader中 内置的 光源空间变换矩阵是用于将世界空间下的位置转换到光源空间下（光源位置为原点）的老版本：\_LightMatrix0新版本：unity_WorldToLight由于我们需要从 \_LightTexture0 光照纹理中取出对应的衰减数据因此我们需要将顶点位置从世界空间中转换到光源空间中然后再来从其中取得衰减数据我们可以通过矩阵运算将世界空间下的点转换到光源空间下mul(unity_WorldToLight, float4(worldPos, 1));

**总结**  
在Shader中进行光照衰减处理时我们是通过从纹理中取出衰减数据不使用灯光遮罩时，从 \_LightTexture0 纹理中获取使用时，从 \_LightTextureB0 纹理中获取在纹理采样之前我们需要将顶点坐标从世界空间中转换到光源空间中变换矩阵为：老版本：\_LightMatrix0新版本：unity_WorldToLight点光源衰减计算注意一般点光源我们不会为其添加cookie光照遮罩一般想要使用光照遮罩都会在聚光灯中使用因此我们不用考虑cookie纹理的问题步骤第一步将顶点从世界空间转换到光源空间float3 lightCoord = mul(unity_WorldToLight, float4(worldPos, 1)).xyz;lightCoord 是光源坐标系下顶点根据光源的范围range规范化后的坐标相当于是一个模长为0~1之间的向量第二步利用该光源空间下的坐标来计算离光源的距离并利用距离参数，从光源纹理中采样fixed atten = tex2D(\_LightTexture0, dot(lightCoord, lightCoord).xx).UNITY_ATTEN_CHANNEL;dot(LightCoord, LightCoord).xx 中dot(LightCoord, LightCoord) 是为了通过点乘得到 结果 x² + y² + z² = 离光源距离 distance²xx是一种特殊写法，目的是构建一个 float2 代表uv坐标这里的 uv坐标 相当于是 (distance², distance²)用distance²做uv坐标，而不是distance为了避免开平方带来性能消耗采用这种平方衰减更符合现实世界中光照的特性因为人眼对亮部不敏感，而对暗部敏感，这样我们就可以将 衰减值的精度 集中在比较远的地方distance是0.5时，distance2是0.25，这样LUT查找表中大部分值都会留给比较远的部分聚光灯衰减计算

**知识点一 聚光灯默认Cookie**  
我们知道灯光组件中有一个Cookie参数，是用来关联光照遮罩图片的对于平行光和点光源，默认是不会提供任何光照遮罩信息的但是对于聚光灯来说Unity会默认为它提供一个Cookie光照遮罩主要是用于模拟聚光灯的区域性而此时 光照纹理中_LightTexture0 存储的是Cookie纹理信息_LightTextureB0 存储的是光照纹理信息，里面包含衰减值因此

1.获取聚光灯衰减值时  
需要从_LightTextureB0中进行采样

2.获取遮罩范围相关数据时  
需要从_LightTexture0中进行采样我们需要获取所有的信息，进行颜色叠加计算

**知识点二 聚光灯衰减计算**  
第一大步将顶点从世界空间转换到光源空间float4 lightCoord = mul(unity_WorldToLight, float4(i.worldPos, 1));注意：这里我们转换后和点光源不同的是点光源只会获取其中的xyz而聚光灯会获取其中的xyzw这是因为在聚光灯光源空间下的w值有特殊含义会参与后续的计算第二大步利用光源空间下的坐标信息我们会通过3个步骤去获取聚光灯的衰减信息fixed atten = (lightCoord.z > 0) * //第一步tex2D(\_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w * //第二步tex2D(\_LightTextureB0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL; //第三步我们首先分析和点光源相同的部分——第三步tex2D(\_LightTextureB0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;这一步的规则和点光源规则一致，直接根据距离光源原点距离的平方从光照纹理中获取衰减值需要注意的是聚光灯的光照衰减纹理为_LightTextureB0dot函数只会计算xyz，w不会计算接着我们来分析用于进行范围判断的部分——第一、二步第一步：(lightCoord.z > 0)CG语法中没有显示的bool类型，一般情况下 0 表示false，1表示true也就是说lightCoord.z > 0的返回值，条件满足时为1，条件不满足为0这里的z代表的其实是 目标点 相对于 聚光灯照射面 距离如果 lightCoord.z <= 0 证明在聚光灯照射方向的背面，就不应该受到聚光灯的影响也就是说这一步的主要作用，是用来决定顶点是否受到聚光灯光照的影响第二步：tex2D(\_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w我们以前在进行纹理采样时都会进行一个 先缩放 后 平移 的操作比如：uv = v.texcoord.xy * MainTexST.xy + MainTexST.zw;而第二步中的 lightCoord.xy / lightCoord.w + 0.5 其实也是在做这样的一个操作（本质是个齐次除法）这样做的主要目的是因为：我们需要把uv坐标映射到0~~1的范围内再从纹理中采样lightCoord.xy / lightCoord.w 进行缩放后 x,y的取值范围是-0.5~~0.5之间再加上0.5后，x,y的取值范围就是0~1之间，便可以进行正确的纹理采样了而lightCoord.xy / lightCoord.w 是因为聚光灯有很多横截面我们需要把各横截面映射到最大的面上进行采样

**总结**  
看似复杂的聚光灯光照衰减计算方式其实就是由 “遮罩衰减” 和 距离衰减 共同决定的第一步：判断是否能有机会照到光 看得到为1，看不到为0fixed atten = (lightCoord.z > 0) \*第二步：缩放平移，映射到遮罩纹理采样 根据遮罩纹理的信息决定衰减叠加tex2D(\_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w \*第三步：从光照衰减纹理中取出按距离得到的衰减值tex2D(\_LightTextureB0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;多种光源综合实现步骤

1.新建一个Shader文件，删除其中无用代码

2.复制Blinn-Phong光照模型的逐片元光照

3.其中已存在的Pass，就是我们的BasePass（基础渲染通道）  
我们需要为它加上一个编译指令#pragma multi_compile_fwdbase该指令可以保证我们在Shader中使用光照衰减等光照等变量可以被正确赋值并且会帮助我们编译 BasePass 中所有变体

4.复制BasePass，基于它来修改我们的Additional Pass（附加渲染通道）

5.LightMode 改为 ForwardAdd

6.加入混合命令Blend One One 表示开启混合 线性减淡效果

7.加入编译指令#pragma multi_compile_fwdadd  
该指令保证我们在附加渲染通道中能访问到正确的光照变量并且会帮助我们编译Additional Pass中所有变体

8.修改相关代码，基于不同的光照类型来计算衰减值  
8-1:光的方向计算方向修改8-2:基于不同光照类型计算衰减值实现

```hlsl
Shader "Unlit/Lesson64_ForwardLighting"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        //高光反射颜色  光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0, 20)) = 1
    }
    SubShader
    {
        //Bass Pass 基础渲染通道
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            //得到兰伯特光照模型计算的颜色 （逐片元）
            fixed3 getLambertFColor(in float3 wNormal)
            {
                //得到光源单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //计算除了兰伯特光照的漫反射颜色
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(wNormal, lightDir));

                return color;
            }

            //得到Blinn Phong式高光反射模型计算的颜色（逐片元）
            fixed3 getSpecularColor(in float3 wPos, in float3 wNormal)
            {
                //1.视角单位向量
                //float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - wPos );
                float3 viewDir = normalize(UnityWorldSpaceViewDir(wPos));

                //2.光的反射单位向量
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);

                //半角方向向量
                float3 halfA = normalize(viewDir + lightDir);
                
                //color = 光源颜色 * 材质高光反射颜色 * pow( max(0, dot(视角单位向量, 光的反射单位向量)), 光泽度 )
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(wNormal, halfA)), _SpecularNum );

                return color;
            }

            v2f vert (appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //计算兰伯特光照颜色
                fixed3 lambertColor = getLambertFColor(i.wNormal);
                //计算BlinnPhong式高光反射颜色
                fixed3 specularColor = getSpecularColor(i.wPos, i.wNormal);

                //衰减值
                fixed atten = 1;
                //物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色
                //衰减值 会和 漫反射颜色 + 高光反射颜色 后 再进行乘法运算
                fixed3 blinnPhongColor = UNITY_LIGHTMODEL_AMBIENT.rgb + (lambertColor + specularColor)*atten; 

                return fixed4(blinnPhongColor.rgb, 1);
            }
            ENDCG
        }

     //Additional Pass 附加渲染通道
        Pass
        {
            Tags
            {
                "LightMode"="ForwardAdd"
            }
            //线性减淡的效果 进行 光照颜色混合
            Blend One One

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdadd

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            v2f vert(appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                return v2fData;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                //兰伯特漫反射
                fixed3 worldNormal = normalize(i.wNormal);
                //平行光 光的方向 其实就是它的位置
                #ifdef USING_DIRECTIONAL_LIGHT
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                #else //点光源和聚光灯 光的方向 是 光的位置 - 顶点位置
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz - i.wPos);
                #endif
                // 漫反射颜色 = 光颜色 * 属性中颜色 * max(0, dot(世界坐标系下的法线, 世界坐标系下的光方向));
                fixed3 diffuse = _LightColor0.rgb * _MainColor.rgb * max(0, dot(worldNormal, worldLightDir));

                //BlinnPhong高光反射
                //视角方向
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos.xyz);
                //半角方向向量
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                // 高光颜色 = 光颜色 * 属性中的高光颜色 * pow(max(0, dot(世界坐标系法线, 世界坐标系半角向量)), 光泽度);
                fixed3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(
                    max(0, dot(worldNormal, halfDir)), _SpecularNum);

                //衰减值
                #ifdef USING_DIRECTIONAL_LIGHT
                    fixed atten = 1;
                #elif defined(POINT)
                    //将世界坐标系下顶点转到光源空间下
                    float3 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1)).xyz;
                    //利用这个坐标得到距离的平方 然后再再光源纹理中隐射得到衰减值
                    fixed atten = tex2D(_LightTexture0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL;
                #elif defined(SPOT)
                    //将世界坐标系下顶点转到光源空间下 聚光灯需要用w参与后续计算
                    float4 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1));
                    fixed atten = (lightCoord.z > 0) * //判断在聚光灯前面吗
                                  tex2D(_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w * //映射到大图中进行采样
                                  tex2D(_LightTextureB0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL; //距离的平方采样
                #else
                fixed atten = 1;
                #endif

                //在附加渲染通道中不需要在加上环境光颜色了 因为它只需要计算一次 在基础渲染通道中已经计算了
                return fixed4((diffuse + specular) * atten, 1);
            }
            ENDCG
        }
    }
}
```


## 第 6 节 阴影

阴影—基本原理阴影—基本原理.pdf要点是光源摄像头深度图的比较然后将目标顶点也转换到光源空间下和深度图里面的深度进行比较。即可得到是否被阴影遮挡Unity中阴影的实现原理Unity中阴影的实现原理.pdf如果用到sssm技术，还会多一个当前屏幕的深度图Unity有使用此技术由于比起普通技术，sssm只需要对屏幕空间像素的顶点进行光源空间转换，所以可以节约性能。（普通技术需要对所有计算的顶点执行空间转换）但是值得一提的是——sssm很容易在某些场合出现bug——比如，它无法得知屏幕之外顶点的信息，这直接导致屏幕外遮挡物的阴影就无法判定了。  
不透明物体阴影——让物体投射阴影

**知识点一 感受Fallback的作用**  
我们新建一个材质球将其的Shader设置为我们之前编写的多种光源综合实现ShaderLesson64_ForwardLighting并将该材质球赋值给较大的立方体使用我们会发现该立方体不再投射阴影也不再接受阴影

1.不投射阴影的原因  
该Shader中没有LightMode为ShaderCaster的Pass，无法参与光源的阴影映射纹理的计算

2.不接收阴影的原因  
该Shader并没有对阴影映射相关纹理进行采样，没有进行阴影相关颜色运算我们之前学习理论知识时提到过Unity会寻找LightMode为ShaderCaster的Pass来进行处理，如果该Shader没有该Pass会在它FallBack指定的Shader中寻找，直到找到为止我们现在在该Shader 最后加上FallBack "Specular"便可以让该立方体投射阴影

**知识点二 让物体投射阴影**  
物体向其它物体投射阴影的关键点是：

1. 需要实现 LightMode(灯光模式) 为 ShadowCaster(阴影投射) 的 Pass(渲染通道)     
   这样该物体才能参与到光源的阴影映射纹理计算中
2. 一个编译指令，一个内置文件，三个关键宏     
   编译指令：#pragma multi_compile_shadowcaster该编译指令时告诉Unity编译器生成多个着色器变体用于支持不同类型的阴影（SM，SSSM等等）可以确保着色器能够在所有可能的阴影投射模式下正确渲染内置文件：#include "UnityCG.cginc"其中包含了关键的阴影计算相关的宏三个关键宏:2-1.V2F_SHADOW_CASTER顶点到片元着色器阴影投射结构体数据宏这个宏定义了一些标准的成员变量这些变量用于在阴影投射路径中传递顶点数据到片元着色器我们主要在结构体中使用2-2.TRANSFER_SHADOW_CASTER_NORMALOFFSET转移阴影投射器法线偏移宏用于在顶点着色器中计算和传递阴影投射所需的变量主要做了2-2-1.将对象空间的顶点位置转换为裁剪空间的位置2-2-2.考虑法线偏移，以减轻阴影失真问题，尤其是在处理自阴影时2-2-3.传递顶点的投影空间位置，用于后续的阴影计算我们主要在顶点着色器中使用2-3.SHADOW_CASTER_FRAGMENT阴影投射片元宏将深度值写入到阴影映射纹理中我们主要在片元着色器中使用

3.利用这些内容在Shader中实现代码

**知识点三 建议**  
由于投射阴影相关的代码较为通用因此建议大家不用自己去实现相关Shader代码直接通过FallBack调用Unity中默认Shader中的相关代码即可实现注意：实现部分只需要看阴影投射的pass即可，其他部分是原来的基础光照实现。  
当然，最终项目可以用fallback Specular快速实现投射。

```hlsl
Shader "Unlit/不透明物体投射阴影"
{
    Properties
    {
        //主纹理色
        _MainColor("MainColor", Color) = (1, 1, 1, 1)
        //高光颜色
        _HightLightColor("HightLightColor", Color) = (1, 1, 1, 1)
        //光泽度幂
        _SpecularNum("SpecularNum", Range(0, 20)) = 0.5
    }
    SubShader
    {
        //基础渲染通道，处理漫反射等主要光
        Pass
        {
            Tags
            {
                "LightMode" = "ForwardBase"
            }

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            float3 _HightLightColor;
            float3 _MainColor;
            float _SpecularNum;

            struct v2f
            {
                float4 vertex : SV_POSITION;
                float4 worldVertex:TEXCOORD0;
                float3 worldNormal:NORMAL;
            };

            fixed3 GetLambertColor(float3 WorldNormal)
            {
                float3 Normal = normalize(WorldNormal); //标准化法线
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz); //标准化光线方向
                fixed3 color = _LightColor0 * _MainColor * max(0, dot(Normal, lightDir));
                return color;
            }

            fixed3 GetBlinPhoneHighLight(float4 WorldVertex, float3 WorldNormal)
            {
                float3 ViewDir = normalize(_WorldSpaceCameraPos - WorldVertex); //标准化视角方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz); //标准化光线方向
                float3 HalfDir = normalize(ViewDir + lightDir); //半角向量

                fixed3 color = _LightColor0 * _HightLightColor * pow(max(0, dot(normalize(WorldNormal), HalfDir)),
                                                                     _SpecularNum);
                return color;
            }

            v2f vert(appdata_base v)
            {
                v2f v2_f;
                v2_f.vertex = UnityObjectToClipPos(v.vertex);
                v2_f.worldVertex = mul(UNITY_MATRIX_M, v.vertex);
                v2_f.worldNormal = UnityObjectToWorldNormal(v.normal);
                return v2_f;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 Color = GetBlinPhoneHighLight(i.worldVertex, i.worldNormal);
                //光源衰减值，但是平行光并不会衰减，只是写着看的
                fixed attend = 1;
                Color += GetLambertColor(i.worldNormal);
                Color += UNITY_LIGHTMODEL_AMBIENT.rgb;
                return fixed4(Color * attend, 1);
            }
            ENDCG
        }

        //Additional Pass 附加渲染通道
        Pass
        {
            Tags
            {
                "LightMode"="ForwardAdd"
            }
            //线性减淡的效果 进行 光照颜色混合
            Blend One One

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdadd

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            v2f vert(appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                return v2fData;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                //兰伯特漫反射
                fixed3 worldNormal = normalize(i.wNormal);
                //平行光 光的方向 其实就是它的位置
                #ifdef  USING_DIRECTIONAL_LIGHT
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                #else //点光源和聚光灯 光的方向 是 光的位置 - 顶点位置
                fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz - i.wPos);
                #endif
                // 漫反射颜色 = 光颜色 * 属性中颜色 * max(0, dot(世界坐标系下的法线, 世界坐标系下的光方向));
                fixed3 diffuse = _LightColor0.rgb * _MainColor.rgb * max(0, dot(worldNormal, worldLightDir));

                //BlinnPhong高光反射
                //视角方向
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos.xyz);
                //半角方向向量
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                // 高光颜色 = 光颜色 * 属性中的高光颜色 * pow(max(0, dot(世界坐标系法线, 世界坐标系半角向量)), 光泽度);
                fixed3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(
                    max(0, dot(worldNormal, halfDir)), _SpecularNum);

                //衰减值
                #ifdef  USING_DIRECTIONAL_LIGHT
                    fixed atten = 1;
                #elif defined(POINT)
                    //将世界坐标系下顶点转到光源空间下
                    float3 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1)).xyz;
                    //利用这个坐标得到距离的平方 然后再再光源纹理中隐射得到衰减值
                    fixed atten = tex2D(_LightTexture0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL;
                #elif defined(SPOT)
                    //将世界坐标系下顶点转到光源空间下 聚光灯需要用w参与后续计算
                    float4 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1));
                    fixed atten = (lightCoord.z > 0) * //判断在聚光灯前面吗
                                  tex2D(_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w * //映射到大图中进行采样
                                  tex2D(_LightTextureB0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL; //距离的平方采样
                #else
                fixed atten = 1;
                #endif

                //在附加渲染通道中不需要在加上环境光颜色了 因为它只需要计算一次 在基础渲染通道中已经计算了
                return fixed4((diffuse + specular) * atten, 1);
            }
            ENDCG
        }

        //实现阴影投影的pass
        Pass
        {
            Tags
            {
                "LightMode"="ShadowCaster"
            }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //不同平台支持不同的阴影(SM,SSSM)
            #pragma multi_compile_shadowcaster
            //包含阴影计算有关的宏
            #include "UnityCG.cginc"

            struct v2f
            {
                //自带一些标准阴影成员变量的宏
                V2F_SHADOW_CASTER;
            };

            v2f vert(appdata_base v)
            {
                v2f data;
                //在顶点着色器对结构体传递阴影投影所需变量
                TRANSFER_SHADOW_CASTER_NORMALOFFSET(data);
                return data;
            }

            float4 frag(v2f i):SV_Target
            {
                //将深度写入阴影映射纹理中
                SHADOW_CASTER_FRAGMENT(i);
                
            }

            
            ENDCG
        }

    }
    //用这个Fallback可以直接开启投射
    //Fallback "Specular"
}
```

效果图不透明物体阴影——让物体接收阴影

**知识点一 让物体接收阴影的思路**  
上节课我们已经能够让物体投射阴影了所谓的投射阴影，其实就是让物体参与到光源的阴影映射纹理计算中最终才能影响其他物体在接收阴影时的采样结果由此可见让物体接收阴影的主要思路其实就是要从阴影隐射纹理中进行采样然后将采样结果用于最终的颜色计算中总体的流程就是:

1.在顶点着色器中进行顶点坐标转换（将顶点坐标 转换为 阴影映射纹理坐标）

2.在片元着色器中使用阴影映射纹理坐标在阴影映射纹理中进行采样  
通过得到的深度值判断片元（像素）是否在阴影中，以计算出阴影衰减值

3.将采样结果参与到最终的颜色计算中

**知识点二 实现物体接收阴影效果**

1.创建一个新的Shader，复制Lesson64_ForwardLighting中的内容到新Shader中  
在此基础上实现接受阴影的效果

2.接受阴影的三剑客（三个宏）  
我们这节课只修改Bass Pass中的代码来感受下接受阴影的流程首先我们需要在Bass Pass当中引用包含内置文件#include "AutoLight.cginc"该内置文件中，有用于计算阴影时需要使用的三剑客2-1.SHADOW_COORDS (阴影坐标宏)该宏在v2f结构体（顶点着色器返回值）中使用本质上就是声明了一个用于对阴影纹理进行采样的坐标在内部实际上就是声明了一个名为_ShadowCoord的阴影纹理坐标变量需要注意的是：在使用时 SHADOW_COORDS(2) 传入参数2表示需要时下一个可用的插值寄存器的索引值2-2.TRANSFER_SHADOW (转移阴影宏)该宏在顶点着色器函数中调用，传入对应的v2f结构体对象该宏会在内部自己判断应该使用哪种阴影映射技术（SM、SSSM）最终的目的就是将顶点进行坐标转换并存储到_ShadowCoord阴影纹理坐标变量中需要注意的是：

1.该宏会在内部使用顶点着色器中传入的结构体  
该结构体中顶点的命名必须是vertex

2.该宏会在内部使用顶点着色器的返回结构体  
其中的顶点位置命名必须是pos2-3.SHADOW_ATTENUATION (阴影衰减宏)该宏在片元着色器中调用，传入对应的v2f结构体对象该宏会在内部利用v2f中的 阴影纹理坐标变量(ShadowCoord)对相关纹理进行采样将采样得到的深度值进行比较，以计算出一个fixed3的阴影衰减值我们只需要使用它返回的结果和 (漫反射+高光反射) 的结果相乘即可

3.注意  
我们目前处理的方式只是让大家大致了解接受阴影的流程我们还没有对 Additional Pass 附加渲染通道进行处理我们将在下节课中统一处理光照衰减和阴影 得到最终效果注意——关闭阴影投射只打开阴影接收如果你关闭了阴影投射，单独开启阴影接收，会发现出现诡异的视觉bug：你莫名其妙可以在非透明物体上看到后方的顶点阴影了目前原因较难推测，因为理论上前方和后方的阴影点在光源空间的采样位置很多情况是不一样的，但是我测试了几个环境下都出现了这个问题。(下图第一张为同时开启，第二张为单独开启阴影接收)目前初步推测是：定向光的阴影范围会紧紧地包裹住相机当前能看到的区域（相机的视锥 Frustum）视锥的阴影图优化可能导致两个顶点的最终采样坐标相近，但等待进一步确定原因。  
实现重点研究知识点二的实现部分代码

```hlsl
Shader "Unlit/Lesson67_Shadow"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        //高光反射颜色  光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0, 20)) = 1
    }
    SubShader
    {
        //Bass Pass 基础渲染通道
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            //引用包含该内置文件 其中有计算阴影会用到的三个宏（阴影三剑客）
            #include "AutoLight.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
                //      该宏在v2f结构体（顶点着色器返回值）中使用
                //      本质上就是声明了一个用于对阴影纹理进行采样的坐标
                //      在内部实际上就是声明了一个名为_ShadowCoord的阴影纹理坐标变量
                //      需要注意的是：
                //      在使用时 SHADOW_COORDS(2) 传入参数2
                //      表示需要时下一个可用的插值寄存器的索引值   
                //阴影坐标宏 主要用于存储阴影纹理坐标
                SHADOW_COORDS(2)
            };

            //得到兰伯特光照模型计算的颜色 （逐片元）
            fixed3 getLambertFColor(in float3 wNormal)
            {
                //得到光源单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //计算除了兰伯特光照的漫反射颜色
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(wNormal, lightDir));

                return color;
            }

            //得到Blinn Phong式高光反射模型计算的颜色（逐片元）
            fixed3 getSpecularColor(in float3 wPos, in float3 wNormal)
            {
                //1.视角单位向量
                //float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - wPos );
                float3 viewDir = normalize(UnityWorldSpaceViewDir(wPos));

                //2.光的反射单位向量
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);

                //半角方向向量
                float3 halfA = normalize(viewDir + lightDir);
                
                //color = 光源颜色 * 材质高光反射颜色 * pow( max(0, dot(视角单位向量, 光的反射单位向量)), 光泽度 )
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(wNormal, halfA)), _SpecularNum );

                return color;
            }

            v2f vert (appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                //      该宏在顶点着色器函数中调用，传入对应的v2f结构体对象
                //      该宏会在内部自己判断应该使用哪种阴影映射技术（SM、SSSM）
                //      最终的目的就是将顶点进行坐标转换并存储到_ShadowCoord阴影纹理坐标变量中
                //      需要注意的是：
                //      1.该宏会在内部使用顶点着色器中传入的结构体
                //        该结构体中顶点的命名必须是vertex
                //      2.该宏会在内部使用顶点着色器的返回结构体
                //        其中的顶点位置命名必须是pos
                //计算阴影映射纹理坐标 它会在内部去进行计算 然后将其存入 v2f中的SHADOW_COORDS中
                TRANSFER_SHADOW(v2fData);

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //计算兰伯特光照颜色
                fixed3 lambertColor = getLambertFColor(normalize(i.wNormal));
                //计算BlinnPhong式高光反射颜色
                fixed3 specularColor = getSpecularColor(i.wPos, normalize(i.wNormal));

                //得到阴影衰减值
                //      该宏在片元着色器中调用，传入对应的v2f结构体对象
                //      该宏会在内部利用v2f中的 阴影纹理坐标变量(ShadowCoord)对相关纹理进行采样
                //      将采样得到的深度值进行比较，以计算出一个fixed3的阴影衰减值
                //      我们只需要使用它返回的结果和 (漫反射+高光反射) 的结果相乘即可
                fixed3 shadow = SHADOW_ATTENUATION(i);

                //衰减值
                fixed atten = 1;
                //物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色
                //衰减值 会和 漫反射颜色 + 高光反射颜色 后 再进行乘法运算
                fixed3 blinnPhongColor = UNITY_LIGHTMODEL_AMBIENT.rgb + (lambertColor + specularColor)*atten*shadow; 

                return fixed4(blinnPhongColor.rgb, 1);
            }
            ENDCG
        }

        //Additional Pass 附加渲染通道
        Pass
        {
            Tags { "LightMode"="ForwardAdd" }
            //线性减淡的效果 进行 光照颜色混合
            Blend One One

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdadd

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
            };

            v2f vert (appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //兰伯特漫反射
                fixed3 worldNormal = normalize(i.wNormal);
                //平行光 光的方向 其实就是它的位置
                #if defined(_DIRECTIONAL_LIGHT)
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                #else //点光源和聚光灯 光的方向 是 光的位置 - 顶点位置
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz - i.wPos);
                #endif
                // 漫反射颜色 = 光颜色 * 属性中颜色 * max(0, dot(世界坐标系下的法线, 世界坐标系下的光方向));
                fixed3 diffuse = _LightColor0.rgb * _MainColor.rgb * max(0, dot(worldNormal, worldLightDir));
                
                //BlinnPhong高光反射
                //视角方向
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos.xyz);
                //半角方向向量
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                // 高光颜色 = 光颜色 * 属性中的高光颜色 * pow(max(0, dot(世界坐标系法线, 世界坐标系半角向量)), 光泽度);
                fixed3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0, dot(worldNormal, halfDir)), _SpecularNum);

                //衰减值
                #if defined(_DIRECTIONAL_LIGHT)
                    fixed atten = 1;
                #elif defined(_POINT_LIGHT)
                    //将世界坐标系下顶点转到光源空间下
                    float3 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1)).xyz;
                    //利用这个坐标得到距离的平方 然后再再光源纹理中隐射得到衰减值
                    fixed atten = tex2D(_LightTexture0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL;
                #elif defined(_SPOT_LIGHT)
                    //将世界坐标系下顶点转到光源空间下 聚光灯需要用w参与后续计算
                    float4 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1));
                    fixed atten = (lightCoord.z > 0) * //判断在聚光灯前面吗
                                  tex2D(_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w * //映射到大图中进行采样
                                  tex2D(_LightTextureB0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL; //距离的平方采样
                #else
                    fixed atten = 1;
                #endif

                //在附加渲染通道中不需要在加上环境光颜色了 因为它只需要计算一次 在基础渲染通道中已经计算了
                return fixed4((diffuse + specular)*atten, 1);
            }
            ENDCG
        }
    }

    FallBack "Specular"
}
```

效果图不透明物体阴影——光照衰减和阴影

**知识点一 光照衰减和阴影**  
通过之前的学习我们发现光照衰减和接受阴影相关的计算是类似的关键点都是通过计算出一个衰减值，参与到颜色计算中都是用 (漫反射+高光反射) 的结果乘以对应的衰减值由于它们对最终颜色影响的计算非常类似，都是通过乘法进行运算因此Unity中专门提供了对应的宏来综合处理光照衰减和阴影衰减的计算AutoLight.cginc 内置文件中的UNITY_LIGHT_ATTENUATION（Unity光照衰减宏）该宏中会统一的对 光照衰减进行计算 并且也会计算出 阴影衰减值最后将两者相乘得到综合衰减值我们只需要利用该宏来处理 光照和阴影的衰减即可我们可以在 Unity安装目录的Editor->Data->CGIncludes中找到该内置文件查看该宏的逻辑

**知识点二 光照衰减和阴影的综合实现**  
我们将利用AutoLight.cginc 内置文件中的UNITY_LIGHT_ATTENUATION（Unity光照衰减宏）来综合处理光照衰减和阴影相关的逻辑

1.创建一个新的Shader 并将Lesson67_Shadow中的代码赋值过来

2.将三剑客中的前两剑客，在Additional Pass 附加渲染通道中也添加上  
注意：需要在附加渲染通道中包含内置文件 AutoLight.cginc（因为不仅三剑客来自于它，UNITY_LIGHT_ATTENUATION-Unity光照衰减宏也来自于它）

3.修改两个Pass的片元着色器中衰减计算相关的代码  
使用 UNITY_LIGHT_ATTENUATION 宏替代原有逻辑该宏需要传入3个参数第一个参数是用来存储最终衰减值的变量名（不用声明，内部会声明）第二个参数是片元着色器中传入的v2f结构体对象第三个参数是顶点相对于世界坐标系的位置最终将得到的衰减结果和 (漫反射+高光反射) 的结果相乘即可

4.为了让Additional Pass 附加渲染通道能够添加阴影效果  
需要将编译指令进行修改将原本的 #pragma multi_compile_fwdadd修改为 #pragma multi_compile_fwdadd_fullshadows这样Unity会生成多个包括支持和不支持阴影的Shader变体从而为额外的逐像素光源计算阴影，并传递给Shader了实现

```hlsl
Shader "Unlit/Lesson68_Attenuation"
{
   Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        //高光反射颜色  光泽度
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0, 20)) = 1
    }
    SubShader
    {
        //Bass Pass 基础渲染通道
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            //引用包含该内置文件 其中有计算阴影会用到的三个宏（阴影三剑客）
            #include "AutoLight.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
                //      该宏在v2f结构体（顶点着色器返回值）中使用
                //      本质上就是声明了一个用于对阴影纹理进行采样的坐标
                //      在内部实际上就是声明了一个名为_ShadowCoord的阴影纹理坐标变量
                //      需要注意的是：
                //      在使用时 SHADOW_COORDS(2) 传入参数2
                //      表示需要时下一个可用的插值寄存器的索引值   
                //阴影坐标宏 主要用于存储阴影纹理坐标
                SHADOW_COORDS(2)
            };

            //得到兰伯特光照模型计算的颜色 （逐片元）
            fixed3 getLambertFColor(in float3 wNormal)
            {
                //得到光源单位向量
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //计算除了兰伯特光照的漫反射颜色
                fixed3 color = _LightColor0.rgb * _MainColor.rgb * max(0, dot(wNormal, lightDir));

                return color;
            }

            //得到Blinn Phong式高光反射模型计算的颜色（逐片元）
            fixed3 getSpecularColor(in float3 wPos, in float3 wNormal)
            {
                //1.视角单位向量
                //float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - wPos );
                float3 viewDir = normalize(UnityWorldSpaceViewDir(wPos));

                //2.光的反射单位向量
                //光的方向
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);

                //半角方向向量
                float3 halfA = normalize(viewDir + lightDir);
                
                //color = 光源颜色 * 材质高光反射颜色 * pow( max(0, dot(视角单位向量, 光的反射单位向量)), 光泽度 )
                fixed3 color = _LightColor0.rgb * _SpecularColor.rgb * pow( max(0, dot(wNormal, halfA)), _SpecularNum );

                return color;
            }

            v2f vert (appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                //      该宏在顶点着色器函数中调用，传入对应的v2f结构体对象
                //      该宏会在内部自己判断应该使用哪种阴影映射技术（SM、SSSM）
                //      最终的目的就是将顶点进行坐标转换并存储到_ShadowCoord阴影纹理坐标变量中
                //      需要注意的是：
                //      1.该宏会在内部使用顶点着色器中传入的结构体
                //        该结构体中顶点的命名必须是vertex
                //      2.该宏会在内部使用顶点着色器的返回结构体
                //        其中的顶点位置命名必须是pos
                //计算阴影映射纹理坐标 它会在内部去进行计算 然后将其存入 v2f中的SHADOW_COORDS中
                TRANSFER_SHADOW(v2fData);

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //计算兰伯特光照颜色
                fixed3 lambertColor = getLambertFColor(normalize(i.wNormal));
                //计算BlinnPhong式高光反射颜色
                fixed3 specularColor = getSpecularColor(i.wPos, normalize(i.wNormal));

                //得到阴影衰减值
                //      该宏在片元着色器中调用，传入对应的v2f结构体对象
                //      该宏会在内部利用v2f中的 阴影纹理坐标变量(ShadowCoord)对相关纹理进行采样
                //      将采样得到的深度值进行比较，以计算出一个fixed3的阴影衰减值
                //      我们只需要使用它返回的结果和 (漫反射+高光反射) 的结果相乘即可
                //fixed3 shadow = SHADOW_ATTENUATION(i);

                //衰减值
                //fixed atten = 1;

                //利用灯光衰减和阴影衰减计算宏统一进行衰减值的计算
                UNITY_LIGHT_ATTENUATION(atten, i, i.wPos)

                //物体表面光照颜色 = 环境光颜色 + 兰伯特光照模型所得颜色 + Phong式高光反射光照模型所得颜色
                //衰减值 会和 漫反射颜色 + 高光反射颜色 后 再进行乘法运算
                fixed3 blinnPhongColor = UNITY_LIGHTMODEL_AMBIENT.rgb + (lambertColor + specularColor)*atten; 

                return fixed4(blinnPhongColor.rgb, 1);
            }
            ENDCG
        }

        //Additional Pass 附加渲染通道
        Pass
        {
            Tags { "LightMode"="ForwardAdd" }
            //线性减淡的效果 进行 光照颜色混合
            Blend One One

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            //用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中
            //#pragma multi_compile_fwdadd
            #pragma multi_compile_fwdadd_fullshadows

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            //材质漫反射颜色
            fixed4 _MainColor;
            fixed4 _SpecularColor;
            float _SpecularNum;

            //顶点着色器返回出去的内容
            struct v2f
            {
                //裁剪空间下的顶点位置
                float4 pos:SV_POSITION;
                //世界空间下的法线位置
                float3 wNormal:NORMAL;
                //世界空间下的 顶点坐标 
                float3 wPos:TEXCOORD0;
                //      该宏在v2f结构体（顶点着色器返回值）中使用
                //      本质上就是声明了一个用于对阴影纹理进行采样的坐标
                //      在内部实际上就是声明了一个名为_ShadowCoord的阴影纹理坐标变量
                //      需要注意的是：
                //      在使用时 SHADOW_COORDS(2) 传入参数2
                //      表示需要时下一个可用的插值寄存器的索引值   
                //阴影坐标宏 主要用于存储阴影纹理坐标
                SHADOW_COORDS(2)
            };

            v2f vert (appdata_base v)
            {
                v2f v2fData;
                //转换模型空间下的顶点到裁剪空间中
                v2fData.pos = UnityObjectToClipPos(v.vertex);
                //转换模型空间下的法线到世界空间下
                v2fData.wNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转到世界空间
                v2fData.wPos = mul(unity_ObjectToWorld, v.vertex).xyz;

                //      该宏在顶点着色器函数中调用，传入对应的v2f结构体对象
                //      该宏会在内部自己判断应该使用哪种阴影映射技术（SM、SSSM）
                //      最终的目的就是将顶点进行坐标转换并存储到_ShadowCoord阴影纹理坐标变量中
                //      需要注意的是：
                //      1.该宏会在内部使用顶点着色器中传入的结构体
                //        该结构体中顶点的命名必须是vertex
                //      2.该宏会在内部使用顶点着色器的返回结构体
                //        其中的顶点位置命名必须是pos
                //计算阴影映射纹理坐标 它会在内部去进行计算 然后将其存入 v2f中的SHADOW_COORDS中
                TRANSFER_SHADOW(v2fData);

                return v2fData;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //兰伯特漫反射
                fixed3 worldNormal = normalize(i.wNormal);
                //平行光 光的方向 其实就是它的位置
                #if defined(_DIRECTIONAL_LIGHT)
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                #else //点光源和聚光灯 光的方向 是 光的位置 - 顶点位置
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz - i.wPos);
                #endif
                // 漫反射颜色 = 光颜色 * 属性中颜色 * max(0, dot(世界坐标系下的法线, 世界坐标系下的光方向));
                fixed3 diffuse = _LightColor0.rgb * _MainColor.rgb * max(0, dot(worldNormal, worldLightDir));
                
                //BlinnPhong高光反射
                //视角方向
                fixed3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos.xyz);
                //半角方向向量
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                // 高光颜色 = 光颜色 * 属性中的高光颜色 * pow(max(0, dot(世界坐标系法线, 世界坐标系半角向量)), 光泽度);
                fixed3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0, dot(worldNormal, halfDir)), _SpecularNum);

                //衰减值
                //#if defined(_DIRECTIONAL_LIGHT)
                //    fixed atten = 1;
                //#elif defined(_POINT_LIGHT)
                //    //将世界坐标系下顶点转到光源空间下
                //    float3 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1)).xyz;
                //    //利用这个坐标得到距离的平方 然后再再光源纹理中隐射得到衰减值
                //    fixed atten = tex2D(_LightTexture0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL;
                //#elif defined(_SPOT_LIGHT)
                //    //将世界坐标系下顶点转到光源空间下 聚光灯需要用w参与后续计算
                //    float4 lightCoord = mul(unity_WorldToLight, float4(i.wPos, 1));
                //    fixed atten = (lightCoord.z > 0) * //判断在聚光灯前面吗
                //                  tex2D(_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w * //映射到大图中进行采样
                //                  tex2D(_LightTextureB0, dot(lightCoord,lightCoord).xx).UNITY_ATTEN_CHANNEL; //距离的平方采样
                //#else
                //    fixed atten = 1;
                //#endif

                //利用灯光衰减和阴影衰减计算宏统一进行衰减值的计算
                UNITY_LIGHT_ATTENUATION(atten, i, i.wPos)

                //在附加渲染通道中不需要在加上环境光颜色了 因为它只需要计算一次 在基础渲染通道中已经计算了
                return fixed4((diffuse + specular)*atten, 1);
            }
            ENDCG
        }
    }

    FallBack "Specular"
}
```

**总结**  
效果图如上一节，这里就不再总结总的来说，这一节介绍的宏主要是可以让我们方便的得到阴影和光照衰减的atteen，不用再写一大串内容。透明物体阴影——透明度测试物体阴影实现

**知识回顾 透明度测试**  
在游戏开发中对象的某些部位完全透明而其他部位完全不透明这种透明需求往往不需要半透明效果相对比较极端，只有看得见和看不见之分比如树叶、草、栅栏等等基本原理：通过一个阈值来决定哪些像素应该被保留，哪些应该被丢弃

**知识点一 让透明度测试Shader投射阴影**

1.同样我们使用FallBack的形式投射阴影  
但是需要注意的是FallBack的内容为：Transparent/Cutout/VertexLit该默认Shader中会把裁剪后的物体深度信息写入到 阴影映射纹理和摄像机深度图中注意：使用该默认Shader计算投射阴影时，需要使用_Cutoff属性 和 \_Color属性来进行相关计算因此我们必须保证我们的Shader当中有名为_Cutoff的阈值属性 和 \_Color的漫反射颜色属性否则无法得到正确阴影结果

2.为了得到正确的阴影效果，我们需要将该物体的Cast Shadows（投射阴影）属性设置为Two Sided（双面）  
强制让Unity计算阴影隐射纹理时计算所有面的深度信息。因为如果不设置，默认将物体渲染到阴影隐射纹理和摄像机深度图时只会考虑物体的正面背对光源的面不会参与计算，设置为双面后即可参与计算，得到正确的结果

**知识点二 透明度测试Shader接收阴影**  
和我们之前处理接收阴影的方式一样主要分5步骤：

1.编译指令  
#pragma multi_compile_fwdbase用于帮助我们编译所有变体 并且保证衰减相关光照变量能够正确赋值到对应的内置变量中

2.包含内置文件  
#include "AutoLight.cginc"

3.结构体中声明阴影坐标宏  
SHADOW_COORDS(n)n为下一个可用的插值寄存器的索引值（结构体前面有几个TEXCOORD就填几）

4.坐标转换宏  
TRANSFER_SHADOW(o);

5.Unity光照衰减计算宏  
UNITY_LIGHT_ATTENUATION(atten, v2f结构体, 顶点世界坐标位置);实现

```hlsl
Shader "Unlit/Lesson69_AlphaTestShadow"
{
    Properties
    {
       //主要就是将单张纹理Shader和Blinn-Phong光照模型逐片元Shader进行一个结合
       _MainTex("MainTex", 2D) = ""{}
       //漫反射颜色、高光反射颜色、光泽度
       _Color("MainColor", Color) = (1,1,1,1)
       _SpecularColor("SpecularColor", Color) = (1,1,1,1)
       _SpecularNum("SpecularNum", Range(0,20)) = 15
       //透明度测试用的阈值
       _Cutoff("Cutoff", Range(0,1)) = 0
    }
    SubShader
    {
        //设置渲染队列 决定对象在何时渲染
        Tags{"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}
        Pass
        {
            Tags{ "LightMode" = "ForwardBase" }
            //关闭剔除 让正面和背面都去进行渲染
            Cull Off

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdbase
            
            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            //纹理贴图对应的映射成员
            sampler2D _MainTex;
            float4 _MainTex_ST;
            //漫反射颜色、高光反射颜色、光泽度
            fixed4 _Color;
            fixed4 _SpecularColor;
            float _SpecularNum;
            //透明度测试用的阈值
            fixed _Cutoff;

            struct v2f
            {
                //裁剪空间下的顶点坐标
                float4 pos:SV_POSITION;
                //UV坐标
                float2 uv:TEXCOORD0;
                //世界空间下的法线
                float3 wNormal:NORMAL;
                //世界空间下的顶点坐标
                float3 wPos:TEXCOORD1;
                //阴影坐标宏
                SHADOW_COORDS(2)
            };

            v2f vert (appdata_base v)
            {
               v2f data;
               //把模型空间下的顶点转换到裁剪空间下
               data.pos = UnityObjectToClipPos(v.vertex);
               //uv坐标计算
               data.uv = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
               //世界空间下的法线
               data.wNormal = UnityObjectToWorldNormal(v.normal);
               //世界空间下的顶点坐标
               data.wPos = mul(unity_ObjectToWorld, v.vertex);

               //坐标转换宏
               TRANSFER_SHADOW(data)

               return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //颜色纹理的颜色信息
                fixed4 texColor = tex2D(_MainTex, i.uv);

                //判断贴图的 颜色信息中的 透明通道 有没有小于阈值 
                //如果小于了 就直接丢弃
                clip(texColor.a - _Cutoff);
                //相当于
                //if(texColor.a - _Cutoff < 0)
                //if(texColor.a < _Cutoff)
                //    discard;

                //新知识点：纹理颜色需要和漫反射材质颜色叠加（乘法） 共同决定最终的颜色
                fixed3 albedo = texColor.rgb * _Color.rgb;
                //光的方向（指向光源方向）
                float3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //兰伯特漫反射颜色 = 光的颜色 * 漫反射材质的颜色 * max(0, dot(世界坐标系下的法线, 光的方向))
                //新知识点：兰伯特光照模型计算时，漫反射材质颜色使用 1 中的叠加颜色计算
                fixed3 lambertColor = _LightColor0.rgb * albedo.rgb * max(0, dot(i.wNormal, lightDir));
                
                // 视角方向
                //float3 viewDir = normalize(_WorldSpaceCameraPos.xyz - i.wPos);
                float3 viewDir = normalize(UnityWorldSpaceViewDir(i.wPos));
                //半角向量 = 视角方向 + 光的方向
                float3 halfA = normalize(viewDir + lightDir);
                //高光反射的颜色 = 光的颜色 * 高光反射材质的颜色 * pow(max(0, dot(世界坐标系下的法线, 半角向量)), 光泽度)
                fixed3 specularColor = _LightColor0.rgb * _SpecularColor * pow( max(0, dot(i.wNormal, halfA)), _SpecularNum);

                //计算光照衰减和阴影衰减的宏
                UNITY_LIGHT_ATTENUATION(atten, i, i.wPos)

                //Blinn-Phong光照颜色 = 环境光颜色 + 兰伯特漫反射颜色 + 高光反射的颜色
                //新知识点：最终使用的环境光叠加时，环境光变量UNITY_LIGHTMODEL_AMBIENT需要和 1 中颜色进行乘法叠加
                //         为了避免最终的渲染效果偏灰
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo + (lambertColor + specularColor)*atten;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }
    Fallback "Transparent/Cutout/VertexLit"
}
```

效果图透明物体阴影——透明度混合物体阴影实现

**知识回顾 透明度混合**  
透明度测试只能处理极端效果（完全透明和完全不透明）而透明度混合可以帮助我们实现半透明效果它的基本原理：关闭深度写入，开启混合，让片元颜色和颜色缓冲区中颜色进行混合计算

**知识点一 透明度混合Shader处理阴影**  
根据上节课的思路我们如果想要物体产生阴影效果无非就两步：

1.投射阴影  
在FallBack中关联内置的对应Shader（"Transparent/VertexLit"）

2.接受阴影  
在Shader中书写计算阴影衰减值的相关代码因此对于透明度混合的Shader也会使用同样的思路去制作但是！！！由于透明度混合需要关闭深度写入而阴影相关的处理需要用到深度值参与计算因此Unity中从性能方面考虑（要计算半透明物体的的阴影表现效果是相对复杂的）所有的内置半透明Shader都不会产生阴影效果（比如 Transparent/VertexLit）因此2-1.透明混合Shader想要 投射阴影时不管你在FallBack中写入哪种自带的半透明混合Shader都不会有投射阴影的效果，因为深度不会写入2-2.透明混合Shader想要 接受阴影时Unity内置关于阴影接收计算的相关宏不会计算处理 透明混合Shader混合因子 设置为半透明效果(Blend SrcAlpha OneMinusSrcAlpha)的Shader因为透明混合物体的深度值和遮挡关系无法直接用传统的深度缓冲和阴影贴图来处理结论：Unity中不会直接为透明度混合Shader处理阴影

**知识点三 强制生成阴影**  
我们可以尝试让透明混合Shader强制投射阴影在FallBack中设置一个非透明Shader，比如VertexLit、Diffuse等用其中的灯光模式设置为阴影投射的渲染通道来参与阴影映射纹理的计算把该物体当成一个实体物体处理但是，这种效果并不真实，不建议使用也就是目前知识点比较难实现一个半透明物体的阴影功能，待后续补充


## 第 7 节 标准光照着色器

标准漫反射Shader

**知识点一 什么是标准漫反射Shader？**  
目前我们已经完成了光源和阴影的主要知识点学习我们学习了多光源、阴影、光照衰减等等知识已经可以在Shader中处理光和阴影相关的效果了那么我们将结合所学的知识实现一个标准的漫反射Shader该Shader其实就是一个带有法线（世界空间中计算-全局效果的表现更准确）的基于Phong光照模型（去掉高光反射）的支持多光源和阴影的Shader说是标准，其实就是一个常用Shader而已

**知识点二 制作常用漫反射Shader**

1.新建一个Shader，取名叫BumpedDiffuse（凹凸漫反射）

2.复制 Lesson52 中Shader代码，粘贴到新建文件中

3.加入渲染标签Tags { "RenderType"="Opaque" "Queue"="Geometry"}  
渲染类型设置为不透明的、渲染队列设置为几何队列（不透明的几何体通常使用该队列）

4.删除高光反射相关代码

5.加入阴影、衰减相关代码

6.加入附加渲染通道

7.加入FallBac "Diffuse"  
实现

```hlsl
Shader "Unlit/BumpedDiffuse"
{
   Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        _MainTex("MainTex", 2D) = ""{}
        _BumpMap("BumpMap", 2D) = ""{}
        _BumpScale("BumpScale", Range(0,1)) = 1
    }
    SubShader
    {
        Tags{ "RenderType"="Opaque" "Queue"="Geometry"}
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            float4 _MainColor;//漫反射颜色
            sampler2D _MainTex;//颜色纹理
            float4 _MainTex_ST;//颜色纹理的缩放和平移
            sampler2D _BumpMap;//法线纹理
            float4 _BumpMap_ST;//法线纹理的缩放和平移
            float _BumpScale;//凹凸程度

            struct v2f
            {
                float4 pos:SV_POSITION;
                //float2 uvTex:TEXCOORD0;
                //float2 uvBump:TEXCOORD1;
                //我们可以单独的声明两个float2的成员用于记录 颜色和法线纹理的uv坐标
                //也可以直接声明一个float4的成员 xy用于记录颜色纹理的uv，zw用于记录法线纹理的uv
                float4 uv:TEXCOORD0;
                //顶点相对于世界坐标的位置 主要用于 之后的 视角方向的计算
                //float3 worldPos:TEXCOORD1;
                //切线 到 世界空间的 变换矩阵
                //float3x3 rotation:TEXCOORD2;

                //代表我们切线空间到世界空间的 变换矩阵的3行
                float4 TtoW0:TEXCOORD1;
                float4 TtoW1:TEXCOORD2;
                float4 TtoW2:TEXCOORD3;

                SHADOW_COORDS(4)
            };

            v2f vert (appdata_full v)
            {
                v2f data;
                //把模型空间下的顶点转到裁剪空间下
                data.pos = UnityObjectToClipPos(v.vertex);
                //计算纹理的缩放偏移
                data.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                data.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
                //得到世界空间下的 顶点位置 用于之后在片元中计算视角方向（世界空间下的）
                //data.worldPos = mul(unity_ObjectToWorld, v.vertex);
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex);
                //把模型空间下的法线、切线转换到世界空间下
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                float3 worldTangent = UnityObjectToWorldDir(v.tangent);
                //计算副切线 计算叉乘结果后 垂直与切线和法线的向量有两条 通过乘以 切线当中的w，就可以确定是哪一条
                float3 worldBinormal = cross(normalize(worldTangent), normalize(worldNormal)) * v.tangent.w;
                //这个就是我们 切线空间到世界空间的 转换矩阵
                //data.rotation = float3x3( worldTangent.x, worldBinormal.x,  worldNormal.x,
                //                          worldTangent.y, worldBinormal.y,  worldNormal.y,
                //                          worldTangent.z, worldBinormal.z,  worldNormal.z);
                data.TtoW0 = float4(worldTangent.x, worldBinormal.x,  worldNormal.x, worldPos.x);
                data.TtoW1 = float4(worldTangent.y, worldBinormal.y,  worldNormal.y, worldPos.y);
                data.TtoW2 = float4(worldTangent.z, worldBinormal.z,  worldNormal.z, worldPos.z);

                TRANSFER_SHADOW(data);

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //世界空间下光的方向
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //世界空间下视角方向
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                //通过纹理采样函数 取出法线纹理贴图当中的数据
                float4 packedNormal = tex2D(_BumpMap, i.uv.zw);
                //将我们取出来的法线数据 进行逆运算并且可能会进行解压缩的运算，最终得到切线空间下的法线数据
                float3 tangentNormal = UnpackNormal(packedNormal);
                //乘以凹凸程度的系数
                tangentNormal.xy *= _BumpScale;
                tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
                //把计算完毕后的切线空间下的法线转换到世界空间下
                //float3x3 rotation = float3x3(i.TtoW0.xyz, i.TtoW1.xyz, i.TtoW2.xyz );
                //float3 worldNormal = mul(rotation, tangentNormal);
                //本质 就是在进行矩阵运算
                float3 worldNormal = float3(dot(i.TtoW0.xyz, tangentNormal), dot(i.TtoW1.xyz, tangentNormal), dot(i.TtoW2.xyz, tangentNormal));

                //接下来就来处理 带颜色纹理的 Blinn-Phong光照模型计算

                //颜色纹理和漫反射颜色的 叠加
                fixed3 albedo = tex2D(_MainTex, i.uv.xy) * _MainColor.rgb;
                //兰伯特
                fixed3 lambertColor = _LightColor0.rgb * albedo.rgb * max(0, dot(worldNormal, normalize(lightDir)));
               
                UNITY_LIGHT_ATTENUATION(atten, i, worldPos);
               
               //Blinn-Phong
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo + lambertColor * atten;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }

        Pass
        {
            Tags { "LightMode"="ForwardAdd" }

            Blend One One

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdadd

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            float4 _MainColor;//漫反射颜色
            sampler2D _MainTex;//颜色纹理
            float4 _MainTex_ST;//颜色纹理的缩放和平移
            sampler2D _BumpMap;//法线纹理
            float4 _BumpMap_ST;//法线纹理的缩放和平移
            float _BumpScale;//凹凸程度

            struct v2f
            {
                float4 pos:SV_POSITION;
                //float2 uvTex:TEXCOORD0;
                //float2 uvBump:TEXCOORD1;
                //我们可以单独的声明两个float2的成员用于记录 颜色和法线纹理的uv坐标
                //也可以直接声明一个float4的成员 xy用于记录颜色纹理的uv，zw用于记录法线纹理的uv
                float4 uv:TEXCOORD0;
                //顶点相对于世界坐标的位置 主要用于 之后的 视角方向的计算
                //float3 worldPos:TEXCOORD1;
                //切线 到 世界空间的 变换矩阵
                //float3x3 rotation:TEXCOORD2;

                //代表我们切线空间到世界空间的 变换矩阵的3行
                float4 TtoW0:TEXCOORD1;
                float4 TtoW1:TEXCOORD2;
                float4 TtoW2:TEXCOORD3;

                SHADOW_COORDS(4)
            };

            v2f vert (appdata_full v)
            {
                v2f data;
                //把模型空间下的顶点转到裁剪空间下
                data.pos = UnityObjectToClipPos(v.vertex);
                //计算纹理的缩放偏移
                data.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                data.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
                //得到世界空间下的 顶点位置 用于之后在片元中计算视角方向（世界空间下的）
                //data.worldPos = mul(unity_ObjectToWorld, v.vertex);
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex);
                //把模型空间下的法线、切线转换到世界空间下
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                float3 worldTangent = UnityObjectToWorldDir(v.tangent);
                //计算副切线 计算叉乘结果后 垂直与切线和法线的向量有两条 通过乘以 切线当中的w，就可以确定是哪一条
                float3 worldBinormal = cross(normalize(worldTangent), normalize(worldNormal)) * v.tangent.w;
                //这个就是我们 切线空间到世界空间的 转换矩阵
                //data.rotation = float3x3( worldTangent.x, worldBinormal.x,  worldNormal.x,
                //                          worldTangent.y, worldBinormal.y,  worldNormal.y,
                //                          worldTangent.z, worldBinormal.z,  worldNormal.z);
                data.TtoW0 = float4(worldTangent.x, worldBinormal.x,  worldNormal.x, worldPos.x);
                data.TtoW1 = float4(worldTangent.y, worldBinormal.y,  worldNormal.y, worldPos.y);
                data.TtoW2 = float4(worldTangent.z, worldBinormal.z,  worldNormal.z, worldPos.z);

                TRANSFER_SHADOW(data);

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //世界空间下光的方向
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //世界空间下视角方向
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                //通过纹理采样函数 取出法线纹理贴图当中的数据
                float4 packedNormal = tex2D(_BumpMap, i.uv.zw);
                //将我们取出来的法线数据 进行逆运算并且可能会进行解压缩的运算，最终得到切线空间下的法线数据
                float3 tangentNormal = UnpackNormal(packedNormal);
                //乘以凹凸程度的系数
                tangentNormal.xy *= _BumpScale;
                tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
                //把计算完毕后的切线空间下的法线转换到世界空间下
                //float3x3 rotation = float3x3(i.TtoW0.xyz, i.TtoW1.xyz, i.TtoW2.xyz );
                //float3 worldNormal = mul(rotation, tangentNormal);
                //本质 就是在进行矩阵运算
                float3 worldNormal = float3(dot(i.TtoW0.xyz, tangentNormal), dot(i.TtoW1.xyz, tangentNormal), dot(i.TtoW2.xyz, tangentNormal));

                //接下来就来处理 带颜色纹理的 Blinn-Phong光照模型计算

                //颜色纹理和漫反射颜色的 叠加
                fixed3 albedo = tex2D(_MainTex, i.uv.xy) * _MainColor.rgb;
                //兰伯特
                fixed3 lambertColor = _LightColor0.rgb * albedo.rgb * max(0, dot(worldNormal, normalize(lightDir)));
               
                UNITY_LIGHT_ATTENUATION(atten, i, worldPos);
               
               //Blinn-Phong
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo + lambertColor * atten;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }

    Fallback "Diffuse"
}
```

效果图标准高光反射

**知识点一 什么是标准高光反射Shader？**  
和上节课一样，所谓的标准高光反射Shader其实就是一个常用的高光反射Shader而已该Shader其实就是一个带有法线（世界空间中计算-全局效果的表现更准确）的基于BlinnPhong光照模型的支持多光源和阴影的Shader

**知识点二 制作常用高光反射Shader**

1.新建一个Shader，取名叫BumpedSpecular（凹凸镜面反射）

2.复制 Lesson52 中Shader代码，粘贴到新建文件中

3.加入渲染标签Tags { "RenderType"="Opaque" "Queue"="Geometry"}  
渲染类型设置为不透明的、渲染队列设置为几何队列（不透明的几何体通常使用该队列）

4.加入阴影、衰减相关代码

5.加入附加渲染通道

6.加入FallBack "Specular"  
实现

```hlsl
Shader "Unlit/BumpedSpecular"
{
    Properties
    {
        _MainColor("MainColor", Color) = (1,1,1,1)
        _MainTex("MainTex", 2D) = ""{}
        _BumpMap("BumpMap", 2D) = ""{}
        _BumpScale("BumpScale", Range(0,1)) = 1
        _SpecularColor("SpecularColor", Color) = (1,1,1,1)
        _SpecularNum("SpecularNum", Range(0,20)) = 18
    }
    SubShader
    {
        Tags{ "RenderType"="Opaque" "Queue"="Geometry"}
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            float4 _MainColor;//漫反射颜色
            sampler2D _MainTex;//颜色纹理
            float4 _MainTex_ST;//颜色纹理的缩放和平移
            sampler2D _BumpMap;//法线纹理
            float4 _BumpMap_ST;//法线纹理的缩放和平移
            float _BumpScale;//凹凸程度
            float4 _SpecularColor;//高光颜色
            fixed _SpecularNum;//光泽度

            struct v2f
            {
                float4 pos:SV_POSITION;
                //float2 uvTex:TEXCOORD0;
                //float2 uvBump:TEXCOORD1;
                //我们可以单独的声明两个float2的成员用于记录 颜色和法线纹理的uv坐标
                //也可以直接声明一个float4的成员 xy用于记录颜色纹理的uv，zw用于记录法线纹理的uv
                float4 uv:TEXCOORD0;
                //顶点相对于世界坐标的位置 主要用于 之后的 视角方向的计算
                //float3 worldPos:TEXCOORD1;
                //切线 到 世界空间的 变换矩阵
                //float3x3 rotation:TEXCOORD2;

                //代表我们切线空间到世界空间的 变换矩阵的3行
                float4 TtoW0:TEXCOORD1;
                float4 TtoW1:TEXCOORD2;
                float4 TtoW2:TEXCOORD3;

                SHADOW_COORDS(4)
            };

            v2f vert (appdata_full v)
            {
                v2f data;
                //把模型空间下的顶点转到裁剪空间下
                data.pos = UnityObjectToClipPos(v.vertex);
                //计算纹理的缩放偏移
                data.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                data.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
                //得到世界空间下的 顶点位置 用于之后在片元中计算视角方向（世界空间下的）
                //data.worldPos = mul(unity_ObjectToWorld, v.vertex);
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex);
                //把模型空间下的法线、切线转换到世界空间下
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                float3 worldTangent = UnityObjectToWorldDir(v.tangent);
                //计算副切线 计算叉乘结果后 垂直与切线和法线的向量有两条 通过乘以 切线当中的w，就可以确定是哪一条
                float3 worldBinormal = cross(normalize(worldTangent), normalize(worldNormal)) * v.tangent.w;
                //这个就是我们 切线空间到世界空间的 转换矩阵
                //data.rotation = float3x3( worldTangent.x, worldBinormal.x,  worldNormal.x,
                //                          worldTangent.y, worldBinormal.y,  worldNormal.y,
                //                          worldTangent.z, worldBinormal.z,  worldNormal.z);
                data.TtoW0 = float4(worldTangent.x, worldBinormal.x,  worldNormal.x, worldPos.x);
                data.TtoW1 = float4(worldTangent.y, worldBinormal.y,  worldNormal.y, worldPos.y);
                data.TtoW2 = float4(worldTangent.z, worldBinormal.z,  worldNormal.z, worldPos.z);

                TRANSFER_SHADOW(data);

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //世界空间下光的方向
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //世界空间下视角方向
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                //通过纹理采样函数 取出法线纹理贴图当中的数据
                float4 packedNormal = tex2D(_BumpMap, i.uv.zw);
                //将我们取出来的法线数据 进行逆运算并且可能会进行解压缩的运算，最终得到切线空间下的法线数据
                float3 tangentNormal = UnpackNormal(packedNormal);
                //乘以凹凸程度的系数
                tangentNormal.xy *= _BumpScale;
                tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
                //把计算完毕后的切线空间下的法线转换到世界空间下
                //float3x3 rotation = float3x3(i.TtoW0.xyz, i.TtoW1.xyz, i.TtoW2.xyz );
                //float3 worldNormal = mul(rotation, tangentNormal);
                //本质 就是在进行矩阵运算
                float3 worldNormal = float3(dot(i.TtoW0.xyz, tangentNormal), dot(i.TtoW1.xyz, tangentNormal), dot(i.TtoW2.xyz, tangentNormal));

                //接下来就来处理 带颜色纹理的 Blinn-Phong光照模型计算

                //颜色纹理和漫反射颜色的 叠加
                fixed3 albedo = tex2D(_MainTex, i.uv.xy) * _MainColor.rgb;
                //兰伯特
                fixed3 lambertColor = _LightColor0.rgb * albedo.rgb * max(0, dot(worldNormal, normalize(lightDir)));
                //半角向量
                float3 halfA = normalize(normalize(viewDir) + normalize(lightDir));
                //高光反射
                fixed3 specularColor = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0, dot(worldNormal, halfA)), _SpecularNum);
                
                UNITY_LIGHT_ATTENUATION(atten, i, worldPos);
                
                //Blinn-Phong
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo + (lambertColor + specularColor)*atten;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }

        Pass
        {
            Tags { "LightMode"="ForwardAdd" }

            Blend One One

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdadd

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            float4 _MainColor;//漫反射颜色
            sampler2D _MainTex;//颜色纹理
            float4 _MainTex_ST;//颜色纹理的缩放和平移
            sampler2D _BumpMap;//法线纹理
            float4 _BumpMap_ST;//法线纹理的缩放和平移
            float _BumpScale;//凹凸程度
            float4 _SpecularColor;//高光颜色
            fixed _SpecularNum;//光泽度

            struct v2f
            {
                float4 pos:SV_POSITION;
                //float2 uvTex:TEXCOORD0;
                //float2 uvBump:TEXCOORD1;
                //我们可以单独的声明两个float2的成员用于记录 颜色和法线纹理的uv坐标
                //也可以直接声明一个float4的成员 xy用于记录颜色纹理的uv，zw用于记录法线纹理的uv
                float4 uv:TEXCOORD0;
                //顶点相对于世界坐标的位置 主要用于 之后的 视角方向的计算
                //float3 worldPos:TEXCOORD1;
                //切线 到 世界空间的 变换矩阵
                //float3x3 rotation:TEXCOORD2;

                //代表我们切线空间到世界空间的 变换矩阵的3行
                float4 TtoW0:TEXCOORD1;
                float4 TtoW1:TEXCOORD2;
                float4 TtoW2:TEXCOORD3;

                SHADOW_COORDS(4)
            };

            v2f vert (appdata_full v)
            {
                v2f data;
                //把模型空间下的顶点转到裁剪空间下
                data.pos = UnityObjectToClipPos(v.vertex);
                //计算纹理的缩放偏移
                data.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                data.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
                //得到世界空间下的 顶点位置 用于之后在片元中计算视角方向（世界空间下的）
                //data.worldPos = mul(unity_ObjectToWorld, v.vertex);
                float3 worldPos = mul(unity_ObjectToWorld, v.vertex);
                //把模型空间下的法线、切线转换到世界空间下
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                float3 worldTangent = UnityObjectToWorldDir(v.tangent);
                //计算副切线 计算叉乘结果后 垂直与切线和法线的向量有两条 通过乘以 切线当中的w，就可以确定是哪一条
                float3 worldBinormal = cross(normalize(worldTangent), normalize(worldNormal)) * v.tangent.w;
                //这个就是我们 切线空间到世界空间的 转换矩阵
                //data.rotation = float3x3( worldTangent.x, worldBinormal.x,  worldNormal.x,
                //                          worldTangent.y, worldBinormal.y,  worldNormal.y,
                //                          worldTangent.z, worldBinormal.z,  worldNormal.z);
                data.TtoW0 = float4(worldTangent.x, worldBinormal.x,  worldNormal.x, worldPos.x);
                data.TtoW1 = float4(worldTangent.y, worldBinormal.y,  worldNormal.y, worldPos.y);
                data.TtoW2 = float4(worldTangent.z, worldBinormal.z,  worldNormal.z, worldPos.z);

                TRANSFER_SHADOW(data);

                return data;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //世界空间下光的方向
                fixed3 lightDir = normalize(_WorldSpaceLightPos0.xyz);
                //世界空间下视角方向
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                //通过纹理采样函数 取出法线纹理贴图当中的数据
                float4 packedNormal = tex2D(_BumpMap, i.uv.zw);
                //将我们取出来的法线数据 进行逆运算并且可能会进行解压缩的运算，最终得到切线空间下的法线数据
                float3 tangentNormal = UnpackNormal(packedNormal);
                //乘以凹凸程度的系数
                tangentNormal.xy *= _BumpScale;
                tangentNormal.z = sqrt(1.0 - saturate(dot(tangentNormal.xy, tangentNormal.xy)));
                //把计算完毕后的切线空间下的法线转换到世界空间下
                //float3x3 rotation = float3x3(i.TtoW0.xyz, i.TtoW1.xyz, i.TtoW2.xyz );
                //float3 worldNormal = mul(rotation, tangentNormal);
                //本质 就是在进行矩阵运算
                float3 worldNormal = float3(dot(i.TtoW0.xyz, tangentNormal), dot(i.TtoW1.xyz, tangentNormal), dot(i.TtoW2.xyz, tangentNormal));

                //接下来就来处理 带颜色纹理的 Blinn-Phong光照模型计算

                //颜色纹理和漫反射颜色的 叠加
                fixed3 albedo = tex2D(_MainTex, i.uv.xy) * _MainColor.rgb;
                //兰伯特
                fixed3 lambertColor = _LightColor0.rgb * albedo.rgb * max(0, dot(worldNormal, normalize(lightDir)));
                //半角向量
                float3 halfA = normalize(normalize(viewDir) + normalize(lightDir));
                //高光反射
                fixed3 specularColor = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0, dot(worldNormal, halfA)), _SpecularNum);
                
                UNITY_LIGHT_ATTENUATION(atten, i, worldPos);
                
                //Blinn-Phong
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb * albedo + (lambertColor + specularColor)*atten;

                return fixed4(color.rgb, 1);
            }
            ENDCG
        }
    }

    Fallback "Specular"
}
```

效果图


## 第 8 节 高级纹理

立方体纹理是什么立方体纹理是什么.pdf总的来说，是由6张2D纹理组成的纹理我们通常用一个向量在上面采样。天空盒

**知识点一 天空盒是什么？**  
天空盒（Skybox）是计算机图形学和三维游戏开发中常用于模拟背景的一种方式,它通常由一个立方体构成字面上理解就是一个用于模拟天空的盒子（但是它不仅可以模拟天空，模拟室内等环境也是可以的）在游戏场景中使用天空盒时，整个场景就被包围在了一个立方体中。  
这个立方体的每个面使用的技术就是立方体纹理技术。它给我们带来的主要好处是：

1.可以创建逼真的背景  
天空盒使场景看起来更加逼真，因为它可以展示复杂和详细的背景

2.可以提高性能  
可以减少渲染远景所需的消耗，只需要要纹理贴图就可以模拟远景的效果，而无需实际的在远处创建这些物体

3.全景视图  
天空盒的六个面（前、后、左、右、上、下）共同构成一个完整的环境，无论玩家向哪个方向看，都能看到连贯的背景图像

4.易于实现  
对于开发者来说，制作和应用天空盒相对简单，只需要准备好六个方向的纹理图像，并将它们应用到立方体纹理中即可等等

**知识点二 通过天空盒来感受立方体纹理的作用**  
在Unity当中使用天空盒非常的简单我们无需写任何代码就可以使用天空盒第一步：创建天空盒材质球

1.创建一个材质球

2.将其Shader设置为Unity自带的天空盒着色器(Skybox/6 Sided)

3.为了让天空盒接缝处不要出现不匹配的现象，我们将准备的纹理的Wrap Mode（循环模式）设置为Clamp（夹紧）

4.将我们准备好的6张纹理赋值给对应的面

5.修改材质球上的各参数：Tint Color（材质整体颜色），Exposure（天空盒亮度），Rotation（天空盒沿正y轴方向的旋转角度）  
第二步：创建测试用场景

1.新建一个测试场景

2.设置天空盒  
2-1:设置全局天空盒（影响所有摄像机）在Lighting面板中设置材质球2-2:单独为摄像机设置天空盒（可以覆盖全局设置）为摄像机添加Skybox组件并关联材质球效果图【立方体纹理】动态生成立方体纹理

**知识点一 为什么要动态生成立方体纹理？**  
我们之前学习的知识中提到过立方体纹理的其中一个最大的作用就是环境映射在实现反射、折射等等效果时，需要用到立方体纹理来制作对应效果而立方体纹理中最重要的就是组成它的6张2D纹理图片对于之前学过的天空盒来说，6张2D纹理图片可以根据想要的美术表现效果来进行自定义制作提前把纹理制作好，直接使用即可这种立方体纹理往往是被提前做好的，是场景中物体们共用的但如果制作反射、折射等效果还是使用这样的立方体纹理，效果肯定不够理想因为物体在场景上的位置不同，产生的对应效果也会是不同的举例说明因此为了更好更真实的表现效果，对于场景中不同位置的物体我们应该为它们在不同位置生成不同的立方体纹理（6张2D纹理贴图）

**知识点二 如何动态生成立方体纹理？**  
我们将结合

1.Unity编辑器拓展

2.Camera中的RenderToCubemap方法  
这两个知识点，在对应位置生成对应的立方体纹理贴图对于一些场景展示类项目，我们不需要实时生成，只需要在编辑器模式下生成一次即可主要要完成的功能为：

1.自定义编辑器窗口，关联对象（通过对象来指定位置）和cubemap变量

2.自定义窗口中有一个生成按钮，点击后使用Camera中的RenderToCubemap自动生成对应的6张2D纹理贴图  
其中编辑器窗口相关功能使用Unity编辑器拓展相关知识自动生成立方体纹理贴图功能使用Camera中的RenderToCubemap方法该方法可以将任意位置观察到的场景图像存储到6张图像中

**知识点三 动态生成立方体纹理**  
按照知识点二中的制作思路来实现逻辑

1.新建一个脚本 RenderToCubemap 放在Editor文件夹中

2.让该类继承EditorWindow 将其作为一个编辑器窗口类

3.实现打开该窗口的静态函数

4.实现OnGUI中的窗口布局，添加以下控件  
4-1:关联位置对象的控件4-2:关联立方体纹理贴图的空间4-3.生成按钮

5.实现具体逻辑  
注意点：

1.我们通过动态创建一个空物体为它添加摄像机组件的形式生成立方体纹理贴图  
该对象为临时对象，使用完毕后即时删除即可

2.cubemap上需要勾选Readable

3.分辨率决定了清晰度

**知识点四 在运行时动态生成**  
Camera中的RenderToCubemap也可以在运行时实时动态生成立方体纹理但是要注意对性能的影响

1.在LateUpdate中使用

2.降低立方体纹理贴图的分辨率

3.分帧渲染，RenderToCubemap有重载，可以一个面一个面的渲染

4.降低更新频率，不要每帧执行  
等等实现

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;

public class RenderToCubeMap : EditorWindow {
    private GameObject obj;//场景上的对象
    private Cubemap cubemap;//生成的立方体纹理

    [MenuItem("立方体纹理动态生成/打开生成窗口")]
    static void OpenWindow() {
        RenderToCubeMap window = EditorWindow.GetWindow<RenderToCubeMap>("纹理生成窗口");
        window.Show();
    }

    private void OnGUI() {
        GUILayout.Label("关联的场景上对应位置对象");
        obj = EditorGUILayout.ObjectField(obj, typeof(GameObject), true) as GameObject;
        
        GUILayout.Label("关联的对应生成立方体纹理资源(需勾选Readable)");
        cubemap = EditorGUILayout.ObjectField(cubemap, typeof(Cubemap), false) as Cubemap;
        
        //点击按钮后执行生成
        if (GUILayout.Button("生成立方体纹理")) {
            if (obj==null || cubemap==null) {
                EditorUtility.DisplayDialog("提醒", "请将上方两个选项执行对应关联", "确定");
                return;
            }
            if (!cubemap.isReadable) {
                EditorUtility.DisplayDialog("提醒", "你的CubMap没有勾选Readable!", "确定");
                return;
            }
            //生成立方体纹理
            GameObject tmpObj=new GameObject("临时对象");
            tmpObj.transform.position=obj.transform.position;
            Camera camera = tmpObj.AddComponent<Camera>();
            camera.RenderToCubemap(cubemap);
            DestroyImmediate(tmpObj);
        }
    }
}
```

效果图【立方体纹理】反射原理反射原理.pdf常常用来模拟镜子或金属表面简单来说，就是通过入射光和反射光角度的计算，最后在立方体纹理中采样，得到结果有人可能会疑惑现实的光人眼接受的不是入射光吗，为什么计算过程貌似反过来求反射光了？请参考pdf里面的光路可逆原则，反射光就是我们接受到的目标光顶点。  
【立方体纹理】反射的基础实现

**知识点一 实现反射基础效果使用的新知识点**  
立方体纹理采样函数CG中提供了内置函数 texCUBE 用于进行立方体纹理采样texCUBE(立方体纹理,反射方向向量) 便可以得到在立方体纹理中的采样结果

**知识点二 实现反射效果**

1.属性声明  
我们将声明2个关键属性1-1:立方体纹理(这里的纹理是固定的，需要我们用上一节课的立方体纹理生成然后传入)1-2:反射率（0~1）之间

2.顶点着色器

**关键步骤：**  
2-1:顶点坐标转裁剪坐标2-2:顶点法线转世界坐标2-3:顶点坐标转世界坐标2-4:世界空间下 视角方向计算2-5:视角反向逆向得到反射方向

3.片元着色器

**关键步骤：**  
3-1:立方体纹理采样(利用texCUBE函数)3-2:结合反射程度返回最终颜色实现

```hlsl
Shader "Unlit/Lesson75_ReflectBase"
{
    Properties
    {
        //立方体纹理
        _Cube("Cubemap", Cube) = ""{}
        //反射率
        _Reflectivity("Reflectivity", Range(0,1)) = 1
    }
    SubShader
    {
        Tags{"RenderType"="Opaque" "Queue"="Geometry"}

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            samplerCUBE _Cube;
            float _Reflectivity;

            struct v2f
            {
                float4 pos:SV_POSITION;//裁剪空间下的顶点坐标
                //世界空间下的反射向量
                //我们将把反射向量的计算放在顶点着色器函数中 节约性能 表现效果也不会太差 肉眼几乎分辨不出来
                float3 worldRefl:TEXCOORD0;
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //计算反射光向量
                //1.计算世界空间下法线向量
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                //2.世界空间下的顶点坐标
                fixed3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //3.计算视角方向 内部是用摄像机位置 - 世界坐标位置 
                fixed3 worldViewDir = UnityWorldSpaceViewDir(worldPos);
                //4.计算反射向量
                o.worldRefl = reflect(-worldViewDir, worldNormal);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //对立方体纹理利用对应的反射向量进行采样
                fixed4 cubemapColor = texCUBE(_Cube, i.worldRefl);
                //用采样颜色 * 反射率 决定最终的颜色效果
                return cubemapColor * _Reflectivity;
            }

            ENDCG
        }
    }
}
```

效果图【立方体纹理】反射结合漫反射

**知识点一 使用的新知识点**  
插值函数CG中提供了内置函数 lerp 用于进行插值计算lerp(a, b, t)a:插值起点值b:插值终点值t:插值因子，0~~1之间内部计算公式：a + t * (b - a)当 t = 0 时，插值结果为a当 t = 1 时，插值结果为b当 t 在 0~~1 之间变化时，返回一个a到b之间的一个线性插值结果我们将利用该函数来决定反射效果的程度用它在漫反射颜色和反射颜色之间进行插值控制反射程度

**知识点二 实现带漫反射的 反射效果**  
新建一个Shader，复制上节课的代码

1.属性声明  
我们将加入2个关键属性1-1:漫反射颜色1-2:反射颜色

2.v2f结构体  
因为要在片元着色器中处理光和阴影需要加入2-1：世界空间法线:2-2：世界空间顶点位置2-3: 阴影宏

3.顶点着色器

**关键步骤：**  
3-1:顶点坐标转裁剪坐标3-2:顶点法线转世界坐标3-3:顶点坐标转世界坐标3-4:世界空间下 视角方向计算3-5:视角反向逆向得到反射方向3-6:阴影相关计算

4.片元着色器

**关键步骤：**  
4-1:得到光的方向4-2:兰伯特漫反射颜色计算4-3:立方体纹理采样(利用texCUBE函数)4-4:衰减计算4-5:最终颜色计算(利用lerp函数)

4.FallBack "Reflective/VertexLit"  
实现

```hlsl
Shader "Unlit/Lesson76_Reflection"
{
    Properties
    {
        //漫反射颜色
        _Color("Color", Color) = (1,1,1,1)
        //反射颜色
        _ReflectColor("ReflectColor", Color) = (1,1,1,1)
        //立方体纹理
        _Cube("Cubemap", Cube) = ""{}
        //反射率
        _Reflectivity("Reflectivity", Range(0,1)) = 1
    }
    SubShader
    {
        Tags{"RenderType"="Opaque" "Queue"="Geometry"}

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed4 _Color;
            fixed4 _ReflectColor;
            samplerCUBE _Cube;
            float _Reflectivity;

            struct v2f
            {
                float4 pos:SV_POSITION;//裁剪空间下的顶点坐标
                //世界空间下法线
                fixed3 worldNormal:NORMAL;
                //世界空间下的顶点位置
                float3 worldPos:TEXCOORD0;
                //世界空间下的反射向量
                //我们将把反射向量的计算放在顶点着色器函数中 节约性能 表现效果也不会太差 肉眼几乎分辨不出来
                float3 worldRefl:TEXCOORD1;
                //阴影相关
                SHADOW_COORDS(2)
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //计算反射光向量
                //1.计算世界空间下法线向量
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                //2.世界空间下的顶点坐标
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //3.计算视角方向 内部是用摄像机位置 - 世界坐标位置 
                fixed3 worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
                //4.计算反射向量
                o.worldRefl = reflect(-worldViewDir, o.worldNormal);

                //阴影相关处理
                TRANSFER_SHADOW(o);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //漫反射光照相关计算
                //得到光的方向
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                //漫反射颜色
                fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(normalize(i.worldNormal), worldLightDir));

                //对立方体纹理利用对应的反射向量进行采样
                fixed3 cubemapColor = texCUBE(_Cube, i.worldRefl).rgb * _ReflectColor.rgb;

                //得到光照衰减以及阴影相关的衰减值
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);

                //我们利用lerp 在漫反射颜色和反射颜色之间 进行插值 0和1就是极限状态 0 没有反射效果 1只有反射效果 0~1之间就是两者叠加
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb + lerp(diffuse, cubemapColor, _Reflectivity) * atten;

                //用采样颜色 * 反射率 决定最终的颜色效果
                return fixed4(color, 1.0);
            }

            ENDCG
        }
    }
    FallBack "Reflective/VertexLit"
}
```

效果图看得出来，主要是多了一层漫反射的光亮感还有通过lerp插值实现了反射和完全不反射效果的可配置。【立方体纹理】折射原理折射原理.pdf一般用来模拟水面、透明玻璃球、眼镜、钻石、水晶球、空气扰动等等效果和反射类似，但是这里使用斯涅耳定律来获取目标向量。  
现实中折射是在多种介质里面多次发生的，但是鉴于性能和计算难度，很多情况下我们会直接默认只发生了一次。【立方体纹理】折射的基础实现

**知识点一 使用的新知识点**  
折射向量计算函数CG中提供了内置函数 refract 用于进行折射向量的计算refract(入射方向单位向量, 顶点法线单位向量, 入射介质折射率/射入介质折射率)便可以得到在目标介质中的折射向量该函数内部就是利用了斯涅耳定律进行的计算我们无需再自己写逻辑计算了

**知识点二 折射的基础实现**

1.属性声明  
我们将声明4个关键属性1-1:介质A折射率1-2:介质B折射率1-3:立方体纹理贴图1-4:折射程度

2.编译指令、内置文件、属性映射、结构体相关

3.顶点着色器

**关键步骤：**  
3-1:顶点坐标转裁剪坐标3-2:顶点法线转世界坐标3-3:顶点坐标转世界坐标3-4:世界空间下 视角方向计算3-5:利用折射函数计算折射向量

4.片元着色器

**关键步骤：**  
3-1:立方体纹理采样(利用texCUBE函数)3-2:结合折射程度返回最终颜色实现

```hlsl
Shader "Unlit/Lesson77_RefractionBase"
{
    Properties
    {
        //介质A折射率
        _RefractiveIndexA("RefractiveIndexA", Range(1,2)) = 1
        //介质B折射率
        _RefractiveIndexB("RefractiveIndexB", Range(1,2)) = 1.3
        //立方体纹理贴图
        _Cube("Cubemap", Cube) = ""{}
        //折射程度
        _RefracAmount("RefracAmount", Range(0,1)) = 1
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "Queue"="Geometry" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            samplerCUBE _Cube;
            fixed _RefractiveIndexA;
            fixed _RefractiveIndexB;
            fixed _RefracAmount;

            struct v2f
            {
                //裁剪空间下顶点坐标
                float4 pos:SV_POSITION;
                //折射向量
                float3 worldRefr:TEXCOORD0;
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //法线转世界
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转世界
                fixed3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //视角方向获取 摄像机 - 顶点位置
                fixed3 worldViewDir = UnityWorldSpaceViewDir(worldPos);
                //计算折射向量
                //第三个参数一定是 介质A/介质B的结果 我们可以声明一个变量在外部算好传进来 这里我们用两个变量只是为了讲解知识
                o.worldRefr = refract(-normalize(worldViewDir), normalize(worldNormal), _RefractiveIndexA/_RefractiveIndexB);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //立方体纹理采样
                fixed4 cubemapColor = texCUBE(_Cube, i.worldRefr);
                //结合折射程度进行计算返回
                return cubemapColor * _RefracAmount;
            }

            ENDCG
        }
    }
}
```

效果图折射结合漫反射及阴影

**知识点 实现带漫反射和阴影的 折射效果**

1.新建一个Shader，复制上节课的代码

2.将折射率A和折射率B 合并为一个 折射率比值变量  
以后在外部算好了传入，避免内部计算浪费性能

3.折射效果结合漫反射和阴影的做法  
和之前反射几乎一模一样我们可以直接复制之前的代码避免书写重复内容3-1:属性相关代码复制漫反射颜色和折射颜色3-2:渲染路径、编译指令、内置文件复制属性映射复制修改3-3:结构体相关内容复制3-4:顶点着色器相关内容修改3-5:片元着色器相关内容复制修改实现

```hlsl
Shader "Unlit/Lesson78_Refraction"
{
    Properties
    {
        //漫反射颜色
        _Color("Color", Color) = (1,1,1,1)
        //折射颜色
        _RefractColor("RefractColor", Color) = (1,1,1,1)
        //折射率比值 介质A折射率/介质B折射率
        _RefractRatio("RefractRatio", Range(0.1, 1)) = 0.5
        //立方体纹理贴图
        _Cube("Cubemap", Cube) = ""{}
        //折射程度
        _RefracAmount("RefracAmount", Range(0,1)) = 1
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" "Queue"="Geometry" }

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed4 _Color;
            fixed4 _RefractColor;
            samplerCUBE _Cube;
            fixed _RefractRatio;
            fixed _RefracAmount;

            struct v2f
            {
                //裁剪空间下顶点坐标
                float4 pos:SV_POSITION;
                 //世界空间下法线
                fixed3 worldNormal:NORMAL;
                //世界空间下的顶点位置
                float3 worldPos:TEXCOORD0;
                //折射向量
                float3 worldRefr:TEXCOORD1;
                //阴影相关
                SHADOW_COORDS(2)
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //法线转世界
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                //顶点转世界
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //视角方向获取 摄像机 - 顶点位置
                fixed3 worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
                //计算折射向量
                //第三个参数一定是 介质A/介质B的结果 我们可以声明一个变量在外部算好传进来 这里我们用两个变量只是为了讲解知识
                o.worldRefr = refract(-normalize(worldViewDir), normalize(o.worldNormal), _RefractRatio);

                //阴影相关处理
                TRANSFER_SHADOW(o);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //漫反射光照相关计算
                //得到光的方向
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                //漫反射颜色
                fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(normalize(i.worldNormal), worldLightDir));

                //立方体纹理采样
                fixed3 cubemapColor = texCUBE(_Cube, i.worldRefr).rgb * _RefractColor.rgb;
                
                //得到光照衰减以及阴影相关的衰减值
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
                
                //我们利用lerp 在漫反射颜色和反射颜色之间 进行插值 0和1就是极限状态 0 没有折射效果 1只有折射效果 0~1之间就是两者叠加
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb + lerp(diffuse, cubemapColor, _RefracAmount) * atten;

                //结合折射程度进行计算返回
                return fixed4(color, 1.0);
            }

            ENDCG
        }
    }
    FallBack "Reflective/VertexLit"
}
```

效果图其实效果蛮违和的....就只放张静态图了。左下方的cube是阴影投射的示范【立方体纹理】菲涅耳反射原理菲涅尔反射原理.pdf也就是现实里面大部分的情况是反射和折射共存的——观察是垂直角度的时候折射较强，而角度越小反射越明显。（如湖面和地上的水坑）菲涅尔公式就是为了模拟这个现象，但是由于计算复杂，我们实现的时候一般只用简化版的公式。  
而这一节我们注意实现的是放射【立方体纹理】菲涅尔反射基础实现

**知识点一 使用的新知识点**  
Schlick 菲涅耳近似等式R(θ) = R0​ + ( 1− R0​ )( 1 − V·𝑵)^𝟓R(θ)：对应角度的菲涅耳反射率R0：介质反射率V：视角单位向量N：法线单位向量使用该等式计算菲涅耳反射率，参与颜色计算

**知识点二 菲涅尔反射的基础实现**

1.新建Shader 复制Lesson75_ReflectBase中代码  
在其基础上进行修改

2.属性声明  
将反射率变量修改为 \_FresnelScale 菲涅耳中介质的反射率

3.结构体  
由于使用Schlick菲涅耳近似等式需要用到世界空间下视角方向、和法线向量因此在结构体中加入这两个变量

4.顶点着色器

**关键步骤：**  
结构体中变量赋值

5.片元着色器

**关键步骤：**  
利用Schlick菲涅耳近似等式计算出菲涅耳反射率 参与最终颜色计算实现值得注意的是，这里并没有折射内容，而是实现了菲涅尔的更高级的反射一个特点就是边缘明显有高亮感——这类似现实里面你侧面看向菲涅尔反射物体的感觉(原因请观察代码里面的schlick菲涅耳近似等式)下面的效果图可以看出更直观的感受

```hlsl
Shader "Unlit/Lesson79_FresnelBase"
{
    Properties
    {
        //立方体纹理
        _Cube("Cubemap", Cube) = ""{}
        //菲涅耳反射中 对应介质的反射率
        _FresnelScale("FresnelScale", Range(0,1)) = 1
    }
    SubShader
    {
        Tags{"RenderType"="Opaque" "Queue"="Geometry"}

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            samplerCUBE _Cube;
            float _FresnelScale;

            struct v2f
            {
                float4 pos:SV_POSITION;//裁剪空间下的顶点坐标
                //世界空间下的法线
                float3 worldNormal:NORMAL;
                //世界空间下视角的方向
                float3 worldViewDir:TEXCOORD0;
                //世界空间下的反射向量
                //我们将把反射向量的计算放在顶点着色器函数中 节约性能 表现效果也不会太差 肉眼几乎分辨不出来
                float3 worldRefl:TEXCOORD1;
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //计算反射光向量
                //1.计算世界空间下法线向量
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                //2.世界空间下的顶点坐标
                fixed3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //3.计算视角方向 内部是用摄像机位置 - 世界坐标位置 
                o.worldViewDir = UnityWorldSpaceViewDir(worldPos);
                //4.计算反射向量
                o.worldRefl = reflect(-o.worldViewDir, o.worldNormal);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //对立方体纹理利用对应的反射向量进行采样
                fixed4 cubemapColor = texCUBE(_Cube, i.worldRefl);

                //利用schlick菲涅耳近似等式 计算菲涅耳反射率
                fixed fresnel = _FresnelScale + (1-_FresnelScale) * pow(1-dot(normalize(i.worldViewDir), normalize(i.worldNormal)), 5);

                //用采样颜色 * 反射率 决定最终的颜色效果
                return cubemapColor * fresnel;
            }

            ENDCG
        }
    }
}
```

效果图【立方体纹理】菲涅尔反射结合漫反射及阴影

**知识点 菲涅尔反射结合漫反射**  
由于菲涅耳反射是基于反射的因此我们在之前实现的反射结合漫反射的基础上修改即可

1.新建Shader 复制LLesson76_Reflection的代码

2.修改属性  
2-1:去掉反射颜色2-2:修改反射率为菲涅耳相关的R0反射率

3.修改v2f结构体  
加入视角方向（需要再片元着色器中用菲涅耳近似公式参与计算）

4.顶点着色器  
修改视角方向临时变量

5.片元着色器  
5-1:计算菲涅耳反射率5-2:用菲涅耳反射率参与lerp计算实现

```hlsl
Shader "Unlit/Lesson80_Fresnel"
{
    Properties
    {
        //漫反射颜色
        _Color("Color", Color) = (1,1,1,1)
        //立方体纹理
        _Cube("Cubemap", Cube) = ""{}
        //反射率
        _FresnelScale("FresnelScale", Range(0,1)) = 1
    }
    SubShader
    {
        Tags{"RenderType"="Opaque" "Queue"="Geometry"}

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma multi_compile_fwdbase

            #include "UnityCG.cginc"
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed4 _Color;
            samplerCUBE _Cube;
            float _FresnelScale;

            struct v2f
            {
                float4 pos:SV_POSITION;//裁剪空间下的顶点坐标
                //世界空间下法线
                fixed3 worldNormal:NORMAL;
                //世界空间下的顶点位置
                float3 worldPos:TEXCOORD0;
                //世界空间下视角的方向
                float3 worldViewDir:TEXCOORD1;
                //世界空间下的反射向量
                //我们将把反射向量的计算放在顶点着色器函数中 节约性能 表现效果也不会太差 肉眼几乎分辨不出来
                float3 worldRefl:TEXCOORD2;
                //阴影相关
                SHADOW_COORDS(3)
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //计算反射光向量
                //1.计算世界空间下法线向量
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                //2.世界空间下的顶点坐标
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //3.计算视角方向 内部是用摄像机位置 - 世界坐标位置 
                o.worldViewDir = UnityWorldSpaceViewDir(o.worldPos);
                //4.计算反射向量
                o.worldRefl = reflect(-o.worldViewDir, o.worldNormal);

                //阴影相关处理
                TRANSFER_SHADOW(o);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //漫反射光照相关计算
                //得到光的方向
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                //漫反射颜色
                fixed3 diffuse = _LightColor0.rgb * _Color.rgb * max(0, dot(normalize(i.worldNormal), worldLightDir));

                //对立方体纹理利用对应的反射向量进行采样
                fixed3 cubemapColor = texCUBE(_Cube, i.worldRefl).rgb;

                //得到光照衰减以及阴影相关的衰减值
                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);

                //菲涅耳反射率计算
                fixed fresnel = _FresnelScale + (1-_FresnelScale)*pow(1 - dot(normalize(i.worldViewDir), normalize(i.worldNormal)), 5);

                //我们利用lerp 在漫反射颜色和反射颜色之间 进行插值 0和1就是极限状态 0 没有反射效果 1只有反射效果 0~1之间就是两者叠加
                fixed3 color = UNITY_LIGHTMODEL_AMBIENT.rgb + lerp(diffuse, cubemapColor, fresnel) * atten;

                //用采样颜色 * 反射率 决定最终的颜色效果
                return fixed4(color, 1.0);
            }

            ENDCG
        }
    }
    FallBack "Reflective/VertexLit"
}
```

效果图【渲染纹理】渲染目标纹理是什么

**知识点一 渲染目标纹理是什么？**  
渲染目标纹理（Render Target Texture）是一种特殊类型的纹理一般摄像机的渲染结果会输出到颜色缓冲区中最终渲染到设备屏幕上，让玩家通过屏幕看见游戏画面而渲染目标纹理允许我们将渲染结果直接写入到某一张纹理中我们可以利用这个纹理来处理各种特殊效果比如：镜子、玻璃、屏幕后处理、阴影映射等等在Unity中渲染目标纹理（Render Target Texture）通常和渲染纹理（Render Texture）可以互换使用，指的相同的概念。  
在实际使用中，我们更常听到的是"渲染纹理"这个术语说人话：渲染纹理就是将渲染结果存储到一个纹理对象中，以便在后续的渲染步骤中使用它的作用体现在：我们在进行Shader开发时我们经常会希望某摄像机不要直接将结果渲染到屏幕上而是可以让我们得到它的渲染结果进行二次处理或利用

**知识点二 Unity中的渲染目标纹理**  
常用方法

1.渲染纹理（Render Texture）  
Unity引擎中提供的一种专门的纹理类型我们可以在Project窗口中右键Create――>Custom Render Texturn创建我们只需要把该纹理进行相关的设置，然后关联到摄像机组件的Target Texture中即可这样对应摄像机渲染的内容就会直接写入到该纹理中我们便可以使用它来进行相关操作注意：除了手动创建，我们也可以通过代码创建并关联

2.GrabPass  
在Unity Shader当中我们可以在Pass渲染通道中使用 GrabPass 抓取指令捕获当前屏幕内容并将其保存为纹理，以便在后续的渲染过程中使用

3.OnRenderImage  
在继承了MonoBehaviour的脚本中我们可以使用OnRenderImage来获取摄像机渲染完成的图像该函数一般用于实现自定义的图像后处理效果相当于将摄像机渲染完成的图像进行二次处理等等【渲染纹理】镜面效果实现贴图参数介绍Custom Render Texturn参数介绍.xmind

**知识点一 镜面效果原理**  
镜面效果的原理就是将摄像机看到的画面渲染到渲染纹理当中再在Shader中利用该渲染纹理进行翻转渲染即可(因为镜子的镜像性)

**知识点二 镜面效果准备工作**

1.创建测试场景

2.创建一个摄像机用来得到镜子看到的画面

3.创建Custom Render Texture 将其和摄像机关联  
知识三 实现镜面效果Shader

1.新建Shader，复制 Lesson48 单张纹理_纹理颜色显示Shader

2.在顶点着色器中，让UV坐标的X轴反向（坐标范围0~1，想要反向直接用 1 - uv.x 即可)

3.将渲染纹理应用到该Shader的纹理中  
实现效果图【渲染纹理】玻璃效果 基本原理玻璃效果原理.pdf由于玻璃经常有折射，反射等效果，基础管线的普通透明混合效果并不容易实现所以，我们通常会在渲染玻璃之前，用GrabPass捕获当前屏幕效果，然后基于当前屏幕渲染玻璃，实现丰富的效果。  
【渲染纹理】基础玻璃效果实现

**知识回顾**

1.玻璃效果基本原理  
在渲染玻璃效果之前，先捕获当前屏幕内容并保存到一张渲染纹理当中，在之后的Shader处理中利用该渲染纹理进行采样，参与最终的颜色计算，实现各种玻璃效果。

2.玻璃效果实现使用的新知识点  
捕获屏幕内容的GrabPass；裁剪坐标转屏幕坐标的ComputeGrabScreenPos方法捕获纹理进行采样时，进行一些偏移计算，模拟折射效果

3.渲染标签中的 渲染队列  
Tags{ "Queue" = "标签值" }常用Unity预先定义好的渲染队列标签值：3-1.Background(背景)(队列号:1000)最早被渲染的物体的队列，一般用来渲染天空盒或者背景Tags{ "Queue" = "Background" }3-2.Geometry(几何)(队列号:2000)不透明的几何体通常使用该队列，当没有声明渲染队列时，Unity会默认使用这个队列Tags{ "Queue" = "Geometry" }3-3.AlphaTest(透明测试)(队列号:2450)有透明通道的，需要进行Alpha测试的几何体会使用该队列当所有Geometry队列实体绘制完后再绘制AlphaTest队列，效率更高Tags{ "Queue" = "AlphaTest" }3-4.Transparent(透明的)(队列号:3000)该队列中几何体按照由远到近的顺序进行绘制，半透明物体的渲染队列，所有进行透明混合的几何体都应该使用该队列比如：玻璃材质，粒子特效等Tags{ "Queue" = "Transparent" }3-5.Overlay(覆盖)(队列号:4000)用是放在最后渲染的队列，于叠加渲染的效果，比如镜头光晕等Tags{ "Queue" = "Overlay" }3-6.自定义队列基于Unity预先定义好的这些渲染队列标签来进行加减运算来定义自己的渲染队列比如：Tags{ "Queue" = "Geometry+1" } 代表的队列号就是 2001Tags{ "Queue" = "Transparent-1" } 代表的队列号就是 2999自定义队列在一些特殊情况下，特别有用比如 一些水的渲染 想要在不透明物体之后，半透明物体之前进行渲染，就可以自定义

**知识点一 如何让玻璃效果对象滞后渲染**  
我们上节课知道在实现玻璃效果之前，需要先捕获当前屏幕内容并保存到一张渲染纹理当中那么要保证玻璃效果对象后面的内容正确渲染，我们必须保证玻璃对象能够滞后渲染想要让一个对象滞后渲染，那么通过我们学习过的知识，自然的联想到了渲染标签Tags中的 渲染队列Queue因此对于玻璃效果对象，虽然它本质上是一个不透明物体但是我们完全可以将它的渲染队列设置为 Transparent(透明的)保证它晚于 背景队列、几何队列、透明测试队列 之后再进行渲染这时我们捕获的屏幕内容，将包含这些更早渲染的内容信息便可以利用GrabPass捕获到相对正确的内容了

**知识点二 基础玻璃效果实现**

1.新建一个Shader，复制反射基础实现Lesson75_ReflectBase中的代码

2.修改相关代码  
2-1:修改属性代码（同时修改CG中的属性映射）加入主纹理属性（用于处理物体本身颜色）加入立方体纹理属性（用于处理反射）将反射率改为折射程度（用于控制折射程度 0表示完全不折射-相当于完全反射，1表示完全折射-相当于完全透明）2-2:修改渲染队列为Transparent，但是RenderType渲染类型不修改，因为它本质上还是一个不透明物体以后使用着色器替换功能时，可以在被正常渲染2-3:加入GrabPass，抓取屏幕图像存储渲染纹理2-4:修改v2f结构体加入相对屏幕坐标float4类型成员加入uv，用于采样物体颜色纹理2-5:修改顶点着色器使用ComputeGrabScreenPos方法，计算结构体中相对屏幕坐标计算纹理的缩放偏移2-6:修改片元着色器用uv采样主纹理颜色将屏幕坐标转为裁剪坐标0~~1范围内，对捕获纹理进行采样用反射在立方体纹理中进行采样，用结果乘以主纹理颜色，进行颜色叠加用折射程度参与最终的颜色计算，折射程度值的变化决定了最终表现效果在 完全反射~~完全折射 之间变化

**知识点三 自定义折射效果**  
目前我们利用屏幕坐标在抓取纹理中采样并没有处理偏移因此呈现出来的效果像是在透明和半透明之前切换我们可以自定义一些简单的偏移计算规则让最终的采样位置发生偏移，模拟折射效果实现

```hlsl
Shader "Unlit/Lesson85_GlassBase"
{
     Properties
    {
        //主纹理
        _MainTex("MainTex", 2D) = ""{}
        //立方体纹理
        _Cube("Cubemap", Cube) = ""{}
        //折射程度 0~1 0代表完全反射（完全不折射）1代表完全折射（透明效果 相当于光全部进入了内部）
        _RefractAmount("RefractAmount", Range(0,1)) = 1
    }
    SubShader
    {
        //将渲染队列改为透明的 目的是让玻璃对象 滞后渲染
        //能够捕获到之前正确的屏幕图像
        Tags{"RenderType"="Opaque" "Queue"="Transparent"}
        //使用它来捕获当前屏幕内容 并存储到默认的渲染纹理变量中
        GrabPass{}

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;
            samplerCUBE _Cube;
            float _RefractAmount;
            //GrabPass默认存储的纹理变量 这个是规则
            sampler2D _GrabTexture;

            struct v2f
            {
                float4 pos:SV_POSITION;//裁剪空间下的顶点坐标
                //用于存储从屏幕图像中采样的坐标（顶点相对于屏幕的位置）
                float4 grabPos:TEXCOORD0;
                //用于在颜色纹理中采样的UV坐标
                float2 uv:TEXCOORD1;
                //世界空间下的反射向量
                //我们将把反射向量的计算放在顶点着色器函数中 节约性能 表现效果也不会太差 肉眼几乎分辨不出来
                float3 worldRefl:TEXCOORD2;
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //屏幕坐标转换相关的内容
                o.grabPos = ComputeGrabScreenPos(o.pos);
                //uv坐标计算相关的内容
                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                //计算反射光向量
                //1.计算世界空间下法线向量
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                //2.世界空间下的顶点坐标
                fixed3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                //3.计算视角方向 内部是用摄像机位置 - 世界坐标位置 
                fixed3 worldViewDir = UnityWorldSpaceViewDir(worldPos);
                //4.计算反射向量
                o.worldRefl = reflect(-worldViewDir, worldNormal);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //反射颜色相关的计算 会叠加主纹理颜色
                //对颜色纹理进行采样
                fixed4 mainTex = tex2D(_MainTex, i.uv);
                //将反射颜色和主纹理颜色进行叠加
                fixed4 reflColor = texCUBE(_Cube, i.worldRefl) * mainTex;
                
                //折射相关的颜色
                //其实就是从我们抓取的 屏幕渲染纹理中进行采样 参与计算
                //抓取纹理中的颜色信息 相当于是这个玻璃对象后面的颜色

                //想要有折射效果 可以在采样之前 进行xy屏幕坐标的偏移
                float2 offset = 1 - _RefractAmount;
                //xy偏移一个位置
                i.grabPos.xy = i.grabPos.xy - offset/10; 

                //利用透视除法 将屏幕坐标转换到 0~1范围内 然后再进行采样
                fixed2 screenUV = i.grabPos.xy / i.grabPos.w;
                //从捕获的渲染纹理中进行采样 获取后面的颜色
                fixed4 grabColor = tex2D(_GrabTexture, screenUV);

                //折射程度 0~1 0代表完全反射（完全不折射）1代表完全折射（透明效果 相当于光全部进入了内部）
                float4 color = reflColor*(1 - _RefractAmount) + grabColor * _RefractAmount;

                return color;
            }

            ENDCG
        }
    }
}
```

效果图带法线纹理的玻璃效果实现（加密）

**知识点一 什么是带法线纹理的玻璃效果**  
就是在上一节的基础上，允许我们指定法线纹理来营造凹凸感和影响折射反射。

**知识点二 带法线纹理的玻璃效果实现**

1.创建Shader 复制上节课的相关代码

2.将 BumpedDiffuse 标准法线漫反射Shader中关于法线相关的计算整合进来  
注意：不再需要_BumpScale来控制凹凸程度，我们默认法线凹凸程度最大化

3.修改反射向量计算规则  
由于法线需要从法线纹理中获取因此需要将反射向量的计算放入到片元着色器中

**知识点三 利用切线空间法线来计算折射偏移**

1.加入一个控制折射扭曲程度的新属性_Distortion 取值范围可以大一些

2.利用切线空间下法线来计算偏移值  
加入两行关键代码第一行float2 offset = tangentNormal.xy * \_Distortion ;使用切线空间下法线的xy * 扭曲值得到一个偏移量代表光线经过法线方向扰动后的偏移程度，确定光线折射的方向和强度第二行屏幕坐标.xy = offset * 屏幕坐标.z + 屏幕坐标.xy;用偏移量和屏幕空间深度值相乘，模拟出真实的折射效果深度值越大（即距离相机越远），折射效果越明显。  
这样可以实现近大远小的效果，使得物体在不同深度上的折射效果有所差异。这种计算方式是图形学前辈们通过实践总结出来的接近真实世界折射效果的方法本人补充：这条式子不是物理折射，而是美术近似：用深度（或深度差）放大偏移，制造在一个点观看玻璃的不同部分的时候，有一种“远处更扭曲 / 近处更清晰”的视觉层次感。(可以观察下面效果图)在视觉上它还能顺便模拟部分玻璃“随视角/距离变化的飘动感”，但真实折射由 IOR、入射角、法线等决定，但是运算难度太大，这是一个取巧的办法。  
实现

```hlsl
Shader "Unlit/Lesson86_GlassRefraction"
{
     Properties
    {
        //主纹理
        _MainTex("MainTex", 2D) = ""{}
        //法线纹理
        _BumpMap("BumpMap", 2D) = ""{}
        //立方体纹理
        _Cube("Cubemap", Cube) = ""{}
        //折射程度 0~1 0代表完全反射（完全不折射）1代表完全折射（透明效果 相当于光全部进入了内部）
        _RefractAmount("RefractAmount", Range(0,1)) = 1
        //控制折射扭曲程度的变量
        _Distortion("Distortion", Range(0,10)) = 0
    }
    SubShader
    {
        //将渲染队列改为透明的 目的是让玻璃对象 滞后渲染
        //能够捕获到之前正确的屏幕图像
        Tags{"RenderType"="Opaque" "Queue"="Transparent"}
        //使用它来捕获当前屏幕内容 并存储到默认的渲染纹理变量中
        GrabPass{}

        Pass
        {
            Tags{"LightMode"="ForwardBase"}
            
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"
            #include "Lighting.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;//法线纹理
            float4 _BumpMap_ST;//法线纹理的缩放和平移
            samplerCUBE _Cube;
            float _RefractAmount;
            //GrabPass默认存储的纹理变量 这个是规则
            sampler2D _GrabTexture;
            float _Distortion;

            struct v2f
            {
                float4 pos:SV_POSITION;//裁剪空间下的顶点坐标
                //用于存储从屏幕图像中采样的坐标（顶点相对于屏幕的位置）
                float4 grabPos:TEXCOORD0;
                //用于在颜色纹理中采样的UV坐标
                float4 uv:TEXCOORD1;
                //世界空间下的反射向量
                //我们将把反射向量的计算放在顶点着色器函数中 节约性能 表现效果也不会太差 肉眼几乎分辨不出来
                //float3 worldRefl:TEXCOORD2;

                //代表我们切线空间到世界空间的 变换矩阵的3行
                float4 TtoW0:TEXCOORD3;
                float4 TtoW1:TEXCOORD4;
                float4 TtoW2:TEXCOORD5;
            };

            v2f vert(appdata_full v)
            {
                v2f o;
                //顶点坐标转换
                o.pos = UnityObjectToClipPos(v.vertex);
                //屏幕坐标转换相关的内容
                o.grabPos = ComputeGrabScreenPos(o.pos);
                //uv坐标计算相关的内容
                o.uv.xy = v.texcoord.xy * _MainTex_ST.xy + _MainTex_ST.zw;
                //法线纹理的uv坐标计算
                o.uv.zw = v.texcoord.xy * _BumpMap_ST.xy + _BumpMap_ST.zw;
                //计算反射光向量
                //1.计算世界空间下法线向量
                float3 worldNormal = UnityObjectToWorldNormal(v.normal);
                //2.世界空间下的顶点坐标
                fixed3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                ////3.计算视角方向 内部是用摄像机位置 - 世界坐标位置 
                //fixed3 worldViewDir = UnityWorldSpaceViewDir(worldPos);
                ////4.计算反射向量
                //o.worldRefl = reflect(-worldViewDir, worldNormal);

                //世界空间下切线
                float3 worldTangent = UnityObjectToWorldDir(v.tangent);
                //计算副切线 计算叉乘结果后 垂直与切线和法线的向量有两条 通过乘以 切线当中的w，就可以确定是哪一条
                float3 worldBinormal = cross(normalize(worldTangent), normalize(worldNormal)) * v.tangent.w;

                o.TtoW0 = float4(worldTangent.x, worldBinormal.x,  worldNormal.x, worldPos.x);
                o.TtoW1 = float4(worldTangent.y, worldBinormal.y,  worldNormal.y, worldPos.y);
                o.TtoW2 = float4(worldTangent.z, worldBinormal.z,  worldNormal.z, worldPos.z);

                return o;
            }

            fixed4 frag(v2f i):SV_TARGET
            {
                //世界空间下视角方向
                float3 worldPos = float3(i.TtoW0.w, i.TtoW1.w, i.TtoW2.w);
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));
                //通过纹理采样函数 取出法线纹理贴图当中的数据
                float4 packedNormal = tex2D(_BumpMap, i.uv.zw);
                //将我们取出来的法线数据 进行逆运算并且可能会进行解压缩的运算，最终得到切线空间下的法线数据
                float3 tangentNormal = UnpackNormal(packedNormal);
                //把计算完毕后的切线空间下的法线转换到世界空间下
                //float3x3 rotation = float3x3(i.TtoW0.xyz, i.TtoW1.xyz, i.TtoW2.xyz );
                //float3 worldNormal = mul(rotation, tangentNormal);
                //本质 就是在进行矩阵运算
                float3 worldNormal = float3(dot(i.TtoW0.xyz, tangentNormal), dot(i.TtoW1.xyz, tangentNormal), dot(i.TtoW2.xyz, tangentNormal));

                //反射颜色相关的计算 会叠加主纹理颜色
                //对颜色纹理进行采样
                fixed4 mainTex = tex2D(_MainTex, i.uv);

                //我们需要在这里计算反射向量 因为这时才把法线纹理中的法线信息 算出来
                float3 refl = reflect(-viewDir, worldNormal);
                //将反射颜色和主纹理颜色进行叠加
                fixed4 reflColor = texCUBE(_Cube, refl) * mainTex;
                
                //折射相关的颜色
                //其实就是从我们抓取的 屏幕渲染纹理中进行采样 参与计算
                //抓取纹理中的颜色信息 相当于是这个玻璃对象后面的颜色

                //想要有折射效果 可以在采样之前 进行xy屏幕坐标的偏移
                float2 offset = tangentNormal.xy * _Distortion;
                //xy偏移一个位置
                i.grabPos.xy = offset*i.grabPos.z + i.grabPos.xy; 

                //利用透视除法 将屏幕坐标转换到 0~1范围内 然后再进行采样
                fixed2 screenUV = i.grabPos.xy / i.grabPos.w;
                //从捕获的渲染纹理中进行采样 获取后面的颜色
                fixed4 grabColor = tex2D(_GrabTexture, screenUV);

                //折射程度 0~1 0代表完全反射（完全不折射）1代表完全折射（透明效果 相当于光全部进入了内部）
                float4 color = reflColor*(1 - _RefractAmount) + grabColor * _RefractAmount;

                return color;
            }

            ENDCG
        }
    }
}
```

效果图【程序纹理】程序纹理是什么

**知识点一 程序纹理是什么**  
程序纹理（Procedural Textures）顾名思义就是通过程序代码生成的纹理我们之前学习的大部分纹理，一般都是美术同学提前制作好的图片比如颜色纹理、法线纹理、渐变纹理、高光遮罩纹理等等就算是在高级纹理中学习的立方体纹理、渲染纹理，他们也是根据摄像机看到的内容生成的而程序纹理是由我们程序员编写代码，动态生成的纹理图片它的可控性和自由度，都远远大于我们之前学习的其他纹理相关内容

**知识点二 程序纹理如何生成**  
一般生成程序纹理由两种方式：

1.通过C#脚本生成纹理后传递给Shader

2.直接在Shader代码中自定义逻辑生成纹理  
我们将在之后的课程当中分别讲解用这两种方式我们将以动态生成国际象棋棋盘格举例

**知识点三 程序纹理的好处**  
程序纹理由于是由我们程序员写代码动态生成的因此它具备以下优点：

1.由于是动态生成，不需要存储大文件，可以在运行时生成任意分辨率的纹理

2.可以根据需求调整自定义参数，实时的更改纹理外观

3.通过适当的函数设计，可以生成无缝平铺的纹理  
总体而言程序纹理的最明显好处就是自由度高，可控性强【程序纹理】C#代码动态生成程序纹理

**知识点一 生成国际象棋棋盘格纹理对象**

1.设置程序纹理可编辑参数  
纹理宽高、棋盘格行列数、棋盘格两种颜色.

2.实现更新纹理方法  
2-1：创建Texture2D对象，通过参数设置尺寸2-2：设置纹理对象每个像素的颜色规则：国际象棋棋盘的 x和y方向，按格子分，格子的行列编号同奇同偶则为白色，不同则为黑色我们只需要判断(x,y)像素所在格子和上面规则的关系即可2-3：应用像素变化

**知识点二 设置材质球主纹理**

1.获取脚本依附对象的Renderer渲染器脚本

2.通过渲染器脚本设置材质球主纹理

**知识点三 在Inspector窗口添加更新纹理按钮**

1.为生成纹理脚本新建自定义Inspector窗口用脚本

2.添加自定义编辑器特性

3.重写OnInspectorGUI()方法，在其中使用DrawDefaultInspector方法显示默认组件

4.新建按钮，用于调用材质更新方法  
注意点(重要补充知识)编辑器/未运行时：在 Inspector 给某个 Material 赋贴图，会持久化到该 Material 资产（即改了原文件/资源）。运行时：通过 renderer.material 修改的是该对象的材质副本（Unity 会为此实例化一个 clone），不会改动共享资产；通过 renderer.sharedMaterial 会修改共享的资产本身（影响所有引用）；使用 MaterialPropertyBlock 则能够在不改动资产或实例化材质的情况下，单独修改某个 Renderer 的显示（仅在运行时、非持久化，且不能改变 shader 本身或 shader keywords，类似一个临时修改的效果）。  
因此，前面的情况可以为场景预设纹理（编辑器时持久化）；后面的情况可以实现游戏里按需动态变化纹理效果——若需要频繁或大量对象的动态差异且要保留批处理/性能，优先用 MaterialPropertyBlock；若只想给单独对象临时改且不在意开销，可以用 renderer.material。

**总结**  
C#代码动态生成程序纹理相对较简单我们只需要按需求用代码绘制纹理图片在需要的时候更新程序纹理即可更新纹理的时机可以根据需求来定可以是在编辑模式下，可以是在运行时实现纹理修改脚本

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MeshCreator : MonoBehaviour {
    //纹理宽高
    public int TextureWidth = 256;
    public int TextureHeight = 256;

    //行列数
    public int TileCount = 8;

    //棋盘两种颜色
    public Color color1 = Color.white;
    public Color color2 = Color.black;

    public void Start() {
    }

    // 更新纹理
    public void UpdateTexture() {
        Texture2D tex = new Texture2D(TextureWidth, TextureHeight);//new一个2D纹理对象
        for (int y = 0; y < TextureHeight; y++) {
            for (int x = 0; x < TextureWidth; x++) {
                //根据格子行和列的奇数偶数判定颜色
                if (x / (TextureWidth / TileCount) % 2 == y / (TextureHeight / TileCount) % 2) {
                    tex.SetPixel(x, y, color1);
                }
                else {
                    tex.SetPixel(x, y, color2);
                }
            }
        }
        tex.Apply();

        Renderer renderer = this.GetComponent<Renderer>();
        if (renderer != null) {
            renderer.material.mainTexture = tex;
        }
    }
}
```

编辑器面板修改脚本

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(MeshCreator))]
public class MeshCreatorEditor : Editor {
    public override void OnInspectorGUI() {
        DrawDefaultInspector();//绘制默认参数

        //获取目标脚本
        MeshCreator script = (MeshCreator)target;
        if (GUILayout.Button("更新程序纹理")) {
            script.UpdateTexture();
        }
    }
}
```

效果图【程序纹理】Shader代码动态生成程序纹理补充新知识点Shader中的内置函数floor（属于UnityCG.cginc）该函数 传入一个数值 floor(数值)会对传入数值向下取整比如：floor(2.6) 返回 2floor(0.4) 返回 0floor(-2.3) 返回 -3

**知识点 Shader代码动态计算国际象棋棋盘格纹理**

1.新建Shader，删除无用代码

2.声明属性  
平铺数量(行列数) \_TileCount格子颜色1 \_Color1格子颜色2 \_Color2

3.v2f结构体  
顶点和uv

4.顶点着色器  
顶点坐标转换uv直接赋值

5.片元着色器  
uv * 行列数 将0~~1范围 变为 0~~_TileCount范围利用floor得到行列格子编号利用奇偶相加规律得到 0、1 值，0代表同奇或同偶，1代表不同利用该值决定该像素使用哪种颜色实现

```hlsl
Shader "Unlit/Lesson89_ShaderPT"
{
    Properties
    {
        //棋盘格行列数
        _TileCount("TileCount", Float) = 8
        //格子颜色1
        _Color1("Color1", Color) = (1,1,1,1)
        //格子颜色2
        _Color2("Color2", Color) = (0,0,0,1)
    }
    SubShader
    {
        Tags { "RenderType"="Opaque" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            float _TileCount;
            float4 _Color1;
            float4 _Color2;

            v2f vert (appdata_base v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = v.texcoord;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                //把uv坐标从0~1范围 缩放到 0~_TileCount
                float2 uv = i.uv * _TileCount;
                //相当于就是得到当前 uv坐标 所在的格子索引位置
                float2 posIndex = floor(uv);
                // posIndex.x + posIndex.y 
                //  情况1：如果它们同奇或者同偶 加起来就是偶数
                //  情况2：如果不同一个奇数，一个偶数，加起来就是奇数
                //  这的结果只会是 0或者1 如果是0 代表满足情况1;如果是1 代表满足情况2
                float value = (posIndex.x + posIndex.y ) % 2;
                //因为value只会是0或1 ，那么我们完全可以利用lerp进行取值
                //取的就是两端的极限值 只有两种情况
                return lerp(_Color1, _Color2, value);
            }
            ENDCG
        }
    }
}
```

效果图【程序纹理】程序材质

**知识点一 程序材质是什么**  
程序材质是通过算法和数学函数生成的材质（主要在Shader代码中实现）它通常包括多个纹理属性和各种其他属性，用于计算模拟现实世界中的各种表面特性总的来说程序材质是由多个程序纹理和材质属性组合而成的他们共同定义了材质的外观和物理属性，模拟出复杂的表现效果

**知识点二 制作程序材质的工具**  
常见的制作程序材质的美术工具有：

1.Substance Designer（物质设计师）

2.Blender

3.Houdini  
等等其中Substance Designer是一个使用非常广泛的的程序材质创建工具在游戏开发、电影制作、建筑可视化、虚拟现实领域都很常用通过它制作的程序纹理不仅可以在各种游戏引擎（Unity、UE等）中使用，还可以在其他领域使用因此我们在此主要讲解如何使用由Substance Designer制作的程序材质

**知识点三 Unity中使用程序材质**  
Substance Designer制作的程序材质后缀为 .sbsar 文件Unity中并不能直接使用.sbsar后缀的程序材质我们需要在Asset Store中搜索Substance找到一个叫：Substance 3D for Unity 的插件并导入到自己的工程中即可这样我们就可以在工程中直接使用.sbsar后缀的程序材质了

**知识点四 获取程序材质**  
我们可以让美术同学使用Substance Designer制作程序材质也可以在一些免费网站中获取程序材质比如:

1.Unity资源商店

2.Substance Share: <https://substance3d.adobe.com/community-assets>

3.GameTextures：<https://gametextures.com/>  
等等
