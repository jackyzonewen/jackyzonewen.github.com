---
layout: post
title: "ruby rvm brew gem bundler梳理"
date: 2014-01-06 16:07
comments: true
categories: other
tags: [ruby]
---


**Ruby**是日本人发明的一种脚本语言。

---

**[Brew][1]**是OS X上提供软件包的管理。Homebrew将软件包安装到单独的目录，然后符号链接到/usr/local 中，完全基于git和ruby。使用gem来安装你的gems，用brew来搞定他们的依赖包。

Mac OS X是基于Unix的操作系统，可以安装大部分为Unix/Linux开发的软件。如果只是以使用为目的，对每个软件都进行手工编译不是很方便，也不利于管理已安装的软件，于是出现了类似于Linux中APT、Yum等类似的软件包管理系统，Mac下比较著名的有MacPorts、Fink、Homebrew等。

MacPorts有个原则，对于软件包之间的依赖，都在MacPorts内部解决（/opt/local），无论系统本身是否包含了需要的库，都不会重用。这使得MacPorts过分的庞大臃肿，导致系统出现大量软件包的冗余，占用不小的磁盘空间，同时稍大型一点的软件编译时间都会难以忍受。

Homebrew恰恰相反，它尽可能地利用系统自带的各种库，使得软件包的编译时间大为缩短；同时由于几乎不会造成冗余，软件包的管理也清晰、灵活了许多。Homebrew的另一个特点是使用Ruby定义软件包安装配置(formula)，定制非常简单。

Fink暂不讨论。

首先确保你的系统满足如下要求：
 1. 基于Intel CPU
 2. 操作系统为Mac OS X 10.5 Leopard或更高版本
 3. 已安装版本管理工具Git（Mac OS X 10.7 Lion已经预安装
 4. 已安装Xcode开发工具1(Command Line Tools)
 5. 已安装Java Developer Update2

注1：Xcode 4.3中，命令行编译工具是可选安装，需要在Preferences > Downloads中激活。<br>
注2：可选，Homebrew本身不依赖于Java，只有部分软件包的安装需要Java支持。

Homebrew的安装:

    ruby -e "$(curl -fsSL https://raw.github.com/mxcl/homebrew/go)"
   
或者

    ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)" 

由于Homebrew的安装地址可能变化，请到[官方网站][2]查看最新的安装方法。

Homebrew的使用: Homebrew的可执行命令是brew，其基本使用方法如下（以软件包wget为例）。

查找软件包

    brew search wget

安装软件包

    brew install wget

列出已安装的软件包

    brew list

删除软件包

    brew remove wget

查看软件包信息

    brew info wget

列出软件包的依赖关系

    brew deps wget

更新brew

    brew update

列出过时的软件包（已安装但不是最新版本）

    brew outdated

更新过时的软件包（全部或指定）

    brew upgrade 或 brew upgrade wget

定制自己的软件包（假定软件包名称是bar，来自foo站点）。

首先找到待安装软件的源码下载地址
http://foo.com/bar-1.0.tgz

建立自己的formula
brew create http://foo.com/bar-1.0.tgz

编辑formula，上一步建立成功后，Homebrew会自动打开新建的formula进行编辑，也可用如下命令打开formula进行编辑:

    brew edit bar

Homebrew自动建立的formula已经包含了基本的configure和make install命令，对于大部分软件，不需要进行修改，退出编辑即可。

输入以下命令安装自定义的软件包

    brew install bar


---

**[RubyGems][3]**是一个包管理框架，提供了ruby社区gem的托管服务，用于方便地下载、安装和使用ruby软件包(gem)，包含了ruby应用或库,要升级到最新的RubyGems，运行：

    $ gem update --system

如果没有安装RubyGems，则需要先下载安装包，然后解压开后运行：

    ruby setup.rb

---

**Gem**也是Ruby软件包管理工具,如果要看安装的gem文档，一是可以用ri，二是可以gem server启动一个web服务。详细的帮助参见[RubyGems Guides][4]。

brew和gem不同，brew用于操作系统层面上软件包的安装，而gem只是管理ruby软件,gem常用命令如下:

	gem -v #gem版本
	gem update #更新所有包 
	gem update --system #更新RubyGems软件 
	gem install rake #安装rake,从本地或远程服务器 
	gem install rake --remote #安装rake,从远程服务器 
	gem install watir -v(或者--version) 1.6.2#指定安装版本的 
	gem uninstall rake #卸载rake包 
	gem list d #列出本地以d打头的包
	gem list --local | grep cocoapods #列出cocoapods相关的set 
	gem query -n ''[0-9]'' --local #查找本地含有数字的包 
	gem search log --both #从本地和远程服务器上查找含有log字符串的包 
	gem search log --remoter #只从远程服务器上查找含有log字符串的包 
	gem search -r log #只从远程服务器上查找含有log字符串的包 
	gem help #提醒式的帮助 
	gem help install #列出install命令 帮助 
	gem help examples #列出gem命令使用一些例子 
	gem build rake.gemspec #把rake.gemspec编译成rake.gem 
	gem check -v pkg/rake-0.4.0.gem #检测rake是否有效 
	gem cleanup #清除所有包旧版本，保留最新版本 
	gem contents rake #显示rake包中所包含的文件 
	gem dependency rails -v 0.10.1 #列出与rails相互依赖的包 
	gem environment #查看gem的环境

注意：安装的时候不要使用sudo，默认的Mac OS安装了Ruby，所以这个时候如果使用sudo，更新的是系统版本的ruby对应的gems。

---

**[RVM][5]**（Ruby enVironment Version  ) Manager）是一个命令行工具，提供在多个ruby环境中方便的安装、管理和工作，包括解释器和gem集合。rvm自己的安装通过curl命令执行，如：curl -L https://get.rvm.io | bash。

RVM有一个非常灵活的gem管理系统，称为Gem Sets。RVM的’gemsets’管理横跨多个Ruby版本的gems包。

这里所有的命令都是再用户权限下操作的，任何命令最好都不要用sudo.

rvm安装:

    bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)
    
设置环境变量:

    echo '[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm" # Load RVM function' >> ~/.bash_profile
    source ~/.bash_profile


修改 RVM 的 Ruby 安装源到国内的 淘宝镜像服务器，这样能提高安装速度

    sed -i -e 's/ftp\.ruby-lang\.org\/pub\/ruby/ruby\.taobao\.org\/mirrors\/ruby/g' ~/.rvm/config/db

列出已知的ruby版本

    rvm list known

安装一个ruby版本

    rvm install 1.9.3

这里安装了最新的1.9.3, rvm list known列表里面的都可以拿来安装。

使用一个ruby版本

    rvm use 1.9.3

如果想设置为默认版本，可以这样

    rvm use 1.9.3 --default 

查询已经安装的ruby

    rvm list

卸载一个已安装版本

    rvm remove 1.9.2
    
rvm连同ruby一起卸载调

    rvm implode

一键将rvm,Ruby连同bundler一起装好

    curl -L https://get.rvm.io | bash -s stable --ruby

---

**[Bundler][6]**为ruby维持一个一致性的环境，跟踪应用代码和所需要的ruby gems，这样一个应用可以有所需要的精确的gems（和版本）安装通过如下命令：

    $ gem install bundler

---

安装的顺序，先安装rvm，之后选择安装一个ruby版本，就可以提供一个完整的ruby运行环境。之后可以安装brew（brew虽然是管理os的，但基于ruby）和gem，分别管理操作系统和ruby的软件包。之后ruby重新编译的时候所依赖的包可以使用brew安装。有了gem之后，bundler只不过就是一个gem，直接通过gem install 即可。


  [1]: https://github.com/Homebrew/homebrew/wiki
  [2]: https://github.com/Homebrew/homebrew/wiki
  [3]: http://rubygems.org/
  [4]: http://guides.rubygems.org/
  [5]: https://rvm.io/
  [6]: http://bundler.io/