（本笔记基于  opencv  4.8.0  版本）


opencv拥有模块化的结构，下面是可用的模块：

Core functionality(core)  定义基本数据结构的模块，包括了多维数组  Mat  和被其他模块使用的基本函数  
Image Processing(imgproc)  一个图像处理模块，包括线性和非线性图像滤波、几何图像转换(调整大小、仿射和透视扭曲、通用的基于表的重新映射)、颜色空间转换、直方图等。  
Video Analysis(video)  一个视频分析模块，包括运动估计，背景减去，和目标跟踪算法。  
Camera Calibration and 3D Reconstruction(calib3d)  基本的多视图几何算法，单个和立体相机校准，目标姿态估计，立体对应算法，以及三维重建的要素。  
2D Features Framework(features2d)  显著特征检测器、描述符和描述符匹配器。  
Object Detection(objdetect)  检测对象和预定义类的实例(例如，面孔、眼睛、杯子、人、汽车等)。  
High-level GUI(highgui)   一个易于使用的界面，简单的UI功能  
Video I/O(videodio)  an easy-to-use interface to video capturing and video codecs.  


# API概念
## cv  命名空间（namespace）

所有的  opencv  的类和函数都被放在  cv  命名空间中。为了在代码中访问，请使用

`cv::`
`using namespace cv;`

注意：现在或将来的一些opencv函数名可能会与  STL  或其他库的同名函数冲突，在这种情况下，使用显式的命名空间限定符来解决名称冲突。  
（这里也说明了尽量不要引用整个命名空间，而是只引用需要的函数，保持引用的头文件是必要且不可删除的，减少不必要头文件的引用，可减少命名冲突的发生）

## 自动内存管理

OpenCV自动处理所有内存。

首先，std::vector、cv::Mat以及函数和方法使用的其他数据结构都有析构函数，可以在需要时释放底层内存缓冲区。  
这意味着析构函数并不总是像Mat那样释放缓冲区。它们考虑到可能的数据共享。析构函数减少与矩阵数据缓冲区相关联的引用计数器。  
当且仅当引用计数器达到零时，即没有其他结构引用相同的缓冲区时，缓冲区被释放。类似地，当复制Mat实例时，没有真正复制实际数据。  
相反，引用计数器是递增的，以记住同一数据的另一个所有者。还有cv::Mat::clone方法，用于创建矩阵数据的完整副本。

## 输出数据的自动分配

OpenCV自动释放内存，并在大多数情况下自动为输出函数参数分配内存。  
因此，如果一个函数有一个或多个输入数组(cv::Mat实例)和一些输出数组，输出数组将被自动分配或重新分配。  
输出数组的大小和类型由输入数组的大小和类型决定。如果需要，这些函数会接受额外的参数来帮助确定输出数组的属性。

## 饱和算法

作为一个计算机视觉库，OpenCV处理大量的图像像素，这些图像像素通常以紧凑的、每通道8位或16位的形式编码，因此具有有限的值范围。  
此外，对图像的某些操作，如色彩空间转换，亮度/对比度调整，锐化，复杂插值(双立方，Lanczos)可以产生超出可用范围的值。  
如果您只存储结果的最低8(16)位，这将导致视觉伪影，并可能影响进一步的图像分析。为了解决这个问题，使用了所谓的饱和算法。  
例如，要将一个操作的结果r存储到一个8位的图像中，您需要在0..255范围:

`I(x,y)=min(max(round(r),0),255)`

类似的规则适用于8位有符号、16位有符号和无符号类型。这种语义在库中无处不在。在c++代码中，使用cv::saturate_cast<>函数完成，类似于标准c++强制转换操作。  
上述公式的执行情况如下:

`I.at<uchar>(y, x) = saturate_cast<uchar>(r);`

其中cv::uchar是OpenCV 8位无符号整数类型。在优化的SIMD代码中，使用了诸如paddusb、packuswb等SSE2指令。它们有助于实现与c++代码完全相同的行为。

注意：当结果为32位整数时，不使用饱和算法。


## 固定的像素类型。模板的有限使用

模板是c++的一个重要特性，它支持实现非常强大、高效且安全的数据结构和算法。然而，模板的广泛使用可能会极大地增加编译时间和代码大小。
此外，当只使用模板时，很难将接口和实现分开。这对于基本算法来说可能没问题，但对于计算机视觉库来说就不太好了，因为一个算法可能跨越数千行代码。
正因为如此，也为了简化其他语言(如Python、Java、Matlab)的绑定开发，这些语言根本没有模板或模板功能有限，目前的OpenCV实现是基于模板上的多态性和运行时调度的。
在那些运行时调度太慢(如像素访问操作符)、不可能(泛型cv::Ptr<>实现)或非常不方便(cv::saturate_cast<>())的地方，当前的实现引入了小型模板类、方法和函数。
在当前OpenCV版本的其他地方，模板的使用是有限的。

因此，标准库可以操作的基本数据类型的固定集合是有限的。也就是说，数组元素应该具有以下类型之一:

* 8-bit unsigned integer (uchar)
* 8-bit signed integer (schar)
* 16-bit unsigned integer (ushort)
* 16-bit signed integer (short)
* 32-bit signed integer (int)
* 32-bit floating-point number (float)
* 64-bit floating-point number (double)
* 由几个元素组成的元组，其中所有元素都具有相同的类型(上述之一)。元素为元组的数组称为多通道数组，与元素为标量值的单通道数组相反。
  通道的最大可能数量由CV_CN_MAX常量定义，目前设置为512。

对于这些基本类型，使用以下枚举:

`enum { CV_8U=0, CV_8S=1, CV_16U=2, CV_16S=3, CV_32S=4, CV_32F=5, CV_64F=6 };`

多通道(n通道)类型可以使用以下选项指定:

* CV_8UC1 ... CV_64FC4常量(用于从1到4的多个通道)
* CV_8UC(n) ... CV_64FC(n)或CV_MAKETYPE(CV_8U, n) ... CV_MAKETYPE(CV_64F, n)宏，当编译时通道数大于4或未知时。

具有更复杂元素的数组不能使用OpenCV构造或处理。此外，每个函数或方法只能处理所有可能数组类型的一个子集。
通常，算法越复杂，支持的格式子集就越小。请看下面这类限制的典型例子:

* 人脸检测算法只适用于8位灰度或彩色图像。
* 线性代数函数和大多数机器学习算法只能处理浮点数组。
* cv::add等基本函数支持所有类型。
* 色彩空间转换函数支持8位无符号、16位无符号和32位浮点类型。

每个函数支持的类型子集都是根据实际需要定义的，将来可以根据用户请求进行扩展。


## 输入数组和输出数组

暂时看不懂.......


## 错误处理

OpenCV使用异常来表示严重错误。
当输入的数据具有正确的格式并且属于指定的值范围，但是算法由于某些原因(例如，优化算法没有收敛)而不能成功时，它返回一个特殊的错误代码(通常只是一个布尔变量)。  
异常可以是cv::Exception类或其派生类的实例。反过来，cv::Exception是std:: Exception的派生。因此，可以使用其他标准c++库组件在代码中优雅地处理它。  
异常通常是通过使用CV_Error(errcode, description)宏或其类似printf的CV_Error_(errcode， (printf-spec, printf-args))变体抛出的，  
或者使用CV_Assert(condition)宏来检查条件并在不满足条件时抛出异常。对于性能关键型代码，CV_DbgAssert(条件)只保留在Debug配置中。  
由于自动内存管理，所有中间缓冲区在突然错误的情况下自动回收。如果需要的话，你只需要添加一个try语句来捕获异常:  

	try
	{
 		... // call OpenCV
	}
	catch (const cv::Exception& e)
	{
 		const char* err_msg = e.what();
 		std::cout << "exception caught: " << err_msg << std::endl;
	}


## Multi-threading and Re-enterability

当前的OpenCV实现是完全可重新进入的。也就是说，可以从不同的线程调用不同类实例的相同函数或相同方法。
同样，同一个Mat可以在不同的线程中使用，因为引用计数操作使用特定于体系结构的原子指令。

关于这一部分可阅读以下文档及其中文版本：

[Writing reentrant and threadsafe code - IBM Documentation](https://www.ibm.com/docs/en/aix/7.3?topic=programming-writing-reentrant-threadsafe-code)

[编写重入和线程安全代码 - IBM 文档](https://www.ibm.com/docs/zh/aix/7.3?topic=programming-writing-reentrant-threadsafe-code)

这一部分属于进阶内容，暂时不需要掌握。

注：文档中提到的  AIX  是  IBM  开发的  UNIX  操作系统

