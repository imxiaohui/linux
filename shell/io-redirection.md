# I/O重定向

这是ABS Guide第20章的内容，我翻译出来。

Unuix/Linux系统有三个文件是默认被打开着的，它们的分别是stdin(标准输入文件，键盘)，stdout（标准输出文件，屏幕）和stderr（标准错误文件，屏幕）。这些文件和其他打开的文件都可以被重定向。I/O重定向简单的说就是用一个脚本来捕获一个文件、命令、程序、脚本甚至是代码块的输出，然后将该输出写入到另一个文件、命令、程序或脚本。

每个打开的文件都被赋予一个文件描述符。0,1,2分别是标准输入、标准输出、标准错误的文件描述符。

## 标准输出 > (COMMAND_OUTPUT >)

重定向标准输出到一个文件，如果这个文件不存在则创建，存在则覆盖。

```
ls -lR > dir-tree.list
```

将当前目录的目录树写入文件dir-tree.list

```
: > filename
```

删除filename里的所有内容（即将filename置空）。如果文件不存在则创建一个空文件（与touch filename命令效果相同）。

```
> filename
```

与上条命令意思相同，但是该命令在某些shell中不可用。

## 标准输出 >> (COMMAND_OUTPUT >>)

重定标准输出到一个文件，如果该文件不存在则创建，存在则追加到该文件末尾。

只对单行重定向命令起作用（即只对命令所在的行起作用）

```
1 >> filename
```

重定向并追加标准输出到filename

```
2 >> filename
```

重定向前追加标准错误输出到filename

```
& > filename
```

重定向标准输出和标准错误输出到filename，该命令可在Bash 4下使用。

```
M > N
```

"M" 是一个文件描述符，默认为1；"N"是一个文件名。意思是将M的内容重定向到N

```
M>&N
```

"M"是一个文件描述符，默认为1；"N"是另一个文件描述符。意思是将文件描述符M重定向到N，即所有M指向的文件输出都会发给N所指向的文件。

```
2>&1
```

将标准错误输出重定向到标准输出，一般用法如下：

```
>> filename 2>&1 
```

意思是将标准错误输出重定向到标准输出，再将标准输出重定向到文件filename

```
>&j
```

等价于 1>&j ，即将标准输出定向到j所指向的文件

```
0<filename 或 <filename
```

以文件filename内容为标准输入。

```
grep search-word < filename
[j]<>filename
```

以读写方式打开filename，并且将j作为文件描述符赋值给filename，如果j没有被指定，默认为0，即标准输入。

下面的例子会让你更清楚理解其含义

```
echo "1234567890" > file # 将字符串写入file
exec 3<>file # 打开file并且将其文件描述符设置成3
read -n 4 <&3 # 仅读取4个字符
echo -n . >&3 # 写入一个英文句号
exec 3>&- # 关于文件描述符3
cat file # 结果会是：1234.567890
```

```
|
```

管道，用来定向命令的，与">"相似，但是更实用。不细说了。

```
ls -yz >> command.log 2>&1
```

捕获非法选项yz的标准错误输出及标准输出，并写入command.log文件。

```
ls -yz 2>&1 >> command.log
```

输出错误信息，但是不写入文件command.log。这一行命令的意思是将标准输出写入command.log，而标准错误输出则定向到标准输出。虽然这条命令和并一条命令一样，都是将标准错误输出定向到标准输出，但是顺序不一样，所以意义也不一样。

## 关闭文件描述符

```
n<&-
```

关闭输入文件描述符n

```
0<&-,<&-
```

关闭标准输入。

```
n>&-
```

关闭标准输出文件描述符n.

```
1>&-,>&-
```

关闭标准标准输出

子进程继承打开的文件描述符，这就是pipe工作的原理，要阻止一个文件描述符被继承，那么请关闭它。

仅重定向标准错误输出到管道

```
exec 3>&1 # 保存当前标准输出的值
ls -l 2>&1 >&3 3>&- | grep bad 3>&- # 为grep关闭文件描述符3（但并不是'ls'命令）
exec 3>&- # 关闭文件描述符3
```

## Reference

英文原文地址：<http://tldp.org/LDP/abs/html/io-redirection.html>