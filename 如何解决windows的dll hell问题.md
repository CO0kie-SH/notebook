# 什么是dll hell问题 ？
首先看一下stackoverflow上给出的解释：
> It's when Application A installs a Shared DLL v1.0, Application B comes and updates the Shared DLL to v1.1 which should be compatible but there are slightly different behaviors, then App A stops working correctly and reinstalls v1.0 then App B stops working ... now imagine this with more than 2 apps let's say a dozen: DLL Hell.

言简意赅的说，有两个名字相同但实现内容不同的dll，程序A依赖第一个dll。如果因为某些原因，程序A在调用dll时调用了第二个dll，那么程序A就有可能会crash，停止工作。这就是dll hell。

1. https://www.wikiwand.com/en/DLL_Hell
2. http://stackoverflow.com/questions/1379287/i-keep-hearing-about-dll-hell-what-is-this

# windows搜索dll路径的顺序
要解决dll hell问题，首先要了解windows搜索dll路径的顺序。
1. The directory from which the application loaded.
2. The system directory. Use the GetSystemDirectory function to get the path of this directory.
3. The 16-bit system directory. There is no function that obtains the path of this directory, but it is searched.
4. The Windows directory. Use the GetWindowsDirectory function to get the path of this directory.
5. The current directory.
6. The directories that are listed in the PATH environment variable. Note that this does not include the per-application path specified by the App Paths registry key. The App Paths key is not used when computing the DLL search path.

**即system directory和windows directory会先于path包含的路径被搜索**。所以如果花费很多时间仍未发现自己程序错误时，就需要考虑是否有系统目录下的dll隐藏了自己需要的dll。

当然用户也可以通过SafeDllSearchMode和LoadLibaryEx来修改搜索路径，详细资料请查看[2](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682586(v=vs.85).aspx).

1. http://stackoverflow.com/a/6546427/692223
2. https://msdn.microsoft.com/en-us/library/windows/desktop/ms682586(v=vs.85).aspx

# 解决思路
当我们了解windows搜索dll路径的顺序之后，就可以进入正题：“如何解决dll hell问题”。

工欲善其事必先利其器，这里我们借助[depends walker](http://www.dependencywalker.com/)和[search everything](https://www.voidtools.com/)解决dll hell问题。
1. 运行目标程序，目标程序crash后会弹出crash界面，点击more details可以得到default module。
2. 运行depends walker查找对应dll，查看是否缺失该dll，或者在64位程序里面调用了32bit的dll。
3. 运行search everything查找该dll，查找是否有多个dll。如有多个，则对应depends walker查看是否存在A模块的dll依赖了B模块的dll。如，qt的dll依赖matlab的dll。

基本上这些方法就能解决大多数dll hell的问题了 :).
