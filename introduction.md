#Introduction
建议观看文档的诸位使用虚拟机进行。在原虚拟机上再添加一块虚拟磁盘，对虚拟磁盘进行操作即可。

LFS（Linux from scratch）
  可以理解为DIY搭建linux,需要一个已有linux系统的机器进行对目标系统的编译。其搭建过程实际上是依赖于宿主机对目标机的硬盘进行直接的挂载读写。
	优点：可自由定制所需系统大小，理解linux表层运作机制。
	缺点：
该过程可分为大体4个部分：
	1.宿主机系统的环境检查与配置
	2.交叉编译工具链的制作
	3.目标系统kernel的编译
	4.目标系统基础应用的编译