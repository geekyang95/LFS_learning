#环境检查  
  
###宿主机需要有以下工具:
*bash 		
*ld			//链接器 
*bision			//语法分析器生成器 
*yacc	(可为软链接)	//同bison,bison由yacc改进而来  
*bzip2		  
*chown		  	
*diff		  	//文本内容比较器
*find	
*gawk	
*awk	(可为软链接)	
*gcc	(需要不支持C99版本)	
*g++	
*ldd			//动态依赖库查询	
*grep			//文本查询工具	
*gzip	
*cat	
*m4			//宏处理器	
*make	
*patch			//文件更改	
*perl			
*sed
*tar
*makeinfo		//Ubuntu下为texinfo
*xz			//xz压缩处理
*texinfo			//文档生成器

###库文件一致性检查
	#!/bin/bash
	for lib in lib{gmp,mpfr,mpc}.la; do
	echo $lib: $(if find /usr/lib* -name $lib|
	grep -q $lib;then :;else echo not;fi) found
	done
	unset lib
	运行以上脚本检查，应为都在或都不在

##目标盘处理
	(不明白的参数man查看）
1.创建新分区
	fdisk 用法:man fdisk
	若宿主系统存在swap分区可以公用
2.文件系统创建
	mkfs -v -t ext4 /dev/(your disk)
	新创建的swap分区需要初始化
	mkswap	/dev/(your swap)
3.文件系统挂载
	export LFS=/mnt/lfs	(一次做不完可以将其写入当前用户的~/.bashrc中，下次进入无需再次声明）
	mkdir -pv $LFS	//挂载点
	mount -v -t ext4 /dev/(your disk) $LFS
	mkdir $LFS/boot	(如果创建了boot分区需要挂载，挂载方法同上)
	mkdir $LFS/home (同上)
	之前若无swap需要启用后来建立的swap
	swapon on /dev/(your swap)
4.相关软件包
	mkdir -v $LFS/sources //创建源码存放地
	以下链接可以置于一个文件之中，使用wget一次性下载
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/acl-2.2.52.src.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/attr-2.4.47.src.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/autoconf-2.69.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/automake-1.15.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bash-4.3.30.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bc-1.06.95.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/binutils-2.25.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bison-3.0.4.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bzip2-1.0.6.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/check-0.9.14.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/coreutils-8.23.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/dbus-1.8.16.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/dejagnu-1.5.2.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/diffutils-3.3.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/e2fsprogs-1.42.12.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/expat-2.1.0.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/expect5.45.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/file-5.22.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/findutils-4.4.2.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/flex-2.5.39.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gawk-4.1.1.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gcc-4.9.2.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gdbm-1.11.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gettext-0.19.4.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/glibc-2.21.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gmp-6.0.0a.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gperf-3.0.4.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/grep-2.21.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/groff-1.22.3.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/grub-2.02~beta2.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/gzip-1.6.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/iana-etc-2.30.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/inetutils-1.9.2.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/intltool-0.50.2.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/iproute2-3.19.0.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/kbd-2.0.2.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/kmod-19.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/less-458.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/libcap-2.24.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/libpipeline-1.4.0.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/libtool-2.4.6.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/linux-3.19.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/m4-1.4.17.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/make-4.1.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/man-db-2.7.1.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/man-pages-3.79.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/mpc-1.0.2.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/mpfr-3.1.2.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/ncurses-5.9.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/patch-2.7.4.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/perl-5.20.2.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/pkg-config-0.28.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/procps-ng-3.3.10.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/psmisc-22.21.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/readline-6.3.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/sed-4.2.2.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/shadow-4.2.1.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/systemd-219.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/tar-1.28.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/tcl8.6.3-src.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/texinfo-5.2.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/tzdata2015a.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/util-linux-2.26.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/vim-7.4.tar.bz2
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/XML-Parser-2.44.tar.gz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/xz-5.2.0.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/zlib-1.2.8.tar.xz
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bash-4.3.30-upstream_fixes-1.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bc-1.06.95-memory_leak-1.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/bzip2-1.0.6-install_docs-1.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/coreutils-8.23-i18n-1.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/glibc-2.21-fhs-1.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/kbd-2.0.2-backspace-1.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/mpfr-3.1.2-upstream_fixes-3.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/readline-6.3-upstream_fixes-3.patch
	http://mirrors.ustc.edu.cn/lfs/lfs-packages/7.7-systemd/systemd-219-compat-1.patch
###编译环境准备
	以下命令权限不够时，请sudo之
	mkdir -v $LFS/tools (用于存放交叉编译工具）
	ln -sv $LFS/tools /tools (创建指向$LFS/tools的软链接)
	groupadd lfs //添加lfs组
	useradd -s /bin/bash -g lfs -m -k /dev/null lfs //添加用户lfs，指定shell,指定组，指定script设备
	passwd lfs //密码设置
	chown -v lfs $LFS/tools		//权限变更
	chown -v lfs $LFS/sources
	visudo	//在其中添加lfs,使其拥有sudo权限
	su - lfs //用户切换
	以下动作都发生在lfs用户下，重新进入系统时，请注意切换用户
	为lfs用户bash进行配置：
		~/.bash_profile:
			exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
		~/.bashrc:
			set +h
			umask 022
			LFS=/mnt/lfs
			LC_ALL=POSIX
			LFS_TGT=$(uname -m)-lfs-linux-gnu
			PATH=/tools/bin:/bin:/usr/bin
			export LFS LC_ALL LFS_TGT PATH
	配置文件启用：
		source ~/.bash_profile
	
	
	
#以上，相关环境配置完毕
