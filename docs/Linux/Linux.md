# Linux

# 用户管理

## 查看用户
`who am i` 

![image-20220630151531650](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301515697.png)

输出的第一列表示打开当前伪终端的用户的用户名，第二列的 pts/0 中 pts 表示伪终端，所谓伪是相对于 /dev/tty 设备而言的，第三列则表示当前伪终端的启动时间。

要查看当前登录用户的用户名，去掉空格直接使用 `whoami` 即可。

## 创建用户
在 Linux 系统里， root 账户拥有整个系统至高无上的权限，比如新建和添加用户。

大部分 Linux 系统在安装时都会建议用户新建一个用户而不是直接使用 root 用户进行登录，当然也有直接使用 root 登录的例如 Kali。

一般我们登录系统时都是以普通账户的身份登录的，要创建用户需要 root 权限，这里就要用到 `sudo` 这个命令了。

> 不过使用这个命令有两个大前提，一是你要知道当前登录用户的密码，二是当前用户必须在 sudo 用户组。shiyanlou 用户也属于 sudo 用户组。

**需要注意 Linux 环境下输入密码是不会显示的。**

`su <user>` 可以切换到用户 user，执行时需要输入目标用户的密码，sudo <cmd> 可以以特权级别运行 cmd 命令，需要当前用户属于 sudo 组，且需要输入当前用户的密码。

`su - <user>` 命令也是切换用户，但是同时用户的环境变量和工作目录也会跟着改变成目标用户所对应的。

现在我们新建一个叫 lilei 的用户：`sudo adduser lilei`

然后是给 lilei 用户设置密码，后面的选项的一些内容你可以选择直接回车使用默认值。

这个命令不但可添加用户到系统，同时也会默认为新用户在 /home 目录下创建一个工作目录。

现在你已经创建好一个用户，并且你可以使用你创建的用户登录了，使用如下命令切换登录用户：

`su -l lilei`

退出当前用户跟退出终端一样，可以使用 exit 命令或者使用快捷键 Ctrl+D。

## 用户组
  在 Linux 里面每个用户都有一个归属（用户组），用户组简单地理解就是一组用户的集合，它们共享一些资源和权限，同时拥有私有资源。

在 Linux 里面如何知道自己属于哪些用户组呢？

 **方法一：使用 groups 命令**

`groups shiyanlou`

![image-20220630151710633](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301517675.png)

其中冒号之前表示用户，后面表示该用户所属的用户组。

这里可以看到 shiyanlou 用户属于 shiyanlou 用户组，每次新建用户如果不指定用户组的话，默认会自动创建一个与用户名相同的用户组。

**方法二：查看 /etc/group 文件**

`cat /etc/group | sort`

这里 cat 命令用于读取指定文件的内容并打印到终端输出，后面会详细讲它的使用。

  | sort 表示将读取的文本进行一个字典排序再输出，然后你将看到如下一堆输出，你可以在最下面看到 shiyanlou 的用户组信息：

![image-20220630151738354](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301517396.png)

没找到？没关系，你可以使用 grep 命令过滤掉一些你不想看到的结果：

`cat /etc/group | grep -E "shiyanlou"`

![image-20220630151751813](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301517847.png)

**🤞如何将其它用户加入 sudo 用户组**

默认情况下新创建的用户是不具有 root 权限的，也不在 sudo 用户组，可以让其加入 sudo 用户组从而获取 root 权限：

```php
  su -l lilei
  sudo ls
```

![image-20220630151815467](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301518506.png)


会提示 lilei 不在 sudoers 文件中，意思就是 lilei 不在 sudo 用户组中，至于 sudoers 文件（/etc/sudoers）你现在最好不要动它，操作不慎会导致比较麻烦的后果。

 `usermod` 命令可以为用户添加用户组，同样使用该命令你必需有 root 权限。

  你可以直接使用 root 用户为其它用户添加用户组，或者用其它已经在 sudo 用户组的用户使用 sudo 命令获取权限来执行该命令。

  > **sudo命令** 用来以其他身份来执行命令，预设的身份为root。在/etc/sudoers中设置了可执行sudo指令的用户。若其未经授权的用户企图使用sudo，则会发出警告的邮件给管理员。用户使用sudo时，必须先输入密码，之后有5分钟的有效期限，超过期限则必须重新输入密码。


这里我用 shiyanlou 用户执行 sudo 命令将 lilei 添加到 sudo 用户组，让它也可以使用 sudo 命令获得 root 权限，首先我们切换回 shiyanlou 用户。

```php
su - shiyanlou
```

当然也可以通过 `sudo passwd shiyanlou` 进行设置，或者你直接关闭当前终端打开一个新的终端。

  ```php
groups lilei
sudo usermod -G sudo lilei
groups lilei
  ```

![image-20220630151858529](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301518568.png)


然后你再切换回 lilei 用户，现在就可以使用 sudo 获取 root 权限了。

![image-20220630151906094](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301519122.png)


## 删除用户和用户组

删除用户是很简单的事：`sudo deluser lilei --remove-home`

![image-20220630151933885](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301519929.png)

使用 `--remove-home` 参数在删除用户时候会一并将该用户的工作目录一并删除。如果不使用那么系统会自动在 /home 目录为该用户保留工作目录。

删除用户组可以使用 `groupdel` 命令，倘若该群组中仍包括某些用户，则必须先删除这些用户后，才能删除群组。

  # 文件权限
  ## 查看文件权限
  > **文件权限**就是文件的访问控制权限，即哪些用户和组群可以访问文件以及可以执行什么样的操作。

 `ls` 命令(list)，我们用它来列出并显示当前目录下的文件，当然这是在不带任何参数的情况下，它能做的当然不止这么多，现在我们就要用它来查看文件权限。

使用较长格式列出文件：`ls -l`

![image-20220630151946450](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301519498.png)    


你可能除了知道最后面那一项是文件名之外，其它项就不太清楚了，那么到底是什么意思呢？

![image-20220630152017736](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301520765.png)


可能你还是不太明白，比如第一项文件类型和权限那一堆东西具体指什么，链接又是什么，何为最后修改时间，下面一一道来：

![image-20220630152031401](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301520443.png)


- 文件类型
  

关于文件类型，这里有一点你必需时刻牢记 Linux 里面一切皆文件，正因为这一点才有了设备文件（ /dev 目录下有各种设备文件，大都跟具体的硬件设备相关）这一说。

  socket：网络套接字；

  pipe 管道，这个东西很重要；

  软链接文件：链接文件是分为两种的，另一种当然是“硬链接”（硬链接不常用，具体内容不作为本课程讨论重点，而**软链接等同于 Windows 上的快捷方式**，你记住这一点就够了）。

- 文件权限
  

读权限，表示你可以使用 `cat <file name>` 之类的命令来读取某个文件的内容；

  写权限，表示你可以编辑和修改某个文件的内容；

执行权限，通常指可以运行的二进制程序文件或者脚本文件，如同 Windows 上的 exe 后缀的文件，不过 Linux 上不是通过文件后缀名来区分文件的类型。

  需要注意的一点是，**一个目录同时具有读权限和执行权限才可以打开并查看内部文件，而一个目录要有写权限才允许在其中创建其它文件**，这是因为目录文件实际保存着该目录里面的文件的列表等信息。

所有者权限，这一点相信你应该明白了，至于所属用户组权限，是指你所在的用户组中的所有其它用户对于该文件的权限，比如，你有一个 iPad，那么这个用户组权限就决定了你的兄弟姐妹有没有权限使用它破坏它和占有它。

- 链接数
链接到该文件所在的 inode 结点的文件名数目

- 文件大小
以 inode 结点大小为单位来表示的文件大小，你可以给 ls 加上 -lh 参数来更直观的查看文件的大小。

明白了文件权限的一些概念，我们顺带补充一下关于 `ls` 命令的一些其它常用的用法：

 显示除了 .（当前目录）和 ..（上一级目录）之外的所有文件，包括隐藏文件（Linux 下以 . 开头的文件为隐藏文件）:`ls -a`

![image-20220630152053925](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301520962.png)

当然，你可以同时使用 -a 和 -l 参数：`ls -al`

查看某一个目录的完整属性，而不是显示目录里面的文件属性：`ls -dl <目录名>`

显示所有文件大小，并以普通人类能看懂的方式呈现：`ls -asSh`

![image-20220630152104555](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301521599.png)

其中小 s 为显示文件大小，大 S 为按文件大小排序，若需要知道如何按其它方式排序，可以使用 man ls 命令查询。

  ## 变更文件所有者

切换到 lilei 用户，然后在 /home/lilei 目录新建一个文件，命名为 iphone11。

  ```
su - lilei
pwd
touch iphone11
ls -alh iphone11
  ```

  可见文件所有者是 lilei ：

![image-20220630152125917](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301521947.png)

现在切换回到 shiyanlou 用户，使用以下命令变更文件所有者为 shiyanlou。

  > **chown** 用来变更文件或目录的拥有者或所属群组

```
cd /home/lilei
ls iphone11
sudo chown shiyanlou iphone11
```

现在查看，发现文件所有者成功修改为 shiyanlou。

![image-20220630152213340](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301522370.png)

## 修改文件权限

 > 如果你有一个自己的文件不想被其他用户读、写、执行，那么就需要对文件的权限做修改。

  文件的权限有两种表示方式：

🤞方式一：二进制数字表示

<img src="https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301522090.png" alt="image-20220630152223046" style="zoom:40%;" />

**每个文件有三组固定的权限，分别对应拥有者，所属用户组，其他用户，记住这个顺序是固定的**。文件的读写执行对应字母 rwx，以二进制表示就是 111，用十进制表示就是 7。

  例如我们刚刚新建的文件 iphone11 的权限是 rw-rw-rw-，换成对应的十进制表示就是 666，这就表示这个文件的拥有者，所属用户组和其他用户具有读写权限，不具有执行权限。

如果我要将文件 iphone11 的权限改为只有我自己可以用那么就可以用这个方法更改它的权限。

为了演示，我先在文件里加点内容：

```
echo "echo \"hello shiyanlou\"" > iphone11
```

然后修改权限：

  `chmod`命令用来变更文件或目录的权限

  ```
chmod 600 iphone11
ls -alh iphone11
  ```

![image-20220630152253116](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301522155.png)


切换到 lilei 用户，尝试写入和读取操作，可以看到 lilei 用户已经不能读写这个 iphone11 文件了：

![image-20220630152301244](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301523279.png)

🤞方式二：加减赋值操作

要完成上述实验相同的效果，你可以：`chmod go-rw iphone11`

![image-20220630152309781](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/202206301523814.png)

g、o 还有 u 分别表示 group（用户组）、others（其他用户） 和 user（用户），+ 和 - 分别表示增加和去掉相应的权限。

  ``QNA``

adduser 和 useradd 的区别是什么?

答：

useradd 只创建用户，不会创建用户密码和工作目录，创建完了需要使用 ``passwd <username>`` 去设置新用户的密码。

adduser 在创建用户的同时，会创建工作目录和密码（提示你设置），做这一系列的操作。其实 useradd、userdel 这类操作更像是一种命令，执行完了就返回。而 adduser 更像是一种程序，需要你输入、确定等一系列操作。



[TOC] 

 # 目录结构

首先了解 Linux 的目录与 Windows 的目录的区别，对于一般操作上的感受来说没有多大不同，但从它们的实现机制来说是完全不同的。

一种不同是体现在目录与存储介质（磁盘，内存，DVD 等）的关系上。

以往的 Windows 一直是以存储介质为主的，主要以盘符（C 盘，D 盘...）及分区来实现文件管理，然后之下才是目录，目录就显得不是那么重要，除系统文件之外的用户文件放在任何地方任何目录也是没有多大关系。所以通常 Windows 在使用一段时间后，磁盘上面的文件目录会显得杂乱无章（少数善于整理的用户除外吧）。

然而 UNIX/Linux 恰好相反，UNIX 是以目录为主的，Linux 也继承了这一优良特性。 **Linux 是以树形目录结构的形式来构建整个系统的**，可以理解为树形目录是一个用户可操作系统的骨架。

虽然本质上无论是目录结构还是操作系统内核都是存储在磁盘上的，但从逻辑上来说 Linux 的磁盘是“挂在”（挂载在）目录上的，每一个目录不仅能使用本地磁盘分区的文件系统，也可以使用网络上的文件系统。举例来说，可以利用网络文件系统（Network File System，NFS）服务器载入某特定目录等。

## FHS 标准
Linux 的目录结构说复杂很复杂，说简单也很简单。

复杂在于，因为系统的正常运行是以目录结构为基础的，对于初学者来说里面大部分目录都不知道其作用，重要与否，特别对于那些曾经的重度 Windows 用户，他们会纠结很长时间，关于我安装的软件在哪里这类问题。

说它简单是因为，其中大部分目录结构是规定好了的（FHS 标准），是死的，当你掌握后，你在里面的一切操作都会变得井然有序。

> FHS（英文：Filesystem Hierarchy Standard 中文：文件系统层次结构标准），多数 Linux 版本采用这种文件组织形式，FHS 定义了系统中每个区域的用途、所需要的最小构成的文件和目录，同时还给出了例外处理与矛盾处理。

FHS 定义了两层规范，第一层是 / 下面的各个目录应该要放什么文件数据，例如 /etc 应该放置设置文件，/bin 与 /sbin 则应该放置可执行文件等等。

第二层则是针对 /usr 及 /var 这两个目录的子目录来定义。例如 /var/log 放置系统日志文件，/usr/share 放置共享数据等等。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/beff0f7e-807f-4fc6-8453-02110820c816.png)

关于上面提到的 FHS，这里还有个很重要的内容你一定要明白，FHS 是根据以往无数 Linux 用户和开发者的经验总结出来的，并且会维持更新，FHS 依据文件系统使用的频繁与否以及是否允许用户随意改动，将目录定义为四种交互作用的形态，如下表所示：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1c9fc4b5-1cc0-4fbc-8718-6b081cebe7ae.png)

## 目录路径  
  **路径**

顾名思义，路径就是你要去哪儿的路线。如果你想进入某个具体的目录或者想获得某个目录的文件（目录本身也是文件）那就得用路径来找到了。

使用 `cd` 命令可以切换目录，在 Linux 里面使用 `.` 表示当前目录， `..` 表示上一级目录（以 . 开头的文件都是隐藏文件，所以这两个目录必然也是隐藏的，你可以使用 ls -a 命令查看隐藏文件），`-` 表示上一次所在目录， `~` 通常表示当前用户的 home 目录。使用 `pwd` 命令可以获取当前所在路径（绝对路径）。

进入上一级目录：`cd ..`

进入你的 home 目录：`cd ~`

或者 `cd /home/<你的用户名>`

使用 pwd 获取当前路径：`pwd`

**绝对路径**

关于绝对路径，简单地说就是以根" / "目录为起点的完整路径，以你所要到的目录为终点，表现形式如： /usr/local/bin，表示根目录下的 usr 目录中的 local 目录中的 bin 目录。

**相对路径**

相对路径，也就是相对于你当前的目录的路径，相对路径是以当前目录 `.`为起点，以你所要到的目录为终点，表现形式如： usr/local/bin （这里假设你当前目录为根目录）。

注意到，我们表示相对路径实际并没有加上表示当前目录的那个 `.` ，而是直接以目录名开头，因为这个 usr 目录为 / 目录下的子目录，是可以省略这个 `.` 的（以后会讲到一个类似不能省略的情况）；

如果是当前目录的上一级目录，则需要使用 `..` ，比如你当前目录为 /home/shiyanlou 目录下，根目录就应该表示为 ../../ ，表示上一级目录（ home 目录）的上一级目录（ / 目录）。

下面我们以你的 home 目录为起点，分别以绝对路径和相对路径的方式进入 /usr/local/bin 目录：

``` 
# 绝对路径
cd /usr/local/bin
# 相对路径
cd ../../usr/local/bin
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/97d28ce4-25d0-4dbe-ace3-cabad86aaa3e.png)

进入一个目录，可以使用绝对路径也可以使用相对路径，那我们应该在什么时候选择正确的方式进入某个目录呢?

就是凭直觉嘛😁，你觉得怎样方便就使用哪一个，而不用特意只使用某一种。

比如假设我当前在 /usr/local/bin 目录，我想进入上一级的 local 目录你说是使用 cd .. 方便还是 cd /usr/local 方便？

而如果要进入的是 usr 目录，那么 cd /usr ，就比 cd ../.. 方便一点了。

提示：在进行目录切换的过程中请多使用 Tab 键自动补全，可避免输入错误，连续按两次 Tab 可以显示全部候选结果。

  # 文件基本操作

  ## 新建
  **新建空白文件**

使用 `touch` 命令创建空白文件。

关于 touch 命令，其主要作用是来更改已有文件的时间戳的（比如，最近访问时间，最近修改时间），但其在不加任何参数的情况下，只指定一个文件名，则可以创建一个指定文件名的空白文件（不会覆盖已有同名文件），当然你也可以同时指定该文件的时间戳。

创建名为 test 的空白文件，因为在其它目录没有权限，所以需要先 cd ~ 切换回 shiyanlou 用户的 Home 目录：

```
cd ~
touch test
```

**新建目录**

使用 `mkdir`（make directories）命令可以创建一个空目录，也可同时指定创建目录的权限属性。

创建名为“ mydir ”的空目录：

```
mkdir mydir
```

使用 `-p` 参数，同时创建父目录（如果不存在该父目录），如下我们同时创建一个多级目录（这在安装软件、配置安装路径时非常有用）：

```
mkdir -p father/son/grandson
```

这里使用的路径是相对路径，代表在当前目录下生成，当然我们直接以绝对路径的方式表示也是可以的。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/07fc77ac-327e-412b-8cc6-fb2f76ba96c5.png)

还有一点需要注意的是，若当前目录已经创建了一个 test 文件，再使用 mkdir test 新建同名的文件夹，系统会报错文件已存在。这符合 Linux 一切皆文件的理念。


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/61ea06f3-cb04-409f-9247-255228be47f2.png)

若当前目录存在一个 test 文件夹，则 touch 命令，则会更改该文件夹的时间戳而不是新建文件。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/f44e1514-e390-4191-bf06-281222fd0603.png)

  ## 复制
**复制文件**

使用 `cp` 命令（copy）复制一个文件到指定目录。

将之前创建的 test 文件复制到 /home/shiyanlou/father/son/grandson 目录中：

```
cp test father/son/grandson
```

是不是很方便啊，如果在图形界面则需要先在源目录复制文件，再进到目的目录粘贴文件，而命令行操作步骤就一步到位了。

**复制目录**

如果直接使用 `cp` 命令复制一个目录的话，会出现如下错误：


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/92dd2caa-bc90-4998-9f98-a7af7d780e9a.png)


要成功复制目录需要加上 `-r` 或者 `-R` 参数，表示递归复制：

```
cd /home/shiyanlou
mkdir family
cp -r father family
```

  ## 删除
  **删除文件**

使用 `rm`（remove files or directories）命令删除一个文件：

```
rm test
```

有时候你会遇到想要删除一些为只读权限的文件，直接使用 rm 删除会显示一个提示，如下：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/6b942238-e5a8-4a0f-9d30-45ef52986a65.png)

你如果想忽略这提示，直接删除文件，可以使用 `-f ` 参数强制删除：

```
rm -f test
```

**删除目录**
跟复制目录一样，要删除一个目录，也需要加上 `-r` 或 `-R` 参数：

```
rm -r family
```

遇到权限不足删除不了的目录也可以和删除文件一样加上 `-f` 参数：

```
rm -rf family
```

  ## 移动文件与文件重命名
  **移动文件**

使用 `mv`（move or rename files）命令移动文件（剪切）。命令格式是 `mv 源目录文件 目的目录`。

例如将文件“ file1 ”移动到 Documents 目录：

```
mkdir Documents
touch file1
mv file1 Documents
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/625e8c80-afa2-4fc7-9ffa-da0f17c652b5.png)

**重命名文件**

`mv` 命令除了能移动文件外，还能给文件重命名，命令格式为` mv 旧的文件名 新的文件名`。

例如将文件“ file1 ”重命名为“ myfile ”：

```
mv file1 myfile
```
**批量重命名**

要实现批量重命名，`mv` 命令就有点力不从心了，我们可以使用一个看起来更专业的命令 `rename` 来实现，不过它要用 `perl` 正则表达式来作为参数。

`rename` 命令并不是内置命令，若提示无该命令可以使用 `sudo apt-get install rename` 命令自行安装。

```
cd /home/shiyanlou/

# 使用通配符批量创建 5 个文件:
touch file{1..5}.txt

# 批量将这 5 个后缀为 .txt 的文本文件重命名为以 .c 为后缀的文件:
rename 's/\.txt/\.c/' *.txt

# 批量将这 5 个文件，文件名和后缀改为大写:
rename 'y/a-z/A-Z/' *.c
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/e66e1d96-39bf-459c-8608-e85f7b39b175.png)

简单解释一下上面的命令，rename 是先使用第二个参数的通配符匹配所有后缀为 .txt 的文件，然后使用第一个参数提供的正则表达式将匹配的这些文件的 .txt 后缀替换为 .c，这一点在我们后面学习了 `sed` 命令后，相信你会更好地理解。

有的同学可能在输入时出现命令未闭合的状态，命令行会出现 `quote>` 开头的提示符。这是因为上述命令中的 ' 未输入完成，这时按下 ctrl+c 即可退出该模式。还有就是注意 ' 必须为英文符号（半角），若输入的是中文符号（全角）也会报错。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/80736788-91d0-48fd-adee-b21ad82ac8b7.png)

  ## 查看文件

  **使用 `cat`，`tac` 和 `nl` 命令查看文件**

前两个命令都是用来打印文件内容到标准输出（终端），其中 `cat` 为正序显示，`tac` 为倒序显示。

标准输入输出：当我们执行一个 shell 命令行时通常会自动打开三个标准文件，即标准输入文件（stdin），默认对应终端的键盘、标准输出文件（stdout）和标准错误输出文件（stderr）。 

后两个文件都对应被重定向到终端的屏幕，以便我们能直接看到输出内容。进程将从标准输入文件中得到输入数据，将正常输出数据输出到标准输出文件，而将错误信息送到标准错误文件中。

比如我们要查看之前从 /etc 目录下拷贝来的 passwd 文件：

```
cd /home/shiyanlou
cp /etc/passwd passwd
cat passwd
```

可以加上 `-n` 参数显示行号：

```
cat -n passwd
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/44ab3dc4-80b2-4937-a3fb-dbd24e1dcb42.png)

`nl` 命令，添加行号并打印，这是个比 `cat -n` 更专业的行号打印命令。

```
这里简单列举它的常用的几个参数：

-b : 指定添加行号的方式，主要有两种：
    -b a:表示无论是否为空行，同样列出行号("cat -n"就是这种方式)
    -b t:只列出非空行的编号并列出（默认为这种方式）
-n : 设置行号的样式，主要有三种：
    -n ln:在行号字段最左端显示
    -n rn:在行号字段最右边显示，且不加 0
    -n rz:在行号字段最右边显示，且加 0
-w : 行号字段占用的位数(默认为 6 位)
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/ea886073-8061-4a2a-8887-12a938b51c27.png)

你会发现使用这几个命令，默认的终端窗口大小，一屏显示不完文本的内容，得用鼠标拖动滚动条或者滑动滚轮才能继续往下翻页，要是可以直接使用键盘操作翻页就好了，那么你就可以使用下面要介绍的命令。

**使用 `more` 和 `less` 命令分页查看文件**

如果说上面的 `cat` 是用来快速查看一个文件的内容的，那么这个 `more` 和 `less` 就是天生用来"阅读"一个文件的内容的，比如说 man 手册内部就是使用的 `less` 来显示内容。

其中 `more` 命令比较简单，只能向一个方向滚动，而 `less` 为基于 `more` 和 `vi` （一个强大的编辑器）开发，功能更强大。`less` 的使用基本和 `more` 一致，具体使用请查看 man 手册，这里只介绍 more 命令的使用。

使用 more 命令打开 passwd 文件：

```
more passwd
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/96a1a9ac-5a5a-424f-87db-03f25d4d5bb1.png)

打开后默认只显示一屏内容，终端底部显示当前阅读的进度。可以使用 `Enter` 键向下滚动一行，使用 `Space` 键向下滚动一屏，按下 `h` 显示帮助，`q` 退出。

**使用 `head` 和 `tail` 命令查看文件**

这两个命令，那些性子比较急的人应该会喜欢，因为它们一个是只查看文件的头几行（默认为 10 行，不足 10 行则显示全部）和尾几行。

还是拿 passwd 文件举例，比如当我们想要查看最近新增加的用户，那么我们可以查看这个 /etc/passwd 文件，不过我们前面也看到了，这个文件里面一大堆乱糟糟的东西，看起来实在费神啊。

因为系统新增加一个用户，会将用户的信息添加到 passwd 文件的最后，那么这时候我们就可以使用 `tail` 命令了：

```
tail /etc/passwd
```

甚至更直接的只看一行， 加上 `-n` 参数，后面紧跟行数：

```
tail -n 1 /etc/passwd
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/85ea6618-7c37-4a0a-aa75-d0431da575b8.png)

关于 `tail` 命令，不得不提的还有它一个很牛的参数 `-f`，这个参数可以实现不停地读取某个文件的内容并显示，这可以让我们动态查看日志，达到实时监视的目的。

  ## 查看文件类型
  我们可以使用 `file` 命令查看文件的类型：

```
file /bin/ls
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/797b3d5c-c668-40d8-a1d0-dab5e83ac73c.png)

说明这是一个可执行文件，运行在 64 位平台，并使用了动态链接文件（共享库）。

与 Windows 不同的是，如果你新建了一个 shiyanlou.txt 文件，Windows 会自动把它识别为文本文件，而 file 命令会识别为一个空文件。

这个前面我提到过，在 Linux 中文件的类型不是根据文件后缀来判断的。当你在文件里输入内容后才会显示文件类型。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/56335591-8ddb-4d05-81ae-45d0e1c8b61b.png)

  ## 编辑文件

  在 Linux 下面编辑文件通常我们会直接使用专门的命令行编辑器比如（emacs，vim，nano），如果你想更加快速地入门，可以直接使用 Linux 内部的 vim 学习教程，输入如下命令即可开始：

```
vimtutor
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/35f63b42-424f-4e75-b640-28607f1806b2.png)

# 环境变量

  要解释环境变量，得先明白变量是什么，准确的说应该是 Shell 变量，所谓变量就是计算机中用于记录一个值（不一定是数值，也可以是字符或字符串）的符号，而这些符号将用于不同的运算处理中。

  通常变量与值是一对一的关系，可以通过表达式读取它的值并赋值给其它变量，也可以直接指定数值赋值给任意变量。

  为了便于运算和处理，大部分的编程语言会区分变量的类型，用于分别记录数值、字符或者字符串等等数据类型。Shell 中的变量也基本如此，有不同类型，但不用专门指定类型名，可以参与运算，有作用域限定。

> 变量的作用域即变量的有效范围（比如一个函数中、一个源文件中或者全局范围），在该范围内只能有一个同名变量。一旦离开则该变量无效，如同不存在这个变量一般。

😗在 Shell 中如何创建一个变量，如何给变量赋值和如何读取变量的值呢？

（1）使用 `declare` 命令创建一个变量名为 tmp 的变量：
`declare tmp`

其实也可以不用 declare 预声明一个变量，直接即用即创建，这里只是告诉你 declare 的作用，这在创建其它指定类型的变量（如数组）时会用到。

（2）使用 = 号赋值运算符，将变量 tmp 赋值为 shiyanlou：
  `tmp=shiyanlou`

 > 注意，与其他语言不同的是， Shell 中的赋值操作，`=` 两边**不可以输入空格，否则会报错**。

 （3） 读取变量的值，使用 `echo` 命令和 `$` 符号：
 `echo $tmp`

  > 注意：`$` 符号用于表示引用一个变量的值，初学者经常忘记输入

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/e4d11f94-2027-4c83-9d99-fe55344301fe.png)

 **注意**：并不是任何形式的变量名都是可用的，变量名只能是英文字母、数字或者下划线，且不能以数字作为开头。

## 环境变量
环境变量的作用域比自定义变量的要大，如 Shell 的环境变量作用于自身和它的子进程。

在所有的 UNIX 和类 UNIX 系统中，每个进程都有其各自的环境变量设置，且默认情况下，当一个进程被创建时，除了创建过程中明确指定的话，它将继承其父进程的绝大部分环境设置。

Shell 程序也作为一个进程运行在操作系统之上，而我们在 Shell 中运行的大部分命令都将以 `Shell的子进程`的方式运行。

![image-20220630162715209](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/image-20220630162715209.png)

通常我们会涉及到的`变量类型`有三种：

- 当前 Shell 进程私有用户自定义变量，如上面我们创建的 tmp 变量，只在当前 Shell 中有效。
- Shell 本身内建的变量。
- 从自定义变量导出的环境变量。

也有三个与上述三种环境变量相关的命令：`set`，`env`，`export`。这三个命令很相似，都是用于打印环境变量信息，区别在于涉及的变量范围不同。详见下表：

| 命 令  | 说 明                                                        |
| :----: | :----------------------------------------------------------- |
|  set   | 显示当前 Shell 所有变量，包括其内建环境变量（与 Shell 外观等相关），用户自定义变量及导出的环境变量。 |
|  env   | 显示与当前用户相关的环境变量，还可以让命令在指定环境中运行。 |
| export | 显示从 Shell 中导出成环境变量的变量，也能通过它将自定义变量导出为环境变量。 |

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1fd6be23-a607-4382-9641-774f0c62e26a.png)

关于哪些变量是环境变量，可以简单地理解成在当前进程的子进程有效则为环境变量，否则不是（有些人也将所有变量统称为环境变量，只是以全局环境变量和局部环境变量进行区分，我们只要理解它们的实质区别即可）。

**永久生效**

但是问题来了，当你关机后，或者关闭当前的 shell 之后，环境变量就没了，怎么才能让环境变量永久生效呢？

按变量的生存周期来划分，Linux 变量可分为两类：

- 永久的：需要修改配置文件，变量永久生效；
- 临时的：使用 export 命令行声明即可，变量在关闭 shell 时失效。

这里介绍两个重要文件 `/etc/bashrc`（有的 Linux 没有这个文件） 和 `/etc/profile` ，它们分别存放的是 shell 变量和环境变量。还有要注意区别的是每个用户目录下的一个隐藏文件：

```
# .profile 可以用 ls -a 查看
cd /home/shiyanlou
ls -a
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/9a453282-a001-4886-b58a-bb0e43c05748.png)

这个 `.profile` **只对当前用户永久生效**。因为它保存在当前用户的 Home 目录下，当切换用户时，工作目录可能一并被切换到对应的目录中，这个文件就无法生效。

而写在` /etc/profile` 里面的是**对所有用户永久生效**，所以如果想要添加一个永久生效的环境变量，只需要打开 `/etc/profile`，在最后加上你想添加的环境变量就好啦。

## 命令的查找路径与顺序
😀我们在 Shell 中输入一个命令，Shell 是怎么知道去哪找到这个命令然后执行的呢？

这是通过`环境变量 PATH` 来进行搜索的，熟悉 Windows 的用户可能知道 Windows 中的也是有这么一个 PATH 环境变量。这个 PATH 里面就保存了 Shell 中执行的命令的搜索路径。

查看 PATH 环境变量的内容：

```
echo $PATH
```
默认情况下你会看到如下输出：

```
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```
通常这一类目录下放的都是可执行文件，当我们在 Shell 中执行一个命令时，系统就会按照 PATH 中设定的路径按照顺序依次到目录中去查找，如果存在同名的命令，则执行先找到的那个。

## 添加自定义路径到 PATH 环境变量
 `PATH` 里面的路径是以 `:` 作为分割符的，所以我们可以这样添加自定义路径：
`PATH=$PATH:/home/shiyanlou/mybin`

注意这里一定要使用`绝对路径`。

你可能会意识到这样还并没有很好的解决问题，因为我给 PATH 环境变量追加了一个路径，它也只是在当前 Shell 有效，我一旦退出终端，再打开就会发现又失效了。

有没有方法让添加的环境变量全局有效？或者每次启动 Shell 时自动执行上面添加自定义路径到 PATH 的命令？下面我们就来说说后一种方式——让它自动执行。

在每个用户的 home 目录中有一个 Shell 每次启动时会默认执行一个配置脚本，以初始化环境，包括添加一些用户自定义环境变量等等。

若环境使用的 Shell 是 `zsh`，它的配置文件是 `.zshrc`，相应的如果使用的 Shell 是 `Bash`，则配置文件为 `.bashrc`。

它们在 etc 下还都有一个或多个全局的配置文件，不过我们一般只修改用户目录下的配置文件。Shell 的种类有很多，可以使用 `cat /etc/shells` 命令查看当前系统已安装的 Shell。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/616f63fb-4b83-41e3-9044-91548c6a45a3.png)

我们可以简单地使用下面命令直接添加内容到 `.zshrc` 中：

```
echo "PATH=$PATH:/home/shiyanlou/mybin" >> .zshrc
```
上述命令中 `>>` 表示将标准输出以`追加`的方式重定向到一个文件中；

注意前面用到的 `>` 是以`覆盖`的方式重定向到一个文件中，使用的时候一定要注意分辨。

在指定文件不存在的情况下都会创建新的文件。


## 修改和删除已有变量
**变量修改**

变量的修改有以下几种方式：

|         变量设置方式         |                     说明                     |
| :--------------------------: | :------------------------------------------: |
|      ${变量名#匹配字串}      | 从头向后开始匹配，删除符合匹配字串的最短数据 |
|     ${变量名##匹配字串}      | 从头向后开始匹配，删除符合匹配字串的最长数据 |
|      ${变量名%匹配字串}      | 从尾向前开始匹配，删除符合匹配字串的最短数据 |
|     ${变量名%%匹配字串}      | 从尾向前开始匹配，删除符合匹配字串的最长数据 |
| ${变量名/旧的字串/新的字串}  |    将符合旧字串的第一个字串替换为新的字串    |
| ${变量名//旧的字串/新的字串} |     将符合旧字串的全部字串替换为新的字串     |

比如我们可以修改前面添加到 `PATH` 的环境变量，将添加的 `mybin` 目录从环境变量里删除。

为了避免操作失误导致命令找不到，我们先将 PATH 赋值给一个新的自定义变量 `mypath`：

```
mypath=$PATH
echo $mypath
mypath=${mypath%/home/shiyanlou/mybin}
# 或使用通配符 * 表示任意多个任意字符
mypath=${mypath%*/mybin}
```

可以看到路径已经不存在了。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/001526e2-0585-40b5-bc5d-8f55eb305663.png)


**变量删除**
可以使用 `unset` 命令删除一个环境变量：

```
unset mypath
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/6dc075be-5f38-4a22-b728-e7ab8ac3adfc.png)

## 如何让环境变量立即生效
  前面我们在 Shell 中修改了一个配置脚本文件之后（比如 zsh 的配置文件 home 目录下的 .zshrc），每次都要退出终端重新打开甚至重启主机之后其才能生效，很是麻烦，我们可以使用 `source` 命令来让其立即生效，如：

```
cd /home/shiyanlou
source .zshrc
```
`source` 命令还有一个别名就是 `.`，上面的命令如果替换成 . 的方式就该是：

```
. ./.zshrc
```
在使用 `.` 的时候，需要注意与表示当前路径的那个点区分开。

注意第一个点后面有一个空格，而且后面的文件必须指定完整的绝对或相对路径名，`source` 则不需要。

# 搜索文件
与搜索相关的命令常用的有 `whereis`，`which`，`find` 和 `locate`。

- `whereis` 简单快速

```
whereis who
whereis find
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/2dca2b39-f05c-4185-953c-ac2b073a56fc.png)

你会看到 `whereis find` 找到了三个路径，两个可执行文件路径和一个 man 在线帮助文件所在路径，这个搜索很快，因为它并没有从硬盘中依次查找，而是直接从`数据库`中查询。

whereis 只能搜索二进制文件（-b），man 帮助文件（-m）和源代码文件（-s）。如果想要获得更全面的搜索结果可以使用 `locate` 命令。

- `locate` 快而全

使用 `locate` 命令查找文件也不会遍历硬盘，它通过查询 `/var/lib/mlocate/mlocate.db 数据库`来检索信息。

不过这个数据库也不是实时更新的，系统会使用定时任务每天自动执行 `updatedb` 命令来更新数据库。

所以有时候你刚添加的文件，它可能会找不到，需要手动执行一次 `updatedb` 命令（在我们的环境中必须先执行一次该命令）。

注意这个命令也不是内置的命令，在部分环境中需要手动安装，然后执行更新。

```
sudo apt-get update
sudo apt-get install locate
sudo updatedb
```
它可以用来查找指定目录下的不同文件类型，如查找 /etc 下所有以 sh 开头的文件：

```
locate /etc/sh
```
> 注意，它不只是在 /etc 目录下查找，还会自动递归子目录进行查找。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/d5943c61-dc6b-4ef6-84e7-2508ebd2cd7a.png)

查找 /usr/share/ 下所有 jpg 文件：

```
locate /usr/share/*.jpg
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/5142a506-6ae3-40c5-8dbf-2b11b6d48401.png)

环境里使用 `zsh`，在 ~/.zshrc 文件里添加了 `setopt nonomatch` 配置，这样就不会自动处理和修复命令，因此可以不使用 `\` 转义。

如果其他环境中执行该命令提示 `zsh: no matches found: /usr/share/*.jpg`，则可以在 .zshrc 中添加上述配置，或者使用 `\` 转义。


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1b36cb34-e5c9-42e7-ad26-ed3ca02cd7d5.png)

如果想只统计数目可以加上 `-c` 参数，`-i` 参数可以忽略大小写进行查找，`whereis` 的 `-b`、`-m`、`-s` 同样可以使用。

- `which` 小而精

`which` 本身是 Shell 内建的一个命令，我们通常使用 `which` 来确定是否安装了某个指定的程序，因为它只从 `PATH` 环境变量指定的路径中去搜索命令并且返回第一个搜索到的结果。

也就是说，我们可以看到某个系统命令是否存在以及执行的到底是哪一个地方的命令。

```
which man
which nginx
which ping
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1a3b095f-7d45-4854-8371-fe36fbd0911c.png)

- `find` 精而细

`find` 应该是这几个命令中最强大的了，它不但可以通过文件类型、文件名进行查找，而且可以根据文件的属性（如文件的时间戳，文件的权限等）进行搜索。

`find` 命令强大到，要把它讲明白至少需要单独好几节课程才行，我们这里只介绍一些常用的内容。

```
sudo find /etc/ -name interfaces
```
这条命令表示去 /etc/ 目录下面 ，搜索名字叫做 `interfaces` 的文件或者目录。

这是 `find` 命令最常见的格式，**千万记住 `find` 的第一个参数是要搜索的地方**。

命令前面加上 sudo 是因为 shiyanlou 只是普通用户，对 /etc 目录下的很多文件都没有访问的权限，如果是 root 用户则不用使用。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/402a8de3-3937-451f-8755-798460d866f6.png)

> 注意：`find` 命令的路径是作为第一个参数的， 基本命令格式为 `find [path][option] [action]` 。

与时间相关的命令参数：

|  参数  |          说明          |
| :----: | :--------------------: |
| -atime |      最后访问时间      |
| -ctime | 最后修改文件内容的时间 |
| -mtime | 最后修改文件属性的时间 |

下面以 `-mtime` 参数举例：

- `-mtime n`：n 为数字，表示为在 n 天之前的“一天之内”修改过的文件
- `-mtime +n`：列出在 n 天之前（不包含 n 天本身）被修改过的文件
- `-mtime -n`：列出在 n 天之内（包含 n 天本身）被修改过的文件
- `-newer file`：file 为一个已存在的文件，列出比 file 还要新的文件名

<img src="https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/d1d3e3f3-26b0-4e6c-88d5-1ddb53915451.png" style="zoom:70%;" />

列出 home 目录中，当天（24 小时之内）有改动的文件：

```
find ~ -mtime 0
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1af6123e-008d-4120-8561-0bb3d74eb8d2.png)

列出用户家目录下比 /etc 目录新的文件：

```
find ~ -newer /etc
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/39977be1-c990-445d-8164-5ad74e906a81.png)

# 文件打包与解压缩

## 概念

首先，了解一下常见常用的压缩包文件格式。在 Windows 上最常见的不外乎这两种 `*.zip`，`*.7z` 后缀的压缩文件。

在 Linux 上面常见的格式除了以上两种外，还有 `.rar`，`*.gz`，`*.xz`，`*.bz2`，`*.tar`，`*.tar.gz`，`*.tar.xz`，`*.tar.bz2`，简单介绍如下：

| 文件后缀名 |              说明              |
| :--------: | :----------------------------: |
|   \*.zip   |     zip 程序打包压缩的文件     |
|   \*.rar   |       rar 程序压缩的文件       |
|   \*.7z    |      7zip 程序压缩的文件       |
|   \*.tar   |   tar 程序打包，未压缩的文件   |
|   \*.gz    | gzip 程序（GNU zip）压缩的文件 |
|   \*.xz    |       xz 程序压缩的文件        |
|   \*.bz2   |      bzip2 程序压缩的文件      |
| \*.tar.gz  | tar 打包，gzip 程序压缩的文件  |
| \*.tar.xz  |  tar 打包，xz 程序压缩的文件   |
| \*tar.bz2  | tar 打包，bzip2 程序压缩的文件 |
| \*.tar.7z  |  tar 打包，7z 程序压缩的文件   |

讲了这么多种压缩文件，这么多个命令，不过我们一般只需要掌握几个命令即可，包括 `zip`，`tar`。下面会依次介绍这几个命令及对应的解压命令。

## zip 压缩打包程序

### 使用 zip 打包文件夹，注意输入完整的参数和路径：

```
cd /home/shiyanlou
zip -r -q -o shiyanlou.zip /home/shiyanlou/Desktop
du -h shiyanlou.zip
file shiyanlou.zip
```

上面命令将目录 `/home/shiyanlou/Desktop` 打包成一个文件，并查看了打包后文件的大小和类型。

第一行命令中，`-r` 参数表示递归打包包含子目录的全部内容，`-q` 参数表示为安静模式，即不向屏幕输出信息，`-o`表示输出文件，需在其后紧跟打包输出文件名。

后面使用 `du` 命令查看打包后文件的大小。

### 设置压缩级别为 9 和 1（9 最大，1 最小），重新打包：

```
zip -r -9 -q -o shiyanlou_9.zip /home/shiyanlou/Desktop -x ~/*.zip
zip -r -1 -q -o shiyanlou_1.zip /home/shiyanlou/Desktop -x ~/*.zip
```

这里添加了一个参数用于设置压缩级别 `-[1-9]`，`1` 表示最快压缩但体积大，`9` 表示体积最小但耗时最久。

最后那个 `-x` 是为了排除我们上一次创建的 zip 文件，否则又会被打包进这一次的压缩文件中，

> 注意：这里只能使用**绝对路径**，否则不起作用。

我们再用 `du` 命令分别查看默认压缩级别、最低、最高压缩级别及未压缩的文件的大小：

```
du -h -d 0 *.zip ~ | sort
```

通过 man 手册可知：

-h， --human-readable（顾名思义，你可以试试不加的情况）
-d， --max-depth（所查看文件的深度）

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/882ceef6-2f1e-4846-bc8a-fc06f03798db.png)

这样一目了然，理论上来说默认压缩级别应该是最高的，但是由于文件不大，这里的差异不明显（几乎看不出差别），不过你在环境中操作之后看到的压缩文件大小可能跟图上的有些不同，因为系统在使用过程中，会随时生成一些缓存文件在当前用户的家目录中，这对于我们学习命令使用来说，是无关紧要的，可以忽略这些不同。

### 创建加密 zip 包

使用 `-e` 参数可以创建加密压缩包：

```
zip -r -e -o shiyanlou_encryption.zip /home/shiyanlou/Desktop
```

**注意：**

关于 `zip` 命令， Windows 系统与 Linux/Unix 在文本文件格式上的一些兼容问题，比如换行符（为不可见字符），在 Windows 为 CR+LF（Carriage-Return+Line-Feed：回车加换行），而在 Linux/Unix 上为 LF（换行）。

如果在不加处理的情况下，在 Linux 上编辑的文本，在 Windows 系统上打开可能看起来是没有换行的。

如果你想让你在 Linux 创建的 zip 压缩文件在 Windows 上解压后没有任何问题，那么你还需要对命令做一些修改：

```
zip -r -l -o shiyanlou.zip /home/shiyanlou/Desktop
```

需要加上 `-l` 参数将 `LF` 转换为 `CR+LF` 来达到以上目的。

## 使用 unzip 命令解压缩 zip 文件

将 shiyanlou.zip 解压到当前目录：

```
unzip shiyanlou.zip
```

使用安静模式，将文件解压到指定目录：

```
unzip -q shiyanlou.zip -d ziptest
```

`-d`：指定文件解压缩后所要存储的目录。若指定目录不存在，将会自动创建。

如果你不想解压，只想查看压缩包的内容你可以使用 `-l` 参数：

```
unzip -l shiyanlou.zip
```

**注意：**

使用 `unzip` 解压文件时我们同样应该注意兼容问题，不过这里我们关心的不再是上面的问题，而是中文编码的问题。

通常 Windows 系统上面创建的压缩文件，如果有包含中文的文档或以中文作为文件名的文件时，默认会采用 `GBK` 或其它编码，而 Linux 上面默认使用的是 `UTF-8` 编码。

如果不加任何处理，直接解压的话可能会出现中文乱码的问题（有时候它会自动帮你处理），为了解决这个问题，我们可以在解压时指定编码类型：

使用 `-O`（英文字母，大写 o）参数指定编码类型：

```
unzip -O GBK 中文压缩文件.zip
```

## tar 打包工具

首先要弄清两个概念：`打包`和`压缩`。

`打包`是指将一大堆文件或目录变成一个总的文件，`压缩`则是将一个大的文件通过一些压缩算法变成一个小文件。

为什么要区分这两个概念呢？

这源于 Linux 中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（`tar`命令），然后再用压缩程序进行压缩（gzip bzip2 命令）。

在 Linux 上面更常用的是 `tar` 工具，`tar` 原本只是一个打包工具，只是同时还是实现了对 7z、gzip、xz、bzip2 等工具的支持。

这些压缩工具本身只能实现对文件或目录（单独压缩目录中的文件）的压缩，没有实现对文件的打包压缩，所以我们也无需再单独去学习其他几个工具，tar 的解压和压缩都是同一个命令，只需参数不同，使用比较方便。

下面先掌握 `tar` 命令一些基本的使用方式，即不进行压缩只是进行打包（创建归档文件）和解包的操作。

- 创建一个 tar 包：

```
cd /home/shiyanlou
tar -P -cf shiyanlou.tar /home/shiyanlou/Desktop
```

上面命令中，`-P` 表示保留绝对路径符，`-c` 表示创建一个 tar 包文件，`-f` 用于指定创建的文件名，注意文件名必须紧跟在 -f 参数之后，比如不能写成 `tar -fc shiyanlou.tar`，可以写成 `tar -f shiyanlou.tar -c ~`。你还可以加上 `-v` 参数以可视的的方式输出打包的文件。

- 解包一个文件（`-x` 参数）到指定路径的已存在目录（`-C` 参数）：

```
mkdir tardir
tar -xf shiyanlou.tar -C tardir
```

- 只查看不解包文件 `-t` 参数：

```
tar -tf shiyanlou.tar
```

- 保留文件属性和跟随链接（符号链接或软链接）

有时候我们使用 `tar` 备份文件，当你在其他主机还原时，希望保留文件的属性（`-p` 参数）和备份链接指向的源文件而不是链接本身（`-h` 参数）：

```
tar -cphf etc.tar /etc
```

对于创建不同的压缩格式的文件，对于 tar 来说是相当简单的，需要的只是换一个参数，这里我们就以使用 `gzip` 工具创建 `*.tar.gz` 文件为例来说明。

我们只需要在创建 tar 文件的基础上添加 `-z` 参数，使用 `gzip` 来压缩文件：

```
tar -czf shiyanlou.tar.gz /home/shiyanlou/Desktop
```

解压 \*.tar.gz 文件：

```
tar -xzf shiyanlou.tar.gz
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/9660fdfe-bc42-40f6-994f-fdbda4ec6746.png)

现在我们要使用其它的压缩工具创建或解压相应文件时，只需要更改一个参数即可：

| 压缩文件格式 | 参数 |
| :----------: | :--: |
|   *.tar.gz   |  -z  |
|   *.tar.xz   |  -J  |
|   *tar.bz2   |  -j  |

# 文件系统操作与磁盘管理
##  查看磁盘和目录的容量
- 使用 `df` 命令查看磁盘的容量：

在实验楼的环境中你将看到如下的输出内容：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/545214aa-5266-4e80-8bd1-d26dfbc4d5dc.png)

但在实际的物理主机上会更像这样：
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/fdac732e-6429-4a22-90a4-55b46ca40779.png)

物理主机上的 `/dev/sda2` 是对应着主机硬盘的分区，后面的数字表示分区号，数字前面的字母 `a` 表示第几块硬盘（也可能是可移动磁盘），你如果主机上有多块硬盘则可能还会出现 `/dev/sdb`，`/dev/sdc` 这些磁盘设备都会在 `/dev` 目录下以文件的存在形式。

接着你还会看到"`1k-块`"这个陌生的东西，它表示以`磁盘块大小`的方式显示容量，后面为相应的以块大小表示的已用和可用容量，以一种你应该看得懂的方式展示：

```
df -h
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/6498735f-d0f6-4836-a595-ac17f4af811f.png)

现在你就可以使用命令查看你主机磁盘的使用情况了。

- 使用 `du` 命令查看目录的容量

```
# 默认同样以块的大小展示
du
# 加上 `-h` 参数，以更易读的方式展示
du -h
```
`-d` 参数指定查看目录的深度

```
# 只查看 1 级目录的信息
du -h -d 0 ~
# 查看 2 级
du -h -d 1 ~
```
常用参数
| 指令  |                           解释                            |
| :---: | :-------------------------------------------------------: |
| du -h | 同 --human-readable 以 K，M，G 为单位，提高信息的可读性。 |
| du -a |            同 --all 显示目录中所有文件的大小。            |
| du -s |      同 --summarize 仅显示总计，只列出最后加总的值。      |


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1bf3d76f-7e8d-4083-b1bf-9a81db3e9007.png)

`du`（estimate file space usage）

`df`（report file system disk space usage）

## 挂载卸载磁盘
### `dd` 命令简介

`dd` 命令用于转换和复制文件，不过它的复制不同于 `cp`。

之前提到过关于 Linux 的很重要的一点，一切即文件，在 Linux 上，硬件的设备驱动（如硬盘）和特殊设备文件（如 /dev/zero 和 /dev/random）都像普通文件一样，只是在各自的驱动程序中实现了对应的功能，`dd` 也可以读取文件或写入这些文件。

这样，`dd` 也可以用在备份硬件的引导扇区、获取一定数量的随机数据或者空数据等任务中。`dd` 程序也可以在复制时处理数据，例如转换字节序、或在 ASCII 与 EBCDIC 编码间互换。

`dd` 的命令行语句与其他的 Linux 程序不同，因为它的命令行选项格式为 `选项=值`，而不是更标准的 `--选项 值` 或 `-选项=值`。

`dd` 默认从标准输入中读取，并写入到标准输出中，但可以用选项 if（input file，输入文件）和 of（output file，输出文件）改变。

我们先来试试用 `dd` 命令从标准输入读入用户的输入到标准输出或者一个文件中：

```
# 输出到文件
dd of=test bs=10 count=1 # 或者 dd if=/dev/stdin of=test bs=10 count=1
# 输出到标准输出
dd if=/dev/stdin of=/dev/stdout bs=10 count=1
# 在打完了这个命令后，继续在终端打字，作为你的输入
```

上述命令从标准输入设备读入用户输入（缺省值，所以可省略）然后输出到 test 文件。

`bs`（block size）用于指定块大小（缺省单位为 Byte，也可为其指定如 K，M，G 等单位），`count` 用于指定块数量。

如上图所示，我指定只读取总共 10 个字节的数据，当我输入了 hello shiyanlou 之后加上空格回车总共 16 个字节（一个英文字符占一个字节）内容，显然超过了设定大小。

使用 `du` 和 `cat` 10 个字节（那个黑底百分号表示这里没有换行符），而其他的多余输入将被截取并保留在标准输入。

前面说到 `dd` 在拷贝的同时还可以实现数据转换，那下面就举一个简单的例子：将输出的英文字符转换为大写再写入文件：

```
dd if=/dev/stdin of=test bs=10 count=1 conv=ucase
```

### 使用 dd 命令创建虚拟镜像文件
下面就来使用 `dd` 命令来完成创建虚拟磁盘的第一步。

从 `/dev/zero` 设备创建一个容量为 256M 的空文件：

```
dd if=/dev/zero of=virtual.img bs=1M count=256
du -h virtual.img
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/42a81eec-cbd2-43e4-9d7e-11cb536a3a94.png)


然后我们要将这个文件格式化（写入文件系统），这里我们要学到一个（准确的说是一组）新的命令来完成这个需求。

- 使用 `mkfs` 命令格式化磁盘（我们这里是自己创建的虚拟磁盘镜像）

你可以在命令行输入 `sudo mkfs `然后按下 `Tab` 键，你可以看到很多个以 `mkfs` 为前缀的命令，这些不同的后缀其实就是表示着不同的文件系统，可以用 `mkfs` 格式化成的文件系统。

我们可以简单的使用下面的命令来将我们的虚拟磁盘镜像格式化为 `ext4` 文件系统：

```
sudo mkfs.ext4 virtual.img
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/6f060d38-0f04-47a7-b3af-e0a5c00e0042.png)

可以看到实际 mkfs.ext4 是使用 `mke2fs` 来完成格式化工作的。


如果你想知道 Linux 支持哪些文件系统，你可以输入 `ls -l /lib/modules/$(uname -r)/kernel/fs` 查看。

### 使用 `mount` 命令挂载磁盘到目录树

用户在 Linux/UNIX 的机器上打开一个文件以前，包含该文件的文件系统必须先进行挂载的动作，此时用户要对该文件系统执行 `mount` 的指令以进行挂载。

该指令通常是使用在 USB 或其他可移除存储设备上，而根目录则需要始终保持挂载的状态。又因为 Linux/UNIX 文件系统可以对应一个文件而不一定要是硬件设备，所以可以挂载一个包含文件系统的文件到目录树。

Linux/UNIX 命令行的 `mount` 指令是告诉操作系统，对应的文件系统已经准备好，可以使用了，而该文件系统会对应到一个特定的点（称为`挂载点`）。挂载好的文件、目录、设备以及特殊文件即可提供用户使用。

我们先来使用 `mount` 来查看下主机已经挂载的文件系统：

```
sudo mount
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/7b51bc56-2108-4829-a058-bb8ebb1a3db9.png)

输出的结果中每一行表示一个设备或虚拟设备，每一行最前面是设备名，然后是 `on` 后面是挂载点，`type` 后面表示文件系统类型，再后面是挂载选项（比如可以在挂载时设定以只读方式挂载等等）。

- 那么我们如何挂载真正的磁盘到目录树呢？

mount 命令的一般格式如下：
`mount [options] [source] [directory]`

一些常用操作：

```
mount [-o [操作选项]] [-t 文件系统类型] [-w|--rw|--ro] [文件系统源] [挂载点]
```

现在直接来挂载我们创建的虚拟磁盘镜像到 /mnt 目录：

```
mount -o loop -t ext4 virtual.img /mnt
# 也可以省略挂载类型，很多时候 mount 会自动识别

# 以只读方式挂载
mount -o loop --ro virtual.img /mnt
# 或者 mount -o loop,ro virtual.img /mnt
```
- 使用 `umount` 命令卸载已挂载磁盘

```
# 命令格式 sudo umount 已挂载设备名或者挂载点，如：
sudo umount /mnt
```

### 使用 fdisk 为磁盘分区

下面以某物理主机为例讲解如何为磁盘分区。

```
# 查看硬盘分区表信息
sudo fdisk -l
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/6bdf3474-aa10-4ce6-b61d-9cda7868a534.png)


输出结果中开头显示了主机上的磁盘的一些信息，包括容量扇区数，扇区大小，I/O 大小等信息。

重点看一下中间的分区信息，`/dev/sda1`，`/dev/sda2` 为主分区，分别安装了 Windows 和 Linux 操作系统，`/dev/sda3` 为交换分区（可以理解为虚拟内存），`/dev/sda4` 为扩展分区其中包含` /dev/sda5`，`/dev/sda6`，`/dev/sda7`，`/dev/sda8 `四个逻辑分区，因为主机上有几个分区之间有空隙，没有对齐边界扇区，所以分区之间不是完全连续的。

```
# 进入磁盘分区模式
sudo fdisk virtual.img
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/316d6849-e12f-4f45-bc7b-3509c47bf8b2.png)

在进行操作前，首先应先规划好分区方案，这里使用 128M（可用 127M 左右）的虚拟磁盘镜像创建一个 30M 的主分区，剩余部分为扩展分区，包含 2 个大约 45M 的逻辑分区。

操作完成后输入 `p` 查看结果如下:

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/25381820-29b8-43bf-b07a-1c4512d4033d.png)


最后不要忘记输入 `w` 写入分区表。

使用 `losetup` 命令建立镜像与回环设备的关联：

```
sudo losetup /dev/loop0 virtual.img
# 如果提示设备忙你也可以使用其它的回环设备，"ls /dev/loop*"参看所有回环设备

# 解除设备关联
sudo losetup -d /dev/loop0
```
然后再使用 `mkfs` 格式化各分区（前面我们是格式化整个虚拟磁盘镜像文件或磁盘），不过格式化之前，我们还要为各分区建立虚拟设备的映射，用到 `kpartx` 工具，需要先安装：

```sudo apt-get install kpartx
sudo kpartx -av /dev/loop0

# 取消映射
sudo kpartx -dv /dev/loop0
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/c97825dc-a127-40c7-adb4-0b0f96b923fb.png)


接着再是格式化，我们将其全部格式化为 ext4：

```
sudo mkfs.ext4 -q /dev/mapper/loop0p1
sudo mkfs.ext4 -q /dev/mapper/loop0p5
sudo mkfs.ext4 -q /dev/mapper/loop0p6
```
格式化完成后在 `/media` 目录下新建四个空目录用于挂载虚拟磁盘：

```
mkdir -p /media/virtualdisk_{1..3}
```
```
# 挂载磁盘分区
sudo mount /dev/mapper/loop0p1 /media/virtualdisk_1
sudo mount /dev/mapper/loop0p5 /media/virtualdisk_2
sudo mount /dev/mapper/loop0p6 /media/virtualdisk_3

# 卸载磁盘分区
sudo umount /dev/mapper/loop0p1
sudo umount /dev/mapper/loop0p5
sudo umount /dev/mapper/loop0p6
```
然后：

```
df -h
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/0e0dddc4-580b-42f2-a095-b911b96c52ed.png)

[toc]
# 命令执行顺序控制与管道

## 命令执行顺序控制

通常情况下，我们每次只能在终端输入一条命令，按下回车执行，执行完成后，我们再输入第二条命令，然后再按回车执行。

有时候我们会一次输入多条命令，这个时候的执行过程又是如何的呢？

### 顺序执行多条命令

当我们需要使用 `apt-get` 安装一个软件，安装完成后立即运行安装的软件或命令工具，又恰巧你的主机才更换的软件源还没有更新软件列表。

比如之前我们的环境中，每次重新开始实验就得 `sudo apt-get update`，否则可能会报错提示 404，那么你可能会有如下一系列操作：

```
sudo apt-get update
# 等待执行完毕，然后输入下面的命令
sudo apt-get install some-tool # 这里 some-tool 需要替换成具体的软件包
# 等待安装完毕，然后输入软件包名称执行
some-tool
```

这时你可能就会想：要是我可以一次性输入完，让它自己去依次执行各命令就好了，这就是我们这一小节要解决的问题。

简单的顺序执行你可以使用 `;` 来完成，比如上述操作你可以：

```
sudo apt-get update;sudo apt-get install some-tool;some-tool # 让它自己运行
```

### 有选择的执行命令

关于上面的操作，如果我们在让它自动顺序执行命令时，前面的命令执行不成功，而后面的命令又依赖于上一条命令的结果，那么就会造成花了时间，最终却得到一个错误的结果，而且有时候直观的看，你还无法判断结果是否正确。

那么我们需要能够有选择性的来执行命令，比如上一条命令执行成功才继续下一条，或者不成功又该做出其它什么处理，比如我们使用 which 来查找是否安装某个命令，如果找到就执行该命令，否则什么也不做，虽然这个操作没有什么实际意义，但可帮你更好的理解一些概念：

```
which cowsay>/dev/null && cowsay -f head-in ohch~
```

你如果没有安装 `cowsay`，你可以先执行一次上述命令，你会发现什么也没发生，你再安装好之后你再执行一次上述命令，你也会发现一些惊喜。

上面的 `&&` 就是用来实现`选择性执行`的，它表示如果前面的命令执行结果（不是表示终端输出的内容，而是表示命令执行状态的结果）返回 0 则执行后面的，否则不执行，你可以从 `$?` 环境变量获取上一次命令的返回结果：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/f90d5d4a-8735-4983-add6-5e4792ca70ae.png)

学习过 C 语言的用户应该知道在 C 语言里面 `&&` 表示逻辑与，而且还有一个 `||` 表示逻辑或，同样 Shell 也有一个 `||`。

它们的区别就在于，shell 中的这两个符号除了也可用于表示逻辑与和或之外，就是可以实现这里的命令执行顺序的简单控制。

`||` 在这里就是与`&&` 相反的控制效果，当上一条命令执行结果为`≠0(\$?≠0)` 时则执行它后面的命令：

```
which cowsay>/dev/null || echo "cowsay has not been install, please run 'sudo apt-get install cowsay' to install"
```

除了上述基本的使用之外，我们还可以结合着 `&&` 和 `||` 来实现一些操作，比如：

```
which cowsay>/dev/null && echo "exist" || echo "not exist"
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/a3b4720e-d56e-4def-9ba5-f7de0d88bc31.png)

画个流程图来解释一下上面的流程：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/244717d2-71ba-4f08-98f1-c6746ef1110a.png)

## 管道

管道是一种通信机制，通常用于进程间的通信（也可通过 `socket` 进行网络通信），它表现出来的形式就是将前面每一个进程的输出（stdout）直接作为下一个进程的输入（stdin）。

管道又分为`匿名管道`和`具名管道`。

我们在使用一些过滤程序时经常会用到的就是`匿名管道`，在命令行中由 `|` 分隔符表示，`|` 在前面的内容中我们已经多次使用到了。`具名管道`简单的说就是有名字的管道，通常只会在源程序中用到具名管道。

下面我们就将通过一些常用的可以使用管道的过滤程序来帮助你熟练管道的使用。

### 试用

先试用一下管道，比如查看 /etc 目录下有哪些文件和目录，使用 `ls` 命令来查看：

```
ls -al /etc
```

有太多内容，屏幕不能完全显示，这时候可以使用滚动条或快捷键滚动窗口来查看。不过这时候可以使用管道：

```
ls -al /etc | less
```

通过管道将前一个命令(`ls`)的输出作为下一个命令(`less`)的输入，然后就可以一行一行地看。

### cut 命令，打印每一行的某一字段

打印 /etc/passwd 文件中以 `:` 为分隔符的第 1 个字段和第 6 个字段分别表示用户名和其家目录：

```
cut /etc/passwd -d ':' -f 1,6
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/647c2b95-c02d-4c16-88ad-27e150dbe2af.png)

打印 /etc/passwd 文件中每一行的前 N 个字符：

```
# 前五个（包含第五个）
cut /etc/passwd -c -5
# 前五个之后的（包含第五个）
cut /etc/passwd -c 5-
# 第五个
cut /etc/passwd -c 5
# 2 到 5 之间的（包含第五个）
cut /etc/passwd -c 2-5
```

### grep 命令，在文本中或 stdin 中查找匹配字符串

`grep` 命令是很强大的，也是相当常用的一个命令，它结合正则表达式可以实现很复杂却很高效的匹配和查找，这里介绍它简单的使用，而关于正则表达式后面将会有单独一小节介绍到时会再继续学习 `grep` 命令和其他一些命令。

grep 命令的一般形式为：
`grep [命令选项]... 用于匹配的表达式 [文件]...`

我们搜索/home/shiyanlou 目录下所有包含"shiyanlou"的文本文件，并显示出现在文本中的行号：

```
grep -rnI "shiyanlou" ~
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/64a91d0e-4190-4879-b8a2-51573afa090a.png)

`-r` 参数表示递归搜索子目录中的文件，`-n` 表示打印匹配项行号，`-I` 表示忽略二进制文件。这个操作实际没有多大意义，但可以感受到 grep 命令的强大与实用。

当然也可以在匹配字段中使用正则表达式，下面简单的演示：

```
# 查看环境变量中以 "yanlou" 结尾的字符串
export | grep ".*yanlou$"
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/ac492769-8923-4be6-81cf-2f45e23f972c.png)

其中`$`就表示一行的末尾。

### wc 命令，简单小巧的计数工具

`wc` 命令用于统计并输出一个文件中行、单词和字节的数目，比如输出 /etc/passwd 文件的统计信息：

```
wc /etc/passwd
```

分别只输出行数、单词数、字节数、字符数和输入文本中最长一行的字节数：

```
# 行数
wc -l /etc/passwd
# 单词数
wc -w /etc/passwd
# 字节数
wc -c /etc/passwd
# 字符数
wc -m /etc/passwd
# 最长行字节数
wc -L /etc/passwd
```

> 注意：对于西文字符来说，一个字符就是一个字节，但对于中文字符一个汉字是大于 2 个字节的，具体数目是由字符编码决定的。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/7d704c16-cc05-4b30-a590-b3783e15bfe3.png)

再来结合管道来操作一下，下面统计 /etc 下面所有目录数：

```
ls -dl /etc/*/ | wc -l
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/0000e869-bca1-4573-afb7-7dfed461fc0b.png)

### sort 排序命令

这个命令前面我们也是用过多次，功能很简单就是将输入按照一定方式排序，然后再输出，它支持的排序有按字典排序，数字排序，按月份排序，随机排序，反转排序，指定特定字段进行排序等等。

默认为字典排序：

```
cat /etc/passwd | sort
```

反转排序：

```
cat /etc/passwd | sort -r
```

按特定字段排序：

```
cat /etc/passwd | sort -t':' -k 3
```

上面的`-t`参数用于指定字段的分隔符，这里是以"`:`"作为分隔符；`-k` 字段号用于指定对哪一个字段进行排序。

这里/etc/passwd 文件的第三个字段为数字，默认情况下是以字典序排序的，如果要按照数字排序就要加上`-n`参数：

```
cat /etc/passwd | sort -t':' -k 3 -n
```

注意观察第二个冒号后的数字：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/f6b3c016-a990-4e00-b2a2-1f765fa00a87.png)

### uniq 去重命令

`uniq` 命令可以用于过滤或者输出重复行。

- 过滤重复行
  我们可以使用 `history` 命令查看最近执行过的命令（实际为读取 \${SHELL}\_history 文件，如我们环境中的 .zsh_history 文件），不过你可能只想查看使用了哪个命令而不需要知道具体干了什么，那么你可能就会要想去掉命令后面的参数然后去掉重复的命令：

```
history | cut -c 8- | cut -d ' ' -f 1 | uniq
```

然后经过层层过滤，你会发现确是只输出了执行的命令那一列，不过去重效果好像不明显，仔细看你会发现它确实去重了，只是不那么明显，之所以不明显是因为 `uniq` 命令**只能去连续重复的行，不是全文去重**，所以要达到预期效果，我们**先排序**：

```
history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq
# 或者
history | cut -c 8- | cut -d ' ' -f 1 | sort -u
```

- 输出重复行

```
# 输出重复过的行（重复的只输出一个）及重复次数
history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq -dc
# 输出所有重复的行
history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq -D
```

### tr 命令 删除或者替换字符
tr 命令可以用来删除一段文本信息中的某些文字，或者将其进行转换。

使用方式：
`tr [option]...SET1 [SET2]`

常用的选项有
| 选项 |                             说明                             |
| :--: | :----------------------------------------------------------: |
|  -d  | 删除和 set1 匹配的字符，注意不是全词匹配也不是按字符顺序匹配 |
|  -s  |               去除 set1 文本中连续并重复的字符               |

操作举例
```
# 删除 "hello shiyanlou" 中所有的'o'，'l'，'h'
$ echo 'hello shiyanlou' | tr -d 'olh'
# 将"hello" 中的ll，去重为一个l
$ echo 'hello' | tr -s 'l'
# 将输入文本，全部转换为大写或小写输出
$ echo 'input some text here' | tr '[:lower:]' '[:upper:]'
# 上面的'[:lower:]' '[:upper:]'你也可以简单的写作'[a-z]' '[A-Z]'，当然反过来将大写变小写也是可以的
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/fa5376ba-ebe0-4600-8b60-8b313b17368b.png)

### col 命令
col 命令可以将`Tab`换成对等数量的空格键，或反转这个操作。

使用方式：
`col [option]`

常用的选项有
| 选项 |            说明             |
| :--: | :-------------------------: |
|  -x  |       将Tab转换为空格       |
|  -h  | 将空格转换为Tab（默认选项） |

操作举例：
```
# 查看 /etc/protocols 中的不可见字符，可以看到很多 ^I ，这其实就是 Tab 转义成可见字符的符号
cat -A /etc/protocols
# 使用 col -x 将 /etc/protocols 中的 Tab 转换为空格，然后再使用 cat 查看，你发现 ^I 不见了
cat /etc/protocols | col -x | cat -A
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/3b695b2e-db84-479b-a7e6-ebb8450088b3.png)


### join 命令 合并文件
`join` 命令用于将两个文件中包含相同内容的那一行合并在一起。

使用方式
join [option]... file1 file2

常用的选项有：
| 选项 |                         说明                         |
| :--: | :--------------------------------------------------: |
|  -t  |                指定分隔符，默认为空格                |
|  -i  |                   忽略大小写的差异                   |
|  -1  | 指明第一个文件要用哪个字段来对比，默认对比第一个字段 |
|  -2  | 指明第二个文件要用哪个字段来对比，默认对比第一个字段 |

操作举例：
```
cd /home/shiyanlou
# 创建两个文件
echo '1 hello' > file1
echo '1 shiyanlou' > file2
join file1 file2
# 将 /etc/passwd 与 /etc/shadow 两个文件合并，指定以':'作为分隔符
sudo join -t':' /etc/passwd /etc/shadow
# 将 /etc/passwd 与 /etc/group 两个文件合并，指定以':'作为分隔符，分别比对第4和第3个字段
sudo join -t':' -1 4 /etc/passwd -2 3 /etc/group
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/589de178-fd48-4d86-a1a7-d534e7d82b6a.png)

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/dc75b09d-ddba-4daa-aa84-4eb5138df0ac.png)

### paste 命令 合并文件

`paste` 命令与 `join` 命令类似，它是在不对比数据的情况下，简单地将多个文件合并一起，以 `Tab` 隔开。

使用方式：
`paste [option] file...`

常用的选项有：

| 选项 |             说明             |
| :--: | :--------------------------: |
|  -d  | 指定合并的分隔符，默认为 Tab |
|  -s  | 不合并到一行，每个文件为一行 |

操作举例：
```
echo hello > file1
echo shiyanlou > file2
echo www.shiyanlou.com > file3
paste -d ':' file1 file2 file3
paste -s file1 file2 file3
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/076e18d7-7d9f-4969-a2c5-8fa6d19b581b.png)

## 校招真题
- 介绍

在 Linux 中，对于文本的处理和分析是极为重要的，现在有一个文件叫做 `data1`，可以使用下面的命令下载：

```
cd /home/shiyanlou
wget https://labfile.oss.aliyuncs.com/courses/1/data1
```
`data1` 文件里记录是一些命令的操作记录，现在需要你从里面找出出现频率次数前 3 的命令并保存在 /home/shiyanlou/result。

- 目标

处理文本文件 /home/shiyanlou/data1
将结果写入 /home/shiyanlou/result
结果包含三行内容，每行内容都是出现的次数和命令名称，如“100 ls”

- 答案
```
cat data1 |cut -c 8-|sort|uniq -dc|sort -rn -k1 |head -3 > /home/shiyanlou/result```
```

# 数据流重定向

`>` 或 `>>` 操作分别是将标准输出导向一个文件或追加到一个文件中，这其实就是`重定向`，将原本输出到标准输出的数据重定向到一个文件中，因为标准输出(/dev/stdout)本身也是一个文件，我们将命令输出导向另一个文件自然也是没有任何问题的。

## 简单的重定向

前面已经提到过 Linux 默认提供了三个特殊设备，用于终端的显示和输出，分别为 `stdin`（标准输入，对应于在终端的输入），`stdout`（标准输出，对应于终端的输出），`stderr`（标准错误输出，对应于终端的输出）。

| 文件描述符 |  设备文件   |   说明   |
| :--------: | :---------: | :------: |
|     0      | /dev/stdin  | 标准输入 |
|     1      | /dev/stdout | 标准输出 |
|     2      | /dev/stderr | 标准错误 |

> 文件描述符：文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于 UNIX、Linux 这样的操作系统。

我们可以这样使用这些文件描述符。例如默认使用终端的标准输入作为命令的输入和标准输出作为命令的输出：

```
cat # 按 Ctrl+C 退出
```
将 `cat` 的连续输出（heredoc 方式）重定向到一个文件：

```
mkdir Documents
cat > Documents/test.c <<EOF
#include <stdio.h>

int main()
{
printf("hello world\n");
return 0;
}

EOF
```
将一个文件作为命令的输入，标准输出作为命令的输出：

```
cat Documents/test.c
```
将 `echo` 命令通过管道传过来的数据作为 `cat` 命令的输入，将标准输出作为命令的输出：

```
echo 'hi' | cat
```
将 echo 命令的输出从默认的标准输出重定向到一个普通文件：

```
echo 'hello shiyanlou' > redirect
cat redirect
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/064797b5-4cdd-4293-85a6-9d2fa193c59f.png)

初学者这里要注意不要将管道和重定向混淆，管道默认是连接前一个命令的输出到下一个命令的输入，而重定向通常是需要一个文件来建立两个命令的连接，你可以仔细体会一下上述第三个操作和最后两个操作的异同点。

## 标准错误重定向
重定向标准输出到文件，这是一个很实用的操作，另一个很实用的操作是将标准错误重定向，标准输出和标准错误都被指向伪终端的屏幕显示，所以我们经常看到的一个命令的输出通常是同时包含了标准输出和标准错误的结果的。比如下面的操作：

```
# 使用cat 命令同时读取两个文件，其中一个存在，另一个不存在
cat Documents/test.c hello.c
# 你可以看到除了正确输出了前一个文件的内容，还在末尾出现了一条错误信息
# 下面我们将输出重定向到一个文件
cat Documents/test.c hello.c > somefile
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/2ae92db1-aa86-40ef-8e3e-2b76bfc24d8a.png)

遗憾的是，这里依然出现了那条错误信息，这正是因为如我上面说的那样，标准输出和标准错误虽然都指向终端屏幕，实际它们并不一样。

那有的时候我们就是要隐藏某些错误或者警告，那又该怎么做呢。这就需要用到我们前面讲的`文件描述符`了：

```
# 将标准错误重定向到标准输出，再将标准输出重定向到文件，注意要将重定向到文件写到前面
cat Documents/test.c hello.c >somefile  2>&1
# 或者只用bash提供的特殊的重定向符号"&"将标准错误和标准输出同时重定向到文件
cat Documents/test.c hello.c &>somefilehell
```

注意你应该在输出重定向文件描述符前加上`&`，否则 shell 会当做重定向到一个文件名为 1 的文件中。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/ac4540f4-452d-4002-a4bc-777a1bb05b39.png)


## 使用 tee 命令同时重定向到多个文件
你可能还有这样的需求，除了需要将输出重定向到文件，也需要将信息打印在终端。那么你可以使用 `tee` 命令来实现：`tee [OPTION]... [FILE]...`


```
echo 'hello shiyanlou' | tee hello
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/a56075d8-2a52-46dc-bc2d-19f6c130421d.png)


## 永久重定向
前面的重定向操作都只是临时性的，即只对当前命令有效，那如何做到永久有效呢？

比如在一个脚本中，你需要某一部分的命令的输出全部进行重定向，难道要让你在每个命令上面加上临时重定向的操作吗？

当然不需要，我们可以使用 `exec` 命令实现永久重定向。

`exec` 命令的作用是使用指定的命令替换当前的 Shell，即使用一个进程替换当前进程，或者指定新的重定向：

```
# 先开启一个子 Shell
zsh
# 使用exec替换当前进程的重定向，将标准输出重定向到一个文件
exec 1>somefile
# 后面你执行的命令的输出都将被重定向到文件中，直到你退出当前子shell，或取消exec的重定向（后面将告诉你怎么做）
ls
exit
cat somefile
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/b64c5a0d-a56f-4a50-ae65-b6eedd373b39.png)


## 创建输出文件描述符
在 Shell 中有 9 个文件描述符。上面我们使用了也是它默认提供的 0，1，2 号文件描述符。

另外我们还可以使用 3-8 的文件描述符，只是它们默认没有打开而已。你可以使用下面命令查看当前 Shell 进程中打开的文件描述符：

```
cd /dev/fd/;ls -Al
```
同样使用 `exec` 命令可以创建新的文件描述符：

```
zsh
exec 3>somefile
# 先进入目录，再查看，否则你可能不能得到正确的结果，然后再回到上一次的目录
cd /dev/fd/;ls -Al;cd -
# 注意下面的命令>与&之间不应该有空格，如果有空格则会出错
echo "this is test" >&3
cat somefile
exit
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/cc487579-057c-4bde-9117-c9c0b01344a1.png)

上面打开的 3 号文件描述符，可以使用如下操作将它关闭：

```
exec 3>&-
cd /dev/fd;ls -Al;cd -
```

## 完全屏蔽命令的输出
在 Linux 中有一个被称为黑洞的设备文件，所有导入它的数据都将被吞噬。

在类 UNIX 系统中，`/dev/null`，或称空设备，是一个特殊的设备文件，它通常被用于丢弃不需要的输出流，或作为用于输入流的空文件，这些操作通常由重定向完成。读取它则会立即得到一个 EOF。

我们可以利用 `/dev/null` 屏蔽命令的输出：

```
cat Documents/test.c 1>/dev/null 2>&1
```
上面这样的操作将使你得不到任何输出结果。


## 使用 xargs 分割参数列表
`xargs` 命令的作用是将参数列表转换成小块分段传递给其他命令，以避免参数列表过长的问题。

这个命令在有些时候十分有用，特别是当用来处理产生大量输出结果的命令如 `find`，`locate` 和 `grep` 的结果。

```
cut -d: -f1 < /etc/passwd | sort | xargs echo
```
上面这个命令用于将 `/etc/passwd` 文件按 `:` 分割取第一个字段排序后，使用 `echo` 命令生成一个列表。

# 正则表达式基础

## 概述

正则表达式，英语：Regular Expression，在代码中常简写为 regex、regexp 或 RE），计算机科学的一个概念。

正则表达式使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。在很多文本编辑器里，正则表达式通常被用来检索、替换那些符合某个模式的文本。

许多程序设计语言都支持利用正则表达式进行字符串操作。例如，在 Perl 中就内建了一个功能强大的正则表达式引擎。正则表达式这个概念最初是由 UNIX 中的工具软件（例如`sed`和`grep`）普及开的。

简单的说形式和功能上，正则表达式和我们前面讲的通配符很像，不过它们之间又有很大差别，特别在于一些特殊的匹配字符的含义上。

假设我们有这样一个文本文件，包含 shiyanlou 和 shilouyan 这两个字符串，同样一个表达式：

```
shi*
```

如果这作为一个正则表达式，它将只能匹配 shi，而如果不是作为正则表达式 `*` 作为一个通配符，则将同时匹配这两个字符串。这是为什么呢？

因为在正则表达式中 `*` 表示`匹配前面的子表达式`（这里就是它前面一个字符）零次或多次，比如它可以匹配 sh，shii，shish，shiishi 等等，而作为通配符表示匹配通配符后面任意多个任意字符，所以它可以匹配 shiyanlou 和 shilouyan 两个字符。

## 基本语法

一个正则表达式通常被称为一个`模式`（pattern），为用来描述或者匹配一系列符合某个句法规则的字符串。

- 选择

`|` 竖直分隔符表示选择，例如 `boy|girl` 可以匹配 boy 或者 girl。

- 数量限定

数量限定除了我们举例用的 `\*` 还有 `+` 加号 `?` 问号，如果在一个模式中不加数量限定符则表示出现一次且仅出现一次：

`+` 表示前面的字符必须出现至少一次(1 次或多次)，例如 `goo+gle` 可以匹配 gooogle，goooogle 等；

  `?` 表示前面的字符最多出现一次（0 次或 1 次），例如，`colou?r`，可以匹配 color 或者 colour;

`*` 星号代表前面的字符可以不出现，也可以出现一次或者多次（0 次、或 1 次、或多次），例如，`0\*42` 可以匹配 42、042、0042、00042 等。

  - 范围和优先级

  `()` 圆括号可以用来定义模式字符串的范围和优先级，这可以简单的理解为是否将括号内的模式串作为一个整体。

  例如，`gr(a|e)y` 等价于 `gray|grey`，（这里体现了优先级，竖直分隔符用于选择 a 或者 e 而不是 gra 和 ey），`(grand)?father` 匹配 father 和 grandfather（这里体现了范围，? 将圆括号内容作为一个整体匹配）。

- 语法（部分）

正则表达式有多种不同的风格，下面列举一些常用的作为 PCRE 子集的适用于 perl 和 python 编程语言及 grep 或 egrep 的正则表达式匹配规则：

> PCRE（Perl Compatible Regular Expressions 中文含义：perl 语言兼容正则表达式）是一个用 C 语言编写的正则表达式函数库，由菲利普.海泽(Philip Hazel)编写。PCRE 是一个轻量级的函数库，比 Boost 之类的正则表达式库小得多。PCRE 十分易用，同时功能也很强大，性能超过了 POSIX 正则表达式库和一些经典的正则表达式库。


| 字符  |                             描述                             |
| :---: | :----------------------------------------------------------: |
|   \   | 将下一个字符标记为一个特殊字符、或一个原义字符。 例如 `n` 匹配字符 `n`，`\n` 匹配一个换行符，序列 `\\` 匹配 `\` ，`\(` 则匹配 `(`。 |
|   ^   |                匹配输入字符串的**开始**位置。                |
|   $   |                匹配输入字符串的**结束**位置。                |
|  {n}  | n 是一个非负整数。**匹配确定的 n 次**。例如 o{2} 不能匹配 Bob 中的 o，但是能匹配 food 中的两个 o。 |
| {n,}  | n 是一个非负整数。**至少匹配 n 次**。例如 o{2,} 不能匹配 Bob 中的 o，但能匹配 foooood 中的所有 o。o{1,} 等价于 o+。o{0,} 则等价于 o\*。 |
| {n,m} | m 和 n 均为非负整数，其中 n<=m。**最少匹配 n 次且最多匹配 m 次**。例如，o{1,3} 将匹配 fooooood 中的前三个 o。o{0,1} 等价于 o?。请注意在逗号和两个数之间不能有空格。 |
`*` |匹配前面的子表达式零次或多次。例如，zo* 能匹配 z、zo 以及 zoo。* 等价于 {0,}。
`+` |匹配前面的子表达式一次或多次。例如，zo+ 能匹配 zo 以及 zoo，但不能匹配 z。+ 等价于 {1,}。
  ? |匹配前面的子表达式零次或一次。例如，do(es)? 可以匹配 do 或 does 中的 do。? 等价于 {0,1}。
  ? |当该字符紧跟在任何一个其他限制符（\*，+，?，{n}，{n,}，{n,m}）后面时，匹配模式是**非贪婪**的。非贪婪模式尽可能少的匹配所搜索的字符串，而默认的贪婪模式则尽可能多的匹配所搜索的字符串。例如，对于字符串 oooo，o+? 将匹配单个 o，而 o+ 将匹配所有 o。
  . |**匹配除 \n 之外的任何单个字符**。要匹配包括 \n 在内的任何字符，请使用类似 `(.｜\n)` 的模式。
  (pattern) |**匹配 pattern 并获取这一匹配的子字符串**。该子字符串用于向后引用。要匹配圆括号字符，请使用 \( 和 \)。
   `x | y` |匹配 x 或 y。例如，“z ｜ food”能匹配 z 或 food。“(z ｜ f)ood”则匹配 zood 或 food。
  [xyz] |字符集合（character class）。**匹配所包含的任意一个字符**。例如，[abc] 可以匹配 plain 中的 a。其中特殊字符仅有反斜线 \ 保持特殊含义，用于转义字符。其它特殊字符如星号、加号、各种括号等均作为普通字符。脱字符^如果出现在首位则表示负值字符集合；如果出现在字符串中间就仅作为普通字符。连字符 - 如果出现在字符串中间表示字符范围描述；如果出现在首位则仅作为普通字符。
  [^xyz] |排除型（negate）字符集合。**匹配未列出的任意字符**。例如，[^abc] 可以匹配 plain 中的 plin。
  [a-z] |字符范围。**匹配指定范围内的任意字符**。例如，[a-z] 可以匹配 a 到 z 范围内的任意小写字母字符。
  [^a-z]| 排除型的字符范围。**匹配任何不在指定范围内的任意字符**。例如，[^a-z] 可以匹配任何不在 a 到 z 范围内的任意字符。


+ 优先级

优先级为从上到下从左到右，依次降低：

|           运算符           |     说明     |
| :------------------------: | :----------: |
|             \              |    转义符    |
|     ()，(?:)，(?=)，[]     | 括号和中括号 |
| \*，+，?，{n}，{n,}，{n,m} |    限定符    |
|    ^，\$，\ 任何元字符     | 定位点和序列 |
|             ｜             |     选择     |

# grep 模式匹配命令

## 基本操作
`grep` 命令用于打印输出文本中匹配的模式串，它使用正则表达式作为模式匹配的条件。grep 支持三种正则表达式引擎，分别用三个参数指定：

| 参数 |           说明            |
| :--: | :-----------------------: |
|  -E  | POSIX 扩展正则表达式，ERE |
|  -G  | POSIX 基本正则表达式，BRE |
|  -P  |   Perl 正则表达式，PCRE   |

不过在你没学过 perl 语言的大多数情况下你将只会使用到 ERE 和 BRE，所以我们接下来的内容都不会讨论到 PCRE 中特有的一些正则表达式语法（它们之间大部分内容是存在交集的，所以你不用担心会遗漏多少重要内容）。

在通过grep命令使用正则表达式之前，先介绍一下它的常用参数：

|     参数     |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|      -b      |                将二进制文件作为文本来进行匹配                |
|      -c      |                     统计以模式匹配的数目                     |
|      -i      |                          忽略大小写                          |
|      -n      |                   显示匹配文本所在行的行号                   |
|      -v      |                   反选，输出不匹配行的内容                   |
|      -r      |                         递归匹配查找                         |
|     -A n     | n 为正整数，表示 after 的意思，除了列出匹配行之外，还列出后面的 n 行 |
|     -B n     | n 为正整数，表示 before 的意思，除了列出匹配行之外，还列出前面的 n 行 |
| --color=auto |              将输出中的匹配项设置为自动颜色显示              |

> 注：在大多数发行版中是默认设置了 grep 的颜色的，你可以通过参数指定或修改GREP_COLOR环境变量。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/36df5bf1-dd99-4a32-8c01-0078904e7daf.png)


## 使用正则表达式
**使用基本正则表达式，BRE**
- 位置

查找 /etc/group 文件中以 shiyanlou 为开头的行

```
grep 'shiyanlou' /etc/group
grep '^shiyanlou' /etc/group
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/bb1f0b87-8206-404d-84cf-7ddd0c653252.png)

- 数量
```
# 将匹配以'z'开头以'o'结尾的所有字符串
echo 'zero\nzo\nzoo' | grep 'z.*o'
# 将匹配以'z'开头以'o'结尾，中间包含一个任意字符的字符串
echo 'zero\nzo\nzoo' | grep 'z.o'
# 将匹配以'z'开头，以任意多个'o'结尾的字符串
echo 'zero\nzo\nzoo' | grep 'zo*'
```
注意：其中 \n 为换行符


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/582137d3-1cdb-4211-a64e-7cfbc2e3411c.png)

- 选择
```
# grep默认是区分大小写的，这里将匹配所有的小写字母
echo '1234\nabcd' | grep '[a-z]'
# 将匹配所有的数字
echo '1234\nabcd' | grep '[0-9]'
# 将匹配所有的数字
echo '1234\nabcd' | grep '[[:digit:]]'
# 将匹配所有的小写字母
echo '1234\nabcd' | grep '[[:lower:]]'
# 将匹配所有的大写字母
echo '1234\nabcd' | grep '[[:upper:]]'
# 将匹配所有的字母和数字，包括0-9，a-z，A-Z
echo '1234\nabcd' | grep '[[:alnum:]]'
# 将匹配所有的字母
echo '1234\nabcd' | grep '[[:alpha:]]'
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/e7956578-a977-4a5f-865a-1df22ca4e289.png)

下面包含完整的特殊符号及说明：

|   特殊符号   |                             说明                             |
| :----------: | :----------------------------------------------------------: |
| [[:alnum:]]  |         代表英文大小写字母及数字，亦即 0-9，A-Z，a-z         |
|  [:alpha:]]  |            代表任何英文大小写字母，亦即 A-Z，a-z             |
| [[]:blank:]] |                 代表空白键与 [Tab] 按键两者                  |
| [[:cntrl:]]  |     代表键盘上面的控制按键，亦即包括 CR，LF，Tab，Del...     |
| [[:digit:]]  |                    代表数字而已，亦即 0-9                    |
| [[:graph:]]  |     除了空白字节（空白键与 [Tab] 按键）外的其他所有按键      |
| [[:lower:]]  |                    代表小写字母，亦即 a-z                    |
| [[:print:]]  |                 代表任何可以被列印出来的字符                 |
| [[:punct:]]  | 代表标点符号（punctuation symbol），即："，'，?，!，;，:，#，$... |
| [[:upper:]]  |                    代表大写字母，亦即 A-Z                    |
| [[:space:]]  |       任何会产生空白的字符，包括空格键，[Tab]，CR 等等       |
| [[:xdigit:]] | 代表 16 进位的数字类型，因此包括： 0-9，A-F，a-f 的数字与字节 |

- 注意

之所以要使用特殊符号，是因为上面的 [a-z] 不是在所有情况下都管用，这还与主机当前的语系有关，即设置在 LANG 环境变量的值，zh_CN.UTF-8 的话 [a-z]，即为所有小写字母，其它语系可能是大小写交替的如，"a A b B...z Z"，[a-z] 中就可能包含大写字母。所以在使用 [a-z] 时请确保当前语系的影响，使用 [:lower:] 则不会有这个问题。

```
# 排除字符
echo 'geek\ngood' | grep '[^o]'
```
 `^` 放到中括号内为排除字符，否则表示行首。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/9e6c02ba-f429-4ede-ad25-539f101f325d.png)

**使用扩展正则表达式，ERE**

要通过 `grep` 使用`扩展正则表达式`需要加上 `-E` 参数，或使用 `egrep`。

- 数量
```
# 只匹配"zo"
echo 'zero\nzo\nzoo' | grep -E 'zo{1}'
# 匹配以"zo"开头的所有单词
echo 'zero\nzo\nzoo' | grep -E 'zo{1,}'
```
推荐掌握 `{n,m}` 即可 +，?，* 这几个不太直观，且容易弄混淆。

- 选择
```
# 匹配"www.shiyanlou.com"和"www.google.com"
echo 'www.shiyanlou.com\nwww.baidu.com\nwww.google.com' | grep -E 'www\.(shiyanlou|google)\.com'
# 或者匹配不包含"baidu"的内容
echo 'www.shiyanlou.com\nwww.baidu.com\nwww.google.com' | grep -Ev 'www\.baidu\.com'
```
因为 `.` 号有特殊含义，所以需要转义。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/cd84cf3c-7cb3-4b66-bf88-c46b37aa8091.png)

# sed 流编辑器
`sed` 工具在 man 手册里面的全名为"sed - stream editor for filtering and transforming text "，意即，用于过滤和转换文本的流编辑器。

在 Linux/UNIX 的世界里敢称为编辑器的工具，大都非等闲之辈，比如前面的 vi/vim（编辑器之神），emacs（神的编辑器），gedit 这些编辑器。

`sed` 与上述的最大不同之处在于它是一个`非交互式`的编辑器，下面我们就开始介绍 `sed` 这个编辑器。

## sed 常用参数介绍
sed 命令基本格式：
`sed [参数]... [执行命令] [输入文件]...`

```
# 形如：
$ sed -i 's/sad/happy/' test # 表示将test文件中的"sad"替换为"happy"
```
|    参数     |                             说明                             |
| :---------: | :----------------------------------------------------------: |
|     -n      |    安静模式，只打印受影响的行，默认打印输入数据的全部内容    |
|     -e      | 用于在脚本中添加多个执行命令一次执行，在命令行中执行多个命令通常不需要加该参数 |
| -f filename |                指定执行 filename 文件中的命令                |
|     -r      |           使用扩展正则表达式，默认为标准正则表达式           |
|     -i      |       将直接修改输入文件内容，而不是打印到标准输出设备       |

## sed 编辑器的[执行命令]
`sed` 执行命令格式：
`[n1][,n2]command`
`[n1][~step]command`

其中一些命令可以在后面加上作用范围，形如：

```
sed -i 's/sad/happy/g' test # g 表示全局范围
sed -i 's/sad/happy/4' test # 4 表示指定行中的第四个匹配字符串
```
其中 `n1`,`n2` 表示输入内容的行号，它们之间为 `,` 逗号则表示从 `n1` 到 `n2` 行，如果为 `~` 波浪号则表示从 `n1` 开始以 `step` 为步进的所有行；`command` 为执行动作，下面为一些常用动作指令：

| 命令 |                说明                |
| :--: | :--------------------------------: |
|  s   |              行内替换              |
|  c   |              整行替换              |
|  a   |         插入到指定行的后面         |
|  i   |         插入到指定行的前面         |
|  p   | 打印指定行，通常与 -n 参数配合使用 |
|  d   |             删除指定行             |

## sed 操作举例
我们先找一个用于练习的文本文件：

```
cp /etc/passwd ~
```
- 打印指定行
```
# 打印2-5行
nl passwd | sed -n '2,5p'
# 打印奇数行
nl passwd | sed -n '1~2p'
# nl命令作用是添加行号
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/0be2e9e1-e589-4c49-97a7-163dfc3f9827.png)

- 行内替换
```
# 将输入文本中"shiyanlou" 全局替换为"hehe"，并只打印替换的那一行，注意这里不能省略最后的"p"命令
sed -n 's/shiyanlou/hehe/gp' passwd
```
行内替换可以结合正则表达式使用。

- 删除某行

```
nl passwd | grep "shiyanlou"
# 删除第30行
sed -i '30d' passwd
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/395773e1-ff02-4fc7-8f51-76e396115fc4.png)

更多 sed 用法详见：https://coolshell.cn/articles/9104.html

# awk 文本处理语言

## awk 的一些基础概念
`awk` 所有的操作都是基于 pattern(模式)—action(动作)对来完成的，如下面的形式：
`pattern {action}`

你可以看到就如同很多编程语言一样，它将所有的动作操作用一对 `{}` 花括号包围起来。

其中 `pattern` 通常是表示用于匹配输入的文本的“关系式”或“正则表达式”，`action` 则是表示匹配后将执行的动作。

在一个完整 `awk` 操作中，这两者可以只有其中一个，如果没有 `pattern` 则默认匹配输入的全部文本，如果没有 `action` 则默认为打印匹配内容到屏幕。

`awk` 处理文本的方式，是将文本分割成一些“字段”，然后再对这些字段进行处理，默认情况下，`awk` 以空格作为一个字段的分割符，不过这不是固定的，你可以任意指定分隔符。

## awk 命令基本格式
`awk [-F fs] [-v var=value] [-f prog-file | 'program text'] [file...]`

其中 `-F` 参数用于预先指定前面提到的字段分隔符（还有其他指定字段的方式），`-v` 用于预先为 `awk` 程序指定变量，`-f` 参数用于指定 `awk` 命令要执行的程序文件，或者在不加 `-f` 参数的情况下直接将程序语句放在这里，最后为 `awk` 需要处理的文本输入，且可以同时输入多个文本文件。

## awk 操作体验
先用 vim 新建一个文本文档：

```
vim test
```
包含如下内容：

```
I like linux
www.shiyanlou.com
```
- 使用 awk 将文本内容打印到终端：
```php
# "quote>" 不用输入
awk '{
quote> print
quote> }' test
# 或者写到一行
awk '{print}' test
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/3eea2e6e-b546-4aa5-ba83-f067491fe853.png)


说明:在这个操作中我是省略了 pattern，所以 awk 会默认匹配输入文本的全部内容，然后在 `{}` 花括号中执行动作，即 print 打印所有匹配项，这里是全部文本内容。

- 将 test 的第一行的每个字段单独显示为一行：
```php
$ awk '{
> if(NR==1){
> print $1 "\n" $2 "\n" $3
> } else {
> print}
> }' test
# 或者
$ awk '{
> if(NR==1){
> OFS="\n"
> print $1, $2, $3
> } else {
> print}
> }' test
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1382af4e-072a-4eef-aea7-77da39d2069a.png)


 `NR` 与 `OFS`，这两个是 awk 内建的变量，`NR` 表示当前读入的记录数，你可以简单的理解为当前处理的行数，`OFS` 表示输出时的字段分隔符，默认为"` `"空格。

如上图所见，我们将字段分隔符设置为 `\n` 换行符，所以第一行原本以空格为字段分隔的内容就分别输出到单独一行了。

然后是 `$N` 其中 N 为相应的字段号，这也是 awk 的内建变量，它表示引用相应的字段，因为我们这里第一行只有三个字段，所以只引用到了 `$3`。除此之外另一个这里没有出现的 $0，它表示引用当前记录（当前行）的全部内容。

- 将 test 的第二行的以点为分段的字段换成以空格为分隔：
```php
awk -F'.' '{
> if(NR==2){
> print $1 "\t" $2 "\t" $3
> }}' test
# 或者
awk '
> BEGIN{
> FS="."
> OFS="\t"  # 如果写为一行，两个动作语句之间应该以";"号分开
> }{
> if(NR==2){
> print $1, $2, $3
> }}' test
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/1d4ce074-6333-4418-8d74-e87ef35abc7f.png)

这里的 `-F` 参数，用来预先指定待处理记录的字段分隔符。

除了指定 `OFS` ，我们还可以在 print 语句中直接打印特殊符号，如这里的 `\t`，print 打印的非变量内容都需要用`""`一对引号包围起来。

上面另一个版本，展示了实现预先指定变量分隔符的另一种方式，即使用 `BEGIN`，这个表达式指示了，其后的动作将在所有动作之前执行，这里 `FS` 赋值了新的 `.` 点号代替默认的空格。


## awk 常用的内置变量
|  变量名  |                             说明                             |
| :------: | :----------------------------------------------------------: |
| FILENAME | 当前输入文件名，若有多个文件，则只表示第一个。如果输入是来自标准输入，则为空字符串 |
|    $0    |                        当前记录的内容                        |
|    $N    |               N 表示字段号，最大值为NF变量的值               |
|    FS    |           字段分隔符，由正则表达式表示，默认为空格           |
|    RS    |         输入记录分隔符，默认为 \n，即一行为一个记录          |
|    NF    |                        当前记录字段数                        |
|    NR    |                       已经读入的记录数                       |
|   FNR    |          当前输入文件的记录数，请注意它与 NR 的区别          |
|   OFS    |                  输出字段分隔符，默认为空格                  |
|   ORS    |                  输出记录分隔符，默认为 \n                   |

更多使用方法见：

AWK 简明教程：https://coolshell.cn/articles/9070.html

awk 程序设计语言：https://awk.readthedocs.io/en/latest/chapter-one.html#awk

# 软件安装
## 开始
### apt 包管理工具介绍
`APT` 是 Advance Packaging Tool（高级包装工具）的缩写，是 Debian 及其派生发行版的软件包管理器，APT 可以自动下载，配置，安装二进制或者源代码格式的软件包，因此简化了 Unix 系统上管理软件的过程。

APT 最早被设计成 dpkg 的前端，用来处理 deb 格式的软件包。现在经过 APT-RPM 组织修改，APT 已经可以安装在支持 RPM 的系统管理 RPM 包。

这个包管理器包含以 apt- 开头的多个工具，如 apt-get apt-cache apt-cdrom 等，在 Debian 系列的发行版中使用。

当你在执行安装操作时，首先 `apt-get` 工具会在本地的一个数据库中搜索关于  软件的相关信息，并根据这些信息在相关的服务器上下载软件安装，这里大家可能会一个疑问：既然是在线安装软件，为啥会在本地的数据库中搜索？要解释这个问题就得提到几个名词了：

- 软件源镜像服务器

- 软件源

我们需要定期从服务器上下载一个软件包列表，使用 `sudo apt-get update` 命令来保持本地的软件包列表是最新的（有时你也需要手动执行这个操作，比如更换了软件源），而这个表里会有软件依赖信息的记录。

对于软件依赖，举个例子：我们安装 w3m 软件的时候，而这个软件需要 libgc1c2 这个软件包才能正常工作，这个时候 `apt-get` 在安装软件的时候会一并替我们安装了，以保证 软件 能正常的工作。

### apt-get
`apt-get` 是用于处理 apt 包的公用程序集，我们可以用它来在线安装、卸载和升级软件包等，下面列出一些 `apt-get` 包含的常用的一些工具：

|     工具     |                             说明                             |
| :----------: | :----------------------------------------------------------: |
|   install    |             其后加上软件包名，用于安装一个软件包             |
|    update    | 从软件源镜像服务器上下载/更新用于更新本地软件源的软件包列表  |
|   upgrade    | 升级本地可更新的全部软件包，但存在依赖问题时将不会升级，通常会在更新之前执行一次 update |
| dist-upgrade |             解决依赖关系并升级（存在一定危险性）             |
|    remove    | 移除已安装的软件包，包括与被移除软件包有依赖关系的软件包，但不包含软件包的配置文件 |
|    purge     |      与 remove 相同，但会完全移除软件包，包含其配置文件      |
|  autoremove  |      移除之前被其他软件包依赖，但现在不再被使用的软件包      |
|    clean     | 移除下载到本地的已经安装的软件包，默认保存在 /var/cache/apt/archives/ |
|  autoclean   |                移除已安装的软件的旧版本软件包                |

下面是一些 `apt-get` 常用的参数：

|        参数        |                             说明                             |
| :----------------: | :----------------------------------------------------------: |
|         -y         | 自动回应是否安装软件包的选项，在一些自动化安装脚本中使用这个参数将十分有用 |
|         -s         |                           模拟安装                           |
|         -q         | 静默安装方式，指定多个 q 或者 -q=#，# 表示数字，用于设定静默级别，这在你不想要在安装软件包时屏幕输出过多时很有用 |
|         -f         |                      修复损坏的依赖关系                      |
|         -d         |                         只下载不安装                         |
|    --reinstall     |            重新安装已经安装但可能存在问题的软件包            |
| --install-suggests |             同时安装 APT 给出的建议安装的软件包              |

### 安装软件包
关于安装，只需要执行 `apt-get install <packagename>` 即可，除了这一点，还应该掌握如何`重新安装`软件包。

很多时候我们需要重新安装一个软件包，比如你的系统被破坏，或者一些错误的配置导致软件无法正常工作。你可以使用如下方式重新安装：
```
sudo apt-get --reinstall install <packagename>
```
另一个需要掌握的是，如何在不知道软件包完整名的时候进行安装。

通常我们使用 `Tab` 键补全软件包名。有时候需要同时安装多个软件包，你还可以使用正则表达式匹配软件包名进行批量安装。

### 软件升级
```php
# 更新软件源
sudo apt-get update
# 升级没有依赖问题的软件包
sudo apt-get upgrade
# 升级并解决依赖关系
sudo apt-get dist-upgrade
```
### 卸载软件
如果你现在觉得 w3m 这个软件不合自己的胃口或者是找到了更好的，你需要卸载它。同样是一个命令加回车 `sudo apt-get remove w3m`，系统会有一个确认的操作，之后这个软件就被卸载了。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/3088e5a9-119e-4508-857d-c21e340062e2.png)

或者，你可以执行
```php
# 不保留配置文件的移除
sudo apt-get purge w3m
# 或者
sudo apt-get --purge remove w3m
# 移除不再需要的被依赖的软件包
sudo apt-get autoremove
```

### 软件搜索
当自己刚知道了一个软件，想下载使用，需要确认软件仓库里面有没有，就需要用到搜索功能了，命令如下：

```php
sudo apt-cache search softname1 softname2 softname3……
```
`apt-cache` 命令则是针对本地数据进行相关操作的工具，search 顾名思义在本地的数据库中寻找有关 softname1，softname2 相关软件的信息。

更多详细内容见：https://www.debian.org/doc/manuals/apt-howto/index.zh-cn.html#contents

## 使用 dpkg
本节讲解如何使用 `dpkg` 从本地磁盘安装 `deb` 软件包。

`dpkg` 是 Debian 软件包管理器的基础，它被伊恩·默多克创建于 1993 年。dpkg 与 RPM 十分相似，同样被用于安装、卸载和供给和 `.deb` 软件包相关的信息。

`dpkg`(Debian Package) 本身是一个底层的工具。上层的工具，像是 APT，被用于从远程获取软件包以及处理复杂的软件包关系。

我们经常可以在网络上见到以 `deb` 形式打包的软件包，就需要使用 `dpkg` 命令来安装。

`dpkg` 常用参数介绍：

| 参数 |                       说明                        |
| :--: | :-----------------------------------------------: |
|  -i  |                  安装指定 deb 包                  |
|  -R  | 后面加上目录名，用于安装该目录下的所有 deb 安装包 |
|  -r  |          remove，移除某个已安装的软件包           |
|  -I  |               显示 deb 包文件的信息               |
|  -s  |               显示已安装软件的信息                |
|  -S  |                搜索已安装的软件包                 |
|  -L  |            显示已安装软件包的目录信息             |

### 使用 dpkg 安装 deb 软件包
我们先使用apt-get加上-d参数只下载不安装，下载 emacs 编辑器的 deb 包：

```php
sudo apt-get update
sudo apt-get -d install -y emacs
```
下载完成后，我们可以查看/var/cache/apt/archives/目录下的内容，如下图：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/68cc9f9f-e158-4421-87ed-d7c764785154.png)

然后我们将第一个deb拷贝到 /home/shiyanlou 目录下，并使用 `dpkg` 安装

```php
cp /var/cache/apt/archives/emacs24_24.5+1-6ubuntu1.1_amd64.deb ~
# 安装之前参看deb包的信息
sudo dpkg -I emacs24_24.5+1-6ubuntu1.1_amd64.deb
```

如你所见，这个包还额外依赖了一些软件包，这意味着，如果主机目前没有这些被依赖的软件包，直接使用 `dpkg` 安装可能会存在一些问题，`因为 dpkg 并不能为你解决依赖关系`。

```php
# 使用dpkg安装
sudo dpkg -i emacs24_24.5+1-6ubuntu1.1_amd64.deb
```
跟前面预料的一样，这里你可能出现了一些错误：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/167993cf-df85-4816-b394-fd166f8cfcbc.png)

我们将如何解决这个错误呢？这就要用到 `apt-get` 了，使用它的`-f`参数了，修复依赖关系的安装：

```
sudo apt-get update
sudo apt-get -f install -y
```
没有任何错误，这样我们就安装成功了，然后你可以运行 emacs 程序


### 查看已安装软件包的安装目录
若想知道到底 linux 将软件安装到了什么地方，可以通过 `dpkg` 找到答案

使用`dpkg -L`查看deb包目录信息

```php
sudo dpkg -L emacs24
```

## 从二进制包安装

二进制包的安装比较简单，我们需要做的只是将从网络上下载的二进制包解压后放到合适的目录，然后将包含可执行的主程序文件的目录添加进 `PATH` 环境变量即可。

# 进程概念
## 概念理解
首先程序与进程是什么？程序与进程又有什么区别？

程序（procedure）：不太精确地说，程序就是执行一系列有逻辑、有顺序结构的指令，帮我们达成某个结果。

进程（process）：进程是程序在一个数据集合上的一次执行过程，在早期的 UNIX、Linux 2.4 及更早的版本中，它是系统进行资源分配和调度的独立基本单位。

简单来说，程序是为了完成某种任务而设计的软件，比如 vim 是程序。什么是进程呢？进程就是运行中的程序。

程序只是一些列指令的集合，是一个静止的实体，而进程不同，进程有以下的特性：

- 动态性：进程的实质是一次程序执行的过程，有创建、撤销等状态的变化。而程序是一个静态的实体。
- 并发性：进程可以做到在一个时间段内，有多个程序在运行中。程序只是静态的实体，所以不存在并发性。
- 独立性：进程可以独立分配资源，独立接受调度，独立地运行。
- 异步性：进程以不可预知的速度向前推进。
- 结构性：进程拥有代码段、数据段、PCB（进程控制块，进程存在的唯一标志）。也正是因为有结构性，进程才可以做到独立地运行。

并发：在一个时间段内，宏观来看有多个程序都在活动，有条不紊的执行（每一瞬间只有一个在执行，只是在一段时间有多个程序都执行过）

并行：在每一个瞬间，都有多个程序都在同时执行，这个必须有多个 CPU 才行

引入`进程`是因为传统意义上的程序已经不足以描述 OS 中各种活动之间的动态性、并发性、独立性还有相互制约性。

`程序`就像一个公司，只是一些证书，文件的堆积（静态实体）。而当公司运作起来就有各个部门的区分，财务部，技术部，销售部等等，就像各个`进程`，各个部门之间可以独立运作，也可以有交互（独立性、并发性）。

而随着程序的发展越做越大，又会继续细分，从而引入了`线程`的概念，当代多数操作系统、Linux 2.6 及更新的版本中，进程本身不是基本运行单位，而是`线程的容器`。

就像上述所说的，每个部门又会细分为各个工作小组（线程），而工作小组需要的资源需要向上级（进程）申请。

`线程`（thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。因为线程中几乎不包含系统资源，所以执行更快、更有效率。

简而言之，一个程序至少有一个进程，一个进程至少有一个线程。线程的划分尺度小于进程，使得多线程程序的并发性高。另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。就如下图所示：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/8346731b-e109-4d18-86fe-a558af96d98d.png)

## 进程的属性
### 进程的分类
大概明白进程是个什么样的存在后，我们需要进一步了解的就是进程分类。可以从两个角度来分：以进程的功能与服务的对象来分；以应用程序的服务类型来分。

- 第一个角度来看，我们可以分为用户进程与系统进程：

用户进程：通过执行用户程序、应用程序或称之为内核之外的系统程序而产生的进程，此类进程可以在用户的控制下运行或关闭。

系统进程：通过执行系统内核程序而产生的进程，比如可以执行内存资源分配和进程切换等相对底层的工作；而且该进程的运行不受用户的干预，即使是 root 用户也不能干预系统进程的运行。

- 第二角度来看，我们可以将进程分为交互进程、批处理进程、守护进程：

交互进程：由一个 shell 终端启动的进程，在执行过程中，需要与用户进行交互操作，可以运行于前台，也可以运行在后台。

批处理进程：该进程是一个进程集合，负责按顺序启动其他的进程。

守护进程：守护进程是一直运行的一种进程，在 Linux 系统启动时启动，在系统关闭时终止。它们独立于控制终端并且周期性的执行某种任务或等待处理某些发生的事件。例如 httpd 进程，一直处于运行状态，等待用户的访问。还有经常用的 cron（在 centOS 系列为 crond）进程，这个进程为 crontab 的守护进程，可以周期性的执行用户设定的某些任务。

### 进程的衍生
进程有这么多的种类，那么进程之间定是有相关性的，而这些有关联性的进程又是如何产生的，如何衍生的？

就比如我们启动了终端，就是启动了一个 bash 进程，我们可以在 bash 中再输入 bash 则会再启动一个 bash 的进程，此时第二个 bash 进程就是由第一个 bash 进程创建出来的，他们之间又是个什么关系？

我们一般称呼第一个 bash 进程是第二 bash 进程的`父进程`，第二 bash 进程是第一个 bash 进程的`子进程`，这层关系是如何得来的呢？

关于父进程与子进程便会提及这两个系统调用 `fork()` 与 `exec()`

> fork() 是一个系统调用（system call），它的主要作用就是为当前的进程创建一个新的进程，这个新的进程就是它的子进程，这个子进程除了父进程的返回值和 PID (Process Identification) 以外其他的都一模一样，如进程的执行代码段，内存信息，文件描述，寄存器状态等等。

> exec() 也是系统调用，作用是切换子进程中的执行程序，也就是替换其从父进程复制过来的代码段与数据段。

子进程就是父进程通过系统调用 fork() 而产生的复制品，fork() 就是把父进程的 PCB 等进程的数据结构信息直接复制过来，只是修改了 PID，所以一模一样，只有在执行 exec() 之后才会不同，而早先的 fork() 比较消耗资源后来进化成 `vfork()`，效率高了不少。

这就是子进程产生的由来。简单的实现逻辑就如下方所示：

```
pid_t p;

p = fork();
if (p == (pid_t) -1)
        /* ERROR */
else if (p == 0)
        /* CHILD */
else
        /* PARENT */
```

既然子进程是通过父进程而衍生出来的，那么子进程的`退出与资源的回收`定然与父进程有很大的相关性。当一个子进程要正常的终止运行时，或者该进程结束时它的主函数 main() 会执行 exit(n); 或者 return n，这里的返回值 n 是一个信号，系统会把这个 SIGCHLD 信号传给其父进程，当然若是异常终止也往往是因为这个信号。

在将要结束时的子进程代码执行部分已经结束执行了，系统的资源也基本归还给系统了，但若是其进程的`进程控制块（PCB）`仍驻留在内存中，而它的 PCB 还在，代表这个进程还存在（因为 PCB 就是进程存在的唯一标志，里面有 PID 等消息），并没有消亡，这样的进程称之为僵尸进程（Zombie）。

如图中第四列标题是 S，`S` 表示的是进程的状态，而在下属的第三行的 `Z` 表示的是 Zombie 的意思。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/feb3405c-dc55-4101-8503-2cd858151d12.png)

正常情况下，父进程会收到两个返回值：`exit code`（SIGCHLD 信号）与 `reason for termination` 。之后，父进程会使用 `wait(&status)` 系统调用以获取子进程的退出状态，然后内核就可以从内存中释放已结束的子进程的 PCB；而如若父进程没有这么做的话，子进程的 PCB 就会一直驻留在内存中，一直留在系统中成为僵尸进程（Zombie）。

虽然`僵尸进程`是已经放弃了几乎所有内存空间，没有任何可执行代码，也不能被调度，在进程列表中保留一个位置，记载该进程的退出状态等信息供其父进程收集，从而释放它。但是 Linux 系统中能使用的 PID 是有限的，如果系统中存在有大量的僵尸进程，系统将会因为没有可用的 PID 从而导致不能产生新的进程。

另外如果父进程结束（非正常的结束），未能及时收回子进程，子进程仍在运行，这样的子进程称之为`孤儿进程`。在 Linux 系统中，孤儿进程一般会被 init 进程所“收养”，成为 init 的子进程。由 init 来做善后处理，所以它并不至于像僵尸进程那样无人问津，不管不顾，大量存在会有危害。

`进程 0` 是系统引导时创建的一个特殊进程，也称之为`内核初始化`，其最后一个动作就是调用 fork() 创建出一个子进程运行 /sbin/init 可执行文件，而该进程就是 PID=1 的进程 1，而进程 0 就转为`交换进程`（也被称为空闲进程），进程 1 （init 进程）是第一个用户态的进程，再由它不断调用 fork() 来创建系统里其他的进程，所以它是所有进程的父进程或者祖先进程。同时它是一个守护程序，直到计算机关机才会停止。

通过以下的命令我们可以很明显的看到这样的结构

```
pstree
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/d3ec3bd4-bdda-428e-ba9f-ec832b72791e.png)

或者从此图我们可以更加形象的看清子父进程的关系


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/260175bc-fb97-446e-964c-261549a92a01.png)

通过以上的显示结果我们可以看的很清楚，`init` 为所有进程的父进程或者说是祖先进程

我们还可以使用这样一个命令来看，其中 pid 就是该进程的一个唯一编号，ppid 就是该进程的父进程的 pid，command 表示的是该进程通过执行什么样的命令或者脚本而产生的：

```php
ps －fxo user,ppid,pid,pgid,command
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/01b8333b-a4fd-4c7d-ada8-bf77712442d6.png)

可以在图中看见我们执行的 ps 就是由 zsh 通过 fork-exec 创建的子进程而执行的。

使用这样的一个命令我们也能清楚的看见 init 如上文所说是由进程 0 这个初始化进程来创建出来的子进程，而其他的进程基本是由 init 创建的子进程，或者是由它的子进程创建出来的子进程。所以 init 是用户进程的第一个进程也是所有用户进程的父进程或者祖先进程。

就像一个树状图，而 init 进程就是这棵树的根，其他进程由根不断的发散，开枝散叶。

### 进程组与 Sessions
每一个进程都会是一个进程组的成员，而且这个进程组是唯一存在的，他们是依靠 `PGID`（process group ID）来区别的，而每当一个进程被创建的时候，它便会成为其父进程所在组中的一员。

一般情况，进程组的 PGID 等同于进程组的第一个成员的 PID，并且这样的进程称为该进程组的领导者，也就是`领导进程`，进程一般通过使用 `getpgrp()` 系统调用来寻找其所在组的 PGID，领导进程可以先终结，此时进程组依然存在，并持有相同的 PGID，直到进程组中最后一个进程终结。

与进程组类似，每当一个进程被创建的时候，它便会成为其父进程所在 Session 中的一员，每一个进程组都会在一个 Session 中，并且这个 Session 是唯一存在的，

Session 主要是针对一个 tty 建立，Session 中的每个进程都称为一个工作(job)。每个会话可以连接一个终端(control terminal)。当控制终端有输入输出时，都传递给该会话的前台进程组。Session 意义在于将多个 jobs 囊括在一个终端，并取其中的一个 job 作为前台，来直接接收该终端的输入输出以及终端信号。 其他 jobs 在后台运行。

`前台`（foreground）就是在终端中运行，能与你有交互的；

`后台`（background）就是在终端中运行，但是你并不能与其任何的交互，也不会显示其执行的过程。

### 工作管理
`bash`(Bourne-Again shell)支持工作控制（job control），而 sh（Bourne shell）并不支持。

并且每个终端或者说 bash 只能管理当前终端中的 job，不能管理其他终端中的 job。比如我当前存在两个 bash 分别为 bash1、bash2，bash1 只能管理其自己里面的 job 并不能管理 bash2 里面的 job

我们都知道当一个进程在前台运作时我们可以用 `ctrl + c` 来终止它，但是若是在后台的话就不行了。

我们可以通过 `&` 这个符号，让我们的命令在后台中运行：

```php
ls &
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/e22d8608-4313-46d0-9ebf-3fadcc3d4969.png)

图中所显示的 [1] 236分别是该 job 的 job number 与该进程的 PID，而最后一行的 Done 表示该命令已经在后台执行完毕。

我们还可以通过 `ctrl + z` 使我们的当前工作停止并丢到后台中去

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/901d744e-10cc-4589-8010-6f032cf13f3d.png)

被停止并放置在后台的工作我们可以使用 `jobs` 命令来查看：

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/a7021dc2-875f-4481-88c7-c4d8d5f161f6.png)

其中第一列显示的为被放置后台 job 的编号，而第二列的 `＋` 表示最近(刚刚、最后)被放置后台的 job，同时也表示预设的工作，也就是若是有什么针对后台 job 的操作，首先对预设的 job，`-` 表示倒数第二（也就是在预设之前的一个）被放置后台的工作，倒数第三个（再之前的）以后都不会有这样的符号修饰，第三列表示它们的状态，而最后一列表示该进程执行的命令。

我们可以通过这样的一个命令将后台的工作拿到前台来：

```php
# 后面不加参数提取预设工作，加参数提取指定工作的编号
# ubuntu 在 zsh 中需要 %，在 bash 中不需要 %
fg [%jobnumber]
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/9943422f-40ec-4134-a2c9-addd0b73fc3e.png)

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/507cd882-8e4d-456c-9c3c-576eabd8e50d.png)


之前我们通过 `ctrl + z` 使得工作停止放置在后台，若是我们想让其在后台运作我们就使用这样一个命令：

```php
#与fg类似，加参则指定，不加参则取预设

bg [%jobnumber]
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/a5e6d61e-c0ff-4a56-8653-ca65e5fdc888.png)

既然有方法将被放置在后台的工作提至前台或者让它从停止变成继续运行在后台，当然也有方法删除一个工作，或者重启等等。

```php
# kill的使用格式如下
kill -signal %jobnumber

# signal从1-64个信号值可以选择，可以这样查看
kill －l
```
其中常用的有这些信号值：

| 信号值 |               作用               |
| :----: | :------------------------------: |
|   -1   | 重新读取参数运行，类似与 restart |
|   -2   |      如同 ctrl+c 的操作退出      |
|   -9   |          强制终止该任务          |
|  -15   |       正常的方式终止该任务       |

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/eb13a1d5-2f3b-45d2-8794-84c3ea7df62b.png)

若是在使用 kill ＋信号值然后直接加 pid，你将会对 pid 对应的进程进行操作。

若是在使用 kill+信号值然后 ％jobnumber，这时所操作的对象是 job，这个数字就是就当前 bash 中后台的运行的 job 的 ID。

# 进程管理
## 进程的查看
不管在测试的时候、在实际的生产环境中，还是自己的使用过程中，难免会遇到一些`进程异常`的情况，所以 Linux 为我们提供了一些工具来查看进程的状态信息。

我们可以通过 `top` 实时的查看进程的状态，以及系统的一些信息（如 CPU、内存信息等），我们还可以通过 `ps` 来静态查看当前的进程信息，同时我们还可以使用 `pstree` 来查看当前活跃进程的树形结构。

### top 工具的使用
`top` 工具是我们常用的一个查看工具，能实时的查看我们系统的一些关键信息的变化。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/01fde29a-9774-4b99-8ac2-c5ef0252558f.png)

top 是一个在前台执行的程序，所以执行后便进入到这样的一个交互界面，正是因为交互界面我们才可以实时的获取到系统与进程的信息。在交互界面中我们可以通过一些指令来操作和筛选。在此之前我们先来了解显示了哪些信息。

- 我们看到 top 显示的第一排：

|             内容             |                  解释                   |
| :--------------------------: | :-------------------------------------: |
|             top              |           表示当前程序的名称            |
|           11:05:18           |          表示当前的系统的时间           |
|       up 8 days,17:12        |      表示该机器已经启动了多长时间       |
|            1 user            |       表示当前系统中只有一个用户        |
| load average: 0.29,0.20,0.25 | 分别对应 1、5、15 分钟内 cpu 的平均负载 |

load average 在 wikipedia 中的解释是 the system load is a measure of the amount of work that a computer system is doing 也就是对当前 CPU 工作量的度量，具体来说也就是指运行队列的平均长度，也就是等待 CPU 的平均进程数相关的一个计算值。

我们该如何看待这个 load average 数据呢？


实际生活中我们需要将这个值除以我们的核数来看。我们可以通过以下的命令来查看 CPU 的个数与核心数：

```php
#查看物理 CPU 的个数
cat /proc/cpuinfo | grep "physical id" | sort | uniq |wc -l

#每个 cpu 的核心数
cat /proc/cpuinfo | grep "physical id" | grep "0" | wc -l
```

通过上面的指数我们可以得知 load 的临界值为 1 ，但是在实际生活中，比较有经验的运维或者系统管理员会将临界值定为 0.7。这里的指数都是除以核心数以后的值，不要混淆了

- 若是 load < 0.7 并不会去关注他；
- 若是 0.7< load < 1 的时候我们就需要稍微关注一下了，虽然还可以应付但是这个值已经离临界不远了；
- 若是 load = 1 的时候我们就需要警惕了，因为这个时候已经没有更多的资源的了，已经在全力以赴了；
- 若是 load > 5 的时候系统已经快不行了，这个时候你需要加班解决问题了

通常我们都会先看 15 分钟的值来看这个大体的趋势，然后再看 5 分钟的值对比来看是否有下降的趋势。

查看 busybox 的代码可以知道，数据是每 5 秒钟就检查一次活跃的进程数，然后计算出该值，然后 load 从 /proc/loadavg 中读取的。

- 来看 top 的第二行数据，基本上第二行是进程的一个情况统计：

|      内容       |         解释         |
| :-------------: | :------------------: |
| Tasks: 26 total |       进程总数       |
|    1 running    | 1 个正在运行的进程数 |
|   25 sleeping   |  25 个睡眠的进程数   |
|    0 stopped    |   没有停止的进程数   |
|    0 zombie     |    没有僵尸进程数    |

- 来看 top 的第三行数据，这一行基本上是 CPU 的一个使用情况的统计了：

|      内容      |                             解释                             |
| :------------: | :----------------------------------------------------------: |
| Cpu(s): 1.0%us |                 用户空间进程占用 CPU 百分比                  |
|    1.0% sy     |                 内核空间运行占用 CPU 百分比                  |
|     0.0%ni     |       用户进程空间内改变过优先级的进程占用 CPU 百分比        |
|    97.9%id     |                       空闲 CPU 百分比                        |
|     0.0%wa     |                等待输入输出的 CPU 时间百分比                 |
|     0.1%hi     |            硬中断(Hardware IRQ)占用 CPU 的百分比             |
|     0.0%si     |            软中断(Software IRQ)占用 CPU 的百分比             |
|     0.0%st     | (Steal time) 是 hypervisor 等虚拟服务中，虚拟 CPU 等待实际 CPU 的时间的百分比 |

`CPU 利用率`是对一个时间段内 CPU 使用状况的统计，通过这个指标可以看出在某一个时间段内 CPU 被占用的情况，而 `Load Average` 是 CPU 的 Load，它所包含的信息不是 CPU 的使用率状况，而是在一段时间内 CPU 正在处理以及等待 CPU 处理的进程数情况统计信息，这两个指标并不一样。

- 来看 top 的第四行数据，这一行基本上是内存的一个使用情况的统计了：

|  内容   |              解释               |
| :-----: | :-----------------------------: |
| 8176740 |      total	物理内存总量      |
| 8032104 |   used	使用的物理内存总量    |
| 144636  |      free	空闲内存总量       |
| 313088  | buffers	用作内核缓存的内存量 |

> 注意：系统中可用的物理内存最大值并不是 free 这个单一的值，而是 free + buffers + swap 中的 cached 的和。

- 来看 top 的第五行数据，这一行基本上是交换区的一个使用情况的统计了：

|  内容  |                             解释                             |
| :----: | :----------------------------------------------------------: |
| total  |                          交换区总量                          |
|  used  |                       使用的交换区总量                       |
|  free  |                        空闲交换区总量                        |
| cached | 缓冲的交换区总量，内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖 |

- 再下面就是进程的一个情况了

|  列名   |                     解释                     |
| :-----: | :------------------------------------------: |
|   PID   |                   进程 id                    |
|  USER   |               该进程的所属用户               |
|   PR    |        该进程执行的优先级 priority 值        |
|   NI    |               该进程的 nice 值               |
|  VIRT   |       该进程任务所使用的虚拟内存的总数       |
|   RES   | 该进程所使用的物理内存数，也称之为驻留内存数 |
|   SHR   |             该进程共享内存的大小             |
|    S    | 该进程进程的状态: S=sleep R=running Z=zombie |
|  %CPU   |             该进程 CPU 的利用率              |
|  %MEM   |              该进程内存的利用率              |
|  TIME+  |              该进程活跃的总时间              |
| COMMAND |               该进程运行的名字               |

- 注意：

`NICE` 值叫做静态优先级，是用户空间的一个优先级值，其取值范围是 -20 至 19。这个值越小，表示进程”优先级”越高，而值越大“优先级”越低。nice 值中的 -20 到 19，中 -20 优先级最高， 0 是默认的值，而 19 优先级最低。

`PR` 值表示 Priority 值叫动态优先级，是进程在内核中实际的优先级值，进程优先级的取值范围是通过一个宏定义的，这个宏的名称是 MAX_PRIO，它的值为 140。Linux 实际上实现了 140 个优先级范围，取值范围是从 0-139，这个值越小，优先级越高。而这其中的 0-99 是实时进程的值，而 100-139 是给用户的。

其中 `PR` 中的 100 to 139 值部分有这么一个对应 PR = 20 + (-20 to +19)，这里的 -20 to +19 便是 nice 值，所以说两个虽然都是优先级，而且有千丝万缕的关系，但是他们的值，他们的作用范围并不相同。

`VIRT` 任务所使用的虚拟内存的总数，其中包含所有的代码，数据，共享库和被换出 swap 空间的页面等所占据空间的总数。

 `top` 是一个前台程序，所以是一个可以交互的：

| 常用交互命令 |                             解释                             |
| :----------: | :----------------------------------------------------------: |
|      q       |                           退出程序                           |
|      I       |               切换显示平均负载和启动时间的信息               |
|      P       |               根据 CPU 使用百分比大小进行排序                |
|      M       |                   根据驻留内存大小进行排序                   |
|      i       |           忽略闲置和僵死的进程，这是一个开关式命令           |
|      k       | 终止一个进程，系统提示输入 PID 及发送的信号值。一般终止进程用 15 信号，不能正常结束则使用 9 信号。安全模式下该命令被屏蔽。 |

好好的利用 `top` 能够很有效的帮助我们观察到系统的瓶颈所在，或者是系统的问题所在。

### ps 工具的使用
`ps` 也是我们最常用的查看进程的工具之一，我们通过这样的一个命令来了解一下，它能给我们带来哪些信息：

```php
ps aux
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/ba043cff-a52f-4db2-a9c4-d778cb810aae.png)

```php
ps axjf
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/3889fa80-5cbc-4c5f-bc40-769901c181fc.png)

我们来总体了解下会出现哪些信息给我们，这些信息又代表着什么：

|                           内容                           |                             解释                             |
| :------------------------------------------------------: | :----------------------------------------------------------: |
|                            F                             | 进程的标志（process flags），当 flags 值为 1 则表示此子程序只是 fork 但没有执行 exec，为 4 表示此程序使用超级管理员 root 权限 |
|                           USER                           |                        进程的拥有用户                        |
|                           PID                            |                          进程的 ID                           |
|                           PPID                           |                        其父进程的 PID                        |
|                           SID                            |                        session 的 ID                         |
|                          TPGID                           |                       前台进程组的 ID                        |
|                           %CPU                           |                    进程占用的 CPU 百分比                     |
|                           %MEM                           |                       占用内存的百分比                       |
|                            NI                            |                        进程的 NICE 值                        |
|                           VSZ                            |                     进程使用虚拟内存大小                     |
|                           RSS                            |                      驻留内存中页的大小                      |
|                           TTY                            |                           终端 ID                            |
|                        S or STAT                         |                           进程状态                           |
|                          WCHAN                           |                      正在等待的进程资源                      |
|                          START                           |                        启动进程的时间                        |
|                           TIME                           |                     进程消耗 CPU 的时间                      |
|                         COMMAND                          |                       命令的名称和参数                       |
| TPGID栏写着-1 的都是没有控制终端的进程，也就是守护进程。 |                                                              |

`STAT` 表示进程的状态，而进程的状态有很多，如下表所示：

| 状态 |                解释                |
| :--: | :--------------------------------: |
|  R   |           Running.运行中           |
|  S   |    Interruptible Sleep.等待调用    |
|  D   | Uninterruptible Sleep.不可中断睡眠 |
|  T   |      Stoped.暂停或者跟踪状态       |
|  X   |          Dead.即将被撤销           |
|  Z   |          Zombie.僵尸进程           |
|  W   |          Paging.内存交换           |
|  N   |           优先级低的进程           |
|  <   |           优先级高的进程           |
|  s   |            进程的领导者            |
|  L   |              锁定状态              |
|  l   |             多线程状态             |
+	|前台进程

其中的 `D` 是不能被中断睡眠的状态，处在这种状态的进程不接受外来的任何 signal，所以无法使用 `kill` 命令杀掉处于 D 状态的进程，无论是 kill，kill -9 还是 kill -15，一般处于这种状态可能是进程 I/O 的时候出问题了。

ps 工具有许多的参数，下面解释部分常用的参数。

使用 `-l` 参数可以显示自己这次登录的 bash 相关的进程信息罗列出来：

```php
ps -l
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/56872c4d-d5b8-49de-9e19-304e54069ce1.png)

相对来说我们更加常用下面这个命令，他将会罗列出所有的进程信息：

```php
ps aux
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/a08178f6-9542-4f5e-9113-00dc49706924.png)

若是查找其中的某个进程的话，我们还可以配合着 `grep` 和正则表达式一起使用：

ps aux | grep zsh


![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/e96466b1-cebc-4105-9b29-6e33068be22a.png)

此外我们还可以查看时，将连同部分的进程呈树状显示出来：

```php
ps axjf
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/5f11c56b-242d-48e2-add3-0376d8a61380.png)

当然如果你觉得使用这样的没有把你想要的信息放在一起，我们也可以是用这样的命令，来自定义我们所需要的参数显示：

```php
ps -afxo user,ppid,pid,pgid,command
```

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/f6c99da1-2fbf-4cda-a184-514cca48a16f.png)

### pstree 工具的使用
通过 `pstree` 可以很直接的看到相同的进程数量，最主要的还是我们可以看到所有进程之间的相关性。

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/0049a8c4-0c6c-421d-8ee4-123662a8dc35.png)

```php
pstree -up
```

| 参数选择 |                解释                 |
| :------: | :---------------------------------: |
|    -A    |     程序树之间以 ASCII 字符连接     |
|    -p    |     同时列出每个 process 的 PID     |
|    -u    | 同时列出每个 process 的所属账户名称 |

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/0ed4bd8a-3164-4e63-9ea5-2c7e2425a48c.png)

## 进程的管理
### kill 命令的掌握
当一个进程结束的时候或者要异常结束的时候，会向其父进程返回一个或者接收一个 SIGHUP 信号而做出的结束进程或者其他的操作，这个 SIGHUP 信号不仅可以由系统发送，我们可以使用 `kill` 来发送这个信号来操作进程的结束或者重启等等。

上节使用 `kill` 命令来管理我们的一些 job，这节课我们将尝试用 `kill` 来操作下一些不属于 job 范畴的进程，直接对 pid 下手。

```php
# 首先我们使用图形界面打开了 gedit、gvim，用 ps 可以查看到
ps aux

# 使用 9 这个信号强制结束 gedit 进程
kill -9 1608

# 我们再查找这个进程的时候就找不到了
ps aux | grep gedit
```
![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/b90949ba-7283-4dea-9364-ed0c55558dcc.png)

![](https://gcore.jsdelivr.net/gh/CARLOSGP2021/myFigures/img/b58db2e2-03b5-4865-b5b8-5ebe34765b80.png)

### 进程的执行顺序
我们在使用 `ps` 命令的时候可以看到大部分的进程都是处于休眠的状态，如果这些进程都被唤醒，那么该谁最先享受 CPU 的服务，后面的进程又该是一个什么样的顺序呢？进程调度的队列又该如何去排列呢？

当然就是靠该进程的`优先级值`来判定进程调度的优先级，而优先级的值就是上文所提到的 `PR` 与 `nice` 来控制与体现了。

而 `nice` 的值我们是可以通过 `nice` 命令来修改的，而需要注意的是 `nice` 值可以调整的范围是 -20 ~ 19，其中 root 有着至高无上的权力，既可以调整自己的进程也可以调整其他用户的程序，并且是所有的值都可以用，而普通用户只可以调制属于自己的进程，并且其使用的范围只能是 0 ~ 19，因为系统为了避免一般用户抢占系统资源而设置的一个限制。

```php
# 这个实验在环境中无法做，因为权限不够，可以自己在本地尝试

# 打开一个程序放在后台，或者用图形界面打开
nice -n -5 vim &

# 用 ps 查看其优先级
ps -afxo user,ppid,pid,stat,pri,ni,time,command | grep vim
```
我们还可以用 `renice` 来修改已经存在的进程的优先级，同样因为权限的原因在实验环境中无法尝试。

```php
renice -5 pid
```











