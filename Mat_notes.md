（基于opencv 4.8.0 版本）  

opencv最初采用 c 接口，由于需要手动管理内存，大型程序很容易出问题，因此改用了 c++ 接口  
c++ 接口的主要缺点是许多的嵌入式平台目前只支持c，移植到这些平台存在麻烦  
Mat 是一个类，包含两个数据部分：矩阵头（包含的信息例如：矩阵大小，存储方法，矩阵存储在哪个地址上，等等）  
和一个指向矩阵的指针。矩阵头的大小是常数，矩阵本身的大小会随图片而变化。  
在向 opencv 中的函数传递图片时，不必要的复制会拖慢程序的运行速度，因此 opencv 采用了**引用计数系统**  
只会复制矩阵头和指向矩阵的指针而非矩阵本身  

	Mat A, C; // creates just the header parts
	A = imread(argv[1], IMREAD_COLOR); // here we'll know the method used (allocate matrix)
	Mat B(A); // Use the copy constructor
	C = A; // Assignment operator
 
上面的所有对象最终都指向同一个图片矩阵，对其中的任何一个进行修改都会影响其他对象。
如果一个矩阵属于多个 Mat 对象，当矩阵不再需要时，谁负责清理工作？答案是最后一个使用它的对象。  
这一部分被引用计数机制处理。如果想要复制矩阵本身，使用 `cv::Mat::clone()`或`cv::Mat::copyTo()`函数  

	Mat F = A.clone();
	Mat G;
	A.copyTo(G);

 有许多不同的颜色系统，每个都有各自的优点：
 * RGB 是最常用的，但是 opencv 标准使用 BGR 色彩空间
 * HSV 和 HLS 把颜色分解为hue（颜色）、saturation（饱和度）和 value/luminance，忽略最后一个组分
   将使算法对输入图片的光照情况更不敏感。
 * YCrCb 是 JPEG 图像格式的颜色系统
 * CIE L*a*b*

