---
layout: post
title:  "肿么安装Effect Game"
date:   2015-07-22 15:52:34
categories: docs
---

Effect Game的安装还是很麻烦的, 原作者用的时代比较久远, 有很多软件包在新的linux系统上都可能有兼容问题, 这里做个安装记录方便后人查阅.

## 环境准备

1. fedora 21
2. 能翻墙的比较快的网络


Effect Game的需要运行在linux系统里, 理论上任何linux发行版都可以, 但是我在ubuntu上折腾了好久, 总归还是放弃了. 

各个发行版虽然内核一样, 但是各种软件包可能不一样, 名字也可能不一样, 包管理工具也不一样, 所以环境这里, 还是建议用 `fedora 21`, 而且不能用最新版, 最新版的fedora把 yum 给替换成了 dnf, 还是很多问题.

这里的安装步骤是基本按照这个E文文档的, [https://github.com/jhuckaby/Effect-Games/wiki/Installation-Guide](), 启用能用yum安装的软件包都直接用yum了.

## 安装各种系统依赖包

作者列了yum install xxx 很多包, 但是其实可以批量安装的

如果是第一次用yum, 先更新下yum的版本库 `yum repolist` 

批量安装, 安装完成后再敲入整个命令确认是否有安装失败的(已经安装了的不会重复安装)

    yum install zlib-devel libxml2-devel libgpg-error-devel libgcrypt-devel libxslt-devel expat-devel db4-devel e2fsprogs-devel krb5-devel openssl-devel aspell-devel rpm-build gcc screen sendmail "gcc-c++" bzip2-devel freetype-devel libpng-devel libtiff-devel libjpeg libjpeg-devel libstdc++-devel curl curl-devel libidn-devel krb5-devel e2fsprogs-devel libgcrypt-devel libgpg-error-devel

## 创建 Effect Game 的安装目录

这里就直接按照作者的目录来设置, 大部分东东都放到这里, 避免各种引用要修改	

	mkdir /effect

## 安装 perl 和 cpan

    yum install perl
    yum install cpan
    
作者把 perl 放到了 /effect/perl 里, 这里做个软连就好

	ln -s /usr/bin/perl /effect/perl/bin/perl 
	ln -s /usr/bin/cpan /effect/perl/bin/cpan

## 安装各种 perl 模块
第一次使用cpan时, 直接敲入 `cpan` 运行一次, 按提示使用自动配置

批量安装各种模块, 安装完成后再敲入整个命令确认是否有安装失败的(已经安装了的不会重复安装)

    cpan LWP::UserAgent Archive::Zip Archive::Tar Class::Loader Digest::SHA Digest::SHA1 Math::Pari Devel::Symdump Module::Build MIME::Lite Test::Simple Text::Wrap CGI Devel::CoreStack Cwd IO::Socket::INET Test::Harness Time::HiRes Unicode::String Date::Parse MIME::Parser Crypt::SSLeay ExtUtils::XSBuilder IO::Socket::SSL File::lockf IPC::ShareLite Syntax::Highlight::Engine::Kate Text::Aspell Net::Twitter::Lite HTTP::Daemon File::Flock

这个需要等很久很久, 下载很久很久.

## 安装图片服务 ImageMagick
ImageMagic不要用yum安装, 编译时需要指定很多环境变量, 直接下载下载自己编译.


安装jpeg和png解码库

    yum install libjpeg-devel libpng-devel

作者用的ImageMagick是6.5版本, 可能会编译失败, 这里下载最新版(目前是6.9).


    wget "http://www.imagemagick.org/download/ImageMagick.tar.gz"
    tar zxf ImageMagick.tar.gz
    cd ImageMagick-*
    
    ln -s /usr/include/freetype2/freetype /usr/include/freetype
    export CPPFLAGS="-I/usr/include/freetype2 -I/usr/local/include"
    export LDFLAGS="-L/usr/local/lib"
    mkdir -p /effect/fonts

    ./configure --prefix=/effect/magick --enable-lzw --without-threads --without-magick-plus-plus --with-perl=perl --with-fontpath=/effect/fonts --without-x --with-quantum-depth=8 --enable-shared=yes --enable-static=no

    make
    make install
   
添加库到系统

    echo "/effect/magick/lib" > /etc/ld.so.conf.d/imagemagick.conf

重新加载配置

	/sbin/ldconfig -v

测试是否已正常安装

	perl -MImage::Magick -e ';'


没有出错就安装OK

创建个软链

    mkdir -p /opt/local/bin
    ln -s /effect/magick/bin/convert /opt/local/bin/convert

## 安装音视频模块 MPG123 and OGG Encoder

安装 MPG

	wget "http://www.effectgames.com/install/mpg123-1.10.0.tar.bz2"
    tar jxf mpg123-1.10.0.tar.bz2
    cd mpg123-1.10.0
    ./configure
    make
    make install
    
安装 OGG

    yum install libvorbis libvorbis-devel vorbis-tools
    
检查下面文件在不在, 有就安装正确了
    
    /usr/local/bin/mpg123
    /usr/bin/oggenc
    
## 安装 Apache

先安装或更新一下openssl

	yum install openssl
	
Apache也有很多安装选项, 自己下载来编译

	wget "http://www.effectgames.com/install/httpd-2.2.14.tar.bz2"
    tar jxf httpd-2.2.14.tar.bz2
    cd httpd-2.2.14

    ./configure --prefix=/effect/apache --with-mpm=prefork --with-ssl=openssl --with-perl=perl --enable-so --enable-expires --enable-headers --enable-mime-magic --enable-rewrite --enable-ssl --enable-mods-shared="all authn_file authn_default authz_host authz_groupfile authz_user authz_default auth_basic include filter log_config env setenvif mime status autoindex asis cgi negotiation dir actions userdir alias expires headers ssl rewrite"
	
    make
    make install

之后Apache会安装到 `/effect/apache`

安装 mod_perl 模块

    yum install perl-ExtUtils-Embed
    
增加www用户组和用户

    /usr/sbin/groupadd www
    /usr/sbin/useradd -m -g www www
    
替换配置文件, 只用用作者的配置

	wget http://www.effectgames.com/install/httpd.conf
	mv /effect/apache/conf/httpd.conf /effect/apache/conf/httpd.conf.bak
	mv httpd.conf /effect/apache/conf/
	

## 配置项目

先下载项目代码下来

    wget "https://github.com/jhuckaby/Effect-Games/tarball/master"
    tar zxf jhuckaby-Effect-Games-*.tar.gz
    rm jhuckaby-Effect-Games-*.tar.gz
    mv jhuckaby-Effect-Games-* /effect
    ln -s /effect/htdocs /effect/apache/htdocs/effect
    
做一些必要配置, 具体项直接参照 [https://github.com/jhuckaby/Effect-Games/wiki/Installation-Guide](安装文档)

Effect Game的主要配置文件在 `/effect/conf/Effect.xml`, 基本上都不需要修改

启动测试一下是否正常

    /effect/apache/bin/apachectl start
    /effect/apache/bin/apachectl stop
    
## 初始化各种环境

创建数据存储目录和配置权限

    mkdir -p /data/effect/storage
    chown www:www /data/effect/storage

    mkdir -p /data/effect/cleanup
    chown www:www /data/effect/cleanup
    
创建日志的目录

    mkdir -p /logs/effect
    chmod 777 /logs/effect

    mkdir -p /logs/apache
    chmod 777 /logs/apache
    
创建初始数据, 执行这个脚本就行, 数据的配置在 `/effect/conf/initial_data_setup.xml`

    /effect/bin/setup_db.pl
    
启动图片服务进程, Effect Game是使用了个后台进程来优化图片, 配置文件在 `/effect/conf/image_service.xml`, 用下面的命令启动

    mkdir -p /data/effect/queue
    chmod 777 /data/effect/queue
    
    /effect/bin/imageservicectl.sh start
    /effect/bin/imageservicectl.sh stop

配置定时任务, 这个任务用来处理日志文件和数据文件, 命令行敲入 `crontab -e`, 输入

    00 * * * * /effect/bin/maint.pl hourly
	30 00 * * * /effect/bin/maint.pl daily
	
定时任务的log输出在 `/logs/effect/maint.log`

过时的日志文件会被打包存储, 默认配置下存在 `/backup/effect/logs/CATEGORY/YYYY/MM/DD/HH.gz`, 路径可以在 `/effect/conf/Effect.xml` 中修改

创建备份目录

    mkdir -p /backup/effect
    chmod 777 /backup/effect
    
## All Done

现在可以启动 Effect Game 啦

    /effect/bin/imageservicectl.sh start
    /effect/apache/bin/apachectl start
    
然后访问 `http://127.0.0.1/effect/`, 默认用户名和密码都是 `admin`

如果要配置域名, 可以在 `/effect/apache/conf/httpd.conf` 中修改 `ServerName` 

So, enjoy it!
