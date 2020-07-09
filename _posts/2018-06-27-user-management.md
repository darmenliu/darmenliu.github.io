---
layout: post
title: "linux 账户及安全管理（一 账户管理）"
author: "Yongqiang Liu"
---

# linux 账户及安全管理（一 账户管理）

![img](https://ithorseman.files.wordpress.com/2019/03/e5beaee4bfa1e4ba8ce7bbb4e7a081-4.jpg?w=258)微信公众号

在服务器及超级计算机领域，linux操作系统（98%以上）占有绝对统治地位，包括现在的互联网相关的各种服务，大部分都是基于linux操作系统，作为IT领域的程序员，话说不懂linux，岂不是一件细思极恐的事情。

基于linux开发的这么多年，有一件事情一直觉得处于一知半解的状态，包括身边的同事，即使是大牛，也是只知其一不知其二，可见这件事情并不是得到大家的重视，尤其是那些刚刚接触linux的新手，更是云里雾里，我说的这件事情其实就是linux的账户和安全管理。很多linux程序员估计一听到root，就会本能的处于兴奋状态，为什么，因为是超级用户啊，在linux世界，root就是钢铁侠，美国队长，无所不能啊。可是一不小心一路root的结果，最终可能变成惨案，使我们的公司及客户遭受巨大损失。因此，我更提倡安全的使用linux，尤其是程序员，更要熟悉和掌握账户和权限的机制，不要总是让我们的程序躺在root的温柔乡里陶醉，而是更合理的限制自己的权限，清楚的知道自己能做什么，不能做什么，这才是进入程序天堂的捷径。

## 账户管理

大家都知道，linux操作系统是典型的多用户多任务操作系统，即同一时间内，允许由多个用户同时登录同一台计算机，运行各自的一个或多个任务，各个用户之间并不一定能明确的感知到其他用户的登录操作。linux上存在一个超级用户，就是root，root账户几乎无所不能，可以用来创建和配置非root账户。

关于账户管理，有两个概念必须理解，用户（User）和用户组（Group）。

用户（User）：一台计算机允许程序或人登录并在一定权限范围之内运行任务的账户配置信息。

用户组（Group）：Linux允许把一类用户（多个）划分到一个组里，这一组的用户拥有相同的操作权限，当然一个用户也可以只对应一个组，同时也支持将一个账户添加到多个组里。

### 如何创建用户及用户组？

创建group：

```
groupadd group1
```

创建group时指定GID(group ID)：

```
groupadd  -g 100  group2
```

创建user：

```
useradd user1
```

创建user 时指定初始的group，如果不指定group，linux会默认创建同名的group:

```
useradd user1 -g group1
```

创建user时，指定UID(user ID):

```
useradd user2 -u 100 -g group2
```

Linux 按照使用场景不同，可以将用户分为**系统账户**和**普通账户**，系统账户主要作为系统进程或程序的账户，普通账户也可以称之交互账户，普通账户主要用于操作系统使用者，跟某个使用人相关，通过该账户，使得使用者具有一定的权限使用该计算机资源，比如登录，浏览网页，运行程序，阅读文档等等。除了使用上的不同外，系统账户和普通账户的UID、GID范围是不同的，系统账户通过SYS_UID_MIN、SYS_UID_MAX和SYS_GID_MIN、SYS_GID_MAX定义，普通账户通过GID_MIN、GID_MAX和UID_MIN、UID_MAX，这些参数的定义可以在/etc/login.defs中找到：

```
#
# Min/max values for automatic uid selection in useradd
#
UID_MIN 1000
UID_MAX 60000
# System accounts
SYS_UID_MIN 201
SYS_UID_MAX 999
#
# Min/max values for automatic gid selection in groupadd
#
GID_MIN 1000
GID_MAX 60000
# System accounts
SYS_GID_MIN 201
SYS_GID_MAX 999
```

创建系统账户和系统组：

```
groupadd -r  -g 100  group2
useradd  -r user2 -u 100 -g group2
```

还可以将一个user添加到两个不同的group中来：

```
groupadd -r  -g 100  group2 
useradd  -r user2 -u 100 -g group2 -G group1
```

创建普通账户时指定home目录，指定登录的交互程序：

```
useradd  user2 -u 100 -g group2 -d /home/user2 -s /bin/bash
```

创建系统账户时禁止登录及home目录：

```
useradd  user2 -u 100 -g group2 -d /dev/null -s /sbin/nologin
```

### 如何修改或者删除账户配置？

修改group的GID和 group name，将gid修改为10000，名称充group2修改为group3：

```
 groupmod –g 10000 -n group3 group2
```

删除group：

```
groupdel group2
```

修改账户属性，此命令将用户user1的登录Shell修改为ksh，主目录改为/home/test，用户组改为group3。

```
usermod -s /bin/ksh -d /home/test –g group3 user1
```

删除账户：

```
userdel user1
```

如果一个用户同时属于多个group时，可以在不同group之间进行切换，以使同一账户具有不同group的权限：

```
newgrp root
```

参考帮助 [useradd](https://ithorseman.wordpress.com/2018/06/29/useradd/) ，[groupadd](https://ithorseman.wordpress.com/2018/06/29/groupadd/)，[usermod](https://linux.die.net/man/8/usermod)，[groupmod](https://linux.die.net/man/8/groupmod)，[userdel](https://linux.die.net/man/8/userdel)，[groupdel](https://linux.die.net/man/8/groupdel)，

## 密码管理

密码是账户访问的一把钥匙，不管使用何种方式登录账户，用户必须输入密码来使用计算机资源，一般交互式账户必须设定密码，设置用户密码有两种方式，一种是在创建账户时使用-p参数设定初始密码，一种是使用passwd命令修改账户密码。普通用户只能修改自己账户的密码，root用户可以修改任意账户的密码。

创建账户时指定密码：

```
useradd  user2 -u 100 -g group2 -d /home/user2 -s /bin/bash -p jlkdakfdf
```

创建账户时指定的密码是明文，虽然Linux最终会加密，但是这种方式还是不安全，建议使用第二种方式。

修改账户密码：

```
(svnenv)user2@d076cf119be7:/mnt/share$ passwd
Current password:
New password:
Retype new password:
passwd: password updated successfully
```

首先会提示用户输入当前密码，然后设定新密码并再次确认。密码应包含6到8个字符，包括一个或多个字符来自以下每个集合的字符：

- 小写字母
- 数字0到9
-  标点符号

必须注意不要包括系统默认擦除或终止字符。 passwd将拒绝任何不合适的密码。

参考帮助[passwd](http://man7.org/linux/man-pages/man1/passwd.1.html)。

### 系统账户信息及密码信息保存在哪里？

如果你想通过某种方式知道系统中所有的用户及group信息时，最简单的方式就是查看用户管理相关的数据文件，一般这些文件对所有用户都是可读的，这些文件包括/etc/passwd, /etc/shadow, /etc/group等。

#### /etc/passwd

不要被该文件的命名迷惑，以前的确会保存密码，但是现在主要保存用户配置信息，你可以通过该文件查询系统所有的账户信息，包括系统账户和普通账户：

```
＃ cat /etc/passwd
root:x:0:0:Superuser:/:
daemon:x:1:1:System daemons:/etc:
bin:x:2:2:Owner of system commands:/bin:
sys:x:3:3:Owner of system files:/usr/sys:
adm:x:4:4:System accounting:/usr/adm:
uucp:x:5:5:UUCP administrator:/usr/lib/uucp:
auth:x:7:21:Authentication administrator:/tcb/files/auth:
cron:x:9:16:Cron daemon:/usr/spool/cron:
listen:x:37:4:Network daemon:/usr/net/nls:
lp:x:71:18:Printer administrator:/usr/spool/lp:
sam:x:200:50:Sam san:/usr/sam:/bin/sh
user2:x:1007:1007::/home/user2:/bin/bash
user1:x:1008:1008::/home/user1:/bin/bash
```

每条记录都对应于一个账户及其属性，通常被冒号“:”分割成7个字段，分别对应：

```
用户名:口令:用户标识号:组标识号:注释:主目录:登录Shell
```

- 用户名：用户账号的字符串，由大小写字母和数字组成。登录名中不能有冒号(:)，因为冒号在这里是分隔符。为了兼容起见，登录名中最好不要包含点字符(.)，并且不使用连字符(-)和加号(+)打头。
- 口令：虽然这个字段存放的只是用户口令的加密串，不是明文，但是由于/etc/passwd文件对所有用户都可读，所以这仍是一个安全隐患。因此，现在许多Linux 系统（如SVR4）都使用了shadow技术，把真正的加密后的用户口令字存放到/etc/shadow文件中，而在/etc/passwd文件的口令字段中只存放一个特殊的字符，例如“x”或者“*”。
- 用户标识号：即UID，用来标识用户，系统用户和普通用户的UID大小范围在配置文件/etc/login.defs中定义。它与用户名是一一对应的，使用useradd添加账户时，如果指定的UID已经被使用，则会提示用户"UID 1008 is not unique"。
- 组标识符：该字段标识该用户初始的group，对应/etc/group中的一条记录，扩展group不会在这里记录。
- 注释：例如用户的真实姓名、电话、地址等，这个字段并没有什么实际的用途。在不同的Linux 系统中，这个字段的格式并没有统一。在许多Linux系统中，这个字段存放的是一段任意的注释性描述文字。
- 主目录：用户的初始工作目录，它是用户在登录到系统之后所处的目录。在大多数系统中，各用户的主目录都被组织在同一个特定的目录下，而用户主目录的名称就是该用户的登录名。各用户对自己的主目录有读、写、执行（搜索）权限，其他用户对此目录的访问权限则根据具体情况设置。当使用useradd添加新的账户时，默认会在/home目录下创建于账户名称相同的目录，如若用户指定了（-d 参数）主目录，则会以指定的目录为准。
- 登录shell：Linux 用户登录后，要启动一个进程，负责将用户的操作传给内核，这个进程是用户登录到系统后运行的命令解释器或某个特定的程序，即Shell。Shell是用户与Linux系统之间的接口。Linux的Shell有许多种，每种都有不同的特点。常用的有sh(Bourne Shell), csh(C Shell), ksh(Korn Shell), tcsh(TENEX/TOPS-20 type C Shell), bash(Bourne Again Shell)等。用户的登录Shell也可以指定为某个特定的程序（此程序不是一个命令解释器）。利用这一特点，我们可以限制用户只能运行指定的应用程序，在该应用程序运行结束后，用户就自动退出了系统。

#### /etc/group

该文件保存所有用户群组的信息，每个用户都属于某个用户组；一个组中可以有多个用户，一个用户也可以属于不同的组。当一个用户同时是多个组中的成员时，在/etc/passwd文件中记录的是用户所属的主组，也就是登录时所属的默认组，而其他组称为附加组。用户要访问属于附加组的文件时，必须首先使用newgrp命令使自己成为所要访问的组中的成员。

```
$ cat /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
user2:x:1007:
user1:x:1008:
user3:x:1009:
user4:x:1010:
user5:x:1011:user1,user2
```

该文件中记录的每条记录使用冒号“:”分割成3个字段：

```
组名:口令:组标识号:组内用户列表
```

- 组名：group名称，由字母和数字组成，规则类似于用户名，组名不能重复。
- 口令：group不会设置口令，因此该字段为空、x或者*。
- 组标识号：即GID，系统组合普通组的大小范围在文件/etc/login.defs中定义。
- 组内用户列表：属于这个组的所有用户的列表，不同用户之间用逗号“,”分隔。这个用户组可能是用户的主组，也可能是附加组。

#### /etc/shadow

/etc/shadow中的记录行与/etc/passwd中的一一对应，它由pwconv命令根据/etc/passwd中的数据自动产生。它的文件格式与/etc/passwd类似，由若干个字段组成，字段之间用":"隔开。这些字段是：

```
登录名:加密密码:上次更改密码的日期:最低密码年龄:最长密码年龄:密码警告期:密码不活动期:账户到期日:保留字段
```

- 登录名：即用户账号。
- 加密密码：加密后的用户密码，该字段可能为空，在这种情况下不需要密码进行验证登录。
- 上次更改密码的日期：上次密码更改的日期，以数字表示，自1970年1月1日以来的几天，空字段表示禁用密码时效功能。
- 最低密码年龄：密码最小使用期限是用户拥有的天数。
- 最长密码年龄：密码最长使用期限是之后的天数,用户必须更改她的密码。经过这几天后，密码可能仍然是有效。应该要求用户在下一次更改密码她将登录的时间。空字段表示没有最大密码期限，没有密码警告期，没有密码不活动期，如果密码最长使用期限低于最小密码年龄，用户无法更改密码。
- 密码警告期：表示的是从系统开始警告用户到用户密码正式失效之间的天数。
- 密码不活动期：表示的是用户没有登录活动但账号仍能保持有效的最大天数。
- 账户到期日：字段给出的是一个绝对的天数，如果使用了这个字段，那么就给出相应账号的生存期。期满后，该账号就不再是一个合法的账号，也就不能再用来登录了。

```
$ cat /etc/shadow
root:$1$eWL4wrs5$W.HzKJ24HVGTyADdFYsOd1vO1:17637:0:99999:7:::
lp:*:13510:0:99999:7:::
nobody:*:13509:0:99999:7:::
tc::13646:0:99999:7:::
dockremap:!:17085:0:99999:7:::
docker:$1$VdAIBSrn$Sx/A85NaQsIYib1JlZex61:17566:0:99999:7:::
user2:$6$pNjbuPVU.FFx56xoQCx$NGfUX/u.h.WssFSLOKH.pgbWNdHldKKFalKL6BNHZIFs9.alunrNuQhatNXl/lk0:17721:0:99999:7:::
user1:!!:17722:0:99999:7:::
user3:!!:17722:0:99999:7:::
user4:!!:17722:0:99999:7:::
user5:!!:17722:0:99999:7:::
```

至此关于linux的账户管理基本上已经覆盖，但是添加账户后linux是如何做到控制用户权限的呢，还有，关于权限的配置应该有哪些呢？关于这些问题，我将在[下一节](https://ithorseman.wordpress.com/2018/07/09/linux-账户及安全管理（二）/)详细讲述。