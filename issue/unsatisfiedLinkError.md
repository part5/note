* 应用的私有`Native library`目录`/data/data/packagename/lib`是一个符号链接。链接到`/data/app/<package name>/lib`目录，System.loadLibrary 是到这个目录去尝试加载 so 的

![](/Users/guoxiaodong/Documents/note/pic/unsatisfiedLinkError_data.png)
![](/Users/guoxiaodong/Documents/note/pic/unsatisfiedLinkError_app.png)

* 