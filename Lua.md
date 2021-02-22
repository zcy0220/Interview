# Lua基础

## lua的基本类型
* nil、boolean、number、string、userdata、function、thread、table

## lua元表（Metatable）
* setmetatable(table, metatable)：对指定table设置元表(metatable)
* getmetatable(table)：返回table的元表
* __index：
	- 1.在表中查找，如果找到，返回该元素，找不到则继续
    - 2.判断该表是否有元表，如果没有元表，返回nil，有元表则继续。
    - 3.判断元表有没有__index方法，如果__index方法为nil，则返回nil；如果__index方法是一个表，则重复1、2、3；如果__index方法是一个函数，则返回该函数的返回值
~~~ lua
local tableA = {k2 = "Hello"}
local tableB = {k1 = "Hi"}
setmetatable(tableB, {__index = tableA})
print(tableB.k2)
----------------------------------------
打印：Hello
----------------------------------------
function fn (table, key)            
    return "Hello"
end
local tableB = {k1 = "Hi"}
etmetatable(tableB,{__index = fn}) ----会将table 和 key作为参数传给fn
print(tableB.k2)
打印：Hello
~~~
* __newindex:  __index用于访问，__newindex用于设置新值。当一个table设置了元表，为这个table设置键值的时候，如果这个键存在，那么直接修改当前键值；如果这个键不存在，那么会操作元表中__newindex相应的值。__newindex的值为一个table或者函数
~~~ lua
local tableA = {}
local tableB = setmetatable({k1 = "Hi"},{__newindex = tableA})
tableB.k1 = "HiHi"
tableB.k2 = "Hello"
print(tableB.k1, tableB.k2, tableA.k2)
----------------------------------------
打印：HiHi nil Hello --(特别注意因为__newindex, 所以赋值tableB.k2设置的是tableA.k2，但又因为没有__index，所以访问tableB.k2是不会访问原表的值，所以为nil)
----------------------------------------
function fn (table, key, value)
	-- 会将table、key、value传给fn作为参数
end
local tableB = {k1 = "Hi"}
setmetatable(tableB,{__newindex = fn})
tableB.k2 = "Good"
~~~
* __call：可以直接像调用普通方法一样传入参数直接调用table，但前提是table设置了元表，并且元表中有__call键
~~~ lua
local table = {1,2,3}
local sum = 0
local fn = function (t, v)        --table会作为第一个参数，其他的为传入的参数
    for i=1,#t do
        sum = sum + t[i] * v      --将table中的每一个元素*5 再相加求和
    end
    return sum
end
setmetatable(table,{__call = fn})
print(table(5))  --调用table，传入一个参数5（可以传入多个参数）
----------------------------------------
打印结果：30
~~~
* __tostring：元方法用于修改表的输出行为
* 运算符：'+'为例，对应的元方法为__add
~~~ lua
local table1 = {1,2,3}
local table2 = {4,5,6}
local fn = function (t1, t2)         -----table会被传入作为参数
  local newtable = {}
  for i=1,3 do
    newtable[i] = t1[i] + t2[i]
  end
  return newtable
end

setmetatable(table2,{__add = fn})

local newtable = table1 + table2
for k,v in pairs(newtable) do
  print("newtable --", k, v)
end
----------------------------------------
打印结果：
newtable -- 1    5
newtable -- 2    7
newtable -- 3    9
~~~
## Lua如何实现面向对象
* 类: 是对具有属性相同的对象的一种归纳，可以看作一个模板或者原型。然后我们用这个原型去造对象。 这个原型也占据着计算机的一段内存空间，只不过它被创造出来就一直在那呆着，只有一份。因此我们可以用一个一直存在的对象去作为一个原型来代表一个类，lua中require一个lua文件时会只执行这个文件一次，并会记录这个文件。由此我们可以推出：可以用一个文件模拟一个类，而这个文件本质上就是一个table占据着一段内存空间
* 用元表实现继承

## Lua中table的底层实现
* 容器功能：与其他语言相似，lua也内置了容器功能，也就是table。而与其他语言不同的是，lua内置容器只有table。正因为如此，为了适配不同的应用需求，table的内部结构也比较考究，分为了数组和哈希表两个部分，根据不同需求来决定使用哪个部分
* 面向对象功能：与其他语言不同的时，lua并没有把面向对象的功能以语法的形式包装给开发者。而是保留了这样一种能力，待开发者去实现自己的面向对象。而这一保留的能力，也是封装在table里的：table里可以组合一个metatable，这个metatable本身也是一个table，它的字段用来描述原table的行为
* 查询操作：查询key是否存在，分为了int和非int类型。如果key是int类型并且小于sizearray，那么直接返回对应slot。否则走hash表查询该key对应的slot
* 新增元素，rehash，迭代操作

## Lua的闭包
* 通过调用含有一个内部函数加上该外部函数持有的外部局部变量（upvalue）的外部函数（就是工厂）产生的一个实例函数
* 闭包组成：外部函数+外部函数创建的upvalue+内部函数（闭包函数）

## lua GC 怎么减少
* 标记清除算法，包含标记和清除两个主要阶段
	- 标记阶段查找所有被跟对象集合(root set)直接或间接引用的对象
	- 清除阶段释放所有没有被标记的对象，将被标记的对象的标记清除

## 了解过lua虚拟机吗
* 由于每个脚本语言都有自己的一套字节码，与具体的硬件平台无关，所以无需修改脚本代码，就能运行在各个平台上。硬件、软件平台的差异都由语言自身的虚拟机解决。
* 由于脚本语言的字节码需要由虚拟机执行，而不像机器代码那样能够直接执行，所以运行速度比编译型语言差不少。

## Lua虚拟机工作流程
* 将源代码编译成虚拟机可以识别执行的字节码，负责将Lua代码进行词法分析、语法分析等，最终生成字节码
* Lua虚拟机执行的主函数luaV_execute，主函数是一个大的循环，依次从字节码中取出指令并执行

## Lua底层模块加载原理
* 1.会去检查package.loaded表格是否已经加载过，如果已经加载过直接返回，所以重复加载同一个模块多次只会加载1次
* 2.如果package.loaded表格没有此模块记录，则会到package.path指定的路径中搜索lua文件，如果找到则会调用loadfile加载，会返回一个函数(loader)。如果找不到lua文件，则会去package.cpath指定的路径中搜索动态库so文件，如果找到则会调用package.loadlib加载并寻找luaopen_modname函数(也就是loadfile的结果)。
* 3.不管是在lua文件中找到还是在动态库中找到，现在都已经有了一个loader加载器(函数)
* 4.此时，require调用加载器函数并传入2个参数(大多数模块会忽略参数)，返回的任意值会存入package.loaded表格以便下次重复加载时直接返回。如果没返回值，也会当做有返回存入package.loaded(来避免下次重复加载时又白跑一遍2-4步骤)

## Lua协程，对称和非对称协程
* 协程（coroutine），意思就是协作的例程，最早由Melvin Conway在1963年提出并实现。跟主流程序语言中的线程不一样，线程属于侵入式组件，线程实现的系统称之为抢占式多任务系统，而协程实现的多任务系统成为协作式多任务系统。线程由于缺乏yield语义，所以运行过程中不可避免需要调度，休眠挂起，上下文切换等系统开销，还需要小心使用同步机制保证多线程正常运行。而协程的运行指令系列是固定的，不需要同步机制，协程之间切换也只涉及到控制权的交换，相比较线程来说是非常轻便的。不过同一时刻可以有多个线程运行，但却只能有一个协程运行。
* 协程具有两个非常重要的特性：
	- 私有数据在协程的间断式运行期间一直有效
	- 协程每次yield后让出控制权，下次被resume后从停止点开始继续执行
* resume／yield语义实现的协程属于非对称协程 A resume B resume C， C yield只能到B，而不能到A或其他协程
* 对称协程只有一个语义可以将控制流直接转到目的协程