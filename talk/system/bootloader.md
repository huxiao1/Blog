现代PC引导操作系统分为多个阶段进行引导（以传统的BIOS+MBR方式为例）：  

1. BIOS-细线：CPU以实模式启动，执行复位向量(Reset vector)的指令，通常指向0xFFFF0（32位）或者0xFFFFFFF0（64位），这个地址是主板ROM上的BIOS/UEFI入口。加载BIOS后，执行一系列硬件的初始化、自检。

2. 第一阶段启动-细绳：BIOS从启动设备列表中按顺序查询，直到查询到一个可以用来启动操作系统的存储设备。以硬盘为例，第一个扇区（0扇区）以0x55AA结束的硬盘被认为是可以启动某个操作系统的（引导签名）。这个扇区就是MBR(Master Boot Record，主引导记录)。  
**传统硬盘的扇区大小是512字节，然后要留出4×16=64字节的主分区记录和2字节的引导签名，所以引导程序最多只有446字节。**  
不同的操作系统对MBR的定义会有一些细节上的差异，例如微软Win9x的DOS 7.1为了支持FAT32、LBA以及加入磁盘时间戳和签名，引导程序只剩下423字节。因此微软对DOS 7.1的引导做了很多限制，例如要求启动文件位于磁盘的特定位置，并且必须连续存放。即便这样，引导程序也只能加载前三个扇区并执行，由这三个扇区上的程序负责把这个文件剩下的数据从硬盘载入内存并执行。

3. 第二阶段启动-粗绳：某些操作系统或者启动技术允许启动多个不同的操作系统，MBR上的引导程序查询设置为启动分区的主分区记录，并加载该分区上的第一个扇区上的引导程序。这个扇区就是VBR(Volume Boot Record，卷引导记录)。某些操作系统则是把第二阶段的启动程序保存在特定的文件中，例如Windows NT的ntldr（5.x及之前版本）或bootmgr（6.x及之后版本）。某些启动技术则是支持把启动程序放在第一阶段或者第二阶段，例如安装Linux的时候经常会询问用户把GRUB安装在MBR还是/boot所在分区的第一扇区。一般来说第二阶段加载的启动程序可以比较大，足以完成加载完整的操作系统内核并执行了，甚至可以提供启动列表供用户选择以启动多个不同的操作系统。

4. 操作系统-铁索：加载并执行操作系统内核以及启动各种服务。例如Windows 9x及之前版本的http://win.com、Windows NT的ntoskrnl.exe（5.x及之前版本）或者winload.exe（6.x及之后版本）、Linux的vmlinuz。




