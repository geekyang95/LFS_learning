#临时系统构建
LFS系统构建是采用host->temp->target的结构

之前编译的交叉编译工具即是temp的部分，此处要做的是利用之前搭建好的工具链搭建一个host无关环境(虽然它仍旧运行在原来的机器上，但是和原来的机器上根文件系统是完全无关的)，之所以这样做，而不是藉由工具链直接编译目标系统，是为了防止在编译过程中可能出现的host依赖问题，这样的问题一般难以检查，在完全熟练linux系统的编译过程后，其实临时系统是不需要的，此处仅是方便学习提高成功率。

此处编译的临时系统，其实也是一个linux系统，只不过该系统并不完善，文件系统结构无，启动相关无，在target系统配置时，其实就是进行文件系统，启动设置，以及一些常用软件的附带编译.

1.Tcl(和Expect,DejaGNU,Check一起，用来提供运行GCC和Binutils测试工具集的环境)
cd $LFS/sources
tar -zxf tcl8.6.3-src.tar.gz
cd tcl8.6.3
cd unix
./configure --prefix=/tools		//prefix指定安装目录，以下不再说明
make
make install
chmod -v u+w /tools/lib/libtcl8.6.so	//更改其库文件为可写，便于之后移除调试符号
make install-private-headers		//安装头文件，Expect需要这部分文件
ln -sv tclsh8.6 /tools/bin/tclsh	//制作软链接便于其他软件调用
rm -rf $LFS/sources/tcl8.6.3

2.Expect
cd $LFS/sources
tar -zxf expect5.45.tar.gz
cd expect5.45
cp -v configure{,.orig}
sed 's:/usr/local/bin:/bin:' configure.orig > configure //使得Expect使用/bin/stty而不是/usr/local/bin/stty
./configure --prefix=/tools \
--with-tcl=/tools/lib \				//tcl指定
--with-tclinclude=/tools/include		//tcl头文件指定
make
make SCRIPTS="" install				//此处不需要脚本
rm -rf $LFS/sources/expect5.45

3.DejaGNU
cd $LFS/sources
tar -zxf dejagnu-1.5.2.tar.gz
cd dejagnu-1.5.2
./configure --prefix=/tools
make install
rm -rf $LFS/sources/dejagnu-1.5.2

4.check
cd $LFS/sources
tar -zxf check-0.9.14.tar.gz
cd check-0.9.14
PKG_CONFIG= ./configure --prefix=/tools //PKG_CONFIG=是为了使config脚本忽略任何与pkg_config相关的可能会导致链接到/tools之外的库的选项
make
make install
rm -rf $LFS/sources/check-0.9.14

5.Ncurses(操作文本终端的库)
cd $LFS/sources
tar -zxf ncurses-5.9.tar.gz
cd ncurses-5.9
./configure --prefix=/tools \
--with-shared \
--without-debug \
--without-ada \				//去掉为ADA提供的支持，避免进入chroot环境时出现问题
--enable-widec \			//widec库支持8位以上机器，通常的库只支持8位机器
--enable-overwrite			//使ncurse库装载位置为/tools/include而不是/tools/include/ncurses,这样才可以被其他程序准确搜索到
make
make install
rm -rf $LFS/sources/ncurses-5.9

6.bash(shell)
cd $LFS/sources
tar -zxf bash-4.3.30.tar.gz
cd bash-4.3.30
./configure --prefix=/tools \
	--without-bash-malloc		//确保bash使用Glibc中的malloc而不是bash的，bash-malloc会造成段错误的问题
make
make install
ln -sv bash /tools/bin/sh
rm -rf $LFS/sources/bash-4.3.30

7.bzip2(.bzip2文件的解压工具)
cd $LFS/sources
tar -zxf bzip2-1.0.6.tar.gz
cd bzip2-1.0.6				//bzip2不含configure,自带make
make
make PREFIX=/tools install
rm -rf $LFS/sources/bzip2-1.0.6

8.coreutils(文件工具集)
cd $LFS/sources
tar -Jxf coreutils-8.23.tar.xz
cd coreutils-8.23
./configure --prefix=/tools \
	--enable-install-program=hostname	//perl的测试集需要，默认不含
make
make install
rm -rf $LFS/sources/coreutils-8.23

9.diffutils(文件差异比较工具)					//不再细说配置，详情可参见./configure --help
cd $LFS/sources
tar -Jxf diffutils-3.3.tar.xz
cd diffutils-3.3
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/diffutils-3.3

10.file(文件类型探测工具)
cd $LFS/sources
tar -zxf file-5.22.tar.gz
cd file-5.22
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/file-5.22

11.findutils(find工具集)
cd $LFS/sources
tar -zxf findutils-4.4.2.tar.gz
cd findutils-4.4.2
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/findutils-4.4.2

12.gawk()
cd $LFS/sources
tar -Jxf gawk-4.1.1.tar.xz
cd gawk-4.1.1
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/gawk-4.1.1

13.gettext()
cd $LFS/sources
tar -zxf gettext-0.19.4.tar.xz
cd gettext-0.19.4
cd gettext-tools
EMACS="no" ./configure --prefix=/tools --disable-shared
make -C gnulib-lib
make -C intl pluralx.c
make -C src msgfmt
make -C src msgmerge
make -C src xgettext
cp -v src/{msgfmt,msgmerge,xgettext} /tools/bin
rm -rf $LFS/sources/gettext-0.19.4

14.grep(文本内容搜索工具)
cd $LFS/sources
tar -Jxf grep-2.21.tar.xz
cd grep-2.21
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/grep-2.21

15.gzip
cd $LFS/sources
tar -Jxf gzip-1.6.tar.xz
cd gzip-1.6
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/gzip-1.6

15.M4(宏处理工具)
cd $LFS/sources
tar -jxf m4-1.4.17.tar.xz
cd m4-1.4.17
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/m4-1.4.17

16.make
cd $LFS/sources
tar -jxf make-4.1.tar.bz2
cd make-4.1
./configure --prefix=/tools --without-guile
make
make install
rm -rf $LFS/sources/make-4.1

17.patch
cd $LFS/sources
tar -Jxf patch-2.7.4.tar.xz
cd patch-2.7.4
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/patch-2.7.4

18.perl
cd $LFS/sources
tar -jxf perl-5.20.2.tar.bz2
cd perl-5.20.2
sh Configure -des -Dprefix=/tools -Dlibs=-lm
make
cp -v perl cpan/podlators/pod2man /tools/bin
mkdir -pv /tools/lib/perl5/5.20.2
cp -Rv lib/* /tools/lib/perl5/5.20.2
rm -rf $LFS/sources/perl-5.20.2

19.sed
cd $LFS/sources
tar -jxf sed-4.2.2.tar.bz2
cd sed-4.2.2
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/sed-4.2.2

20.tar
cd $LFS/sources
tar -jxf tar-1.28.tar.xz
cd tar-1.28
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/tar-1.28\

21.texinfo
cd $LFS/sources
tar -Jxf texinfo-5.2.tar.xz
cd texinfo-5.2
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/texinfo-5.2

22.util-linux
cd $LFS/sources
tar -Jxf util-linux-2.26.tar.xz
cd util-linux-2.26
./configure --prefix=/tools \
--without-python \
--disable-makeinstall-chown \
--without-systemdsystemunitdir \
PKG_CONFIG=""
make
make install
rm -rf $LFS/sources/util-linux-2.26

23.xz
cd $LFS/sources
tar -Jxf xz-5.2.0.tar.xz
cd xz-5.2.0
./configure --prefix=/tools
make
make install
rm -rf $LFS/sources/xz-5.2.0

#无用内容处理
cd ~
strip --strip-debug /tools/lib/*
/usr/bin/strip --strip-unneeded /tools/{,s}bin/*
rm -rf /tools/{,share}/{info,man,doc}

#属主变更
接下来的操作是在root身份下执行，因此tools及其子体都要进行相应的属主变更以适应接下来的操作，在此之前你可以备份刚才的tools，以免接下来的误操作带来的影响
exit
chown -R root:root $LFS/tools

临时系统构建完成，接下来就是进入临时系统去构建真正的LFS系统(其实主要就是文件系统，启动配置，软件编译)
