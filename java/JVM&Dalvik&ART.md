[TOC]

### JVM(Java Virtual Machine)
* 将字节码文件翻译成机器码

#### Dalvik
* Android平台的虚拟机
* 应用运行字节码需要通过即时编译器(`just in time` ，`JIT`)转换为机器码，这会拖慢应用的运行效率
* `JIT`会将经常运行的代码编译成机器码，存入运存，所以并不是每次运行都要即时编译
* `dex`格式是专为`Dalvik`设计的一种压缩格式，适合内存和处理器速度有限的系统
* `Dalvik`虚拟机可以在编译时提前优化代码而不是等到运行时
* 虚拟机很小，使用的空间也小，被设计来满足可高效运行多种虚拟机实例。
* 常量池已被修改为只使用32位的索引，以简化解释器
* 标准Java字节码实行8位堆栈指令,Dalvik使用16位指令集直接作用于局部变量。局部变量通常来自4位的“虚拟寄存器”区。这样减少了Dalvik的指令计数，提高了翻译速度。

##### 应用程序启动过程
> 每启动一个应用程序，都会相应地启动一个 `dalvik` 虚拟机，并建立`JIT` 线程，一直在后台运行。当某段代码被调用时，虚拟机会判断将需要编译成机器码的做标记，`JIT` 线程不断判断此标记，如果发现被标记就把它编译成机器码，并将其机器码地址及相关信息放入 `entry table` 中，下次执行到此就跳到机器码段执行，而不再解释执行，从而提高速度。

##### 寄存器&栈
* `Dalvik`虚拟机是基于寄存器的架构，而 `JVM` 是基于栈的架构
* 基于寄存器架构的VM执行起来更快，可以做到更好的提前优化，但是代价是更大的代码长度
* JVM不用寄存器架构是为了跨平台，而`Dalvik`只需支持`ARM`架构的寄存器就行了
* 基于堆栈的机器必须使用指令才能从堆栈上的加载和操作数据，它们需要更多的指令才能实现相同的性能。
* 基于寄存器机器上的指令必须经过编码,因此,它们的指令往往更大。
![](pic/register_stack.png)

#### ART(Android Runtime Tool)
* ART 环境中，应用在第一次安装的时候，字节码就会预先编译成机器码，使其成为真正的本地应用，这个过程叫做预编译（`AOT,Ahead-Of-Time`）
* 优点：
	* 系统性能的显著提升。
	* 应用启动更快、运行更快、体验更流畅、触感反馈更及时。
	* 更长的电池续航能力。
	* 支持更低的硬件。
* 缺点：
	* 机器码占用的存储空间更大，字节码变为机器码之后，可能会增加`10%-20`（不过在应用包中，可执行的代码常常只是一部分。比如最新的 Google+ APK 是 28.3 MB，但是代码只有 6.9 MB。）
	* 应用的安装时间会变长。

### class、dex、odex、ELF相爱相杀
文件 | 虚拟机 | 生成工具
:---: | :---: | :---:
class | JVM | javac
dex | Dalvik | dx
oat | ART | dex2oat(安装时预处理成可直接执行的oat文件)

### AOT(Ahead-of-time)

`ART` 推出了预先 (`AOT`) 编译，可提高应用的性能。`ART` 还具有比 `Dalvik` 更严格的安装时验证。在安装时，`ART` 使用设备自带的 `dex2oat` 工具来编译应用。该实用工具接受 [DEX](http://source.android.com/devices/tech/dalvik/dex-format.html) 文件作为输入，并针对目标设备生成已编译应用的可执行文件。之后打开 `App` 的时候，不需要额外的翻译工作，直接使用本地机器码运行，因此运行速度提高。

### 混合运行时(Android N 24)

从`Android N`(`API 24`)包含了一个混合模式的运行时。应用在安装时不做编译，而是解释字节码，所以可以快速启动。`ART`中有一种新的、更快的解释器，通过一种新的 JIT 完成，但是这种 `JIT` 的信息不是持久化的。取而代之的是，代码在执行期间被分析，分析结果保存起来。然后，当设备空转和充电的时候，ART 会执行针对“热代码”进行的基于分析的编译，其他代码不做编译。为了得到更优的代码，`ART` 采用了几种技巧包括深度内联。

对同一个应用可以编译数次，或者找到变“热”的代码路径或者对已经编译的代码进行新的优化，这取决于分析器在随后的执行中的分析数据。这个步骤仍被简称为 `AOT`，可以理解为“全时段的编译”（`All-Of-the-Time compilation`）。

这种混合使用 AOT、解释、JIT 的策略的全部优点如下。

- 即使是大应用，安装时间也能缩短到几秒;
- 系统升级能更快地安装，因为不再需要优化这一步;
- 应用的内存占用更小，有些情况下可以降低 50%;
- 改善了性能;
- 更低的电池消耗;

### odex(optimized dex)
因为 `apk` 实际为 `zip` 压缩包，虚拟机每次加载都需要从 `apk` 中读取`classes.dex` 文件，这样会耗费很多的时间，而如果采用了 `odex` 方式优化的 `dex` 文件，他包含了加载 `dex` 必须的依赖库文件列表，只需要直接加载而不需要再去解析。

在 `Android N` (API 24)之前，对于在 `dalvik` 环境中 使用 `dexopt` 来对 `dex` 字节码进行优化生成 `odex` 文件最终存在手机的 `data/dalvik-cache` 目录下，最后把 `apk` 文件中的 `dex` 文件删除。

在 `Android O` 之后，`odex` 是从 `vdex` 这个文件中 提取了部分模块生成的一个新的**可执行二进制码**文件 ， `odex` 从 `vdex` 中提取后，`vdex` 的大小就减少了。

- 第一次开机就会生成在 `/system/app/<packagename>/oat/` 下
- 在系统运行过程中，虚拟机将其 从 `/system/app` 下 `copy` 到 `/data/davilk-cache/`下;
- `odex + vdex = apk` 的全部源码 （`vdex` 并不是独立于 `odex` 的文件 `odex + vdex` 才代表一个 `apk` ）;

https://dandanlove.blog.csdn.net/article/details/102620708