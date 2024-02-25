---
markmap:
  colorFreezeLevel: 2
---

# 计算机图形学
## 变换
### 齐次坐标系
* 仿射变换 = 线性 + 平移
* 定义  <!-- markmap: fold -->
  * 点：(x,y,z,w) = (x/w,y/w,z/w,1)
  * 向量：(x,y,z,0)
### 3d变换
#### 3d 旋转
* 旋转矩阵的逆矩阵是置换矩阵
* 欧拉角
  * 问题：万向锁
  * Rodrigues' Rotation Formula： 把任意旋转表示成矩阵形式
* （拓展）四元数
### MVP (Model View Projection)
#### Model变换
* =模特站位
#### View变换
* =相机站位，把物体转换到相机的坐标系
#### Projection变换
* =拍照
* 两种投影方式
  * 正交投影
  * 透视投影
* 定义视锥  <!-- markmap: fold -->
  * 可以根据这些算出 Projection变换矩阵中的l,r,b,t
  * 需要定义的量
    * 长宽比=aspect ratio
    * 垂直可视角度 = vertical field of view
    * 近平面距离 = n
    * 远平面距离 = f
#### MVP之后：映射到屏幕空间

## 光栅化
### 如何光栅化三角形
* 方法：采样
* 问题
  * 走样
    * 走样成因：信号变化得太快，采样的频率跟不上
    * 例子：摩尔纹
    * 方法：滤波（去掉高频信号）
      * MSAA <!-- markmap: fold -->
        * supersampling -> 先用4x4 sample （先提高像素），再算平均（减少像素）
      * （拓展）FXAA（Fast approximate AA），TAA（Temporal AA）
  * 遮挡
    * 方法：深度缓存
    * 细节 <!-- markmap: fold -->
      * 考虑 super sample -> 得每个采样点都有z-buffer
      * 深度插值
        * 只知道三角形顶点的深度，不知道某个像素点位置的深度，需要插值
### Shading 着色
####
* 定义：对物体应用不同材质
#### Blinn-Phong Reflectance Model
* 高光项 (Specular term) <!-- markmap: fold -->
  * 取决于半程向量和法线夹角
* 漫反射项（Lambertian term）<!-- markmap: fold -->
  * 取决于光源和法线夹角
* 环境光照项 (Ambient term) <!-- markmap: fold -->
  * 常数
#### 着色频率
* 逐平面 (flat shading)
* 逐顶点 (gouraud shading)
* 逐像素 (phong shading)  <!-- markmap: fold -->
  * 应用法线重心插值
#### 纹理映射
* 算法 <!-- markmap: fold -->
  * 三角形每个顶点都对应一个(u,v)，可以找到texcolor
  * 求逐像素的texcolor可以应用重心插值
* 问题
  * 纹理太小了
    * 方法：纹理放大
    * 最邻近插值，双线性插值，Bicubic插值
  * 纹理太大了，产生走样
    * 方法
      * 超采样
        * 问题：计算量很大
      * Mipmap <!-- markmap: fold -->
        * 定义：快速近似正方形范围查询 =  image pyramid
          * 需要查询像素覆盖范围内纹理的平均值
        * 原理：存储范围查询的计算结果，简化计算（空间换时间）
          * 多了1/3的存储空间
        * 细节 
          * 计算需要查询第几层mipmap
          * Trilinear插值
            * 解决查询不连续问题
            * 1.8层 = 0.8 * 2层 + 0.2 * 1 层
      * 各向异性过滤
        * 可以查询长方形
* 应用
  * Environment map <!-- markmap: fold -->
    * 把环境光记录在物体上
    * Cubemap
  * 凹凸贴图 Bump/normal mapping <!-- markmap: fold -->
    * 可以定义任意一个点相对表面的高度，重新计算法线
  * 位移贴图 <!-- markmap: fold -->
    * 真的改变三角形的位置，但要求原来的模型就很细腻（不能在一个三角形上生成不同的位移）
  * 高级纹理
    * 3d纹理 <!-- markmap: fold -->
      * 定义空间任意一个点的颜色
      * 可以做体积渲染 （Volume rendering）
    * 函数纹理 （Procedural Noise）<!-- markmap: fold -->
      * 定义一个噪声函数当纹理
    * Precomputed Shading <!-- markmap: fold -->
      * 提前把纹理算好，之后直接用
### 图形渲染管线
* 步骤 <!-- markmap: fold -->
  * 逐顶点处理
  * 逐三角形处理
  * 光栅化
  * 逐像素处理（Fragment processing）
  * 逐像素操作 (Framebuffer operations)
    * 包含z-buffer
### 光栅化的问题
#### 阴影
* 问题  <!-- markmap: fold -->
  * 光栅化只考虑了局部。涉及到全局光线的问题，光栅化都不太好处理，比如阴影
* 方法：shadow mapping <!-- markmap: fold -->
  * 主要想法
    * 如果一个点不在阴影里，那么摄像机和光源都能看到这个点
  * 算法 
    * Pass 1：建立shadow map，从光源渲染一张图，记录深度
    * Pass 2：从眼睛渲染一张图，变换回光源视图，判断光源可否看到
  * 问题
    * 走样
      * shadowmap的分辨率
    * 需要渲染两遍，开销大
    * 只能做硬阴影
    * 经典的shadow mapping只能处理点光源
### 数学方法：插值
* 重心插值
  * 透视矫正插值 <!-- markmap: fold -->
    * 透视变换后，重心坐标会改变，需要透视矫正
* 双线性插值

## 几何
### 隐式几何
* 定义
  * 满足特定关系 <!-- markmap: fold -->
    * f(x,y,z) = 0
    * 比如函数式可以写出一个球表面
* 例子 
  * constructive solid geometry
  * 距离函数
  * 水平集
  * 分形
### 显式几何
#### 
* 定义
  * 曲面参数化，每个点都能写出来
* 例子
  * 点云
  * Polygon Mesh <!-- markmap: fold -->
    * 存储三角形（或四边形）的顶点
    * .obj 格式
#### 曲线
##### 贝塞尔曲线
* 算法：de Casteljau Algorithm
* 代数表达 <!-- markmap: fold -->
  * 伯恩斯坦多项式
* 性质  <!-- markmap: fold -->
  * 仿射不变性
    * 仿射变换（线性变换+平移）下，只需要把所有控制点都变换完再画一条贝塞尔曲线（等价于变换贝塞尔曲线上每个点）
  * 形成的曲线在所有控制点的凸包内
* Piecewise（逐段）贝塞尔曲线   <!-- markmap: fold -->
  * 最常用，四个控制点
  * 连续的定义
* 贝塞尔曲面 <!-- markmap: fold -->
  * 用贝塞尔曲线在水平方向和竖直方向分别应用
##### 其他spline
* spline = 一个可控的曲线
* B-splines
* NURBS (B-splines的进化版)
#### 网格操作
##### 细分
* 目的： 分出更多地三角形，使得原来的模型更光滑
* 算法
  * Loop subdivision <!-- markmap: fold -->
    * （Loop是人名）
    * 先细分再调整
    * 只能细分三角形网格
  * Catmull-Clark Subdivision  <!-- markmap: fold -->
    * 可以细分四边形
##### 简化
* 目的：减少三角形的数量，物体离得远的时候，可以用简化的模型
* 算法
  * edge collapsing <!-- markmap: fold -->
    * 把边变成点
    * 选取哪条边collapse？
      * 标准：quadric error metrics
        * 可以用贪心算法选边
##### 正则化 Regularization
* 目的：让三角形更一致（大小形状差不多）
## 光线追踪
### Whitted-Style Ray Tracing
* 算法  <!-- markmap: fold -->
  * 从视线cast ray
  * 如果打到反射/折射物体，就recursively cast两条光线，按比例把结果加起来
  * 如果打到漫反射物体，就判断是否能被光源照到，如果可以就计算光栅化结果（比如用blinn-phong模型）
* 备注  <!-- markmap: fold -->
  * 不用深度测试，但是还是不能渲染出软阴影
* 基础光线追踪的技术细节 
  * 光线和物体求交
    * 隐式表面：解方程
    * 显式表面
      * 三角形网格
        * 遍历所有三角形网格，求交
        * 快速的三角形和光线求交算法：Moller Trumbore Algorithm
  * 三角形网格求交的加速：Bounding Volumes
    * 主要想法  <!-- markmap: fold -->
      * 如果光线连包围盒都碰不到，它一定碰不到物体；所以可以把物体用包围盒简化
    * 光线和Axis-Aligned Bounding Box (AABB)求交算法
    * 算法 （如何利用AABB加速）
      * Uniform Spatial Partitions  <!-- markmap: fold -->
        * 把场景分割成相同大小的格子
      * Spatial Partition
        * 物体稀疏的地方用少的格子，密的地方用更多的格子
        * 具体方法
          * 八叉树 <!-- markmap: fold -->
            * 把选中的AABB切成八块，横着切一刀，竖着切一刀
          * KD-tree  <!-- markmap: fold -->
            * 永远只沿着某个轴在某个位置切一刀 （这样就和维度无关了）
            * 类似二叉树
          * BSP-tree <!-- markmap: fold -->
            * 可以斜着切
        * 问题 <!-- markmap: fold -->
          * 一个物体可能属于多个包围盒
          * 三角形和bounding box求交很困难，我们难以判断物体是否属于一个bounding box
            * 但是知道一个三角形，算出它的AABB很简单（算每个坐标的最大最小值）
      * Object Partition & Bounding Volume Hierachy（BVH）
        * 不从空间做划分，而是从物体做划分
        * 常用的物体集合切割方法 <!-- markmap: fold -->
          * Heuristic
            * 选一个维度切割（选最长的axis）
            * 选处于该轴中位数的物体切割
            * 根据Surface Area Heuristic划分（作业）
          * 停止条件：物体集合中物体数量足够小
### 现代光线追踪的物理基础：Radiometry 辐射度量学
#### 概念
* Radient Energy and Flux (Power) <!-- markmap: fold -->
  * 光是energy（单位：焦耳）
  * Radient flux ->单位时间的能量 （功率）（单位：瓦特，另一个单位：lm=lumen）
* Solid Angle  <!-- markmap: fold -->
  * $\Omega=A/r^2$, 锥在单位球面上的面积
* Radiant Intensity  <!-- markmap: fold -->
  * 如何描述光源
  * 单位立体角的power
* Irradiance  <!-- markmap: fold -->
  * 如何描述光照到表面
  * 单位面积的power
* Radiance  <!-- markmap: fold -->
  * 如何描述运动过程中的光
    * 从某个方向打到某个表面，又从某个方向离开
  * 单位面积单位立体角的的power
  * =Irradiance per solid angle
  * =Intensity per projected unit area
  * vs irradiance
    * irradiance是单位表面总的power，radiance是某个方向的power，radiance对方向积分得到irradiance
#### BRDF （Bidirection Reflection Distribution Function）
* 定义：用辐射度量学描述反射
  * 入射方向的radiance如何改变出射方向的radiance <!-- markmap: fold -->
    * $f_r(\omega_i, \omega_r) = \frac{\mathrm{d}L_r(\omega_r)}{\mathrm{d}E_i(\omega_i)}= \frac{\mathrm{d}L_r(\omega_r)}{L_i(\omega_i)\cos\theta_i\mathrm{d}\omega_i}$
### 路径追踪 Path Tracing
#### 渲染方程和全局光照
  * 渲染方程
    * 出射方向的radiance是所有方向radiance影响的积分 <!-- markmap: fold -->
      * $L_r(p, \omega_r) = \int_{H^2}f_r(p,\omega_i\rarr \omega_r)L_i(p, \omega_i)\cos\theta_i\mathrm{d}\omega_i$
  * 全局光照 vs 光栅化
      * 全局光照：Emission + 直接光照 + 间接光照
      * 光栅化：Emission + 直接光照
  * 计算渲染方程的工具：Monte Carlo Integration (蒙特卡洛积分) <!-- markmap: fold -->
    * 思想：积分变采样
#### 算法 <!-- markmap: fold -->
* 从视线cast ray
  * 采样光源，如果光源能照到，计算直接光照的影响
  * 用俄罗斯轮盘赌判定终止（Russian Rolette - RR）
  * 如果没终止，往随机采样的方向recursively cast ray，计算间接光照
### （拓展）高级光线追踪
####
* 分类
  * unbiased Monte Carlo techniques  <!-- markmap: fold -->
    * 期望是真实值（不管用了多少sample）
  * biased  <!-- markmap: fold -->
    * 期望不是真实值，但随着样本量的增加，会converge到真实值
      * consistent
#### Unbiased
* Bidirectional Path Tracing (BDPT)
* Metropolis Light Transport (MLT)
#### Biased
* Photon Mapping
* Vertex Connection and Merging
* Instant Radiosity
### 材质和外观
####
* 材质=BRDF
#### 各种材质
* 漫反射材质
  * 最简单的漫反射材质：均匀地向出射方向反射
* 理想折射反射材质 (Ideal reflective / refractive material )
  * BTDF(transmit 折射) + BRDF(reflect 反射) = BSDF（scatter 散射）
  * 镜面反射
  * 折射
    * 折射方向的计算：Snell's law
  * 光线反射的比例：菲涅尔项（Fresnel term）<!-- markmap: fold -->
    * 导体的反射比例很高
    * 和光线打到物体的方向有关，光线和法线越平行，反射比例越小
* 微表面材质 (Microfacet material)
  * 思想 <!-- markmap: fold -->
    * 微观看物体表面，是无数镜面反射的微表面
  * 微表面材质的BRDF（作业）
* 各向异性材质（Anisotropic Materials）
#### BRDF的性质 <!-- markmap: fold -->
* 非负性
* 线性：可以拆成很多项，分别套进渲染方程，再把结果加起来
* 可逆性：交换入射方向和出射方向，得到的BRDF一定一致
* 能量守恒：经过brdf计算，不能让能量变多，只会变少
* 各向同性的brdf取决于相对方位角
#### （拓展）高级材质
* Participating Media（散射介质）
* Hair Appearance
  * Kajiya-Kay Model
  * Marschner Model
  * Double Cylinder Model
* 布料模拟
* Detailed material
* Procedural Appearnace
## 动画和模拟
### 介绍
#### Keyfram animation
#### Physical simulation
* 弹簧
  * Hooke's Law
* 应用弹簧做布料模拟
  * 别的方法：FEM（Finite element method）
* Particle system  <!-- markmap: fold -->
  * 模拟粒子中力的作用
  * 吸引力，排斥力
    * 电磁力
    * 弹簧
  * Damping forces
    * 摩擦力
  * 碰撞
    * 墙，容器
#### Kinematics（运动学）
  * 用于定义骨骼运动
  * 正运动学：定义关节运动位置
  * 逆运动学：定义端点最终位置，自动生关节变化
#### Rigging（绑定） <!-- markmap: fold -->
  * Blend shapes：定义两个关键帧，插值
#### 动捕
### 方法
#### Single particle simulation
* 描述速度场，写出ordinary differential equation (ODE)
* ODE的解法
  * Euler's method （forward Euler， explicit euler）
  * Runge-Kutta Families <!-- markmap: fold -->
    * 很多种接ODE的方法
    * RK4，4阶的方法
#### Fluid simulation
* 质点法：模拟所有单个物体
* 网格法：把物体分成很多组，以组为单位模拟

## 其他话题
### 相机和透镜
#### 相机的概念
* FOV （field of view） <!-- markmap: fold -->
  * $\mathrm{FOV} = 2\arctan(\frac{h/2}{f})$ h是sensor大小，f是焦距
* 亮度 = 曝光时间 x 到sensor的irradiance
  * 曝光时间
    * 影响因素：快门控制 <!-- markmap: fold -->
      * shutter speed：快门开放速度，速度越快，更少光进来，越不会运动模糊
  * 到sensor的irradiance
    * 影响因素
      * F-Number（F-Stop）<!-- markmap: fold -->
        * 由光圈大小和焦距决定，写为F**N**，N = 焦距/光圈直径
        * 光圈越大，景深效果越明显（非对焦位置物体越模糊）
      * ISO：感光度 <!-- markmap: fold -->
        * 感光度越高，越亮，噪声越大
#### 透镜
* Thin Lens Approximation
  * 可以用来近似相机成像原理
* Defocus Blur <!-- markmap: fold -->
  * 不在focal plane 的物体被接收到会变成一个⚪（circle of confusion），就呈现出模糊效果
  * 取决于光圈的大小（A）
  * Depth of field 
    * 当物体的focal plane距离ideal focal plane（焦距所在的平面）一定距离内的时候，CoC都是足够小的（接近一个pixel的大小，可以认为是锐利的），这段距离就是depth of focus

### 光场
* 全光函数 <!-- markmap: fold -->
  * 整个世界（能看到的东西）是一个七个维度的函数
* 光场图 <!-- markmap: fold -->
  * 用全光函数表示图片
* 光场相机 <!-- markmap: fold -->
  * 光场照出的结果包含了环境中所有光线的信息
  * 可以得到任意景深的照片
### 颜色
* 颜色是人的感知
* 人眼感知颜色的锥形细胞有三类
  * S-Cone （短波长）
  * M-Cone （中波长）
  * L-Cone （长波长）
  * 每个人感光细胞的数量分布差异很大
* Tristimulus theory of color（三刺激 SML） 
* Matamerism （同色异谱） <!-- markmap: fold -->
  * 光的光谱不一样，但是看起来一样（SML一样）
* 色彩空间
  * 标准RGB系统（sRGB）
  * CIE XYZ
  * CIE LAB
  * CMYK <!-- markmap: fold -->
    * 减色系统
    * 用于打印