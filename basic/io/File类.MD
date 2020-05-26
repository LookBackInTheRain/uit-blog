
# File 类
在学习I/O流之前先了解下`File`类的使用，在接下来的例子中我们I/O操作都是基于文件数据进行的. `File` 一个文件或文件集的抽象.

# 常用方法说明

方法名 | 返回值 |说明
---|---|---
File(String pathname) |- | 构造方法
File(String parent, String child) |-|构造方法 parent 表示父路径是一个目录，child 子路径表示目录或文件名
File(File parent, String child) |-|构造方法  parent 表示父路径是一个目录，child 子路径表示目录或文件名
File(URI uri) |-|构造方法:new File(new URI("file:///User/yuit/a.json"))
createNewFile() | boolean |创建文件，如果文件不存在并创建成功返回true、否则返回false
mkdir() | boolean | 文件创建成功返回true |
getName() | String | 获取文件名
getAbsolutePath() | String | 返回此抽象路径名的绝对路径名字符串
canRead() | boolean | 是否可读
canWrite() | boolean | 是否可写
exists() | boolean | 是否存在
isDirectory() | boolean | 是否是文件夹
isFile() | boolean | 是否为普通文件
isHidden() | boolean | 是否为隐藏文件
lastModified() | long | 最后修改时间 单位毫秒
length() | long | 文件长度 单位字节
delete() | boolean | 直接删除文件
deleteOnExit() | - | 在java虚拟机结束运行时删除文件
list(FilenameFilter filter) | String [] | 过滤获取当前文件夹下的文件 `FilenameFilter` 中 boolean accept(File dir, String name) 方法实现过滤
listFiles() | File[] | 获取当前文件夹下的文件对象数组
listFiles(FilenameFilter filter) | File[] | 过滤获取文件对象数组
listFiles(FileFilter filter) | File[] | 过滤获取当前文件夹下的文件对象数组


