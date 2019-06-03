---
title: Vulkan
mathjax: true
date: 2019-05-28 21:36:35
tags:
    - Vulkan
    - 计算机图形学
categories: Vulkan
---
以下是Vulkan中的特性和改进，相较于OpenGL具有更多的优势：
- 降低驱动程序的开销以及CPU使用率：Vulkan旨在更接近底层的图形硬件。因此，它为上层的应用程序提供了对主机计算资源的直接控制，以便GPU尽可能地进行渲染，这种方式同时也允许软件直接访问图形处理器，从而获得更好地性能。
- 多线程可扩展性：OpenGL中的多线程扩展效果非常差，要想利用线程的特性，从而更好地利用CPU是一件非常困难地事情。然后，Vulkan对此进行了专门设计，用于允许终端用户以一种非常透明地方式利用CPU的多线程特性，不存在隐式的全局状态。从创建作业以及提交作业（用于执行）时开始，不同线程下的作业之间会保持分离。
- 一套“显示”的API：OpenGL是一套“隐式”的API，其中的资源管理是驱动程序的责任。驱动程序需要应用程序的提示并跟踪资源的状态，这是一种不必要的开销。Vulkan是一套“显式”的API；在这里，驱动程序不负责跟踪资源以及它们之间的关系，把这项任务分配给了应用程序。这种干净的方法更可预测；驱动程序不会再幕后执行某些操作来管理资源（就像在OpenGL中一样）。因此，作业的处理简化且直接，从而实现最佳性能和可预测的行为。
- 预编译的中间着色语言：与需要着色器shader作为OpenGL(GLSL)源代码提供的OpenGL不同，SPIR-V(Standard Portable Intermediate Language：标准可移植中间语言)是Vulkan用于并行计算和图形操作的标准中间语言。
>注意：用于源语言的编译器，例如GLSL，HLSL和LLVM必须符合SPIR-V规范，并提供实用的工具程序来提供SPIR-V的输入。Vulkan采用Vulkan这种即时执行的二进制中间输入形式并会在着色器阶段使用。
- 驱动程序层和应用程序层：在OpenGL中，与驱动程序层相比，应用程序层更薄，因为驱动程序的自动化考虑了资源管理和状态跟踪，Vulkan正好与此相反。它会确保驱动程序更接近底层的硬件且开销更小。管理逻辑、资源和状态是应用程序的责任。下图显示了这两个API的驱动程序和应用程序代码库的厚度：

{% asset_img 1.jpg %}
- 内存控制：Vulkan能够在系统上暴露若干种内存类型，并要求应用程序开发人员为每个资源的预期用途选择适当的内存类型。相比之下，OpenGL驱动程序则会根据内部启发模式决定资源的放置位置，不同供应商之间的启发模式存在一定的差异，如果稍后驱动程序移动了资源，OpenGL可能会产生次优的放置或出现意外的故障。
- 可预测性：与OpenGL相比，Vulakn具有高度的可预测性；它在渲染时不会导致任何滞后或挂起。一旦将作业提供给驱动程序，就会立即提交，而OpenGL作业提交不是预先提供的，而是受到驱动调度程序的支配。
- 一套API：OpenGL为桌面API(OpenGL)和嵌入式API(OpenGL ES)提供了不同的版本。Vulkan很干净，只有一套适用所有平台的API。
- 直接访问GPU：Vulkan通过公开其功能和硬件设施，为应用程序用户提供了很多控制权。它公开了各种可用的物理设备、内存类型、命令缓冲区以及扩展。这种行为可以确保软件层更接近真实的硬件。
- 错误检查和验证：当使用OpenGL时，运行良好的应用程序在检查错误会付出一些代价，而错误在执行的时候根本不会出发。相比之下，Vulkan将这些检查和验证作为附加服务来提供，可以在需要时启用合计禁用。这些检查是可选的，可以通过启用错误检查和其他的验证层注入到运行时。因此，通过避免不必要的检查，可以减少CPU开销。理想情况下，这些错误和验证层必须在开发阶段的调试器打开，并在发布期间关闭。
- 支持各种GPU硬件：Vulkan支持移动设备光栅化器和桌面光栅化器作为实现的集成部分。它支持嵌入式平台的基于瓦片或延期的光栅化器以及本地基于平铺的前反馈光栅化器。

在深入探讨基本细节之前，先来看看Vulkan中用到的一些比较重要技术术语。随着我们的进一步深入，本书还会覆盖更多的技术术语。

- 物理设备和设备：系统可能包含多个具有Vulkan功能的物理硬件设备。物理设备表示唯一的设备，而设备device则是指应用程序中物理设备的逻辑表示，即逻辑设备。
- 队列：队列表示执行引擎和应用程序之间的口。物理设备始终包含了一个或多个队列（图形队列、计算队列、DMA队列/传输队列等）。队列的职责是收集作业（命令缓冲区）并将其分派给物理设备进行处理。
- 内存类型：Vulkan公开了各种内存类型。在更广泛的层面上，有两种类型的内存：主机内存和设备内存。
- 命令：命令是做某种行为的指令。命令可以大致分为动作，设置状态或者同步。
- 动作命令：这些命令可用于绘制图元、清除表面、复制缓冲区，查询/时间戳操作以及开始/结束子通道操作。这些命令能够更改帧缓冲区附件，读取或写入内存（缓冲区或图像）以及编写查询池。
- 同步命令：同步有助于满足两个或多个操作命令的要求，这些操作命令可能会争夺资源或具有一些内存依赖性。其中包括设置事件、等待时间，插入管线屏障以及渲染通道或子通道的依赖关系。
- 命令缓冲区：命令缓冲区是一组命令；它会记录这些命令并将它们提交给队列。

# Vulkan的执行模型
具有Vulkan功能的系统能够查询并显示系统上可用的物理设备的数量。每个物理设备会暴露一个或多个队列。这个队列会分为不同的族，每个族都有特定的功能。例如，这些功能包括图形，计算，数据传输以及稀疏内存管理。队列族的每个成员都可以包含一个或多个类似的队列，从而使它们互相兼容。例如，给定的实现可能支持同一队列上的数据传输和图形操作。

Vulkan允许开发人员通过应用程序对内存控制进行显式的管理，它公开了设备上可用的各种类型的堆，其中的每个堆属于不同的内存区域。Vulkan的执行模式非常直接，此处，命令缓冲区会被提交到队列中实现的。然后由物理设备使用，以便进行各种处理。

Vulkan应用程序负责控制各种Vulkan设备，这是通过把大量的命令记录到命令缓冲区并将它们提交到队列中实现的。该队列由驱动程序读取，驱动程序会按提交的顺序预先执行作业。命令缓冲区的构建非常昂贵，因此，一旦构建完成，就可以对它进行缓存以及将其提交到队列中，以便根据具体的需求执行若干次。此外，在应用程序中，可以使用多线程同时并行构建多个命令缓冲区。
下图显示了执行模型的简化图示：

{% asset_img 2.jpg %}

在这里，应用程序记录了两个包含多个命令的命令缓冲区。然后根据工作性质将这些命令提供给一个或多个队列。队列将这些命令缓冲区做呕也提交到设备进行处理。最后，设备处理结构并将其显示在输出显示屏上，或将它们返回给应用程序进行一部的处理。

在Vulkan中，应用程序负责以下内容：
- 为成功执行命令提供所有必要的先决条件：这其中可能包括准备资源、预编译的着色器以及将资源附加到着色器、指定渲染状态、构建管线以及绘制调用。
- 内存管理
- 同步
- 主机和设备之间的同步
- 设备上可用的不同队列之间的同步
- 危害管理

# Vulkan的队列

队列是Vulkan中的媒介，就是通过它将命令缓冲区送入设备的。命令缓冲区会记录一个或多个命令并将它们提交到所需的队列。设备也可能会公开多种队列，因此，应用程序有责任将命令缓冲区提交给正确的队列。

可以将命令缓冲区提交到以下几项：
- 一个队列----命令缓冲区的提交顺序以及执行、回放都将保持不变，命令缓冲区以串行方式执行；
- 多个队列----允许在两个或多个队列中并行执行命令缓冲区。除非明确指定，否则无法保证命令缓冲区的提交和执行顺序，同步它们的顺序是应用程序的责任，如果没有进行同步，执行的顺序可能会完全超出预期。

Vulkan提供了几种同步方式，是程序可以在单个队列或跨越多个队列对作业的执行进行相对控制。
这些同步方式有：
- **信号量(Semaphore)**：该同步机制可以跨多个队列进行同步，火灾单个队列中同步粗粒度的命令缓冲区提交。
- **事件(Events)**：事件用来控制细粒度的同步并且被应用于单个队列，允许我们对提交给单个队列的一个命令缓冲区或若干个命令缓冲区序列之间进行同步工作。宿主机也可以参与基于事件的同步。
- **栅栏(Fences)**：该方式能够在主机和设备之间进行同步操作。
- **管线屏障(Pipeline barriers)**：管线屏障是一个插入指令，用于确保在该命令缓冲区中，在管线屏障之前的命令必须在被指定的、管线屏障之后的命令之前执行。

# 对象模型

在应用程序级别，所有的实体（包括设备、队列、命令缓冲区、帧缓冲区、管线等）都被称为Vulkan对象。在内部，在API级别，这些Vulkan对象用句柄进行识别。这些句柄有两种类型：可分发句柄和不可分发句柄。
- **可分发句柄(a dispatchable handle)**：这是一个指针，指向了内部不透明形状的实体(opaque-shaped entity)。不透明的类型不允许您直接访问这种结构的字段。只能使用API例程访问这些字段。每个可分发句柄都有一个关联的可分发类型(dispatchable type)，用于在API命令中作为参数传递。下面是一些示例：
`VKInstance|VkCommandBuffer|VkPhysicalDevice|VkDevice|VkQueue`
- **不可分发句柄(Non-dispatchable handles)**：这些是64位整型类型的句柄，可以包含对象信息本身，而不是指向结构的指针。示例如下：
`VKSemaphore|VkFence|VkQueryPool|VkBufferView`
`VkDeviceMemory|VkBuffer|VkImage|VkPipeline`
`VkShaderModule|VkSampler|VkRenderPass|VkDescriptorPool`

# 对象生命周期和命令语法
在Vulkan中，根据每个应用程序的逻辑，都要显示创建和销毁对象，并且应用程序要负责管理这些对象。
Vulkan中的对象使用Create创建并使用Destroy命令销毁：
- **创建语法**：对象使用`vkCreate *`形式的命令创建的；这类命令接受一个`Vk *CreateInfo`结构作为输入参数。
- **销毁语法**：对应的，使用`vkCreate`命令生成的对象要使用`vkDestroy`进行销毁。
作为现有对象池或堆的一部分而创建的对象要使用`Allocate`命令创建并使用`Free`命令从池或者堆中进行释放。
- **分配语法**：作为对象池的一部分创建的对象使用`vkAllocate *`形式的命令，并且使用`Vk* AllocateInfo`作为输入参数。
- **销毁语法**：相应的，使用`vkFree*`命令把对象释放回池或者内存中。

使用`vkGet`命令可以轻松访问任何给定的实现信息。`vkCmd`形式的API实现用于在命令缓冲区中记录命令。

# 错误检查和验证

Vulkan专门提供高性能而设计，这是通过保持错误检查和验证功能作为一种可选项的形式实现的。在运行时，错误检查和验证的部分少之又少，从而使得构建命令缓冲区和提交变得更加高效。这些可选功能可以通过Vulkan的分层体系结构来实现，该分层体系结构允许把各种层（调试层和验证层）动态注入到正在运行的系统中。

# 了解 Vulkan应用

以下框图显示了系统中不同组件模块以及对应的联系：

{% asset_img 3.jpg %}

# 驱动程序
具有Vulkan功能的系统至少包含一个CPU和GPU。IHV的供应商为其专用GPU架构提供了指定Vulkan规范实现的驱动程序。驱动程序充当应用程序设备本身之间的接口。它为应用程序提供了一些高级设施，以便能够与设备进行通信。例如，驱动程序通知了系统上可用的设备数量、它们的队列和队列功能、可用的堆及其相关属性等。

# 应用程序
应用程序是指用户编写的程序，旨在利用Vulkan API来执行图形或计算任务。应用程序从硬件和软件的初始化开始，它会检测驱动程序并加载所有的Vulkan API。展示层(presentation layer)使用Vulkan的窗口系统集成----Window System Integration----(WSI)API进行初始化；WSI将用于渲染期间在显示表面(display surface)上绘制图形图像。应用程序创建资源并使用描述符(descripors)将它们绑定到着色器阶段(shader stage)。描述符集布局(descriptor set layout)用于把创建的资源绑定到创建的底层管线对象pipeline object(图形或计算类型graphics or compute type)。最后，记录命令缓冲区并将其提交到队列进行处理。
# WSI
窗口系统集成WSI(Windows System Integration)是来自Khronos的一组扩展，用于跨平台(如Linux,Windows和Android)。
# SPIR-V
SPIR-V提供了一种预编译的二进制格式，用于指定Vulkan着色器。编译器可用于各种着色器语言，其中包括能够生成SPIR-V的GLSL和HLSL变种。
# LunarG SDK
LunarG的Vulkan SDK包含了各种工具和资源，用以辅助Vulkan应用程序的开发。这些工具和资源包括Vulkan加载程序、验证层、跟踪和回放工具、SPIR-V工具、Vulkan运行时安装程序、文档，示例以及演示。
# Vulkan编程模型入门

下图显示了Vulkan应用程序编程模型自顶向下的方法；
{%asset_img 4.jpg%}

# 硬件初始化
当Vulkan应用程序启动的时候，它的第一个工作就是硬件的初始化。在这个阶段，应用程序通过与加载器进行通信来激活Vulkan驱动程序。下图展示了一个加载器Loader及其子组件的框图：

{% asset_img 5.jpg %}

**Loader**：加载程序是应用启动时使用的一段代码，可以跨平台、以一种统一的方式在系统中定位Vulkan驱动程序。以下是加载Loader的职责：

- **定位驱动程序(Locating dirvers)**：作为其主要的任务，加载程序知道在给定系统中到哪里搜索驱动程序。它会找到正确的驱动程序并加载。
- **不依赖平台(Platform-independent)**：初始化Vulkan在所有平台上都是一致的。这与OpenGL不同，OpenGL创建上下文需要针对每个环境(EGL,GLX,WGL)使用不同的窗口系统API。Vulkan中的平台差异以扩展名表示。
- 可注入的层(Injectable layers)：加载器支持层次化的体系结构并提供在运行时注入各层的能力。最大的改进就是驱动程序在确定应用程序堆API的使用是否有效时不需要执行任何工作，也不会保留执行该工作所需的任何状态。因此，建议在开发阶段根据应用要求打开选定的可注入层，并在部署阶段将其关闭。例如，可注入层可以以下内容：
   - 跟踪Vulkan API命令
   - 捕获要渲染的场景并在稍后执行场景的渲染
   - 用于调试目的的错误检查和验证

Vulkan应用程序首先执行与加载程序库的握手并初始化Vulkan具体实现的驱动程序。加载程序库会动态加载Vulkan API。加载程序还提供了一种机制，允许将特定的层自动加载到所有Vulkan应用程序中；这被称为隐式启用层。

一旦加载程序找到驱动程序并成功链接到API，应用程序就要复杂以下操作：
- 创建一个Vulkan实例
- 查询物理设备的可用队列
- 查询扩展并将它们存储位函数指针，如WSI或特殊功能的API
- 启用可注入曾，进行错误检查、调试或验证操作

# 窗口展示表面(Window presentation surfaces)
一旦加载器找到Vulkan具体实现的驱动程序，