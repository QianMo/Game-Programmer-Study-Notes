
![](media/title.jpg)


# 【《Real-Time Rendering 3rd》 提炼总结】(六) 第七章 · 高级着色：BRDF及相关技术

在计算机图形学中，BRDF（Bidirectional Reflectance Distribution
Function，双向反射分布函数）是真实感图形学中最核心的概念之一，它描述的是物体表面将光能从任何一个入射方向反射到任何一个视点方向的反射特性，即入射光线经过某个表面反射后如何在各个出射方向上分布。BRDF模型是绝大多数图形学算法中用于描述光反射现象的基本模型。

这篇文章，将专注于总结和提炼《Real-Time Rendering
3rd》（实时渲染图形学第三版）中第七章“Advanced Shading ·
高级着色”的内容，并对这章中介绍BRDF的内容进行适当补充和引申，构成全文，成为一个对BRDF进行近乎系统式总结的文章。

![](media/10ab01213dba8ed11f44985e84f2683e.jpg)

图1 基于BRDF渲染的场景图 ©Disney 2014.《超能陆战队》

一、导读
========

简而言之，通过阅读这篇总结式文章，你将对BRDF的以下要点有所了解：

-   一、BRDF的前置知识 · 数学篇

    -   球面坐标 Spherical Coordinate

    -   立体角 Solid Angle

    -   投影面积 Foreshortened Area

-   二、BRDF前置知识 · 辐射度量学篇

    -   辐射度量学基本参数表格

    -   辐射通量/光通量 Radiant Flux

    -   辐射强度/发光强度 Radiant Intensity

    -   辐射率/光亮度 Radiance

    -   辐照度/辉度 Irradiance

-   三、BRDF的定义与理解

    -   BRDF的定义式

    -   BRDF的非微分形式

    -   BRDF与着色方程

    -   BRDF的可视化表示

-   四、BRDF的性质

    -   亥姆霍兹光路可逆性

    -   能量守恒性质

    -   线性特征

-   五、BRDF模型的分类

    -   BRDF经验模型

    -   数据驱动的BRDF模型

    -   基于物理的BRDF模型

-   六、基于物理的BRDF · 前置知识

    -   次表面散射 Subsurface Scattering

    -   菲涅尔反射 Fresnel Reflectance

    -   微平面理论 Microfacet Theory

-   七、基于物理的BRDF · 常见模型

    -   Cook-Torrance BRDF模型

    -   Ward BRDF模型

-   八、BRDF与其引申

    -   BSSRDF

    -   SBRDF(SVBRDF)

    -   BTDF与BSDF

二 、BRDF前置知识· 数学篇
=========================

在正式了解BRDF的概念之前，有必要先了解数学和辐射度量学相关的前置基础知识。

2.1 球面坐标 SphericalCoordinate
--------------------------------

由于光线主要是通过方向来表达，通常用球面坐标表达它们比用笛卡尔坐标系更方便。

如图，球面坐标中的向量用三个元素来指定：

![](media/bca8b01eb43dacb1beddeeb548b910bb.jpg)

图2 球面坐标

其中：

-   r表示向量的长度

-   θ表示向量和Z轴的夹角

-   Φ表示向量在x-y平面上的投影和x轴的逆时针夹角。

2.2 立体角 Solid Angle
----------------------

立体角描述了从原点向一个球面区域张成的视野大小，可以看成是弧度的三维扩展。

![](media/5ff50dbf74077e2f704a812d67473940.jpg)

图3 立体角

我们知道弧度是度量二维角度的量，等于角度在单位圆上对应的弧长，单位圆的周长是2π，所以整个圆对应的弧度也是2π
。立体角则是度量三维角度的量，用符号Ω表示，单位为立体弧度（也叫球面度，Steradian，简写为sr），等于立体角在单位球上对应的区域的面积（实际上也就是在任意半径的球上的面积除以半径的平方ω=
s/r2 ），单位球的表面积是4π ，所以整个球面的立体角也是4π 。

![](media/4f72f91f846b5a51b5daf31155dc96be.jpg)

图4 立体角

立体角ω有如下微分形式：
<div  align="center"> 
 <img src="media/5.jpg" width = "122" height = "71" alt="name" align=center />
</div>
其中*dA*为面积微元。而面积微元*dA*在球面坐标系下可以写成：
<div  align="center">   
 <img src="media/6.jpg" width = "350" height = "41" alt="name" align=center />
</div>
因此：
<div  align="center">   
 <img src="media/7.jpg" width = "220" height = "58" alt="name" align=center />
</div>

2.3 投影面积 Foreshortened Area
-------------------------------

投影面积描述了一个物体表面的微小区域在某个视线方向上的可见面积。

对于面积微元A，则沿着与法向夹角为θ方向的A的可见面积为：

<div  align="center">    
 <img src="media/8.jpg" width = "182" height = "47" alt="name" align=center />
</div>

![](media/66e11d2f41165001d2bbb09ae9d4ca15.jpg)

图5 投影面积


三、BRDF前置知识· 辐射度量学篇
==============================

3.1 辐射度量学基本参数表格
--------------------------

如下是截取的wiki（[https://en.wikipedia.org/wiki/Radiometry](http://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Radiometry))）上的辐射度量学国际单位制的辐射量参数表格：

![](media/51b29f898497fa2e2a7d70bd6545e8b8.jpg)

下面对几个核心的基本量进行分别介绍。

3.2 辐射通量/光通量 Radiant Flux
--------------------------------

辐射通量（Radiant
Flux，又译作光通量，辐射功率）描述的是在单位时间穿过截面的光能，或每单位时间的辐射能量，通常用Φ来表示，单位是W，瓦特。
<div  align="center">    
 <img src="media/11.jpg" height = "70" alt="name" align=center />
</div>


其中的Q表示辐射能(Radiant energy)，单位是J，焦耳。

3.3 辐射强度/发光强度 Radiant Intensity
---------------------------------------

对一个点（比如说点光源）来说，辐射强度表示每单位立体角的辐射通量，用符号I表示，单位  ：


<div  align="center">    
 <img src="media/12.jpg" height = "76" alt="name" align=center />
</div>

概括一下：辐射强度(Radiant
intensity，又译作发光强度)，表示每单位立体角的辐射通量，通常用符号I表示，单位  ，瓦特每球面度。

3.4 辐射率/光亮度 Radiance
--------------------------

辐射率（Radiance，又译作光亮度，用符号L表示）,表示物体表面沿某一方向的明亮程度，它等于每单位投影面积和单位立体角上的辐射通量，单位是W·sr−1·m−2，瓦特每球面度每平方米。在光学中，光源的辐射率，是描述非点光源时光源单位面积强度的物理量，定义为在指定方向上的单位立体角和垂直此方向的单位面积上的辐射通量。光亮度L也可以理解为发光程度I在表面dA上的积分。

一种直观的辐射率的理解方法是：将辐射率理解为物体表面的微面元所接收的来自于某方向光源的单位面积的光通量，因此截面选用垂直于该方向的截面，其面积按阴影面积技术计算。

辐射率的微分形式：
<div  align="center">    
<img src="media/13.jpg" height = "90" alt="name" align=center />
</div>


其中：Φ是辐射通量，单位瓦特（W）；Ω是立体角，单位球面度（sr）。

另外需要注意的是，辐射率使用物体表面沿目标方向上的投影面积，而不是面积。

概括一下：辐射率（Radiance，又译作光亮度），表示每单位立体角每单位投影面积的辐射通量，通常用符号L表示,单位是  ，瓦特每球面度每平方米。

3.5 辐照度/辉度 Irradiance
--------------------------

辐照度（Irradiance，又译作辉度，辐射照度，用符号E表示），指入射表面的辐射通量，即单位时间内到达单位面积的辐射通量，或到达单位面积的辐射通量，也就是辐射通量对于面积的密度，

用符号E表示，单位  ，瓦特每平方米。

辐照度可以写成辐射率（Radiance）在入射光所形成的半球上的积分：
<div  align="center">    
<img src="media/14.jpg" height = "70" alt="name" align=center />
</div>

其中，Ω是入射光所形成的半球。L(ω)是沿ω方向的光亮度。

概括一下：辐照度（Irradiance，又译作辉度，辐射照度），表示单位时间内到达单位面积的辐射通量，也就是辐射通量对于面积的密度，通常用符号E表示，单位  ，瓦特每平方米。

四、BRDF的定义与理解
====================

4.1 BRDF的定义式
----------------

可以将给一个表面着色的过程，理解为给定入射的光线数量和方向，计算出指定方向的出射光亮度（radiance）。，在计算机图形学领域，BRDF
（Bidirectional Reflectance Distribution Function，译作双向反射分布函数
)是一个用来描述表面如何反射光线的方程。顾名思义，BRDF就是一个描述光如何从给定的两个方向（入射光方向l和出射方向v）在表面进行反射的函数。

BRDF的精确定义是出射辐射率的微分（differential outgoing
radiance）和入射辐照度的微分（differential incoming irradiance）之比：
<div  align="center">    
<img src="media/15.jpg" height = "70" alt="name" align=center />
</div>

要理解这个方程的含义，可以想象一个表面被一个来自围绕着角度**l**的微立体角的入射光照亮，而这个光照效果由表面的辉度dE来决定。

表面会反射此入射光到很多不同的方向，在给定的任意出射方向v，光亮度dLo与辐照度dE成一个比例。而两者之间的这个取决于l和v的比例，就是BRDF。

![](media/86a26c1f917c5730b32f223fa7bf33b9.jpg)

图6 BRDF图示

一个最常见的疑问是，BRDF为什么要取这样的定义。BRDF为什么被定义为辐射率（radiance）和辐照度（irradiance）之比，而不是radiance和radiance之比，或者irradiance和irradiance之比呢？

首先，我们分别重温它们的定义：

-   辐照度（Irradiance，又译作辉度，辐射照度），表示单位时间内到达单位面积的辐射通量，也就是辐射通量对于面积的密度，通常用符号E表示，单位  ，瓦特每平方米。

-   辐射率（Radiance，又译作光亮度），表示每单位立体角每单位投影面积的辐射通量，通常用符号L表示,单位是  ，瓦特每球面度每平方米。

那么，关于这个问题，我们可以这样理解：因为照射到入射点的不同方向的光，都可能从指定的反射方向出射，所以当考虑入射时，需要对面积进行积分。而辐照度irradiance正好表示单位时间内到达单位面积的辐射通量。所以BRDF函数，选取入射时的辐照度Irradiance，和出射时的辐射率Radiance，可以简单明了地描述出入射光线经过某个表面反射后如何在各个出射方向上分布。而直观来说，BRDF的值给定了入射方向和出射方向能量的相对量。

概括一下：BRDF（Bidirectional Reflectance Distribution
Function，译作双向反射分布函数 )，定义为出射辐射率的微分（differential outgoing
radiance）和入射辐照度的微分（differential incoming
irradiance）之比，描述了入射光线经过某个表面反射后如何在各个出射方向上分布，给定了入射方向和出射方向能量的相对量，单位是  ，每球面度。

4.2 BRDF的非微分形式
--------------------

这里的讨论仅限于非区域光源，如点光源或方向光源。在这种情况下，BRDF定义可以用非微分形式表示：

<div  align="center">    
<img src="media/17.jpg" height = "70" alt="name" align=center />
</div>

其中：

-   EL是光源在垂直于光的方向向量L平面测量的辐照度（irradiance）。

-   Lo（v）是在视图矢量v的方向上产生的出射辐射率（radiance）。

4.3 BRDF与着色方程
------------------

根据上文所了解了BRDF的定义，现在，就很容易得到BRDF是如何用n个非区域光来拟合一般的着色方程的：

<div  align="center">    
<img src="media/18.jpg" height = "70" alt="name" align=center />
</div>

其中k是每个光源的索引。使用⊗符号（分段向量乘法），是因为BRDF和辉度（irradiance）都是RGB向量。考虑到入射和出射方向都拥有两个自由度（通常参数化是使用两个角度：相对于表面法线的仰角θ和关于法线的旋转角度*φ*），一般情况下，BRDF是拥有四个标量变量的函数。

另外，各向同性BRDFs（Isotropic
BRDFs）是一个重要的特殊情况。这样的BRDF在输入和输入方向围绕表面法线变化（保持相同的相对夹角）时保持不变。所以，各向同性BRDF是关于三个标量的函数。

4.4 对BRDF的可视化表示
----------------------

一种理解BRDF的方法就是在输入方向保持恒定的情况下对它进行可视化表示，如下图。对于给定方向的入射光来说，图中显示了出射光的能力分布：在交点附近球形部分是漫反射分量，因此出射光来任何方向上的反射概率相等。椭圆部分是一个反射波瓣（Reflectance
Lobe）。它形成了镜面分量。显然，这些波瓣位于入射光的反射方向上，波瓣厚度对应反射的模糊性。根据互易原理，可以将这些相同的可视化形成认为是每个不同入射光方向对单个出射方向的贡献量大小。

![](media/897dc5430a09096c357353c16f6672e5.jpg)

图7 BRDF的可视化表示

五、BRDF的性质
==============

5.1 可逆性
----------

BRDF的可逆性源自于亥姆霍兹光路可逆性（Helmholtz Recoprpcity Rule）。

BRDF的可逆性即，交换入射光与反射光，并不会改变BRDF的值：
<br>
<br>
<div  align="center">    
<img src="media/19.1.gif" height = "25" alt="name" align=center />
</div>

5.2 能量守恒性质
----------------

BRDF需要遵循能量守恒定律。能量守恒定律指出：入射光的能量与出射光能量总能量应该相等。能量守恒方程如下：
<div  align="center">    
<img src="media/20.jpg" height = "45" alt="name" align=center />
</div>
由此可知：

<div  align="center">    
<img src="media/21.jpg" height = "45" alt="name" align=center />
</div>
因此BRDF必须满足如下的积分不等式，也就是能量守恒性质：
<div  align="center">    
<img src="media/22.1.png" height = "55" alt="name" align=center />
</div>


5.3 线性特征
------------

很多时候，材质往往需要多重BRDF计算以实现其反射特性。表面上某一点的全部反射辐射度可以简单地表示为各BRDF反射辐射度之和。例如，镜面漫反射即可通过多重BRDF计算加以实现。

六、BRDF的模型分类
==================

根据BRDF的定义来直接应用，会有一些无从下手的感觉。而为了方便和高效地使用BRDF数据，大家往往将BRDF组织成为各种参数化的数值模型。

有各式各样的BRDF模型，如：

-   Phong (1975)

-   Blinn-Phong (1977)

-   Cook-Torrance (1981)

-   Ward (1992)

-   Oren-Nayar (1994)

-   Schlick (1994)

-   Modified-Phong (Lafortune 1994)

-   Lafortune (1997)

-   Neumann-Neumann (1999)

-   Albedo pump-up (Neumann-Neumann 1999)

-   Ashikhmin-Shirley (2000)

-   Kelemen (2001)

-   Halfway Vector Disk (Edwards 2006)

-   GGX (Walter 2007)

-   Distribution-based BRDF (Ashikmin 2007)

-   Kurt (2010)

-   etc.

这些BRDF的模型可以分为如下几类：

-   经验模型（Empirical Models）：使用基于实验提出的公式对BRDF做快速估计。

-   数据驱动的模型（Data-driven
    Models）：采集真实材质表面在不同光照角度和观察角将BRDF按照实测数据建立查找表，记录在数据库中，以便于快速的查找和计算。

-   基于物理的模型（Physical-based
    Models）：根据物体表面材料的几何以及光学属性建立反射方程，从而计算BRDF，实现极具真实感的渲染效果。

6.1 BRDF经验模型
----------------

关于BRDF的经验模型，有如下几个要点：

-   经验模型提供简洁的公式以便于反射光线的快速计算。

-   经验模型不考虑材质特性，仅仅提供一个反射光的粗糙近似。

-   经验模型不一定满足物理定律，比如Helmholtz可逆性或能量守恒定律。

-   经验模型因为其简洁和高效性被广泛运用。

常见的BRDF经验模型有：

-   Lambert漫反射模型

-   Phong模型

-   Blinn-Phong模型

-   快速Phong模型

-   可逆Phong模型

Lambert模型，Phong模型、Blinn-Phong模型和其改进模型都是常见的光照模型，由于篇幅原因，就不展开论述了。


6.2 数据驱动的BRDF模型
----------------------

数据驱动的BRDF模型可以理解为，度量一个大的BRDF材质集合，并将其记录为高维向量，利用降维的方法从这些数据中计算出一个低维模型，这样基于查表的方式，可以直接找到渲染结果，省去大量的实时计算。代表工作如：A
Data-Driven Reflectance Model,ACM
SIGGRAPH,2003 [http://people.csail.mit.edu/wojciech/DDRM/index.html](http://link.zhihu.com/?target=http%3A//people.csail.mit.edu/wojciech/DDRM/index.html)

另外，MERL等实验室使用各类仪器测量了上多种真实材质表面在不同光照角度和观察角度下的反射数据，并记录在数据库中，如MERL
BRDF Database。

![](media/b10e282149a8d128616c43622d591511.jpg)

图8 一个名为“MERL
100”的BRDF数据库。其中，含50种“光滑材质”（例如金属和塑料），和50种“粗糙材质”（例如纺织物）

需要注意的是，由于这些数据由于采集自真实材质，即便渲染出来的结果很真实，但缺点是没有可供调整效果的参数，无法基于这些数据修改成想要的效果，另外部分极端角度由于仪器限制，无法获取到数据，而且采样点密集，数据量非常庞大，所以并不适合游戏等实时领域，一般可用在电影等离线渲染领域，也可以用来做图形学研究，衡量其他模型的真实程度。

这边提供一些数据库的链接供参考：

-   MERLBRDF
    Database： [http://www.merl.com/brdf/](http://link.zhihu.com/?target=http%3A//www.merl.com/brdf/)

-   MITCSAIL: [http://people.csail.mit.edu/addy/research/brdf/](http://link.zhihu.com/?target=http%3A//people.csail.mit.edu/addy/research/brdf/)

-   CAVE
    Database:[http://www1.cs.columbia.edu/CAVE/databases/tvbrdf/about.php](http://link.zhihu.com/?target=http%3A//www1.cs.columbia.edu/CAVE/databases/tvbrdf/about.php)

6.3 基于物理的BRDF模型
----------------------

基于物理的渲染(PBR, Physically-based
rendering)是计算机图形学中用数学建模的方式模拟物体表面各种材质散射光线的属性从而渲染照片真实图片的技术，是近年来是实时渲染领域的大趋势。

基于物理的BRDF模型通过包含材质的各种几何及光学性质来尽可能精确的近似现实世界中的材料。而一个基于物理的BRDF要必须满足至少如下两条BRDF的特性：能量守恒、亥姆霍兹光路可逆性。

常见的基于物理的BRDF模型有：

-   Cook-Torrance BRDF模型

-   Ward BRDF模型

下文将先介绍基于物理的BRDF常常用到到的菲涅尔反射，次表面散射和微平面理论等理论，分别概括这两种基于物理的BRDF模型。

七、基于物理的BRDF · 前置知识
=============================

7.1 次表面散射 Subsurface Scattering
------------------------------------

在真实世界中许多物体都是半透明的，比如皮肤、玉、蜡、大理石、牛奶等。当光入射到透明或半透明材料表面时，一部分被反射，一部分被吸收，还有一部分经历透射。这些半透明的材质受到数个光源的透射，物体本身就会受到材质的厚度影响而显示出不同的透光性，光线在这些透射部分也可以互相混合、干涉。

次表面散射，Subsurface
Scattering，简称SSS(又简称3S)，就是光射入半透明材质后在内部发生散射，最后射出物体并进入视野中产生的现象，是指光从表面进入物体经过内部散射，然后又通过物体表面的其他顶点出射的光线传递过程。

![](media/1cdecce356f6f3f5f39169ea569cad48.jpg)

图9 次表面散射示例1

![](media/12db40a81c96bfa9d3e553dfba7682a5.jpg)

图10 次表面散射示例2

又如文章开头，贴出的《超能陆战队》中大白的渲染照中，大白的白色身体，略微有些透明的感觉，就是典型的次表面散射。

简而言之：次表面散射，即光射入表面，在材质里散射，然后从与射入点不同的地方射出表面的一种现象。

![](media/6280ca245be9ddec829da28a51589276.jpg)

图11
光与表面的相互作用。左图，可以看到次表面相互作用导致光从入口点重新射出。红色和绿色圆圈代表两个不同尺度下的像素所覆盖的区域。在右边，所有的次表面散射光都是从入口点发出的，忽略了表面之下的细节。

7.2 菲涅尔反射 Fresnel Reflectance
----------------------------------

菲涅耳方程（Fresnel
equations）是一组用于描述光在两种不同折射率的介质中传播时的反射和折射的光学方程。方程中所描述的反射被称作“菲涅耳反射”。

菲涅尔反射（Fresnel Reflectance）或者菲涅尔效果（Fresnel
Effect），即当光入射到折射率不同的两个材质的分界面时，一部分光会被反射，而我们所看到的光线会根据我们的观察角度以不同强度反射的现象。

菲涅尔反射能够真实地模拟真实世界中的反射。在真实世界中，除了金属之外，其它物质均有不同程度的菲涅尔反射效果。

关于菲涅尔反射，一个很好的例子是一池清水。从水池上笔直看下去（也就是与法线成零度角的方向）的话，我们能够一直看到池底。而如果从接近平行于水面的方向看去的话，水池表面的高光反射会变得非常强以至于你看不到池底。

![](media/c1ee1c4ea5e293d9d0ec43ffe6c55c57.jpg)

图12 菲涅尔反射效果

简单来说，视线垂直于表面时，反射较弱，而当视线并非垂直表面时，夹角越小，反射越明显。

对于粗糙表面来说，在接近平行方向的高光反射也会增强但不够达到100%的强度.为何如此是因为影响菲涅尔效应的关键参数在于每个微平面的法向量和入射光线的角度，而不是宏观平面的法向量和入射光线的角度。因此我们在宏观层面看到的实际上是微平面的菲涅尔效应的一个平均结果。

根据菲涅尔反射，若你看向一个圆球，那么圆球中心的反射会较弱，而靠近边缘是反射会较强。另外需注意，这种关系也受折射率影响。

![](media/f39d56cb64eab66f423c5098b2509a1d.jpg)

图13 基于菲涅尔反射，在Unreal Engine 4中实现的边缘光Shader

7.3 微平面理论 Microfacet Theory
--------------------------------

微表面理论假设表面是由不同方向的微小细节表面，也就是微平面（microfacets）组成。每一个微小的平面都会根据它的法线方向在一个方向上反射光线。

表面法线朝向光源方向和视线方向中间的微表面会反射可见光。然而，不是所有的表面法线和半角法线（half
normal）相等的微表面都会反射光线，因为其中有些会被遮挡，如下图所示。

![](media/c83ab9adf0f18f082e0060fb6ac16afd.jpg)

图14 微平面理论图示

我们用法线分布函数（Normal Distribution
Function，简写为NDF），D(h)来描述组成表面一点的所有微表面的法线分布概率。则可以这样理解：向NDF输入一个朝向h，NDF会返回朝向是h的微表面数占微表面总数的比例，比如有8%的微表面朝向是h，那么就有8%的微表面可能将光线反射到v方向。

![](media/fae6d41ba5ea0051c455432f2bc05627.jpg)

图15
由微平面组成的表面。仅红色微平面的表面法线和半矢量h对齐，能参与从入射光线向量l到视线向量v的光线反射

NDF的定义式：
<div  align="center">    
<img src="media/32.jpg" height = "65" alt="name" align=center />
</div>


在微观层面上不规则的表面会造成光的漫反射。例如，模糊的反射是由于光线的散射造成的。而反射的光线并不均匀，因此我们得到的高光反射是模糊的。如下图。

![](media/2e4a602140263296a55d7632b69fb4ae.jpg)

图16 模糊的微平面高光反射

八、基于物理的BRDF · 常见模型
=============================

8.1 Cook-Torrance BRDF模型
--------------------------

Cook-Torrance模型作为图形学中最早的基于物理的BRDF模型，由Cook和Torrance提出，是Torrance-Sparrow模型的一个应用版本。现今，Cook-Torrance模型已经成为基于物理着色的标准模型之一。Cook-Torrance模型将物理学中的菲涅尔反射引入了图形学，实现了比较逼真的效果。

Cook-Torrance微平面着色模型（Cook-Torrance microfacet specular shading
model），即Microfacet Specular BRDF，定义为：

<div  align="center">    
<img src="media/34.jpg" height = "75" alt="name" align=center />
</div>

其中：

-   F为菲涅尔反射函数( Fresnel 函数 )

-   G为阴影遮罩函数（Geometry Factor，几何因子），即未被shadow或mask的比例

-   D为法线分布函数(NDF)

篇幅原因，就不展开论述了，若大家有兴趣，可以参考历年的SIGGRAPH Course:
Physically Based Shading in Theory and Practice系列与其他相关文章。

如SIGGRAPH 2013 Course: Physically Based Shading in Theory and Practice:

[http://blog.selfshadow.com/publications/s2013-shading-course/](http://link.zhihu.com/?target=http%3A//blog.selfshadow.com/publications/s2013-shading-course/)

具体的课程中的两篇推荐：

1.  Background:Physics and Math of
    Shading: [http://blog.selfshadow.com/publications/s2013-shading-course/hoffman/s2013_pbs_physics_math_slides.pdf](http://link.zhihu.com/?target=http%3A//blog.selfshadow.com/publications/s2013-shading-course/hoffman/s2013_pbs_physics_math_slides.pdf)

2.  Physically Based Lighting in Call of Duty: Black Ops
    ：[http://blog.selfshadow.com/publications/s2013-shading-course/lazarov/s2013_pbs_black_ops_2_slides_v2.pdf](http://link.zhihu.com/?target=http%3A//blog.selfshadow.com/publications/s2013-shading-course/lazarov/s2013_pbs_black_ops_2_slides_v2.pdf)

8.2 Ward BRDF模型
-----------------

一般情况下，我们可以将BRDF分为两类：

各项同性（Isotropic）的BRDF

-   反射不受与给定表面法向夹角的约束

-   随机表面微结构

各向异性（Anisotropic）的BRDF

-   反射比随着与某个给定的表面法向之间的夹角而变化

-   图案的表面微结构

-   金属丝，绸缎，毛发等

Phong和Cook-Torrance BRDF模型都不能处理各项异性的效果，Ward模型却可以。

Ward模型由Ward于1992年提出[Measuringand Modeling Anissropic Reflection,ACM
SIGGRAPH,1992]。Ward模型介绍了更一般的表面法向表达方式：通过椭圆体（ellipsoids）这种允许各向异性反射的形式来表达。

然而，由于没有考虑菲涅耳因子（Fresnel factor）和几何衰减因子（geometric
attenuation factor），该模型更像是一种经验模型，但还是属于基于物理的BRDF模型。

各向同性的Ward模型定义为：

<div  align="center">    
<img src="media/35.jpg" height = "75" alt="name" align=center />
</div>



九、BRDF与其引申
================

有不少与BRDF类似的函数：

-   BSSRDF：Bidirectional Surface Scattering Reflectance
    Distribution，双向表面散反射分布函数

-   SBRDF(SVBRDF): spatially varying BRDF(spatial BRDF) 空间BRDF

-   BTDF：Bidirectional Transmittance Distribution Function 双向透射分布函数

-   BSDF : Bidirectional Scattering Distribution Function，双向散射分布函数

下面分别进行介绍。

9.1 BSSRDF
----------

BRDF只是更一般方程的一种近似,这个方程就是BSSRDF（Bidirectional
scattering-surface reflectance distribution
function，双向表面散反射分布函数）。BSSRDF描述了射出辐射率与入射通量之间的关系，BSSRDF函数通过把入射和出射位置作为函数的输入，从而来包含这些现象，它描述了沿入射方向从物体表面的一点到另外一点，最后顺着出射方向出去的光线的相对量。注意，这个函数还考虑了物体表面的一点到另外一点，最好顺着出射方向出去的光线相对量。注意，这个函数还考虑了物体表面不一致的情况，因为随着位置的变化，反射系数也会发生变化。在实时绘制中，物体表面上的位置可以用来获取颜色纹理、光泽度，以及凹凸纹理图等信息。

![](media/398183f217f8be62b6cd386243c485ce.jpg)

图17 BRDF渲染图

![](media/1fb4e0458939444804da12336e640802.jpg)

图18 BSSRDF渲染图

9.2 SBRDF(SVBRDF)
-----------------

一个捕获基于空间位置BRDF变化的函数被称为空间变化的BRDF（Spatially Varying BRDF
,SVBRDF）或称空间BRDF，空间双向反射分布函数（Spatial BRDF ，SBRDF）。

9.3 BTDF与BSDF
--------------

即使一般的BSSRDF函数，无论其多么复杂，仍然忽略了现实世界中非常重要的一些变量，比如说光的偏振。此外，也没有处理穿过物体表面的光线传播，只是对反射情况进行了处理。为了处理光线传播的问题，对物体表面定义了两个BRDF和两个BTDF（T表示传播“Transmittance”），每侧各有一个，这样就组成了BSDF（S表示散射“Scattering”）。

![](media/d084fcee93a2803cf9565f8519099598.jpg)

图19
当用于镜面反射的BRDF和用于镜面透射的BTDF使用菲涅尔公式进行调制渲染时，得到了如真实玻璃视觉上精确的反射和透射的角度变化。来自《Physically
Based Rendering, Third Edition》Figure 8.10

而在实践中，这些更复杂的函数很少使用，BRDF和SVBRDF足以胜任一般情况下表面渲染的效果。

十、其他参考
============

[1] [http://graphics.stanford.edu/\~henrik/images/subsurf.html](http://link.zhihu.com/?target=http%3A//graphics.stanford.edu/%7Ehenrik/images/subsurf.html)

[2] [http://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-shading/reflection-refraction-fresnel](http://link.zhihu.com/?target=http%3A//www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-shading/reflection-refraction-fresnel)

[3] [http://wiki.nuaj.net/index.php?title=BRDF](http://link.zhihu.com/?target=http%3A//wiki.nuaj.net/index.php%3Ftitle%3DBRDF)

[4] [https://www.allegorithmic.com/pbr-guide](http://link.zhihu.com/?target=https%3A//www.allegorithmic.com/pbr-guide)

[5] [http://www.icourses.cn/coursestatic/course_2987.html](http://link.zhihu.com/?target=http%3A//www.icourses.cn/coursestatic/course_2987.html)

[6] [https://docs.unrealengine.com/latest/INT/Engine/Rendering/Materials/HowTo/Fresnel/index.html](http://link.zhihu.com/?target=https%3A//docs.unrealengine.com/latest/INT/Engine/Rendering/Materials/HowTo/Fresnel/index.html)

[7] [https://en.wikipedia.org/wiki/Fresnel_equations](http://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Fresnel_equations)

[8] <https://zhuanlan.zhihu.com/p/21376124>

[9] [https://en.wikipedia.org/wiki/Radiometry](http://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Radiometry)

[10] Real Shading in Unreal Engine
4：[https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf](http://link.zhihu.com/?target=https%3A//cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf)

[11] SIGGRAPH 2013 Course: Physically BasedShading in Theory and
Practice：[http://blog.selfshadow.com/publications/s2013-shading-course/](http://link.zhihu.com/?target=http%3A//blog.selfshadow.com/publications/s2013-shading-course/)

[12] Physically Based Rendering, Third Edition
