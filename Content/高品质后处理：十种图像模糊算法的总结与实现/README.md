![](media/b05cff0fb8d1036d1972f82d35c4da6f.jpg)

# 高品质后处理：十种图像模糊算法的总结与实现

本文的知乎专栏版本：https://zhuanlan.zhihu.com/p/125744132

<br>

后处理（Post-Processing），在图形学和游戏开发等领域是提升最终画面呈现品质的重要渲染技术。后处理渲染技术的好坏，往往决定了游戏画面是否能够达到令人惊艳的级别。

图像模糊算法在后处理渲染领域中占据着重要的地位。很多产品级后处理的实现，都会直接或间接依赖于图像模糊算法中的一种或多种。无论是基于高斯模糊（Gaussian Blur）或其改进算法的Bloom特效，还是基于径向模糊（Radial Blur）的Sun Shaft（God Ray），或是基于方向模糊（Directional Blur）的镜头眩光光晕（Glare Lens Flare），抑或是景深（Depth of Field）特效中摄影级失焦感的散景模糊（Bokeh Blur），都以模糊算法作为重要的支撑。所以说，后处理中所采用模糊算法的优劣，决定了后处理管线最终的渲染品质和消耗性能的多少。


本文将对后处理管线中会使用到的如下十种模糊算法进行总结、对比和盘点，以及提供了这十种模糊算法以及其衍生算法对应的Unity Post Processing Stack v2版本的实现：

-   高斯模糊（Gaussian Blur）

-   方框模糊（Box Blur）

-   Kawase模糊（Kawase Blur）

-   双重模糊（Dual Blur）

-   散景模糊（Bokeh Blur）

-   移轴模糊（Tilt Shift Blur）

-   光圈模糊（Iris Blur）

-   粒状模糊（Grainy Blur）

-   径向模糊（Radial Blur）

-   方向模糊（Directional Blur）


另外，本文暂不涉及运动模糊（Motion Blur），因为其主要作用于帧之间的运动变化，不属于静态型模糊。还有一些其他的模糊算法由于不太适用于实时渲染，本文也暂不涉及，如Moving Averages filter。


下面先放一组使用了依赖于模糊算法的后处理特效的实时渲染截图，然后开始我们的正文。

![](media/6b40938cfddc691005bb05822a5c398e.jpg)

图《巫师2》 中基于径向模糊（Radial Blur）的Sun Shaft

![](media/130ec1a6dbdee035c956306b98b39bcb.jpg)

图 UE4场景中，间接基于高斯模糊（Gaussian Blur）的Bloom特效给画面带了更好的光感 @ UE4 CyberNeon @Junliang Zhang

![](media/dc5b0bb30b61379a55a98b9d2cca8acf.jpg)

图 Sun Shaft Forest @UE4

![](media/513005a757dc352083a8cec5889259f2.png)

图 《鬼泣5》中的镜头眩光光晕（Glare Lens Flare）

![](media/bcac2c9252b814447e87f5bc88268be8.jpg)

图 Tom Clancys The Division中的基于散景模糊（Bokeh Blur）的景深

这里也放一个《赛博朋克:霓虹中国(CyberNeon)》的视频（ArtStation原贴：https://www.artstation.com/artwork/Z5RkbZ）
，其中对于赛博朋克风夜中国风城市的展现，如果缺少了Bloom和Glare Lens Flare等依赖于本文讲到的模糊算法的后处理特效，展现出来的品质将少了很多韵味：

<https://www.youtube.com/watch?v=CZ4MOBSx2xw>

![](media/8bc10ccaba3d1cb963c7c53e4c07ae96.jpg)

![](media/f20159279e99b4174740875b9e33a28b.jpg)




<br>

# 十种图像模糊算法横向对比


在展开全文，对这十种图像模糊算法进行分别介绍之前，这一节中先做一个总览，即一个横向的对比。要评判一种模糊算法的好坏，主要有三个标准：

-   **模糊品质（Quality）** 。模糊品质的好坏是模糊算法是否优秀的主要指标。

-   **模糊稳定性（Stability）** 。模糊的稳定性决定了在画面变化过程中，模糊是否稳定，不会出现跳变或者闪烁。

-   **性能（Performance）** 。性能的好坏是模糊算法是否能被广泛使用的关键所在。

以下是本文涉及的十种模糊算法在标准情况下以上述三个指标作为评判标准的横向对比：

![](media/b82ba2aaa60f290da438450a86ccb48f.png)

从上表的对比可以看到，除了Grainy Blur因其模糊质感的特殊性获得了“一般”的模糊品质评级之外，另外九种模糊算法在模糊品质和稳定性这两方面都获得了不错的评级。这是因为给到足够的迭代次数，且不做RT的DownSample，他们都可以收敛到一个高品质的模糊质感。

最终的分化在于性能，这才是评判一种算法性价比是否高，能否广泛用于实时渲染的关键因素。其中，可以看到仅双重模糊（Dual Blur）和粒状模糊（Grainy Blur）两种算法，获得了高的性能评级。当然，这是针对标准的算法而言，其他八种算法如果进行进一步的性能优化，也能具有更佳的性能。

<br>

# 关于X-PostProcessing Libray

X-PostProcessing Libray，简称XPL，是本人开发的Unity引擎下的高品质开源后处理算法库，旨在提供业界主流的高品质后处理特效的完整解决方案，目前已完美支持Unity
Post-processing Stack v2。后续也将提供对Unity引擎URP/LWRP/HDRP的兼容支持。

【GitHub地址】https://github.com/QianMo/X-PostProcessing-Library

![](media/e230629ffe1b2da2325caec0a6401b96.jpg)

截止本文发表，目前已以开源形式放出了17种Blur算法的后处理实现。而随着后续更多内容的公开，X-PostProcessing Libray将成型为一个具有100+种后处理特效的高品质后处理开源算法库。

<br>

OK，下面我们开始正文。先从最热门，最为大众所熟知的高斯模糊开始。

<br>

# 一、高斯模糊（Gaussian Blur）


高斯模糊（Gaussian Blur），也叫高斯平滑（Gaussian smoothing），作为最经典的模糊算法，一度成为模糊算法的代名词。

![](media/1623d27dec6f63ee9790dda63a60b125.jpg)

高斯模糊在图像处理领域，通常用于减少图像噪声以及降低细节层次，以及对图像进行模糊，其视觉效果就像是经过一个半透明屏幕在观察图像。

从数字信号处理的角度看，图像模糊的本质一个过滤高频信号，保留低频信号的过程。过滤高频的信号的一个常见可选方法是卷积滤波。从这个角度来说，图像的高斯模糊过程即图像与正态分布做卷积。由于正态分布又叫作“高斯分布”，所以这项技术就叫作高斯模糊。而由于高斯函数的傅立叶变换是另外一个高斯函数，所以高斯模糊对于图像来说就是一个低通滤波器。

用于高斯模糊的高斯核（Gaussian Kernel）是一个正方形的像素阵列，其中像素值对应于2D高斯曲线的值。

![](media/3ce94d027109baba427440e515a99999.png)

图 一个典型的高斯核

图像中的每个像素被乘以高斯核，然后将所有这些值相加，得到输出图像中此处的值。

![](media/4556f53e98a5ef378e1fa1926bd496cb.jpg)

N维空间高斯模糊方程可以表示为：

![](media/657a242a1f1beb689df33a82d88b6d5c.png)

在二维空间定义为：

![](media/9b214c420a347a25ac11f6e4edb236ef.png)

其中r是模糊半径

![](media/4da60429ba567bef2df17b9d38b65585.png)

下图为高斯函数的3维图示：

![](media/3aedd63b1819614bb73a76196ceec706.jpg)


高斯模糊也可以在二维图像上对两个独立的一维空间分别进行计算，即满足线性可分（Linearly separable）。这也就是说，使用二维矩阵变换得到的效果也可以通过在水平方向进行一维高斯矩阵变换加上竖直方向的一维高斯矩阵变换得到。从计算的角度来看，这是一项有用的特性，因为这样只需要

![](media/a5f93c9c118c0b991fc6a988e5a23d9e.png)的计算复杂度，而原先的计算复杂度为![](media/2f47eace291ac9e6217c0e124c395b92.png)，其中M,N是需要进行滤波的图像的维数，m、n是滤波器的维数。

以下为一个Gaussian Kernel的线性分解过程：

![](media/4101dff5d82408c4c404b5d90f6ab22c.png)


下图很好的对Gaussian Kernel的线性可分进行了描述：

![](media/1ad248b2a04bfe2d3791a3c89cd88656.png)

实现方面，可以采用经过线性分解的高斯核的方案，且用乒乓RT交互blit的方法。高斯模糊对应的Fragment Shader的实现为：

	float4 FragGaussianBlur(v2f i): SV_Target
	{
		half4 color = float4(0, 0, 0, 0);
		
		color += 0.40 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);
		color += 0.15 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.xy);
		color += 0.15 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.zw);
		color += 0.10 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.xy);
		color += 0.10 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.zw);
		color += 0.05 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.xy);
		color += 0.05 * SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.zw);
		
		return color;
	}


篇幅原因，在这里以及后文中，对应后处理的Runtime + Shader的完整的实现就不贴了，但会给出X-PostProcessing Libray中的完整实现链接。

**完整的高斯模糊Runtime + Shader实现可见**：
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/GaussianBlur



<br>
以下是开启高斯模糊后处理前后的对比图：

![](media/3386e01c8b105ef7e8bf8427f426155c.png)

![](media/8a70626a23b1640b2a0a1ba74304541e.png)

以及展示了BlurRadius为3，Iteration为6，RTDownScale为1的设置下，经过横纵线性分解的高斯模糊的渲染过程的动图：

![](media/ef3648c8276d045ffcd8b58b7b23e624.gif)

对模糊半径（Blur Radius）参数的调节，可以控制高斯模糊的程度：

![](media/f5668bf755f6b60c22c596220253d3e8.gif)


<br>

# 二、方框模糊（Box Blur）

方框模糊（Box Blur），又常被称为盒式模糊，其中所得到的图像中的每个像素具有的值等于其邻近的像素的输入图像中的平均值。和高斯模糊一样，Box Blur也是低通滤波器的一种形式。在图像处理领域，Box Blur通常用于近似高斯模糊。因为根据中心极限定理，重复应用Box Blur可以得到和高斯模糊非常近似的模糊表现。

可以将3 x 3的box blur的kernel表示为如下矩阵

![](media/072d5a18279fac83dd465dfe20ccbc27.png)

而2x2的box blur的kernel表示为如下矩阵：

![](media/b86eda5e0fb7b8c0643adf78adf98925.png)

Box Blur和高斯模糊的性质对比可见下图：

![](media/328bcaa82bbb793c5ab72adad9f92bf1.png)

图 3D结构，2D结构和示例矩阵对比（a）Box Blur（b）Gaussian Blur

以下是Box Blur的作用过程：

![](media/6c89d617142422ac6c7b6b9a4129d983.png)

Box Blur也是线性可分的，如有需要，也可以借助其此性质，如下所示：

![](media/03872365a0d205fe7d1d9c05cf982709.png)

另外box blur也有不少扩展与变体，比如Tent Blur（两次Box Blur）、Quadratic Blur（三次Box Blur）等，具体本文暂时就不展开了。

其中，Tent Blur也已在XPL中进行了实现，具体可见：[https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/TentBlur](https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/TentBlur)



以下是一个4 x 4的box filter的shader实现，low level optimize方面，可以采用乘加组合的书写方式，即MAD指令友好的形式，以在部分GPU上实现指令数的优化：

	half4 BoxFilter_4Tap(TEXTURE2D_ARGS(tex, samplerTex), float2 uv, float2 texelSize)
	{
		float4 d = texelSize.xyxy * float4(-1.0, -1.0, 1.0, 1.0);
		
		half4 s = 0;
		s = SAMPLE_TEXTURE2D(tex, samplerTex, uv + d.xy) * 0.25h;  // 1 MUL
		s += SAMPLE_TEXTURE2D(tex, samplerTex, uv + d.zy) * 0.25h; // 1 MAD
		s += SAMPLE_TEXTURE2D(tex, samplerTex, uv + d.xw) * 0.25h; // 1 MAD
		s += SAMPLE_TEXTURE2D(tex, samplerTex, uv + d.zw) * 0.25h; // 1 MAD
		
		return s;
	}

**完整的Runtime + Shader实现可见：** [https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/BoxBlur](https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/BoxBlur)

另外也可以考虑将uv计算放到vertex层。

Box Blur的渲染效果接近高斯模糊，但性价比并不高，需要较多的迭代次数才能达到高品质的模糊效果：

![](media/9fe9182c3ccb44b95c4feaf5060307cf.png)


以下是Box Blur在BlurRadius为1.6，Iteration为6，RTDownScale为1的设置下模糊过程的动图：

![](media/ce27882013b6c4c1c5e2fc6c4e2feaf8.gif)


<br>

# 三、Kawase模糊（Kawase Blur）


Kawase Blur于Masaki Kawase 在GDC2003的分享《Frame Buffer Postprocessing Effects in DOUBLE-S.T.E.A.L (Wreckless)》中提出。Kawase Blur最初用于Bloom后处理特效，但其可以推广作为专门的模糊算法使用，且在模糊外观表现上与高斯模糊非常接近。
Kawase Blur的思路是对距离当前像素越来越远的地方对四个角进行采样，且在两个大小相等的纹理之间进行乒乓式的blit。创新点在于，采用了随迭代次数移动的blur kernel，而不是类似高斯模糊，或box blur一样从头到尾固定的blur kernel。

![](media/8833561d489ea70dc3e4a45d54e86c24.png)

![](media/7e04eb8d9007d9e91f5f6fe4e7770759.png)

实践数据表明，在相似的模糊表现下，Kawase Blur比经过优化的高斯模糊的性能约快1.5倍到3倍。

具体思路是在runtime层，基于当前迭代次数，对每次模糊的半径进行设置，而Shader层实现一个4 tap的Kawase Filter即可：

	half4 KawaseBlur(TEXTURE2D_ARGS(tex, samplerTex), float2 uv, float2 texelSize, half pixelOffset)
	{
		half4 o = 0;
		o += SAMPLE_TEXTURE2D(tex, samplerTex, uv + float2(pixelOffset +0.5, pixelOffset +0.5) * texelSize); 
		o += SAMPLE_TEXTURE2D(tex, samplerTex, uv + float2(-pixelOffset -0.5, pixelOffset +0.5) * texelSize); 
		o += SAMPLE_TEXTURE2D(tex, samplerTex, uv + float2(-pixelOffset -0.5, -pixelOffset -0.5) * texelSize); 
		o += SAMPLE_TEXTURE2D(tex, samplerTex, uv + float2(pixelOffset +0.5, -pixelOffset -0.5) * texelSize); 
		return o * 0.25;
	}

**完整的Runtime + Shader实现可见：** https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/KawaseBlur

Kawase Blur渲染效果接近高斯模糊，但具有更好的性能：

![](media/9baae89bbc953dd6431475a91df4d322.png)

以下是在初始RT DownScale为1、Iteration为6的设置下，Dual Kawase Blur的渲染步骤：

![](media/KawaseBlur-1.gif)

同样，对模糊半径（Blur Radius）参数的调节，可以控制Kawase模糊的程度：

![](media/KawaseBlur-2.gif)


<br>

# 四、双重模糊（Dual Blur）


Dual Kawase Blur，简称Dual Blur，是SIGGRAPH 2015上ARM团队提出的一种衍生自Kawase Blur的模糊算法。其由两种不同的Blur Kernel构成，如下图所示。

![](media/dbd46f6ebbe1a64e080f2667b55cdd8e.png)

相较于Kawase Blur在两个大小相等的纹理之间进行乒乓blit的的思路，Dual Kawase Blur的核心思路在于blit过程中进行降采样和升采样,即对RT进行了降采样以及升采样。如下图所示：

![](media/2ebf4981ecdc7d4c81bf5c2f02edfb31.png)

由于灵活的升降采样带来了blit RT所需计算量的减少等原因， Dual Kawase Blur相较于上文中提到的Gauusian Blur、Box Blur、Kawase Blur等Blur算法,有更好的性能，下图是相同条件下的性能对比。

![](media/c995ed9cebd5b0e2760b0556f36b2149.png)

可以看到，Dual Kawase Blur具有最佳的性能表现。

为了带来更好的性能表现，可以将uv的偏移放在Vert Shader中进行，而Fragment Shader中基本上仅进行采样即可。

Dual Kawase Blur的Fragment Shader实现为：

	
	half4 Frag_DownSample(v2f_DownSample i): SV_Target
	{
		half4 sum = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv) * 4;
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.xy);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.zw);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.xy);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.zw);
		
		return sum * 0.125;
	}

	half4 Frag_UpSample(v2f_UpSample i): SV_Target
	{
		half4 sum = 0;
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.xy);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv01.zw) * 2;
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.xy);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv23.zw) * 2;
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.xy);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv45.zw) * 2;
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv67.xy);
		sum += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv67.zw) * 2;
		
		return sum * 0.0833;
	}


**完整的Runtime + Shader实现可见：** https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/DualKawaseBlur


XPL中也提供了启发自Dual Kawase Blur的Dual Gaussian Blur、Dual Box Blur、Dual Tent Blur的实现。

- **Dual Gaussian Blur**：https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/DualGaussianBlur

- **Dual Box Blur**：https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/DualBoxBlur

- **Dual Tent Blur**：https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/DualTentBlur



Dual Kawase Blur最终的模糊效果截图如下，可以看到其与高斯模糊的模糊表现也非常接近：

![](media/ef952edc96ab383fa45bd443b24d84c4.png)

以下是在初始RT DownScale为1、Iteration为5的设置下，Dual Kawase Blur的渲染步骤：

![](media/734619e4f5594f567b6009d25ebb7c60.gif)

同样，对模糊半径（Blur Radius）参数的调节，可以控制Dual Kawase Blur模糊的程度：

![](media/7c81e8b40e8c86a286003012a2ef7eea.gif)


<br>

# 五、散景模糊（Bokeh Blur）


散景（Bokeh）亦称焦外成像，是一个摄影名词，一般表示在景深较浅的摄影成像中，落在景深以外的画面，会有逐渐产生松散模糊的效果。散景效果有可能因为摄影技巧或光圈孔形状的不同，而产生各有差异的效果。例如镜头本身的光圈叶片数不同（所形成的光圈孔形状不同），会让圆形散景呈现不同的多角形变化。此外，反射式镜头焦外的散景，会呈现独有的甜甜圈形状。

![](media/6e9680adbf5220b3abb9d20bdede07f3.jpg)

图 不同相机参数下得到的不同散景模糊（Bokeh Blur）

散景（Bokeh）在摄影学中被称为焦外成像，而在光学上被称为Circle of Confusion, CoC（弥散圆/散光圈/散射圆盘 ），即下图橙色Image Plane 中的蓝色C所示区域。由于不同的物距（物体到镜头的距离）投影到镜头所形成的焦点不同，但Image Plane 只能放在某个点上，所以就形成了Circle of Confusion, CoC（弥散圆）。

![](media/96f4ff7c533d389d523cddea4a62f2e6.jpg)

图 散景（Bokeh）成因 （图片来自GPU Gems 1）

![](media/b5b10ba32bec1e755e541cc4de8d015e.jpg)

图 散景（Bokeh）大小不同的成因，即Circle of Confusion,
CoC（弥散圆）的大小与人眼分辨率不同的区域。

镜头本身的光圈叶片数不同（所形成的光圈孔形状不同），会让散景形状呈现不同的多角形变化。从最初的多边形，过渡到最终的圆形。

![](media/d6fa04bb9b48e400b803adf63f42416a.jpg)

图 不同光圈叶片数的镜头，决定了不同的散景形状

![](media/f07c1990dbfce8bd3da82617de62c8f5.jpg)

图 不同光圈叶片数的镜头，决定了不同的散景形状

![](media/89eb65f434ae50051cb07cc3c802ca7d.jpg)

图 不同光圈数值的镜头形态，决定了不同的散景形态

从上图可以看出， 由于光圈大小和叶片数量的不同，散景（Bokeh）的形态各异。

在图形学领域模拟散景（Bokeh）的方法有很多，本文将以最标准的圆形散景为例，采用Golden angle(<https://en.wikipedia.org/wiki/Golden_angle>)的思路进行散景模糊（Bokeh Blur）算法的实现。

具体而言，算法思路是基于对大量离散螺旋型（spiral）分布的点的渲染，来模拟出圆形Bokeh的形状。

![](media/7f0e8d10c7e89fddba1ea3249fd2883a.png)

核心的Shader算法实现为：

	half4 BokehBlur(VaryingsDefault i)
	{
		half2x2 rot = half2x2(_GoldenRot);
		half4 accumulator = 0.0;
		half4 divisor = 0.0;

		half r = 1.0;
		half2 angle = half2(0.0, _Radius);

		for (int j = 0; j < _Iteration; j++)
		{
			r += 1.0 / r;
			angle = mul(rot, angle);
			half4 bokeh = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, float2(i.texcoord + _PixelSize * (r - 1.0) * angle));
			accumulator += bokeh * bokeh;
			divisor += bokeh;
		}
		return accumulator / divisor;
	}

即对于每一次迭代，让采样uv旋转一个角度，经过足够次数的迭代后，众多以圆形分散开来的点，将一起组成合适的Bokeh形状。

**完整的Runtime + Shader实现可见：**

https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/BokehBlur


下图为最终实现的效果图：

![](media/ba1760758cd9f00e7b65d12ecf8b7c8a.jpg)

不同模糊半径（Blur Radius）变化，可以控制不同的Bokeh半径的变化：

![](media/d920a7ba9b40b5511769db942c469371.gif)



<br>

# 六、移轴模糊 （Tilt Shift Blur）


移轴模糊（Tilt Shift Blur），又称镜头模糊（Lens Blur） ，是源自摄影领域的一种模糊算法。

在摄影领域，移轴摄影（Tilt-Shift
Photography）泛指利用移轴镜头创作的作品，所拍摄的照片效果就像是缩微模型一样，非常特别。移轴镜头的作用，本来主要是用来修正以普通广角镜拍照时所产生出的透视问题，但后来却被广泛利用来创作变化景深聚焦点位置的摄影作品。移轴镜摄影是将真实世界拍成像假的一样，使照片能够充分表现出“人造都市”的感觉。

![](media/0aca0e7a74b644d1a1232d7311ae8473.jpg)

图 移轴摄影作品

在后处理渲染中进行移轴摄影的模拟，可以采用Grident uv算法控制画面区域模糊强度，配合全屏模糊算法的方式来实现。

采用Grident uv算法控制画面区域模糊强度的Shader核心实现如下：

	float TiltShiftMask(float2 uv)
	{
		float centerY = uv.y * 2.0 - 1.0 + _Offset; // [0,1] -> [-1,1]
		return pow(abs(centerY * _Area), _Spread);
	}

得到的屏幕模糊强度的mask图如下：

![](media/b505f4fd7a72fc5b8b6b7be0b78229c8.png)

接着，配合合适的全屏图像模糊算法，如Bokeh Blur，便可以营造出移轴摄影的画面感：

![](media/28030aa3848315f385c9b9cb06807485.png)

**完整的Runtime + Shader实现可见：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/TiltShiftBlurV2


**XPL中也提供了另一个版本的实现：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/TiltShiftBlur


对模糊半径（Blur Radius）参数的调节，可以用于控制移轴Bokeh半径的变化：

![](media/736728074657aa9df6a2ec3d00a2a65a.gif)

在一定的区域平滑度（Area Smooth）设置下，调节区域尺寸（Area Size）可以控制移轴模糊区域的变化：

![](media/9b9c38f9730affe157566310955b80a3.gif)

<br>

# 七、光圈模糊（Iris Blur）


光圈模糊（Iris Blur）是Photoshop CS6中新增的功能，用于模拟浅景深的效果。

可以根据用户不同的输入参数，将普通照片模拟出景深以及散景的效果。（PS: Photoshop中也同样有Tilf-Shift Blur功能）

![](media/6216cbf3cdcb7ba42679545ca664e96a.jpg)

图 Photoshop中的光圈模糊（Iris Blur）功能

![](media/a6b05dac4ef3d1d4da0efed0a793baec.jpg)

Photoshop中的光圈模糊（Iris Blur）功能

在后处理渲染中进行光圈模糊的模拟，可以采用一个径向的Grident uv算法沿轴心控制画面区域模糊强度，并配合全屏模糊算法的方式来实现。

采用径向Grident uv算法控制画面区域模糊强度的Shader核心实现如下：

	float IrisMask(float2 uv)
	{
		float2 center = uv * 2.0 - 1.0 + _Offset; // [0,1] -> [-1,1] 
		return dot(center, center) * _AreaSize;
	}
	

得到的屏幕模糊强度的mask图如下：

![](media/357c5fae13a41a96cabc2c8e5cd27d20.png)

同样，配合合适的全屏图像模糊算法，如Bokeh Blur，便可以营造出移轴摄影的画面感：

![](media/ea82e6cf88735637243708803e393275.png)

**光圈模糊（Iris Blur）完整的Runtime + Shader实现可见：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/IrisBlurV2



**XPL中也提供了另一个版本的实现：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/IrisBlur


对模糊半径（Blur Radius）参数的调节，可以用于控制光圈Bokeh半径的变化：

![](media/9d7e3b7a0d89a504390557f88541961e.gif)

同样，调节区域尺寸（Area Size）可以控制光圈模糊区域的变化：

![](media/1eda0da557b6d9808c3f966b0f629490.gif)



<br>

# 八、粒状模糊（Grainy Blur）


粒状模糊（Grainy Blur）是一种低成本的模糊方法，在单pass下即可有合适的模糊表现，性能出色，且其模糊质感有点类似在画面上蒙了一层细碎的冰霜。

![](media/ef8abce69b1e9f7987f0e9d203d9e690.png)

其思路是基于随机uv进行采样的抖动，以对粗粒度的模糊进行模拟。核心算法的Shader实现如下：

	float Rand(float2 n)
	{
		return sin(dot(n, half2(1233.224, 1743.335)));
	}
	
	half4 GrainyBlur(VaryingsDefault i)
	{
		half2 randomOffset = float2(0.0, 0.0);
		half4 finalColor = half4(0.0, 0.0, 0.0, 0.0);
		float random = Rand(i.texcoord);
		
		for (int k = 0; k < int(_Iteration); k ++)
		{
			random = frac(43758.5453 * random + 0.61432);;
			randomOffset.x = (random - 0.5) * 2.0;
			random = frac(43758.5453 * random + 0.61432);
			randomOffset.y = (random - 0.5) * 2.0;
			
			finalColor += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, half2(i.texcoord + randomOffset * _BlurRadius));
		}
		return finalColor / _Iteration;
	}

**粒状模糊（Grainy Blur）完整的Runtime + Shader实现可见：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/GrainyBlur

这边是一个模糊半径更大的粒状模糊（Grainy Blur）的渲染效果：
![](media/61.1.png)

对模糊半径（Blur Radius）参数的调节，也可以控制粒状模糊的程度：

![](media/0c68c66628bfa24d6a4a60b291bf364b.gif)

<br>

# 九、径向模糊（Radial Blur）


径向模糊（Radial Blur）可以给画面带来很好的速度感，是各类游戏中后处理的常客，也常用于Sun Shaft等后处理特效中作为光线投射的模拟。

![](media/870224e670b3157d7082d9e37f468c8d.png)

径向模糊的原理比较直接，首先选取一个径向轴心（Radial
Center），然后将每一个采样点的uv基于此径向轴心进行偏移（offset），并进行一定次数的迭代采样，最终将采样得到的RGB值累加，并除以迭代次数。

其核心算法的Shader代码实现如下：

	half4 RadialBlur(VaryingsDefault i)
	{
		float2 blurVector = (_RadialCenter - i.texcoord.xy) * _BlurRadius;
		
		half4 acumulateColor = half4(0, 0, 0, 0);
		
		[unroll(30)]
		for (int j = 0; j < _Iteration; j ++)
		{
			acumulateColor += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.texcoord);
			i.texcoord.xy += blurVector;
		}
		
		return acumulateColor / _Iteration;
	}

**完整的Runtime + Shader实现可见：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/RadialBlur



另外需要注意的是，如果迭代次数不够多，而又设置了较高的Offset值，则会在屏幕四周出现较为明显的折痕，但一般情况下都还可以接受：

![](media/7f0736f2cb5251bb41477f277d252bee.png)

同样，对模糊半径（Blur Radius）参数的调节，可以控制模糊的程度：

![](media/06f1d0823ec0a797fe4999778ade5a59.gif)

<br>

# 十、方向模糊（Directional Blur）


方向模糊（Directional Blur）可以看做是径向模糊（Radial Blur）的一个变体。其主要思路是传入一个角度，然后在runtime层计算出对应的矢量方向：

	float sinVal = (Mathf.Sin(settings.Angle) * settings.BlurRadius * 0.05f) / settings.Iteration;
	float cosVal = (Mathf.Cos(settings.Angle) * settings.BlurRadius * 0.05f) / settings.Iteration;
	sheet.properties.SetVector(ShaderIDs.Direction, new Vector2(sinVal, cosVal));

然后，在Shader层，将每一个采样点的uv基于此方向进行正负两次偏移（offset），接着进行一定次数的迭代采样，最终将采样得到的RGB值累加，并除以迭代次数，得到最终的输出。

核心算法的Shader代码实现如下：

	half4 DirectionalBlur(VaryingsDefault i)
	{
		half4 color = half4(0.0, 0.0, 0.0, 0.0);

		for (int k = -_Iteration; k < _Iteration; k++)
		{
			color += SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.texcoord - _Direction * k);
		}
		half4 finalColor = color / (_Iteration * 2.0);

		return finalColor;
	}


**完整的Runtime + Shader实现可见：**
https://github.com/QianMo/X-PostProcessing-Library/tree/master/Assets/X-PostProcessing/Effects/DirectionalBlur


方向模糊后处理的渲染效果如下：
![](media/7623f49a2849b081d674afffa3201fee.png)

同样，对模糊半径（Blur Radius）参数的调节，可以控制模糊的程度：

![](media/45bee7e1482fd0ea1a60ed71c0e5c854.gif)

而调节Angle参数，可以控制模糊旋转的方向：

![](media/bfc435caf5f6681f35a2bf9718a57c1e.gif)



<br>

# 总结


本文开头已经提到，模糊算法在后处理渲染领域占据着非常重要的地位。很多产品级后处理的实现，都会直接或间接依赖于一种或多种图像模糊算法。后处理管线中所采用的模糊算法的优劣，决定了产品最终的渲染品质和消耗的性能大小。

让我们重新看一下这十种模糊算法的横向对比，相信此时，大家对这十种模糊算法已经有了更全面的理解，以及自己的选择。

![](media/b82ba2aaa60f290da438450a86ccb48f.png)

The End.


<br>

# Reference

[1] GDC 2003, Frame Buffer Postprocessing Effects in DOUBLE-S.T.E.A.L (Wreckless)

[2] SIGGRAPH 2015, Bandwidth-Efficient Rendering

[3] SIGGRAPH 2001，Fast Image Convolutions

[4] Gonzalez R C, Woods R E. Digital image processing, 4th edn. ISBN: 9780133356724[J]. 2017.

[5] https://computergraphics.stackexchange.com/questions/39/how-is-gaussian-blur-implemented

[6] http://datahacker.rs/opencv-average-and-gaussian-filter/

[7] https://towardsdatascience.com/image-processing-class-egbe443-4-filters-aa1037676130


[8] https://www.zhihu.com/question/20813608/answer/261346592

[9] https://en.wikipedia.org/wiki/Box_blur

[10] https://en.wikipedia.org/wiki/Gaussian_blur

[11] http://blog.marmakoide.org/?p=1

[12] https://developer.nvidia.com/gpugems/gpugems/part-iv-image-processing/chapter-23-depth-field-survey-techniques

[13] https://www.flickr.com/photos/valpil58/9425151785

[14] https://en.wikipedia.org/wiki/Bokeh

[15] https://commons.wikimedia.org/wiki/File:Lenses_with_different_apertures.jpg

[16] https://improvephotography.com/29529/aperture-blades-many-best/

[17] https://www.webucator.com/how-to/how-apply-an-iris-blur-effect-adobe-photoshop.cfm

[18] 题图来自：https://www.artstation.com/artwork/Z5RkbZ

