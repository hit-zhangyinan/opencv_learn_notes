**在github的markdown编辑器中，如果想插入代码块，必须在代码块的前后都插入空行**<br>

避免空指针错误：不要向零地址写入数据  

避免野指针错误：<br>
任何指针变量刚被创建时不会自动成为NULL指针，它的缺省值是随机的，它会乱指一气。
所以，指针变量在创建的同时应当被初始化，要么将指针设置为NULL，要么让它指向合法的内存。<br>

	char *p = NULL;
	char *str = (char *) malloc(100);

free和delete只是把指针所指的内存给释放掉，但并没有把指针本身干掉。<br>
free以后其地址仍然不变（非NULL），只是该地址对应的内存是垃圾，p成了“野指针”。<br>
如果此时不把p设置为NULL，会让人误以为p是个合法的指针。<br>

会导致出错的代码：<br>

	char *p = (char *) malloc(100);
	strcpy(p, "hello");
	free(p);   // p 所指的内存被释放，但是p所指的地址仍然不变
	...
	if(p != NULL)      // 没有起到防错作用
	{
		strcpy(p, "world");      // 出错
	}

因此在使用堆区内存时，要注意遵循以下步骤：<br>
1. 使用malloc开辟一段指定大小的内存，并将内存的首地址返回给声明的指针变量，这样完成了指针变量和堆内存的绑定（一行代码做了两件事）<br>
2. 使用free释放堆区开辟的内存空间，此时该段内存已经不可访问，但指针变量仍保存内存空间的首地址，变成了指向非法地址的指针<br>
3. 将指针变量赋值为空指针（系统和编译器决定了空指针不可访问）<br>

代码段表示：<br>

	xxx = malloc xxx
	free(xxx)
	xxx = NULL

（三部曲：malloc、free、NULL）（new、delete、nullptr）<br>

如果先将指针变量置为空，就再也找不到开辟的那块内存了，也就无法释放，造成内存泄漏<br>

因此上面三条语句的顺序是定死的，不能修改

[NULL指针、零指针、野指针](https://www.cnblogs.com/fly1988happy/archive/2012/04/16/2452021.html)  
[Linux下segmentation fault总结](https://zhuanlan.zhihu.com/p/205579221)  
[Segmentation Fault的产生原因及调试方法](https://www.cnblogs.com/linux-37ge/p/12781176.html)
