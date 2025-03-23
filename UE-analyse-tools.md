# UE分析测试工具

## 补充概念

### 如果GPU Thread率先完成了它的工作，而其他二者仍在工作中，那么GPU会因为等待下一帧数据的指令而延迟显示；
如果GPU耗时严重，导致CPU输送的数据没有被及时处理，使得当前帧还没被渲染完毕就进入下一帧渲染线程导致卡顿

### 常用的性能分析工具

- Unreal Frontend Profiler

- Unreal Insights

### UE最常用的GPU分析工具

- 内置的GPU Visualizer

- [通过插件启动的RenderDoc](https://zhuanlan.zhihu.com/p/354097312#:~:text=%E3%80%90UE4%E7%9A%84Rende)

## cmd

### stat fps

- 显示fps与当前帧的总耗时

### stat unit

- 具体内容

	- frame:一帧所耗费的总时间

	- game:游戏逻辑一帧所耗费时间

		- stat game--显示Tick的耗时情况

		- dumpticks--可将所有正在tick的actor打印到log中

		- 对于复杂的问题，则需要借助Unreal Frontend Profiler/Unreal Insights等工具对游戏逻辑中开销较大的代码进行定位

	- draw:准备好所有必要的渲染所需的信息，并从CPU发送GPU所耗费的时间

		- stat initviews--显示Visibility Culling的耗时情况，同事还能显示当前场景中可见的Static Mesh的数量

		- stat SceneRendering--可查看Mesh Draw Call的数量/Translucency的消耗情况

	- gpu:接受到渲染所需信息之后，将像素最终的表现画在屏幕上的耗时

		- Viewport中选择Optimization Viewmodes -> Shader Complexity，可以可视化Shader造成的开销；-> Quad Overdraw，显示GPU对每个Quad的绘制次数；->Light Complexity,查看灯光引起的性能开销

		- ShowFlag.DynamicShadow 0--使用该指令可关闭场景内动态阴影（0 关闭 ；1 开启）

		- r.ScreenPercentage 50--表示将渲染的像素数量减半（也可替换成0-100的数），以此判断瓶颈是否在Pixel-bound

		- ShowFlag.Translucency 0--关闭所有半透明效果（0 关闭；1 开启）

		- stat streaming overview,查看当前纹理对内存的占用情况--TODO优化纹理

- 理解

	- GameThread:
首先会对整个游戏世界进行逻辑层面的计算和模拟（e.g.Spawn多少个新的actor，每个actor在这一帧位于何处、角色移动、动画状态等等），所有这些信息都会被输送到DrawThread

		- 此线程的开销一般可以归因于C++和蓝图的逻辑处理，瓶颈常见于Tick和代价昂贵的逻辑实现

	- DrawThread(RenderingThread):
根据接收到的信息，剔除Culling掉不需要显示的部分（e.g.处于屏幕外的物体），接着创建一个列表，其中包含了渲染每个物体必备的关键信息（e.g.如何被着色，映射哪些纹理等等），再将这个列表输送给GPUThread

		- 主要开销来源于Visibility Culling和Draw Call

		- Visibility Culling:
基于深度缓存Depth Buffer信息，剔除位于相机的视锥体之外的物体和被遮挡住Occluded的物体，当游戏世界中可见的物体过多，剔除所需的计算量也将变大，导致耗时过长

		- Draw Call:
一般理解：CPU准备好一系列渲染所需的信息，通知GPU进行一次渲染的过程

	- GPU Thread:
在获取了这个列表后，会计算出每个像素最终需要如何被渲染在屏幕上，形成这一帧的画面

		- 顶点处理导致的瓶颈

			- Dynamic Shadow:目前动态阴影的生成主要依赖Shadow Mapping，一种在光栅化阶段计算阴影的技术，Shadow Mapping每生成一次阴影需要进行两次光栅化，因此当顶点数过多时，Dynamic Shadow将成为GPU在光栅化阶段的一大性能瓶颈

		- 着色导致的瓶颈

			- 渲染像素过多

			- Shader Complexity:
显示对每一个像素所执行的着色指令数量越多，消耗越大。场景中存在过多的半透明物体，会显著增加Pixel Shader的计算压力

			- Quad Overdraw:
着色期间GPU的大部分操作不是具有单个像素，二十一块一块地绘制，这个块就叫Quad，是由4个像素（2*2）组成的像素块
当模型存在较多狭长细小的三角形时，有效面积较小，但可能占用了很多的Quad，Quad被多次重复绘制，会导致大量像素参与到无意义的计算中，引起不必要的性能开销

			- Light Complexity:
场景内的动态光源数量过多时，会产生大量动态阴影
动态光源的半径过大，导致多个光源的范围出现大量交叠，也可能导致严重的Overdraw问题

			- 内存引起的瓶颈：
有时性能瓶颈还在于过高的内存占用，其中最常见的是大量的纹理加载和采样

### stat RHI

- Rendering Hardware Interface 硬件渲染接口，可查看G-Buffer大小

## 性能分析工具

### Unreal Frontend Profiler（UnrealFrontend.exe）---5.3版本以后没有Profiler了

- 只能检测到CPU侧的开销信息，无法用于分析GPU方面的性能；
可以精确定位到游戏逻辑内开销较大的某个方法，并清晰显示其调用和被调用关系

### Unreal Insights

- 操作：
stat namedEvents:显示trace event的名字和开销问题
Trace.Start:在/Saved/Profiling/目录生成相应的.utrace文件并写入信息。可以指定开启trace（e.g. Trace.Start frame,cpu,gpu,...）
Trace.Stop:结束性能检测，停止写入
通过Unreal Insights打开指定的.utrace文件，进行性能分析

- [性能分析工具：Unreal Insights](https://zhuanlan.zhihu.com/p/444191961)

## 渲染管线理解

### 渲染

- Deferred Rendering延迟渲染（着色）：
基本思想：先将所有物体的几何信息绘制到屏幕空间的缓冲区（G-Buffer）,再逐光源地对屏幕空间的各像素计算光照并着色。这里的第一步称作“Geometry Pass",第二步称作”Lighting Pass"

	- 优劣：
避免不必要的开销，支持大量光源的场景渲染
先深度测试，后着色：首先将三维空间中物体的几何信息存储到了二维的缓冲区中，这里已经包含了深度测试，所以缓冲区中的像素信息，最终都会呈现在屏幕上，不存在冗余。就是保证了在计算光照时，每个像素都只被处理一次，避免了不必要的开销。
将场景内物体的数量与光源的数量解耦。（在Forward Rendering中，场景物体的数量一旦增加，则需要考虑每个光源对新增物体的影响;而对于Deferred Rendering，每个光源中只需要处理固定大小的缓冲区即可，即使场景中物体数量增多，每个光源处理的像素点也不会增加）
对缓冲区的读写带来更高的内存占用和带宽要求，而完全基于屏幕空间的渲染会导致材质信息的缺失

	- 改进：为了解决Deferred Rendering的缺陷，又衍生出Deferred Lighting等方法

- ForwardRendering正向（前向）渲染（着色）：
基本思想：依次遍历场景中的每个物体，将所有光源对它的影响考虑在内，计算光照结果并渲染该物体

	- 优劣：
实现思想简单直接、符合直觉，收到广泛硬件支持
对性能影响大，Overdraw严重。（e.g.比如一个物体受到场景中多个光源的影响，那么相应的像素点可能被重复绘制多次，而最终显示的是最后一次的结果。）
是先着色，再做深度测试。对于那些部分或完全地被另一些物体遮挡地、实际上不可见的物体，针对这部分像素进行的光照计算也属于浪费

	- 改进：提出了Deferred Rendering,就是为了解决其开销消耗问题

### Buffers

- Z-Buffer = Depth Buffer:
z表示纵深，存储着各像素在该方向上的深度值Z-Value
填充Z-Buffer的过程就叫深度测试，在此过程中，只存储像素对应的最浅深度值，作为判断此像素是否被遮挡的依据

	- 在早期，深度值的存储遵循近小远大的原则，但由于传统方法导致不同深度的物体分布不均匀，且容易出现Z-fighting现象，如今Unity和UE均采用近处为1，远处为0的原则，称作Reverse-Z

- G-buffer =Geometric Buffer:
存储每个像素对应的几何信息（Base Color,Normal,Specular,Roughness等），这些信息将在Deferred Rendering的Light Pass阶段被读取，用于光照计算

	- 上述几何信息将以纹理的形式被存储，因此G-Buffer实际上泛指 一系列存储着几何信息的纹理

	- 并非一类信息就会存成一张纹理，例如Specular,Metallic,Roughness这样的灰度信息可以写入同一种纹理的不同通道上
关于G-Buffer的具体结构，UE源码【DeffereShadingCommon.ush】

- D-Buffer =Decal Buffer:
UE中用于存储延迟贴花材质信息的缓冲区，与G-Buffer类似，由多种纹理组成。

	- D-Buffer用于渲染Deferred Decal，此时还需要得到场景的深度信息，因此也会读取Z-Buffer

	- D-Buffer将作为输入信息，参与到渲染G-Buffer(往G-Buffer中写入数据)的过程中

### Passes

- 整个渲染管线可被拆分成一系列的Pass，每个Pass在这个过程中都有所贡献，协力呈现出最终的渲染结果

- Pass本质上GPU执行一系列Draw Call(蘸上哪种颜料，往哪些物体上涂抹)的过程，内部包含多个子步骤，将这些步骤组织成单个Pass再依次执行，像是将多条语言封装成一个个独立的函数再依次调用，从而保证渲染管线能够合理、有序地运转

- 每个Pass读取所需地输入，经过计算，再将输出结果写入相应的缓冲区，也可以简单理解为更新对应的纹理；如果单独研究某些pass，可能会看到它的输出结果使纹理呈现出非常奇怪、不合理的颜色，那是由于这些Pass并不直接影响最终呈现在屏幕上的颜色结果，而是作为一些关键信息参与到背后的运算中，视觉无法直接感知这些信息的真正含义，但它们同等重要

- Base Pass

	- 将处理场景中所以可见的3D模型，执行它们所有相关材质的shader逻辑，然后将计算好的几何信息写入G-Buffer中

		- 在Deferred Rendering中Base Pass就是它的第一阶段：Geometry Pass,写入G-buffer的信息后续将被用于计算光照，以及基于屏幕空间的pass（e.g. Dynamic Reflection,Ambient Occlusion等）

		- 在Forward Rendering中Base Pass既处理几何，也处理光照，直接影响最终的着色结果 

	- 其执行过程中还包括对延迟贴花 Deferred Decal的处理，由于贴花会影响场景物体的几何效果，Base Pass阶段会读取D-Buffer中的信息，并讲它和即将写入G-Buffer的信息融合，使物体在融合了贴花效果后，再在后续接受光照处理

	- 只针对BlendMode为Opaque或Masked的材质进行渲染

- PrePass
有很多别名，比如Z-PrePass,Depth Only Pass,Early-Z Pass等
UE中填充Z-Buffer的阶段就称作PrePass

	- Z-Buffer是后续渲染（Deferred Decal,Occlusion Culling等）中重要的参考信息，因此在整个渲染管线中PrePass的执行顺序考前

	- 默认情况下，只针对场景中的Opaque和Masked物体进行深度测试
进入Project Settings -> Engine -> Rendering -> Optimization -> Early Z-pass可修改相应配置

	- PrePass的开销主要源于场景中Opaque和Masked物体的多边形数量

- BeginOcclusionTests
负责一帧内的遮挡性查询，即在运行时动态查询场景中的物体是否被其他物体所遮挡

	- UE中支持动态遮挡性查询的方法有多种，默认采用的是Hardware Occlusion Queries

		- Hardware Occlusion Queries:
每一帧CPU都会向GPU发送请求，查询场景中的每个Actor是否可见，GPU会在执行完这次查询（此过程总会用到Z-Buffer）的下一帧将结果回读给CPU。

GPU如果没能及时处理完某一次查询，CPU则会持续等待，知道GPU完成查询的下一帧才能判断眼前的物体是否可见。而在相机快速移动的情况下，这样的延迟可能导致场景中某些原本不可见的物体“意外”地突然出现

		- 由于查询粒度是pre-frame pre-Actor,CPU通知GPU执行指令的次数过多，即DrawCall数量太大，会带来显著的性能开销

	- 该过程一般包含几个子项：
ShadowFrustumQueries
GroupedQueries
IndividualQueries

		- 为了尽可能降低DrawCall数量，UE采取了一种类似于“合批”的机制控制查询的力粒度
细粒度：首先还是基于pre-frame pre-Actor的粒度进行查询，如果结果是此帧此物体visible，表面此物体接下来需要被绘制，这种情况下的查村就叫IndividualQueries

		- 粗粒度：而如果结果是invisible,不需要被绘制，就把这个Actor添加到一个Group，之后就针对这个Group进行查询（粒度变大）；直到整个Group都可见，再把它打散清零，粒度重回pre-frame pre-Actor。随着Group中的物体数量增多，DrawCall的数量就会减少。对Group的查询就称作GroupedQueries

		- ShadowFrustumQueries是BeginOcclusionTests的收尾阶段，此阶段将会对local lights(Spot lights & Point Lights)的包围盒对应的mesh发出遮挡性查询（无论Cast Shadows是否开启），如果光源被遮挡了，后续就没有必要对它们进行光照计算了

- BuildHZB(Hierarchical Z-Buffer)
实质上就是带Mipmap的Z-Buffer，基于PrePass输出的Z-Buffer，生成它的一系列Mipmap

	- HZB主要用于另一种叫做Hierarchical Z-Buffer Occlusion的动态剔除方法，以及SSR（Screen Space Reflections）和SSAO（Screen Space Ambient Occlusion）

	- Hierarchical Z-Buffer Occlusion的工作原理与Hardware Occlusion Queries类似，但在对每个Actor做可见性查询时，会先根据Actor的包围盒大小选择适当的Mipmap，再从这种Mipmap中进行采样，得到查询结果。这种方法相对于Hardware Cooclusion Queries更加保守，查询结果精确度更低，实际剔除的物体会更少；但好处是减少了对纹理的采样次数，降低了读写带宽的消耗

	- BuidlHZB只负责生成z-buffer的各级mipmap,直到分辨率的长或宽为1为止，因此它的开销主要取决于z-buffer本身的大小，即渲染分辨率的大小

- ParticleSimulation
计算GPU sprites类型的粒子的运动情况，并记录到两种纹理上
【ParticlesStatePosition】:粒子的世界坐标【ParticlesStateVelocity】：粒子的运动速度

	- 该Pass是Unreal渲染一帧的过程中最先执行的Pass

	- 性能主要开销源于GPU Sprites类型的粒子数量

- RenderVelocities
针对由GPU驱动的、处于运动状态的物体，保存在当前帧物体各顶点的运动速度

	- 主要用于Motion Blur和TAA(Temporal Anti-Aliasing)

	- 性能开销主要源于运动物体的数量，及其这些物体的多边形数量

## UFE(只能作用独立应用程序启动)

### Session Frontend:
可通过UFE打开，也可通过Editor打开
能够监测到所有运行中的游戏会话（包括Editor,Standalone以及连接上PC的Android会话）

- Profiler

	- LiveData:
贮存在内存中的实时数据，只能此刻预览，无法保存以反复查看

	- CaptureData:
将实时信息输出成本地文件后的数据，可加载到Profiler中反复查看

- Automation

- Console

- Screen Comparsion

### Project Launcher

- Build

- Cook 

- Deploy 

- Launch

### Device Output Log

### Device Manager

