## PHP 扩展开发环境和工具

**工欲善其事，必先利其器**

在开发 PHP 扩展之前，我们先得配置好开发环境，选取好开发工具，才好开始工作。

### 配置开发环境

如果我们开发扩展不仅仅是为了自己用的话，那么我们就应该考虑让它支持尽可能多的 PHP 版本。但是不同版本的 PHP 在 API 上会一些不同。尤其是 PHP 5 和 PHP 7 之间的差别更大。所以我们最好是搭建一个安装有多版本 PHP 的开发环境。

我使用的操作系统是 Mac OS X，并在上面安装了一个 ubuntu 的虚拟机，又在 ubuntu 里[用 dockor 安装了一个 centos](http://www.webmaster.me/server/installing-docker-on-debian-wheezy-in-60-seconds.html)。

我使用的 Mac OS X 是 10.10.2 版本，可以[使用内置的 PHP](http://php.net/manual/zh/install.macosx.bundled.php)，但是要开发扩展，还需要安装一下 [XCode](https://developer.apple.com/xcode/downloads/)。Linux 下首先要保证安装了 gcc，make 这些基本开发工具。然后再用 `apt-get install php-dev`（debian/ubuntu 下）或 `yum install php-devel`（redhat/centos 下）安装 PHP 扩展开发环境就可以了。

但是仅有这些内置的开发环境还是不够用的。要是自己手动下载 [PHP 源码](https://github.com/php/php-src)来编译安装各个不同版本的开发环境呢，倒是也可以。但我是个懒人，所以我找了两个更方便的工具来安装各个不同版本的 PHP 开发环境，并且可以随意切换。它们分别是 [phpenv](https://github.com/chh/phpenv) 和 [php-build](https://github.com/chh/php-build)，至于使用方法呢，这两个工具的使用说明上都写的很清楚了，我就不多说了。

这三个系统下都可以使用这两个工具来编译安装 PHP
开发环境。至于 Windows 下是否支持，我就不知道了，因为我还没有尝试过在 Windows 下开发。所以 Windows 下如何配置开发环境，我就不介绍了。

### 选取开发工具

牛 A 和牛 C 之间的程序员通常喜欢使用 VIM 或 Emacs 加上一堆插件来做开发，但是对我这种连 [Eclipse](http://www.eclipse.org/) 都用不惯的菜鸟级的程序员来说，我更喜欢 [Netbeans](http://www.netbeans.org/)，只因为它配置简单，使用方便。毕竟我不会直接在服务器上用命令行做开发，而且我的电脑也没有差到只能跑字符界面系统的程度，所以就怎么简单怎么来咯。其实开发工具除了上面这几个以外还有很多，选你自己喜欢的就行了。

调试工具呢，在 Linux 下用 gdb， Mac OS X 下用 lldb 就可以了，它俩的用法差不多。通常就是你的测试程序跑崩了之后，会生成一个 core 文件，之后用 `gdb /path/to/php core` 打开 core 文件，用 bt 命令查看是从哪行崩溃的就可以了。这里的 `/path/to/php` 你得替换成实际的 php 命令的路径。如果你的程序跑崩了却没有产生 core 文件，你可以试试执行 `ulimit -c unlimited` 这条命令，然后再执行你的程序，你一般就会找到那个 core 文件了，如果还找不到，那就到 /cores，/tmp 或 /var/tmp 下看看有没有，在这些目录下的 core 文件通常会带一个数字的后缀，你只要知道就是那个东西就行了。你也可以先用 `gdb /path/to/php` 来启动 gdb，然后用 `run test.php` 来运行你的测试程序，如果执行崩了，你会直接看到崩溃信息。这个调试工具的功能当然不是只有这么多，不过我就会这么多，但感觉已经足够用了。如果你想要了解更多，可以去查这方面的资料，反正这种资料是不缺的。

不过有的时候，你为扩展写的测试程序虽然跑崩了，但是用 gdb 却很难定位出错的位置；或者你的测试程序没有跑崩，但是运行的结果却出现了一些莫名其妙的情况；又或者你的程序既没崩溃，结果也很正常，但是却存在内存泄漏（memory leak）[^1]。这个时候，你就需要另外一个工具 [valgrind](http://valgrind.org/) 来定位这些错误了。这个工具的使用也很简单。下面是我曾经用过一条命令行：

```sh
USE_ZEND_ALLOC=0 valgrind /home/andot/.phpenv/versions/master/bin/php  -n -c '/mnt/hgfs/Work/Git/hprose-pecl/tmp-php.ini'  -d "output_handler=" -d "open_basedir=" -d "safe_mode=0" -d "disable_functions=" -d "output_buffering=Off" -d "error_reporting=32767" -d "display_errors=1" -d "display_startup_errors=1" -d "log_errors=0" -d "html_errors=0" -d "track_errors=1" -d "report_memleaks=1" -d "report_zend_debug=0" -d "docref_root=" -d "docref_ext=.html" -d "error_prepend_string=" -d "error_append_string=" -d "auto_prepend_file=" -d "auto_append_file=" -d "magic_quotes_runtime=0" -d "ignore_repeated_errors=0" -d "precision=14" -d "memory_limit=128M" -d "extension_dir=/mnt/hgfs/Work/Git/hprose-pecl/modules/" -d "extension=hprose.so" -d "session.auto_start=0" -d "tidy.clean_output=0" -d "zlib.output_compression=Off" -d "mbstring.func_overload=0" -f "/mnt/hgfs/Work/Git/hprose-pecl/tests/writer.php"
```

也许你会惊呼，怎么这条命令这么长，这让我怎么输入啊，啊，啊……

其实我也不是一个字符一个字符敲进去的，而是复制上去的。如果你的测试程序没有跑通过，在测试目录下，就会生成一个跟测试文件同名，但扩展名是 sh 的文件。你打开这个文件，把里面的命令行复制出来，贴到 `USE_ZEND_ALLOC=0 valgrind` 的后面就可以了。也许你可能会说，那我直接在后面跟脚本的名字不就行了。你可以试一下，虽然也可以出结果，但你会发现结果是不一样的。

如果你的程序中有内存泄漏，或者有重复的内存释放，或者释放了不该释放的值。那么你就可以看到这些错误的详细信息了。

不过 valgrind 这个工具似乎对 Mac OS X 10.10 支持的不是很好，至少还没有官方版本，通过 [homebrew](http://brew.sh/) 也安装不了。这也是我为啥还要搞个 Linux 虚拟机的原因之一[^2]。

开发环境和工具先介绍到这儿，以后要是有新发现再补充。接下来我们就进入实战阶段吧。

[^1] 如果你在编译 PHP 开发环境时开启了 debug 模式，而你的扩展又存在内存泄漏，有些时候你是可以在运行结果中看到内存泄漏提示的。

[^2] 另一个原因是在不同系统下编写扩展时，本身也有一些差别，所以需要安装多个系统来做测试。
