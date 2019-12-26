
### Part1：Introduce
略
### Part2：Generating Embedded Code

#### 定义
对于一个应用程序，基本部分如下：
1. 模型算法
2. 对于的测试程序harness
3. 存在一个实时运行环境，能够运行模型应用
4. 存在一个数据采集接口
5. 存在数据交换接口
6. 存在驱动程序，交互I/O设备

编译进程：
1. 模型编译
- 模型通过代码翻译器生成.rtw文件（ASCII 文件）
2. 代码集成
- rtw文件通过TLC（目标语言编译器），生成应用.c/.h文件并且生成_private.h文件
3. 可执行文件集成
- 上面的文件，通过makefile文件生成exe文件，makefile文件可修改（使用的是汇编）


#### 操作
1. Scope的使用，直接键入，能够监测运行中的数据
2. 软件的代码集成，基本的知识，固定补偿，离散型和ert.tlc格式
3. Report 可以勾选双向可追溯部分的参数，模型和函数互相对应

#### 总结
1. 了解了一个模型生成代码的过程
2. 了解了模型生成的代码，基本构造，包括信号，数据类型，结结构体之类的

### Part3：Integrating Generated Code with Externed Code

#### 定义
一般来说，外部代码是手写代码。\
外部代码集成的两种方式：
1. 模型嵌入已生成的代码
2. 导出模型代码到外部代码中，作为功能的一部分（harness使用的必要性）
3. 这里的Harness就是Main函数，一般都不用

集成记录数据接口函数，主要是用来验证代码的，在最终的代码中，可以被清除
1. 使能logging操作，和对应的MAT-file，集成代码和对应的接口
2. 在IDE上创建工程
3. 在工程找那个添加生成的代码
4. 编译运行工程，会生成一个MAT-file包含运行数据
5. 上载MAT-file文件比较模型仿真后的结果
6. 在实际的开发工程中，禁用logging代码

#### 操作
1. File->Mode Properties->Mode Properities 可以定义模型的一些属性，比如调用函数，变量初始化等等
2. 代码打包到zip格式 Code Generation ->Package code and artifacts，这个好处就是不需要费力寻找.c/.h文件了
3. 嵌入外部.c/.h文件， Code Generation ->Custom Code ->Source/Header file -> 空白地方复制填写路径\Added.c
4. 添加数据记录的接口函数：Code Generation->Interface->non-finite numbers && Advanced parameters(MAT-file logging)选上
5. 只生成被调用的代码的方式（不打包到zip） Code Generation -> Interface ->Shared code placement(shared location)；这个操作生成的代码在slprj\ert\_sharedutilts中

#### 总结
1. 了解如何将生成的代码嵌入外部工程
2. 了解数据记录接口的使用，及对应的代码生成的方式



### Part4：Real-Time Execution
#### 定义
通常不同的模块有不同的速率，或者说是不同的调用周期，那么针对这种情况，在模型中需要增加代码
1. 确保每个模块在它自己的调用周期中，这参考速率列表
2. 管理在不同速率下变量的发送，这参考速率反馈

- 单速率模型（多个模块同一速率），基于基本的速率执行，条件性的更新子速率模块
- 多速率模型，会使能任务优先级功能，高优先级先执行，低优先级后执行

高优先级的任务先执行，执行完后，执行低优先级的任务，如果基本的任务周期时间，低于两个任务顺序执行所需要的时间，那么周期性都会先执行高优先级的任务，再继续执行低优先级上次没有执行完的部分

#### 操作
1. 在模块之间增加Rate Transition模块，可以看到模块之间的数据传输关系
2. 不同Task模块之间，确保数据能够正确的传输，需要勾选 Solver -> **Treat each discrete rate as a seperate task** && **Automatically handle rate transition for data transfer**
3. 可以在模块属性上直接更改它的调用周期，调用周期必须是基本周期的倍数

#### 总结
了解了不同任务周期的模块如何在同一模型中的实现


### Part5：Controlling Function Prototypes
#### 定义
可重入代码：就是代码可以被多线程使用，举例，子函数被线程1调用，子函数内的局部变量在下一个线程使用的时候，可以继续往下走。***一般应操作用不到***

#### 操作
修改Func在代码中有输入输出的方式（各种原型都可设置）
1. 打开 Interface -> Configura Model Functions
2. 自定义函数的初始化函数和主函数，并且可以定义主函数的原型
3. 最后，需要认证一下，是否可用

可重入代码操作
1. Interface —> Code Interface Package -> Reuse Function  (一般不用)

#### 总结
1. 了解了生成不同代码原型的方法

### Part6：Optimizing Generated Code
#### 定义
嵌入式代码的要求：
- 可调试
- 可追溯
- 有效率，充分利用RAM & ROM
- 安全预防

#### 操作
1. Report -> Static Code Metrics(静态代码指标) 会生成Static Code Metrics Report 里面包含，代码的行数，全局变量的大小，堆栈的大小
2. 清除不需要的初始化代码和内部变量的初始化 Optimization ->Code Generation ->Data Inilization(全部勾选) ，这样操作，会将初始化值取消；影响是，提高效率，不过对安全性可能会降低。***慎用***
3. 清除Termination Code, Interface ->Adavanced Parameters -> Terminate functions require(不勾选)；不需要这个函数，有这函数会影响效率和安全性

#### 总结
按照需求，主要是清除一些可能不需要的功能，对集成的代码进行优化


### Part7：Customizing Data Characteristics in Simulink
#### 定义
数据的特征由两个属性来体现：
1. Date Type
   - 数据元素的存储大小
   - 数据元素的数值范围
   - 数据元素分辨率精度
2. Storage class
   - 数据元素的实例化格式
   - 内部或者外部的数据分配
   - 数据定义的分配地方
  
数据在Simulink中可以代表**信号，参数和状态**

信号：的数据类型的可以由以下方式得到
1. 自己指定
2. 内部规则，自动生成
3. 继承其它模块

状态：必须是继承其它模块

参数：数据类型，以下方式得到
1. 自定义
2. 内部规则生成
3. 继承或者从其它模块得到
4. 从Matlab中得到（比如加载Mat文件）

#### 操作
1. Optimization ->Default parameter bahavior(Tunable)，这样操作，参数会以结构体的形式体现，表明参数可变

#### 总结
了解数据的概念，及基本的一些操作


### Part8：Customizing Data Characteristics Using Data Objects
#### 定义
数据的通用属性包括：
- Name
- Value
- Date Type
- Description
- Storage class
- Units
- Rang

Symbols几种格式对应的意思
- $R：Root Model__________________根模型
- $N：Object Name_________________对象名称
- $M：Name Mangling String_______名称管理字符串
- $F：Method Name_________________ 方法名称
- $H：System hierarchy level________系统层次结构
- $A：Data Type acronym______________数据类型的首字母缩写

Data Dictionary：数据字典
- 后缀是.sldd
- 可以容纳全局设计数据（信号，参数，全局数据对象）和配置（Simulink.ConfigSet）
- 数据字典还有以下的优势：
   - 阻止数据在base workspace被杂波干扰或者覆盖
   - 不同模型可以使用不同数字字典，它们之间没有冲突
   - 内部变化可追溯和比较
   - 可以参考其他数据字典更快，更有效率

#### 操作
打开Model Explorer，其中有几个特殊的信号可以注意一下：
1. Simulink.AliasType ：类型别号，就是自定义基础数据类型，可被信号调用
2. Simulink.NumericTypr：类型别号，既可以使用基础数据类型，也可以使用自定义长度的类型，比如说长度为14等
3. Signal -> Code Generation 可以定义结构体封装几个相同类型的信号
4. 同样在Symbols中可以定义集成代码的数据类型 
   - signal naming  -> Force lower case代表信号使用类型别号
   - parameter naming -> Force upper case 代表参数没有数据类型别号被使用

数据字典建立（包含数据和配置）：
1. File -> Model Properties -> Link to Data Dictionary 建立完成后，可以选择删除base workspace 的数据
2. Model Explore -> configuration 右击 Convert to Configuration reference。右击数据字典加载Configuration
3. Save Changes 完成一个完整的数据字典

#### 总结
1. 主要是对数据类型的一些配置，很有借鉴意义
2. 是数据字典的使用，就像之前使用的.Mat文件，数据字典的优势更加强大

### Part9：Creating Custom Storage Classes
#### 定义
CSC(Custom Storage Class)主要是手动生成结构体的方式，有以下的好处：
1. 加载已有的结构体，观察它们的属性
2. 创造新的结构体
3. 创造参考其他Package的结构体
4. 复制和修改已有的结构体
5. 检测和确认结构体的状态
6. 控制结构体的存储位置
7. 预览伪代码
8. 保存自定义的结构体


#### 操作
1. 键入cscdesigner 找到安装路径的模板
2. 自定义文件名和信号参数名
3. 配置对应结构体参数，保存
4. 在Generated Code ->Custom Code build imformation中增加.c文件
5. 生成代码，结构体已存在

#### 总结
这是使用cscdesigner来自定义结构体，感觉用的比较少，而且比较费劲，不推荐，不如使用 bus edit

### Part10：Bus Object and Model Referencing
#### 定义
总线信号分为虚拟和非虚拟的，只有非虚拟的才能生成代码结构体
#### 操作
1. Edit -> Bus edit 打开对应的界面，按照需要生成对应的结构体
2. 使用Bus Creater 接入对应的参数
#### 总结
这个用的比较多，基本已经掌握，也必须熟悉



### Part11：Customizing Generated Code Articulture
#### 定义
一个经典的Simulink模型，除了基础的模型外，还可以包含：
1. 非虚拟子系统
2. MATLAB功能模块
3. S-Function
4. Stateflow
5. 模型参考

模块代码生成几种格式：
1. Auto：只存在一个子系统就会默认为Inlined，否则判断是否多个子函数存在
2. Inline：子系统代码嵌入它的父层
3. Nonreusable Function：子系统封装独立的初始化和输出/更新功能，每一个函数是独一被使用的（会生成独立的.c/.h文件）
4. Reusable Function：可以重复使用的功能，封装库，可以被其它模型调用

配置Reusable Function的方法：
1. 配置模块是原子子系统
2. 配置模块生成可重复使用的代码
3. 指定功能名
4. 封装可重复使用子系统，为他配置可调参数
5. 把重复可利用的子系统留在库中，阻止各种情况改变它

#### 操作
Reusable Function 的使用：
1. 新建一个模型，添加原子子系统模块
2. 右击模块属性，Code Generation -> Function Packaging (reusable Function)
3. Function Name -> User Specified
4. File Name Option -> Use Function Name 
5. 右击模块Mask->Create Mask，自己发挥，完成
6. File -> New ->Library 新建库，并将之前完成的模型拷进去，保存
7. 打开文件，复制库文件，调用多次都可以

Code Generation -> Code Placement
1. Modular：生成的代码有.c/.h/ _private.h/ _types.h
2. Compact (with seperate data file)：生成.c/ .h/ _data.c
3. Compact：生成的文件.c/.h

#### 总结
1. 主要了解了模型的一个重复利用代码库的使用
2. 代码生成文件的格式的不同方式

### Part12：Advanced Customization Techniques
#### 定义
TLC(目标语言编辑器)，rtw文件
1. 更改特定模块生成代码的方式
2. 在模型中内嵌S-function
3. 改变全局意义上的代码生成方式
4. 执行大规模的自定义变量

#### 操作
1. Code Generation -> Templates ：这部分是设置生成代码的样式

#### 总结
这部分主要是更改生成代码的样式，了解一下，变更CGT和CFP文件，感觉暂时用不到深入


### Part13：Deploying Generated Code
#### 定义
生成的代码一般有以下的原因：
1. 用在CPU或者指定的开发板上（编译/连接/调试）
2. 支持目标硬件上的驱动程序
3. 配置指定编译器的编译进程，或者开发出一个调试环境

适合的产品交付包含：
1. 给盒子提供有意义的I/O驱动
2. 第三方调试器能够容易下载
3. 用户能够控制暗访Flash或者RAM的存储位置
4. 支持目标可调可视化


转换成嵌入式代码的生成匹配对应的的硬件和平台，必须要提供以下客户部件：
1. 内嵌能和I/O交互的S-Function程序，用作采样和驱动
2. 有个主程序（harness）
   - 配置硬件和中断
   - 安装中断服务程序
   - 管理时间和中断
3. 一个Make 机制，三种之一
   - 一个样本makefile 包含平台编译器
   - 客户工具链，集成Simulink环境
   - 一个第三方的IDE
 


#### 操作
略

#### 总结
这部分，主要是集中在，编译等工具链上，这部分很难，不过一般一个产品，对应的工具链都已固定，到时候再看吧


### Part14：Integrating Device Drivers
#### 定义
通常意义上的驱动程序，包含
1. 初始化 I/O驱动 ，主要是控制寄存器到希望的模式和操作
2. 计算模块输出
3. 终止程序


#### 总结
这部分主要讲硬件驱动，都是一些概念性的东西，了解即可
### Part15：Improving Code Efficiency and Compliance
#### 定义
Model Advisor 模型顾问，可以检验模型及其子系统的条件，配置等

#### 操作
1. Analysis ->Model Advisor -> Model Advisor -> By Product -> Simulink Coder (全部打钩)
2. 运行右侧 Run Selected Checks

#### 总结
这章主要是对Model Advisor操作，查看到底是否模型，代码存在异常