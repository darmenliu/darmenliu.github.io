---
layout: post
title: "构建GNU Autotools项目"
author: "Yongqiang Liu"
---

# 构建GNU Autotools项目

1. [TOC]

Autoconf和Automake提供了一个有效的构建系统来维护你的软件，通常在别人的系统上。Automake检查源文件，确定它们如何相互依赖，并生成一个Makefile，以便可以按正确的顺序编译这些文件。Autoconf允许自动配置软件安装，处理大量系统怪癖以增加可移植性。Libtool（这里未讨论）是编译器和链接器的命令行界面，可以轻松生成静态库和共享库。

## 基本文件

最小的项目要求您只提供两个文件：

- Makefile.am - automake的输入文件，它指定了项目的构建要求：需要构建的内容以及安装后的位置。
- configure.in - autoconf的输入文件，提供autoconf用于构建*配置* 脚本的宏调用和shell代码片段。

GNU Autotools将生成构建项目所需的其余文件。

## 目录结构

在为新项目编写任何代码之前，您需要确定项目将使用的目录结构。

- 该*顶级*目录用于配置文件，如configure.in，和其他杂项文件，如更新日志，COPY（项目执照副本），并自述。
- 任何独特的库都应该有自己的子目录， 其中包含所有的头文件和源文件，Makefile.am文件以及任何其他库特定的文件。
- 主应用程序的头文件和源文件应位于另一个子目录中，通常称为*src。*
- 其他目录可以包括：*配置*为中间文件，*文档*项目文档和*测试* 该项目自测试套件。

以下步骤将带您完成创建和构建HelloWorld项目。HelloWorld的顶级目录是<tests / project>。您可以在*src*子目录中找到项目的标题和来源。有三个文件：helloworld.cc，helloworld.h和main.cc.

 

## Makfile.am

您必须为源代码树中的每个目录提供一个Makefile.am文件。Makefile.am用于顶层目录很简单。在<tests / project>目录下创建一个名为Makefile.am的新文本文件。将以下行添加到该文件并保存它：

 

```
SUBDIRS = src
```

SUBDIRS变量用于列出必须构建的子目录。

接下来，在<tests / project / src>子目录中创建另一个名为Makefile.am的文本文件。将以下行添加到文件并保存它：

`bin_PROGRAMS = helloworld`  AM_CXXFLAGS = $(INTI_CFLAGS) helloworld_SOURCES = main.cc helloworld.cc helloworld.h helloworld_LDADD = $(INTI_LIBS)

 

bin_PROGRAMS变量指定当*make install*运行时，我们需要构建一个名为helloworld的程序并将其安装在bin目录中。

AM_CXXFLAGS宏设置编译器标志。您不应该在Makefile.am中使用CXXFLAGS，因为它不安全。CXXFLAGS是用户期望能够覆盖的用户变量。

helloworld_SOURCES变量指定用于构建helloworld目标的源文件。请注意，目标的SOURCES变量以目标名称作为前缀，在本例中为helloworld。

最后一个变量helloworld_LDADD指定了必须传递给链接器来构建目标的库。该变量仅供程序和库使用。请注意，LDADD使用与SOURCES变量相同的命名规则。

 

## configure.in

configure.in文件必须位于项目的顶层目录中。切换到<tests / project>目录并创建一个名为configure.in的文本文件。将以下行添加到文件并保存它：

`AC_INIT(src/main.cc)`  PACKAGE=helloworld VERSION=0.1.0 AM_INIT_AUTOMAKE($PACKAGE, $VERSION) INTI_REQUIRED_VERSION=1.0.7 PKG_CHECK_MODULES(INTI, inti-1.0 >= $INTI_REQUIRED_VERSION) AC_SUBST(INTI_CFLAGS) AC_SUBST(INTI_LIBS) AC_PROG_CXX AC_OUTPUT(Makefile src/Makefile)

 

AC_INIT宏对生成的配置脚本执行基本初始化。它将源目录中的文件名作为参数，以确保源目录已被正确指定。

PACKAGE和VERSION变量分别声明包的名称和版本。

AM_INIT_AUTOMAKE宏执行Automake所需的所有标准初始化，并且接受两个参数，即包名称和版本号。

INTI_REQUIRED_VERSION变量指定所需的最小Inti版本，在本例中为1.0.7。

PKG_CHECK_MODULES宏会检查Inti库的指定版本，如果找到，则必须在$（INTI_CFLAGS）中放置必要的包含标志，并将库与$（INTI_LIBS）链接。如果找不到正确的版本，配置会报告错误。

AC_PROG_CXX检查C ++编译器并设置变量CXX，GXX和CXXFLAGS。

必须在configure.in的最后调用最后一个宏AC_OUTPUT来创建Makefiles。

 

## 生成输出文件

现在我们需要从两个输入文件*configure.in*和*Makefile.am中*生成所需的输出文件。首先，我们需要收集configure.in中的所有宏调用，Autoconf将需要构建配置脚本。这是通过以下命令完成的：

 

```
$ aclocal
```

这会生成文件aclocal.m4并将其添加到当前目录。

接下来，运行autoconf：

```
$ autoconf
```

 

运行*autoconf后，*您将在当前目录中找到配置脚本。首先运行*aclocal*非常重要，因为automake依赖configure.in和aclocal.m4中的内容。

 

GNU标准所说的一些文件必须出现在顶层目录中，如果没有找到，Automake会报告一个错误。输入以下命令来创建这些文件：

 

```
$ touch AUTHORS NEWS README ChangeLog
```

现在我们可以运行automake来创建Makefile.in。--add-missing参数将Automake安装中的一些样板文件复制到当前目录中。

```
$ automake --add-missing
```

到目前为止，<tests / project>目录的内容应该看起来很像您以前可能安装的GNU软件包的顶级目录：

```
aclocal.m4 autom4te.cache config.h.in configure.in depcomp install-sh  Makefile.in mkinstalldirs README AUTHORS   ChangeLog    configure  COPYING    INSTALL Makefile.am missing   NEWS      src
```

## 建立和安装项目

此时，您应该能够将源代码树打包到tarball中，并将其提供给其他用户以安装在他们自己的系统上。用户只需解压缩tarball并运行以下命令：

 

```
$ ./configure --prefix=some_directory $ make $ make install
```

 

如果你运行上面的命令并查看你的bin目录，你会发现helloworld。看看可执行文件的大小。哇！它的588千字节。这是因为它包含调试程序所需的所有调试和编译器符号。

现在运行以下命令：

```
$ make install-strip
```

 

如果你现在看helloworld的大小，它会小很多，只有35.7千字节。命令*make install-strip*去除所有调试符号。由此产生的可执行文件更小更快，但您将无法调试程序。通常情况下，只有在稳定时才应该剥离程序。

 

## 维护输入文件

每次编辑软件包中的任何GNU Autotools输入文件时，都必须重新生成输出文件。如果您将新的源文件添加到Makefile.am中的helloworld_SOURCES变量，则必须重新生成Makefile.in。如果你正在构建你的软件包，你将需要重新运行configure来重新生成Makefile文件。许多项目维护人员将必要的命令放入名为*autogen.sh*的脚本中，并在需要重新生成输出文件时运行此脚本。

在顶层目录中创建一个名为autogen.sh的文本文件，并确保更改其文件模式以使其可执行。将以下命令添加到该文件并保存：

 

`#! /bin/sh`  aclocal \ && automake --add-missing \ && autoconf

```
 `现在，您可以轻松运行以下命令来更新项目的输出文件，并重建项目：
` 
$./autogen.sh $ ./configure --prefix=/some_directory $ make $ make install 
```

## 一些有用的链接

本教程应该让你开始使用GNU Autotools，这应该足够了，一段时间。最终，您需要了解更多信息，例如如何构建共享库或应将哪些宏添加到configure.in中。我发现以下链接非常有用：

- [在C ++中使用Automake和Autoconf](http://www.murrayc.com/learning/linux/automake/automake.shtml)
- [在Automake和Autoconf中使用C / C ++库](http://www.murrayc.com/learning/linux/using_libraries/using_libraries.shtml)
- [使用automake和autoconf构建C / C ++库](http://www.murrayc.com/learning/linux/building_libraries/building_libraries.shtml)
- [GNU Autoconf，Automake和Libtool](http://sources.redhat.com/autobook/)
- [GNU的automake和autoconf手册](http://www.gnu.org/manual/manual.html)

 

注：本文翻译自

[Building a GNU Autotools Project](http://inti.sourceforge.net/tutorial/libinti/autotoolsproject.html)