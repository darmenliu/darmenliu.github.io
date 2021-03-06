---
layout: post
title: "linux 账户及安全管理（二 文件权限）"
author: "Yongqiang Liu"
---

# linux 账户及安全管理（二 文件权限）

![img](https://ithorseman.files.wordpress.com/2019/03/e5beaee4bfa1e4ba8ce7bbb4e7a081-4.jpg?w=258)微信公众号

Linux是典型的多用户多任务操作系统，因此对于linux来说，同一台计算机资源可以提供给多个人使用是件很普通的事情，但是问题来了，多个人同时使用一台计算机，会不会相互影响和干扰，如果其中有人不小心，做了破坏性的动作，其他人是不是也要玩完？还有如果每个人都有一些隐私的文档保存在这台计算机上，那这些文档会不会被其他人浏览或获取？作为一个安全的操作系统，Linux是如何解决这些问题的呢？作为root用户，具有超级权限，那到底有哪些权限呢？

### 如何理解用户权限呢

你可以将一台计算机想象为一家酒店，那么这家酒店的root用户是谁呢？毫无疑问，当然是酒店的老板，老板手下的员工可以认为是管理员，而入住者可以认为是普通用户，当你在前台付费登记，定好了自己的房间后，前台会给你指定房间的钥匙或房卡，而这两样就相当于你的账户密码，有了钥匙和房卡，你就可以打开房门，使用该房间提供的所有物品或者服务，当然你使用的东西只能局限于该房间内，你的房卡和钥匙当然不能打开其他房间的房门，除非你是老板。

这是个形象的比喻，也是个缩影，我们周围所有存在的资源都是有限的，没有人能够无限制的使用，因此我们生活和工作的世界里，都存在着大大小小的规则来限制你行动范围和自由，你对资源的使用也需要付出一定的代价来获取。计算机资源也是一样的，对于多人共享的情况下，账户的管理和设计也要符合这种最基本的规则。

权限，从字面意思来理解，权，就是你的权利，规定你能做什么，限，就是限制，规定了你不能做什么，在一定范围内行使你的权利。Linux账户可以根据权限大小的不同，分为超级用户（root），管理员，普通用户。不同用户根据对系统资源的需求，会有不同的权限。那这些权限是谁赋予的呢？这要看超级用户（root）掌握在谁手里，那他就是这些账户的分配者，包括管理员，都需要root用户来分配，在这里管理员账户和普通账户其实没有特别不同，只是一般管理员帐户的权限可能会更多。

Linux的账户权限有很多种，最基本的就是对文件系统的访问权限，即文件或文件夹权限。sudo权限，可以赋予特定账户能够执行root账户的一些命令。ssh权限，控制用户是否可以通过ssh远程登录。

### 关于文件权限

由于Linux的设备和文档都是以文件的形式存在，因此Linux最基本的安全控制就是对文件的访问控制。Linux文件或文件夹权限是用来控制用户访问文件、文件夹或执行程序的一套机制。

根据用户对文件或目录操作种类的不同，linux定义了三种基本的权限：

- 读，用r标识，对文件来说，表示可以查看文件内容，对文件夹来说，表示可以查看文件夹下的文件，可以用数值4来表示；
- 写，用w标识，对文件来说，表示可以向文件中写入或者插入内容，对文件夹来说，表示可以在文件夹下删除、添加或者修改文件名称，可以用数值2来表示；
- 执行，用x标识，对文件来说，表示文件可以以程序来执行，对文件夹来说，表示可以访问子目录及文件及shell中cd到此目录，可以用数值1来表示。

同时，用户和文件存在以下关系：

- 文件所有者（owner）：拥有文件的用户。创建文件时是创建文件的用户（可用`whoami`命令查看）。
- 所属组（group）：文件所属的组。创建文件时用户的组（可用`id`命令查看）。
- 其他用户（other）：既不是文件所有者，也不是文件所属的组的成员的其他用户。

一个文件或文件夹的权限，其实就是对这三种关系的用户规定是否具有可读，可写，可执行的权限，通常文件或文件夹的权限可以通过ls -l命令来查看：

![ls-l](https://ithorseman.files.wordpress.com/2018/07/ls-l.png)

- mode：标识了owner、group及other用户对该文件的操作权限；
- owner：规定文件的拥有者；
- group：规定文件归属的用户组。

mode用长度为10的字符串来表示 “-rw-rw-r--”，第一位字符表示类型“-”表示文件，“d”表示目录，“l”表示链接文件。后面9位需要每三位为一组来理解：

![mode](https://ithorseman.files.wordpress.com/2018/07/mode.png)

如上图所示，每一组的第一位标识是否具有读权限，‘r’表示具有读权限，‘-’表示禁止读权限，第二位标识是否具有写权限，‘w’表示具有写权限，‘-’表示禁止写权限，第三位标识是否具有可执行权限，‘x’表示具有可执行权限，‘-’表示禁止可执行权限。从左往右分为三组，第一组标识owner的权限，第二组表示group的权限，第三组标识其他用户的权限。

举个例子，如上图所示的文件 public_excutable权限为"-rwxr-xr-x", 怎么理解呢？首先拆开来看，owner的权限是"rwx"，表示对文件具有可读可写可执行的权限，group的权限是"r-x"，表示对该文件只有可读和可执行的权限，other用户的权限也是"r-x"，表示对该文件只有可读和可执行的权限。

文件权限除了用字符串表示外，也可使用八进制数来表示权限，即用一个四位八进制数来表示，其中最高位表示特殊权限，随后的三位依次是所有者权限、组权限和其他人权限。每一个八进制位的权限数值是文件具有的相应权限所对应的数值之和，还是以文件public_excutable为例子：

```
0755 = rwxr-xr-x = 0(4 + 2 + 1)(4 + 1)(4 + 1)
```

如果要查看文件的数值权限，可以通过stat命令进行查看：

```
root@d076cf119be7:/ $ stat test_file
File: test_file
Size: 0 Blocks: 0 IO Block: 4096 regular empty file
Device: 29h/41d Inode: 13429 Links: 1
Access: (0644/-rw-r--r--) Uid: ( 0/ root) Gid: ( 0/ root)
Access: 2018-07-26 02:26:26.723187697 +0000
Modify: 2018-07-26 02:26:26.723187697 +0000
Change: 2018-07-26 02:26:26.723187697 +0000
Birth: -
```

### 如何创建并修改文件权限呢？

#### umask

当我们登录系统，创建一个新的文件或者目录时，会被分配一个默认权限，这个最初的权限是如何被分配的呢？在Linux系统中，通常管理员通过设置umask值来定义创建目录和文件时的默认权限。

什么事umask？系统管理员必须要为你设置一个合理的 umask值，以确保你创建的文件具有所希望的缺省权限，防止其他非同组用户对你的文件具有写权限。一般来说，umask命令是在/etc /profile文件中设置的，每个用户在登录时都会引用这个文件，所以如果希望改变所有用户的umask，可以在该文件中加入相应的条目。如果希望永久 性地设置自己的umask值，那么就把它放在自己$HOME目录下的.profile或.bash_profile或.bashrc文件中。

umask值类似于文件的权限值，使用三位八进制数来表示，比如002，设置范围（000~777），以掩码的形式，定义需要禁止那些用户的何种权限，第一位表示禁止owner的权限值，第二位表示禁止group用户的权限值，第三位表示禁止other用户的权限值。

#### 如何通过umask值来计算默认的文件或者目录的权限？

以umask值为002为例，翻译成权限字符串 "--- --- -w-"，表示禁止other用户使用写权限，对于文件来说，默认的最大权限为666 即"rw- rw- rw-"，则根据掩码，去掉other用户的写权限，文件默认的权限为"rw- rw- r--"，值为664，对于文件夹来说，默认的最大权限为777，即"rwx rwx rwx"，则根据掩码，去掉other用户的写权限，文件的权限为"rwx rwx r-x", 值为775。

使用umask命令：

查询系统umask值

```
root $ umask
0022
```

设置umask

```
root $ umask 033
root $ umask
0033
```

### 修改文件或文件夹权限

上文讲述了如何通过umask来定义文件或者文件夹的默认权限，那文件创建后，权限如何变更呢？Linux提供了[chmod](https://linux.die.net/man/1/chmod)命令来修改文件权限，chmod可以给文件或者目录的指定用户增加或者降低权限，比如给owner和group增加执行权限：

```
chmod ug+x test_file
```

以上的例子中u和g分别表示user，group，是用户类型:

- u - user
- g - group
- o - other
- a - all

'+' 表示增加权限，是执行的动作：

- \+ 增加权限
- \- 降低权限

'x' 表示要增加的文件权限：

- r 读
- w 写
- x 可执行

一些常见的例子：

给所有用户增加写权限：

```
chmod a+w test_file
```

给other用户增加执行权限：

```
chmod o+x test_file
```

禁止group和other用户的执行权限：

```
chmod go-x test_file
```

禁止所有用户的执行权限：

```
chmod a-x test_file
```

也可以直接修改文件的权限值：

```
chmod 644 test_file
```

### 修改文件或文件夹的归属

文件的归属决定了文件属于那个用户，隶属于那个用户组，创建一个文件时，默认的情况下，该文件的owner即为创建该文件的用户，group为该用户的初始group，

```
root:/home/user5 $ su user5
user5:~ $ groups
user5
user5:~ $ touch test
user5:~ $ ls -la
-rw-r--r-- 1 user5 user5 0 Aug 2 01:11 test
```

通常情况下对于root和owner用户来说，可以随意修改文件的权限属性，但是对owner来说，决定于该用户是否可以访问chmod命令。

文件的归属可以被root账户修改，chown 命令修改文件的owner，chgrp 修改文件归属的group，

```
root:/home/user5 $ chown user4 test
root:/home/user5 $ ls -la
-rw------- 1 user4 user5 0 Aug 2 01:11 test

root:/home/user5 $ chgrp user2 test
root:/home/user5 $ ls -la
-rw------- 1 user4 user2 0 Aug 2 01:11 test
```

### 文件及文件夹权限的控制

如果一个用户要访问一个文件或文件夹时，Linux首先会检查该用户和访问文件之间的归属关系，其次检查该用户对文件的操作属于什么操作（读，写，执行），最后根据用户的角色，检查该角色对文件是否拥有所操作的权限，比如上例中 test 文件权限如下所示：

```
root:/home $ ls -la
-rw------- 1 user4 user2 0 Aug 2 01:11 test
```

那如果是user2 用户想查看该文件，Linux是否会允许吗？不妨做个实验：

```
root:/home $ su user2
user2:/home $ cat test
cat: test: Permission denied
```

为什么会被拒绝呢？从以上权限来分析，虽然user2账户是属于user2 group的，而且该文件也归属于user2 group，但是文件的权限只对owner 开放了可读，可写权限，group是没有任何权限的，因此user2 是没有读权限的。

文件夹的权限比较特殊，x表示在shell中可以通过cd命令访问该目录及子目录，r表示可以浏览该目录下的文件及子目录，w表示可以在该文件夹下添加删除文件。

```
drwx------ 3 testuser1 test2 4096 Aug 15 01:18 testuser1dir
drwx------ 2 testuser2 testuser2 4096 Jun 28 01:44 testuser2dir
```

比如以上文件夹testuser1dir的权限只对owner用户testuser1开放了读写及访问权限，当我们切换到testuser2，是否能访问testuser1dir呢,

```
testuser2@d1db4cc29365:/home $ cd testuser1dir
bash: cd: testuser1dir: Permission denied
```

实践验证是不行的，原因是testuser1dir对于group用户和other用户是没有开放任何权限的，而testuser2属于other用户。如果需要other用户能够访问该目录，我们应该怎么做呢？

```
root@:/home $ chmod o+rx testuser1dir
root@:/home $ ls -la
drwx---r-x 3 testuser1 test2 4096 Aug 15 01:18 testuser1dir
testuser2@:/home $ cd testuser1dir/
testuser2@:/home/testuser1dir $ ls
testdir
```

首先切换到root账户，对testuser1dir给other用户增加可读可访问权限，然后再切换到testuser2账户，再次尝试cd到该目录，这次就OK了。

至此，基本介绍完了文件权限，我自己也从头到尾又梳理了一遍，Linux文件权限的控制相对松散，要保证系统的安全性，系统管理员必须制订严格复杂的策略，同时需要控制对root权限的使用范围，尤其是对一些系统服务或进程，要避免使用root账户。