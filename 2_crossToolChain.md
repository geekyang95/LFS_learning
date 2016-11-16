#交叉编译工具链制作
#由于LFS系统只是针对x86体系进行的版本，此处的交叉仅指不同系统的交叉，并非体系架构的交叉
1.binutils-2.25(Gnu二进制工具集)
cd $LFS/sources
tar -jxf binutils-2.25.tar.bz2
cd binutils-2.25
mkdir -v ../binutils-build 	//建议在源码目录之外一个专门的编译目录里面编译
cd ../binutils-build		
../binutils-2.25/configure \
--prefix=/tools \		//程序安装目录
--with-sysroot=$LFS \		//目标系统所需库搜索目录
--with-lib-path=/tools/lib \	//链接器所用库路径
--target=$LFS_TGT \		//目标系统指定
--disable-nls \			//只支持英文，去掉对别的语言的支持
--disable-werror		//阻止编译器的警告中断编译过程
make -j4
//如果是在 x86_64 上编译，创建符号链接，以确保工具链的完整性
case $(uname -m) in
x86_64) mkdir -v /tools/lib && ln -sv lib /tools/lib64 ;;
esac
make install
rm -rf $LFS/sources/binutils-build
rm -rf $LFS/sources/binutils-2.25

2.gcc-4.9.2(注意该gcc是不含Glibc的，因此不支持包含对系统调用的编译,仅用于内核和bootloader等不需系统调用的编译)
cd $LFS/sources
tar -jxf gcc-4.9.2.tar.bz2
cd gcc-4.9.2
tar -xf ../mpfr-3.1.2.tar.xz	//mpfr gmp mpc都是GNU浮点运算库
mv -v mpfr-3.1.2 mpfr
tar -xf ../gmp-6.0.0a.tar.xz
mv -v gmp-6.0.0 gmp
tar -xf ../mpc-1.0.2.tar.gz
mv -v mpc-1.0.2 mpc
for file in \				//确保gcc使用的是刚才由Binutils编译出的/tools中的动态链接器
	$(find gcc/config -name linux64.h -o -name linux.h -o -name sysv4.h)
	do
	cp -uv $file{,.orig}
	sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
	    -e 's@/usr@/tools@g' $file.orig > $file
	echo '
	#undef STANDARD_STARTFILE_PREFIX_1			
	#undef STANDARD_STARTFILE_PREFIX_2
	#define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
	#define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
	touch $file.orig
	done
sed -i '/k prot/agcc_cv_libc_provides_ssp=yes' gcc/configure	//GCC栈保护，避免GLIBC时编译出错
mkdir -v ../gcc-build
cd ../gcc-build
../gcc-4.9.2/configure \
--target=$LFS_TGT \			//该gcc编译出的程序的运行环境指定
--prefix=/tools \			//安装目录
--with-sysroot=$LFS \			//目标系统库搜索目录
--with-newlib \				//系统调用库包括Glibc以及Newlib,指定为Newlib可以确保不用宿主系统的GLibc，保证编译出的gcc是干净的
--without-headers \			//交叉编译工具不需要目标系统头文件支持，去掉
--with-local-prefix=/tools \		//指定本地安装库为/tools
--with-native-system-header-dir=/tools/include \	//生成库置于/tools/include
--disable-nls \				//语言仅支持英语即可
--disable-shared \			//关闭动态链接，避免搜索到宿主机的Glibc动态库造成的冲突
--disable-multilib \			//x86_64系统不支持multilib
--disable-decimal-float \
--disable-threads \
--disable-libatomic \
--disable-libgomp \
--disable-libitm \
--disable-libquadmath \
--disable-libsanitizer \
--disable-libssp \
--disable-libvtv \
--disable-libcilkrts \
--disable-libstdc++-v3 \		//由于当前还没有编译系统库，而以上特性都需要libc的支持，因此去掉
--enable-languages=c,c++		//只有C/C++会被构建
make
make install				//若是此时用该GCC进行编译，会报ld找不到crt1.o,crti.o的错，在glibc编译后，该GCC可以进行编译，因此crt1.o,crti.o应该是glibc而来，而不是gcc而来.
rm -rf $LFS/sources/gcc-build
rm -rf $LFS/sources/gcc-4.9.2

3.linux API headers
cd $LFS/sources
tar -Jxf linux-3.19.tar.xz
cd linux-3.19
make mrproper					
make INSTALL_HDR_PATH=dest headers_install	//系统头文件提取，置于dest文件夹
cp -rv dest/include/* /tools/include
rm -rf $LFS/sources/linux-3.19

4.Glibc(GNU c运行库)
cd $LFS/sources
tar -Jxf glibc-2.21.tar.xz
cd glibc-2.21
if [ ! -r /usr/include/rpc/types.h ]; then	//确保宿主机系统中含有RPC相关库
su -c 'mkdir -pv /usr/include/rpc'
su -c 'cp -v sunrpc/rpc/*.h /usr/include/rpc'
fi
sed -e '/ia32/s/^/1:/' \
-e '/SSE2/s/^1://' \
-i sysdeps/i386/i686/multiarch/mempcpy_chk.S
mkdir -v ../glibc-build					//此处编译时并未指明CC等为之前编译的GCC恰好证明该glibc只是本机同架构的glibc,与开篇所说吻合
cd ../glibc-build
../glibc-2.21/configure \
--prefix=/tools \					//安装目录
--host=$LFS_TGT \					//运行环境指定
--build=$(../glibc-2.21/scripts/config.guess) \
--disable-profile \
--enable-kernel=2.6.32 \				//最低内核支持版本指定
--with-headers=/tools/include \				//指向刚才编译的内核API
libc_cv_forced_unwind=yes \				//之前编译的binutils需要Glibc建立之后才能用，因此此处需手动指明forced_Unwind可用(二者互相依赖）
libc_cv_ctors_header=yes \
libc_cv_c_cleanup=yes					//同Unwind
make
make install
rm -rf $LFS/sources/glibc-build
rm -rf $LFS/sources/glibc-2.21

交叉编译环境检测
	echo "main(){}">test.c
	$LFS_TGT-gcc test.c
	readelf -l a.out | grep ': /tools'
	结果应为：[Requesting program interpreter: /tools/lib/ld-linux.so.2]

5.libstdc++-4.9.2(c++运行库)
cd $LFS/sources
tar -jxf gcc-4.9.2.tar.bz2
cd gcc-4.9.2
mkdir -pv ../gcc-build
cd ../gcc-build
../gcc-4.9.2/libstdc++-v3/configure \
--host=$LFS_TGT \			//指定运行时环境
--prefix=/tools \			//安装路径
--disable-multilib \			//x86_64不支持multilib
--disable-shared \			//动态库支持关闭
--disable-nls \
--disable-libstdcxx-threads \		//由于编译的gcc不支持线程，g++也不能支持
--disable-libstdcxx-pch \		//避免之前编译的头文件的干扰，此处不需要
--with-gxx-include-dir=/tools/$LFS_TGT/include/c++/4.9.2 //编译出的g++的库搜索路径
make	-j4
make install
rm -rf $LFS/sources/gcc-build
rm -rf $LFS/sources/gcc-4.9.2

6.Binutils_2(第二次编译使其不依赖于宿主机系统)
cd $LFS/sources
tar -jxf binutils-2.25.tar.bz2
cd binutils-2.25
mkdir -v ../binutils-build
cd ../binutils-build
CC=$LFS_TGT-gcc \			//指定编译器为刚才编译的交叉编译工具，此处的交叉从架构上来讲并无意义，真正的目的是将其lib进行严格的限定
AR=$LFS_TGT-ar \			
RANLIB=$LFS_TGT-ranlib \
../binutils-2.25/configure \
--prefix=/tools \			//安装目录
--disable-nls \				//信息仅支持英语
--disable-werror \
--with-lib-path=/tools/lib \		//链接器库搜索目录
--with-sysroot				//库搜索路径
make
make install
rm -rf binutils-build
rm -rf binutils-2.25

7.gcc-4.9.2(含Glibc的GCC)			//该GCC也是一个本架构GCC，只是其库路径限定与$LFS中，与宿主脱离
cd $LFS/sources
tar -jxf gcc-4.9.2.tar.bz2
cd gcc-4.9.2
cat gcc/limitx.h gcc/glimits.h gcc/limity.h > \		//完全体GCC编译，上次是不完全编译
`dirname $($LFS_TGT-gcc -print-libgcc-file-name)`/include-fixed/limits.h
for file in \
$(find gcc/config -name linux64.h -o -name linux.h -o -name sysv4.h)
do
	cp -uv $file{,.orig}
	sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
 	   -e 's@/usr@/tools@g' $file.orig > $file
	echo '
	#undef STANDARD_STARTFILE_PREFIX_1
	#undef STANDARD_STARTFILE_PREFIX_2
	#define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
	#define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
	touch $file.orig
done
tar -xf ../mpfr-3.1.2.tar.xz
mv -v mpfr-3.1.2 mpfr
tar -xf ../gmp-6.0.0a.tar.xz
mv -v gmp-6.0.0 gmp
tar -xf ../mpc-1.0.2.tar.gz
mv -v mpc-1.0.2 mpc
mkdir -v ../gcc-build
cd ../gcc-build
CC=$LFS_TGT-gcc \				//编译器指定为交叉编译器，同无意义，理由见上
CXX=$LFS_TGT-g++ \
AR=$LFS_TGT-ar \
RANLIB=$LFS_TGT-ranlib \
../gcc-4.9.2/configure \
--prefix=/tools \				//配置同上GCC,但需要Glibc的特性允许
--with-local-prefix=/tools \
--with-native-system-header-dir=/tools/include \
--with-sysroot=$LFS		\		//在一处邮件列表中发现的选项，该选项可以严格指定lib路径
--enable-languages=c,c++ \
--disable-libstdcxx-pch \
--disable-multilib \				//x86_64不支持
--disable-bootstrap \				//阻止gcc进行自我复制校验
--disable-libgomp
make
make install
ln -sv gcc /tools/bin/cc			//创建CC的软链接
rm -rf $LFS/sources/gcc-build
rm -rf $LFS/sources/gcc-4.9.2

编译结果校验
echo 'int main(){}'>test.c
cc test.c
readelf -l a.out | grep ': /tools'
	结果应为：[Requesting program interpreter: /tools/lib64/ld-linux-x86-64.so.2]
	32位系统应为lib/ld-linux.so.2

#至此，交叉编译工具成功运作
