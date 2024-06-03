
## 一、shell的基本形式

> shell是一个命令解释器，它可以用来启动、挂起、停止甚至编写程序。shell是Linux操作系统的一个整体组成部分，也是Linux操作系统和UNIX设计的一部分。

![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/682bb3cd9fb62114df13a443fea74be5.png)

----
### 1.shell的种类 :sh、bash、csh、tcsh、ash 等。

#### (1).sh shell

全称是 Bourne shell，由 AT&T 公司的 Steve Bourne开发，为了纪 念他，就用他的名字命名了。sh 是 UNIX 上的标准 shell，很UNIX 版本都配有 sh。sh 是第一个流行的 Shell

#### (2). csh shell

sh 之后另一个广为流传的 shell 是由柏克莱大学的 Bill Joy 设计这个 shell 的语法有点类似C语言，所以才得名为 C shell ，简称为 csh。

#### (3). tcsh shell

是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持。

#### (4). ash shell

一个简单的轻量级的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容。

#### (5). bash shell

bash shell 是 Linux 的默认 shell，本教程也基于 bash 编写。bash 由 GNU 组织开发，保持了对 sh shell 的兼容性，是各种 Linux 发行版默认配置的 shell。

### 2. shell命令的基本格式

> 命令名 [选项] <参数1> <参数2> … [选项]是对命令的特别定义，以减号(-)开始，多个选项可以用一个减号(-)连起来， ls -l -h 与 ls -lh 相同。 <参数>提供命令运行的信息，或者是命令执行过程中所使用的文件名。

### 3.注意

#### 1.Linux严格区分大小写， a A 不同

![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/8a42bd9211698d9dac415cad971b1359.png)

#### 2.使用分号( ; ) 一行中输入多个命令。

![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/dc5c1cfc652b690ea3235b7d726d922b.png)



![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/21e005180cc078f49277ee593ee4bd5e.png)

#### 3.按下Table键，自动补齐命令、目录或文件名。

#### 4.系统会把过去输入过的命令记忆下来，只要按方向键中的上下箭头

### 4.默认的文本界面 Shell 提示符有两种：

> root 用户登录后的提示符： #

![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/674e512aa9c9ad349d514b24e4c2b9f5.png)

10分钟就能学会，Linux操作系统21个shell常用命令

> 普通用户登录后的提示符： $

![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/f128f6cb5f3aeebc7102739ccf2298c5.png)

10分钟就能学会，Linux操作系统21个shell常用命令

### 5.输入输出重定向：

> 输入定向：<, << inputfile
> 输出定向： >, >>

![10分钟就能学会，Linux操作系统21个shell常用命令|300](https://ask.qcloudimg.com/http-save/yehe-8223537/fdb0efe129c85bf59e36e74d9c297267.png)

### 6.管道

> 可以将多个命令组合到一起，把管道左边的命令的输出 作为右边命令的输入

![10分钟就能学会，Linux操作系统21个shell常用命令|500](https://ask.qcloudimg.com/http-save/yehe-8223537/031cdf4c74f4552ea257c35bf146464c.png)


```
grep: grep word filename
```

## 二、shell常用命令

### 1.切换工作目录命令cd

#### 命令：

```javascript
[cd : Change Directory]
```

> 所谓工作目录，就是当前操作所在的目录。用户在使用Linux的时候，经常需要更换工作目录。cd命令可以帮助用户切换工作目录，后面可跟绝对路径，也可以跟相对路径。 (1).如果省略目录，则默认切换到当前用户的主目录。 (2).还可以使用“～”、“.”和“..”作为目录名， cd 目录名 例如，切换到/usr/bin/可用如下命令： [root@myhost root]# cd /usr/bin 切换到当前用户的主目录可用如下命令： [root@myhost root]# cd ～

### 2.显示当前路径命令pwd

#### 命令：

```javascript
[pwd: print work directory]
```


> 打印当前目录 显示出当前工作目录的绝对路径。

### 3.查看目录命令ls [ls: list]

#### 命令：

```javascript
[ls: list]
```


> ls是英文单词list的简写，其功能为列出目录的内容，使用相应的参数可以查看文件的相关信息，是用户最常用的命令之一，它类似于DOS下的dir命令。该命令的语法如下： 该命令的语法 ls常用的参数及含义 参 数 含 义 -a 显示指定目录下所有子目录与文件，包括隐藏文件 -c 按文件的修改时间排序 -F 在列出的文件名后以符号表示文件类型：目录文件后加“/”，可执行文件后加“*”，符号链接文件后加“@”，管道文件后加“|”，socket文件后加“=” -h 以用户习惯的单位表示文件的大小，K表示千，M表示兆。通常与-l选项搭配使用 -l 以长格式显示文件的详细信息。每行列出的信息依次是：文件类型与权限、链接数、文件属主、文件属组、文件大小、文件建立或修改的时间、文件名。对于符号链接文件，显示的文件名后有“—>”和引用文件路径名；对于设备文件，其“文件大小”字段显示主、次设备号，而不是文件大小。目录中总块数显示在长格式列表的开头，其中包含间接块 -r 从后向前地列举目录中的内容 -s 按文件大小排序 -t 按文件建立的时间排序，越新修改的越排在前面 -u 按文件上次存取时间排序

#### 例程：

> 使用ls命令查看root目录下的文件信息。在命令提示符下执行如下命令，执行结果如下图所示。

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/b4e03932cbc4c60f87a7119e5df09cb6.png)


### 4. 显示文件命令 [cat :concatenate连锁]

#### 命令：

```javascript
[cat :concatenate连锁]
```


> 文件查看和连接命令cat: cat命令：可以用来查看文件内容， ：可以用于即合并文件。 该命令格式如下： cat [选项] 文件名 常用参数及含义 选项 含 义 -b 显示文件中的行号，空行不编号 -E 在文件的每一行行尾加上“$”字符 -T 将文件的Tab键用字符“^I”来显示 -n 在文件的每行前面显示行号 -s 将连续的多个空行用一个空行来显示 -v 显示除Tab和Enter之外的所有字符

#### 例程：

> 使用cat命令查看文件内容。

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/5d97fedd20619092199884173326a1fc.png)

### 5.分屏显示命令 more:

#### 命令：

```javascript
more [选项] 文件名
```

> 和cat命令类似，more可将文件内容显示在屏幕上， 1.每次只显示一页， 2.按下空格键可以显示下一页， 3.按下q键退出显示. 文件中搜索指定的字符串。 其格式如下： more [选项] 文件名

### 6.按页显示命令less

#### 命令：

```javascript
less  [选项]  文件名
```


> less 命令作用和more命令类似，可用于浏览文本文件的内容。 (1).less命令允许用户使用光标键反复浏览文本。 (2).less可以不读入整个文本文件，因此在处理大型文件时速度较快。 (3).与more命令相比，less命令的功能更加前大。

### 7.复制命令cp：[cp:Copy file]

#### 命令：

```javascript
[cp:Copy file]
```


> cp 命令的功能是将给出的文件或目录复制到另一个文件或目录中，相当于DOS下的copy命令。 该命令可以同时复制多个源文件到目标目录中，在进行文件复制的同时，可以指定目标文件的名称。 其基本使用格式如下： cp [选项] 源目录或文件 目标目录或者文件 常用选项及含义如下表所示。 选项 含 义 -a 该选项通常在复制目录时使用，它保留链接、文件属性，并递归地复制目录 -d 复制时保留链接 -f 删除已经存在的目标文件而不提示 -i 交互式复制，在覆盖目标文件之前将给出提示要求用户确认 -p 此时cp命令除复制源文件的内容外，还将把其修改时间和访问权限也复制到新文件中 -r 若给出的源文件是目录文件，则cp将递归复制该目录下的所有子目录和文件，目标文件必须为一个目录名 -l 不作复制，只是链接文件

#### 注意：

> 为防止用户在不经意的情况下用cp命令破坏另一个文件，建议用户在使用cp命令复制文件时，最好使用i选项。 例：创建文件file3，使用cp命令将文件file3复制到/tmp目录，并改名成file4。在终端提示符下执行如下命令，执行结果如下图所示。 [root@myhost root]# touch file3 [root@myhost root]# cp –i file3 /tmp/file4

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/63d135f464278d26e21e9d0a7b5a5e24.png)

### 8.删除命令rm：[rm :Remove（删除目录或文件）]

#### 命令：

```javascript
[rm :Remove（删除目录或文件）]
```


> rm 命令可以删除一个目录中的一个或多个文件或目录，也可以将某个目录及其下的所以文件及子目录均删除。删除链接文件时，只是断开了链接，原文件保持不变。该命令的基本使用格式如下： rm [选项] 文件名 常用参数及含义如下表所示。 选项 含 义 -i 以进行交互式方式执行 -f 强制删除，忽略不存在的文件，无需提示 -r 递归地删除目录下的内容

#### 例程：

> 使用rm命令分别进行交互式删除和强制删除。在命令提示符下分别执行如下命令，执行结果如下图所示。 [root@myhost root]# rm –i file1 [root@myhost root]# rm –f file1

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/cd29193682c9be29f1b7e8df339e24ee.png)

#### 注意：

> 使用“rm -i file1”命令时采用了交互式执行方式，询问是否删除file1文件。 “rm -f file1”命令时采用了强制执行方式，直接删除指定的文件。

### 9.移动或重命名命令

#### 命令：

```javascript
mv: [mv:Move file]
```


> 用户可以使用 mv 命令来移动文件或目录，也可以给文件或目录重命名。它的用法相当于DOS下的ren和move的组合。 该命令格式如下： mv [选项] 源文件或目录A 目标文件或目录B 常用参数及含义如下表所示。 参 数 含 义 -i 交互方式操作，如果mv操作将导致对已存在的目标文件的覆盖，系统会询问是否重写，要求用户回答以避免误覆盖文件 -f 禁止交互式操作，如有覆盖也不会给出提示
> 
> 如果mv命令格式为“mv 源文件 目标文件”，且两个文件在同一目录下，则表示将源文件重命名为目标文件；
> 
> mv命令是移动文件或目录还是重命名文件或目录，视源文件和目标文件的类型而定。
> 
> 如果源文件和目标文件的类型都为文件，且两个文件同在一个目录，则是将源文件重命名为目标文件。
> 
> 如果源文件为目录，目标文件为不存在的目录，它们同在一个父目录，则是将源目录重名为目标目录。
> 
> 如果目标文件为已存在的目录，源文件可以是多个文件或目录，mv命令将指定的源文件或目录均移至目标目录中。

#### 例程：

> 使用mv命令将file3文件移动到/home目录下，并用ls命令查看结果。 在终端提示符下输入如下命令，执行结果如下图所示。 [root@myhost root]# ls ←查看移动前当前目录下文件 [root@myhost root]# mv file3 /home ←移动file3文件到/home目录 [root@myhost root]# ls ←查看移动后当前目录下文件 [root@myhost root]# ls /home ←查看移动后/home目录下文件

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/f9b358ca4d34a2f79a0deacb882daa2d.png)

#### 注意：

> 使用mv命令跨文件系统移动文件时，先复制文件，再将原有文件删除，而链接至该文件的链接也将丢失。

### 10.创建目录命令

#### 命令：

```javascript
mkdir [mkdir :Make Directory(创建目录]
```


> 可使用mkdir命令创建一个新的目录。需要注意的是新建目录的名称不能与当前目录中已有的目录或文件同名，并且目录创建者必须对当前目录具有写权限。 该命令格式如下： m kdir [参数] 目录名 常用参数及含义如下表所示。 参 数 含 义 -m 对新建目录设置存取权限 -p 如果欲建立的目录的上层目录尚未建立，则一并建立其上的所有祖先目录
> 
> 例： 使用mkdir命令分别创建目录dir1、dir2，在dir1中创建目录dir3，在dir2中创建目录dir4，并使用touch命令在dir2中创建文件file2。
> 
> 在终端提示符下执行如下命令，如下图所示。

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/20e619ae66783a4f6abd768b3981e58b.png)

10分钟就能学会，Linux操作系统21个shell常用命令

### 11.删除空目录命令

#### 命令：

```javascript
rmdir [rmdir :Remove directory]
```

复制

#### 讲解：

> 删除空目录可以使用rmdir命令，该命令是从一个目录中删除一个或多个子目录项。需要注意的是，一个目录被删除之前必须是空的。删除某一个目录时，必须具有对其父目录的写权限。如果要删除的目录不空，将产生错误提示。 该命令的基本使用格式如下： rmdir [-p] 目录 命令中选项含义如下： 参数-p表示递归删除目录，当子目录删除后，其父目录为空时也一同被删除。命令执行完毕后，显示相应信息。 此外，使用 rm –r 也可删除目录及其下的文件和子目录。

#### 例程：

> 使用 rmdir -p递归删除dir1和dir3目录，使用 rm –r命令删除dir2目录及其下的所有文件和子目录。 首先用ls命令查看root主目录下的文件，然后执行过删除目录的命令后再用ls查看一下root目录。在命令提示符中下执行rmdir命令和rm命令，删除完成后再用ls查看一下root目录，结果如下图所示。 [root@myhost root]# rmdir –p /root/dir1/dir3 [root@myhost root]# rm –r dir2

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/0e6ec69788628b98e8f9f3d17e6ab18e.png)

10分钟就能学会，Linux操作系统21个shell常用命令

执行rmdir –p命令递归删除目录时，首先从最里层的子目录进行删除，当dir3删除后，dir1为空目录，所以能够递归删除，而dir1删除后，root目录下还有其它文件，故而会显示提示语句“rmdir：‘/root’：目录非空”，说明不能删除root目录。使用rm –r命令删除目录，则会给出提示信息要求确认删除。 执行删除命令之前，用ls命令可以查看到root主目录里有dir1和dir2两个蓝色字体显示的目录文件，成功删除目录后，用ls命令可以看到root目录的这两个目录文件已经被删除了。

### 12. 查找文件或者目录命令find

#### 命令：

```javascript
 find [路径] [选项]
```

复制

#### 讲解：

> find 命令功能非常强大，通常用来在特定的目录下搜索符合条件的文件，也可以用来搜索特定用户属主的文件。 其格式如下： find [路径] [选项] 常用的选项及含义如下表所示。 选项 含 义 -name < filename > 指定搜索的文件名，输出搜索结果 -user < username > 搜索指定用户搜索所属的文件 -atim < time > 搜索在指定的时间内读取过的文件 -ctim < time > 搜索在指定的时间内修改过的文件

#### 例程：

> 使用find命令从根目录开始查找httpd.conf文件；从根目录搜索tom用户的文件。 在终端提示符下输入如下命令： [root@myhost root]# find / -name httpd.conf [root@myhost root]# find / -user tom 命令的执行结果如下图所示。

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/7b759dcf71f6ef98e2e25862a29a3925.png)

### 13.文件定位命令

#### 命令：

```javascript
locate/slocate
```


> 该命令用于通过文件名或扩展名搜索文件。 locate命令是利用事先在系统中建立系统文件索引资料库的，然后再检查资料库的方式工作的。 为了提高locate命令的查出率，在使用该命令前必须拥有最新的资料[数据库](https://cloud.tencent.com/solution/database?from_column=20065&from=20065)。 可以使用如下的命令更新系统的索引资料数据库： [root@myhost root]# updatedb locate命令的格式如下： locate [参数] 文件名 常用的参数和含义如下表所示。 参 数 含 义 -u 建立资料数据库，从根目录开始 -U < dir > 建立资料数据库，从< dir >目录开始 -e < dir > 排除< dir >目录搜索

#### 例程：

> 例： 首先建立资料数据库，然后搜索vsftpd.conf文件的存放路径。 在终端提示符下输入如下命令： [root@myhost root]# updatedb [root@myhost root]# locate vsftpd.conf 如下图所示。

![10分钟就能学会，Linux操作系统21个shell常用命令](https://ask.qcloudimg.com/http-save/yehe-8223537/4e8d3d1540e39fcdffcfc9e0fde8de21.png)

### 14.文件内容检索命令grep

#### 命令：

```javascript
grep [选项]  < string >  文件名
```


> rm 命令可以删除一个目录中的一个或多个文件或目录，也可以将某个目录及其下的所以文件及子目录均删除。删除链接文件时，只是断开了链接，原文件保持不变。该命令的基本使用格式如下： rm [选项] 文件名 常用参数及含义如下表所示。 选项 含 义 -v 显示不包含匹配文本的所有行 -n 显示匹配行及行号

#### 例程：

> 例 搜索/etc/vsftpd目录下后缀为.conf文件中，其内容中包含“anon”字符串的文本行。 在终端提示符下输入如下命令：
> 
> [root@myhost root]# grep anon /etc/vsftpd/*.conf 如下图所示。

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/f4b1f63bf7059233903baf9a6e113355.png)
### 15.链接命令

#### 命令：

```javascript
In
```


![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/0a85e40c7cd884c6c1d9ca8ea31f5f1a.png)

### 16.创建文件、改变文件或目录生成时间命令 touch

#### 命令：

```javascript
touch
```


![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/ac79f947112b673083f25a4c5d8e2db5.png)

### 17.打包命令 tar

#### 命令：

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/1a04a7234fb023755bb977e335823970.png)

#### 讲解：

![10分钟就能学会，Linux操作系统21个shell常用命令|500](https://ask.qcloudimg.com/http-save/yehe-8223537/3b18e0d9941fb1f58e8d823a3ead693e.png)

### 18. 压缩命令 zip 和gzip

### 命令：

```javascript
解压缩命令unzip 和gunzip
```


> zip是将文件打包为zip格式的压缩文件 unzip是从zip包中解压出某个文件 gzip是将文件打包为tar.gz格式的压缩文件 gunzip从tar.gz包中解压出某个文件
> 
> gzip 命令： # gzip test.txt 它会将文件压缩为文件 test.txt.gz，原来的文件则没有了，解压缩也一样 # gunzip test.txt.gz 它会将文件解压缩为文件 test.txt，原来的文件则没有了，为了保留原有的文件，我们可以加上 -c 选项并利用 linux 的重定向 # gzip -c test.txt > /root/test.gz 这样不但可以将原有的文件保留，而且可以将压缩包放到任何目录中，解压缩也一样 # gunzip -c /root/test.gz > ./test.txt zip 命令： # zip test.zip test.txt 它会将 test.txt 文件压缩为 test.zip ，当然也可以指定压缩包的目录，例如 /root/test.zip # unzip test.zip 它会默认将文件解压到当前目录，如果要解压到指定目录，可以加上 -d 选项 # unzip test.zip -d /root

### 19.压缩文件命令 gzip gunzip

在18里面已经讲了，看上一条。

### 20.修改时间 date; 日历 cal ; 显示时间命令 clock

#### 命令：

```javascript
修改时间 date;    日历 cal ;  显示时间命令 clock 
命令：
```

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/18e7f4ccea1a1feac2e22123bccef5bb.png)

10分钟就能学会，Linux操作系统21个shell常用命令

### 21.帮助命令

#### 命令及讲解：

![10分钟就能学会，Linux操作系统21个shell常用命令|400](https://ask.qcloudimg.com/http-save/yehe-8223537/8be6634f791aacd87a6741ce2407cf68.png)
