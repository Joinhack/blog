pack
=====================================
Archive 工具

archive文件是放在$GOPATH/pkg下的文件， 这些文件是通过(go tool [6c/8c/5c] [6g/8g/5g] [6a/8a/5a])编译生成的对象文件， 使用go tool pack 归档生成的。


pack例子:

$ go tool pack p [archive]

打印详细内容

$ go tool pack t [archive]

打印条目


$ go tool pack x [archive]

提取




