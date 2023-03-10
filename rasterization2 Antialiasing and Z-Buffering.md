# Rasterization2 :Antialiasing and Z-Buffering

----

# Antialiasing

--- 

### Artifacts in CG

一切觉得不太对劲的情况。瑕疵：jaggies，锯齿；摩尔纹；wagon wheel illusion 马车轮效应

### 傅里叶变换

傅里叶级数展开 fx转换为正余弦线性组合加常数项

傅里叶变换：把函数变成不同频率的段。采样的频率要跟上fx变化的频率。

傅里叶变换后实现spatial domain 到 frequency domain的的变换。

### Filtering = getting rid of certain frequency contents

**滤波**：把某个特定的频段删掉

**图片的频率信息**：内部低频多，周围高频少。

>  感受时域和频域，图片的不同频率信号。
> 
> 1. High-pass filter抹去低频的高通滤波，剩下边界
> 
> 2. Low-pass filter 模糊图片

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-52-18-image.png)

### Filtering = Convolution = Averaging

> Convolution in the spatial domain is equal to multiplication
> in the frequency domain
> 
> > Option 1:
> > • Filter by convolution in the spatial domain
> > 
> > 用一个滤波得到一个卷积结果。在信号任何一个地方，每一个像素取周围的平均Example: 3x3 box filter；box filter取得越大，越模糊，取更多的像素加入进来进行平均。
> > 
> > Option 2:
> > 
> > • Transform to frequency domain (Fourier transform)
> > 
> > • Multiply by Fourier transform of convolution kernel
> > • Transform back to spatial domain (inverse Fourier)

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-53-08-image.png)

### Sampling theory

**Sampling = Repeating Frequency Contents**

傅里叶变换后，采样的频率要跟上fx的频率变化。

> 1. *函数乘以冲激函数等于频域上的卷积。* 否则存在信号变化太快，采样太慢的情况（**Sparse sampling** ）； Aliasing = Mixed Frequency Contents

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-53-35-image.png)

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-54-18-image.png)

--- 

## How Can We Reduce Aliasing Error?

两种做法：

1. 增加采样率。像素小，像素间隔小，采样频率高，频谱上的搬移 间隔大，存在重叠少，走样就少。但限制于物理情况

> Option 1: Increase sampling rate
> • Essentially increasing the distance between replicas in the
> Fourier domain
> • Higher resolution displays, sensors, framebuffers…
> • But: costly & may need very high resolution

2. 先做模糊，再做采样。模糊就是低通滤波把高频信息先拿掉，再采样，即在频域复制前先把内容变得更窄。 模糊操作（如何滤波）： 用滤波器进行卷积。

> Option 2: **Antialiasing**
> • Making Fourier contents “**narrowe**r” before repeating
> • i.e. Filtering out high frequencies before sampling

---- 

## Antialiased Sampling

先模糊（Pre-Filter）然后再sample。

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-54-50-image.png)

## Antialiasing by Computing Average Pixel Value

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-56-02-image.png)

--- 

#### 抗锯齿

1. **MSAA ：** 像素内部多几个采样点，根据多个样本中被覆盖的情况来判断该像素在图像内的情况。并没有增加分辨率，只是用更多的点得到更准确的三角形覆盖情况，起到抗锯齿的antialiasing。cost：计算量大

> ![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-08-21-56-55-image.png)

其他方法

1. FXAA 先把有锯齿的图片得出来，找到边界并替换掉，去掉锯齿。

2. TAA 复用上一帧的情况，采样点分布在时间上，不在像素内。

---

# Z-Buffer

**深度缓冲**的核心思想是用一块额外的缓冲区域（称为 z buffer）记录每一个像素值当前的最小深度（为方便讨论，我们这里认为离相机近的深度值比较小，且深度值恒为非负数）。首先，我们将 z buffer 中每一个值都初始化为 `float.MaxValue`，当决定是否要绘制这个点时，我们先比较待绘制点的深度值和 z buffer 中的深度值的大小，如果比之前记录的要小，那么就绘制（将 frame buffer 中对应像素的颜色值替换为当前颜色值），并将当前的深度值记录在 z buffer 中：

---

在光栅化的时候如何解决可见性 、遮挡问题。

用近的覆盖远的 。

> 画家算法 对每个图像进行比较 排序，复杂度O(n log n)
> 
> > 存在复杂的覆盖关系，难以处理

### 引入**ZBuffer**

每个图案的前后关系不好排布（画家算法），于是考虑每个像素的情况，缓存其深度。

> **Idea:** • Store current min. z-value for each sample (pixel)
> 
> > • Needs an additional buffer for depth values
> > 
> > 1. frame buffer stores color values
> > 
> > 2. depth buffer (z-buffer) stores depth
> 
> 相机放在原点，往场景看，z is always positive (smaller z -> closer, larger z -> further)

针对某一个像素，新的信息和之前depth buffer的信息进行对比远近，来决定是否更新像素的颜色和深度信息。

> 起初每个像素深度信息 都是无限远
> 
> 这样的比较和更新，就不用区分绘画的前后顺序。
> 
> ![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-09-20-26-14-image.png)

![](C:\Users\admin\AppData\Roaming\marktext\images\2023-03-09-20-37-30-image.png)
