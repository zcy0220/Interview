# Unity基础

## Unity回调方法执行顺序
* Awake：初始化时调用
* OnEnable：对象处于活动状态时调用
* Start：第一帧更新之前
* FixedUpdate：OnTriggerXXX、OnCollisionXXX等物理相关
* Update：yield StartCoroutine等协程相关
* LateUpdate：OnGUI等渲染相关
* Yield WaitForEndOfFrame：等待帧结束
* OnDisable：不在活动状态，与OnEnable对应
* OnDestroy：销毁时调用

## Animator和Animation的区别
* Animation：控制单一动画的播放
* Animator：控制多个动画状态的切换

## Unity的路径有哪些
* Resources：文件夹下的资源不管是否有用，全部会打包进.apk或者.ipa，并且打包时会将里面的资源压缩处理
* Application.dataPath：程序的数据文件所在文件夹的路径，例如在Editor中就是项目的Assets文件夹的路径，通过这个路径可以访问项目中任何文件夹中的资源，但是在移动端它是完全没用
* Application.streamingAssetsPath：文件夹中的资源在打包时会原封不动的打包进去，不会压缩，初始化包体的Bundle包放在这里
* Application.persistentDataPath：回一个持久化数据存储目录的路径，可以在此路径下存储一些持久化的数据文件。这个路径可读、可写，但是只能在程序运行时才能读写操作，不能提前将数据放入这个路径，沙盒路径。热更新的Bundle放在这里

## 状态机和行为树
* 状态机：
	- 1.基本节点是状态。他包含了一系列运行在该状态的行为以及离开这个状态的条件
	- 2.状态可以任意跳转,实现简单,但是对于大的状态机很难维护.状态逻辑的重用性低.
	- 3.状态机状态的复用性很差，一旦一些因素变化导致这个环境发生变化。你只能新增一个状态，并且给这个新状态添加连接他以及其他状态的跳转逻辑
* 行为树：
	- 1.高度模块化状态，去掉状态中的跳转逻辑，使得状态变成一个“行为”。
	- 2."行为"和"行为"之间的跳转是通过父节点（Composite）的类型来决定的（例如sequence或者selector) 。比如并行处理两个行为，在状态机里面无法同时处理两个状态。
    - 3.通过增加控制节点的类型，可以达到复用行为的目的
    
## 定点数和浮点数的区别
* 定点数：固定小数点的位置
* 浮点数：小数点的位置可按需浮动，这种设计可以在某个固定长度的存储空间内表示定点数无法表示的数

## 四元数和欧拉角的区别
* 欧拉角：
	- 优点：三个角度组成，直观，容易理解
	- 优点：可以进行从一个方向到另一个方向旋转大于180度的角度
	- 弱点：万向锁问题
* 四元数：
	- 优点：四元旋转不存在万向节锁问题
	- 优点：存储空间小，计算效率高
	- 弱点：单个四元数不能表示在任何方向上超过180度的旋转
	- 弱点：四元数的数字表示不直观

## 如何处理抖动
* 物体抖动的原因就是不匀速，所有渲染帧移动物体位置不能直接根据逻辑帧的位置赋值，而是渲染位置匀速的追逻辑位置
* 相机抖动的处理：放在LateUpdate里做平滑处理

## 游戏同步如何保证准确性
* 输入一致性：每个客户端发送输入指令给中转服务器，服务器按接收的顺序统一发送所有客户端
* 输出一致性：随机种子，定点数，用List遍历

## 如何做断线重连
* 断线客户端重连上后，服务器向其他玩家请求一份全量数据，转发给重连上的客户端，重连客户端加速追上最新帧

## 网络同步框架
* 状态同步：服务器执行逻辑，把计算结果同步给所有客户端，客户端根据服务器下发的结果跟新自己的状态
* 帧同步：所有客户端执行相同的帧率，同步输入指令，通过相同的输出达到一致的状态

## Android图片压缩

## 自动寻路算法 A*算法
* 搜索区域网格化
* 开启列表：周围所有可到达或者可通过的方格
* 关闭列表：所有不需要再次检查的方格
* 路径权重：F = G + H
	- G：从起点A，沿着产生的路径，移动到网格上指定方格的移动耗费
	- H：从网格上那个方格移动到终点B的预估移动耗费。曼哈顿方法，它计算从当前格到目的格之间水平和垂直的方格的数量总和，忽略对角线方向。然后把结果乘以10
* 流程：
	- 寻找开启列表中F值最低的格子。我们称它为当前格
	- 把它切换到关闭列表
	- 检测对相邻的8格中的每一个，如果它不可通过或者已经在关闭列表中，略过它。反之如下，如果它不在开启列表中，把它添加进去。把当前格作为这一格的父节点。记录这一格的F,G,和H值。如果它已经在开启列表中，用G值为参考检查新的路径是否更好。更低的G值意味着更好的路径。如果是这样，就把这一格的父节点改成当前格，并且重新计算这一格的G和F值。如果你保持你的开启列表按F值排序，改变之后你可能需要重新对开启列表排序
	- 停止，当你把目标格添加进了开启列表，这时候路径被找到，或者没有找到目标格，开启列表已经空了。这时候，路径不存在。保存路径。从目标格开始，沿着每一格的父节点移动直到回到起始格。这就是你的路径。

## 如何解决UI遮挡物体的问题
* Camera的TargetTexture，RenderTexture和UI里面的RawImage，把3D物体渲染到UI上，就可以控制层级
* 多Canvas控制层级

## 图形渲染管道
* 应用程序阶段
* 几何阶段
* 光栅化阶段

## 高斯模糊
* 每个像素的颜色值都是由其本身和相邻像素的颜色值进行加权平均得到的，越靠近像素本身，权值越高，越偏离像素的，权值越低。而这种权值符合我们比较熟悉的一种数学分布-正态分布，又叫高斯分布，所以这种模糊就是高斯模糊啦

## 光照模型
* 漫反射 + 环境光 + 高光

## mono和il2cpp
* mono：动态语言->IL（虚拟机可以识别的中间语言）-> 在mono VM上运行
* il2cpp：动态语言->IL->c++ code -> IL2CPP VM
* 1.将il变成cpp的首要原因除了是cpp在各大平台的可移植性高之外还有cpp在各个平台编译时会做相应的代码优化，提高运行效率
* 2.为什么由il编译成cpp后还需要il2cpp vm的存在呢，是因为动态语言里转换成静态语言时仍然有内存回收问题的存在，需要靠动态语言的gc去解决，还有线程的创建工作，所以il2cpp vm只包含了一小部分工作，相比mono vm剔除了不必要的IL加载和动态解析的工作，使得IL2CPP VM可以做的很小，并且使得游戏载入时间缩短

## mono怎么优化
* GC优化
* 场景切换时，主动调用System.GC.Collect(),及时清理内存

## CPU优化：
CPU的方面的优化：
* DrawCalls
* 物理组件（Physics）
* GC（虽然GC是处理内存问题，但是CPU使用GC去处理内存）
* 当然，还有代码质量

## GPU优化：
* 填充率，可以简单的理解为图形处理单元每秒渲染的像素数量。
* 像素的复杂度，比如动态阴影，光照，复杂的shader等等
* 几何体的复杂度（顶点数量）
* 当然还有GPU的显存带宽

## 如何减少Drawcall
* Drawcall：CPU调用底层图形接口的操作
* Drawcall影响的是CPU的效率
* 比如有上千个物体，每一个的渲染都需要去调用一次底层接口，而每一次的调用CPU都需要做很多工作，那么CPU必然不堪重负。但是对于GPU来说，图形处理的工作量是一样的。所以对DrawCall的优化，主要就是为了尽量解放CPU在调用图形接口上的开销。所以针对drawcall我们主要的思路就是每个物体尽量减少渲染次数，多个物体最好一起渲染。所以，按照这个思路就有了以下几个方案：
	- 使用Draw Call Batching，也就是描绘调用批处理。Unity在运行时可以将一些物体进行合并，从而用一个描绘调用来渲染他们。具体下面会介绍
	- 通过把纹理打包成图集来尽量减少材质的使用
	- 尽量少的使用反光啦，阴影啦之类的，因为那会使物体多次渲染