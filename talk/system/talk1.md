- [书上说代码地址总是从0x400000开始，但是查看编译好的elf头起始地址是从0开始的，这是为什么？](#书上说代码地址总是从0x400000开始但是查看编译好的elf头起始地址是从0开始的这是为什么)

# 书上说代码地址总是从0x400000开始，但是查看编译好的elf头起始地址是从0开始的，这是为什么？
8位和16位CPU时代，计算机系统是实地址模式。程序直接使用物理地址，操作系统和应用程序运行在相同的地址空间。先加载操作系统再在更高的某一个偏移地址上加载应用程序，因此所有应用程序都会被加载到同样的地址。从而任何时候只能跑一个应用程序，否则它们之间会相互覆盖。随着要处理的数据越来越大，单任务系统也出现了问题。因为8位的CPU只有256字节、16位的CPU只有64K的寻址范围，在这个范围当中要放下程序和要处理的数据，以及运行时需要的堆栈等等，这个限制很大。    
**如果每次都将程序加载到内存当中不同的位置，那么不是就可以同时执行多个任务了吗？**  
然而，简单的这样做，可执行文件当中的地址就会发生错误。因为这些地址是在程序编译链接的时候就已经决定好的。  
## 8086
为了能够使用更多的内存，CPU增加了外部地址总线的宽度。最为著名的就是8086，[它内部是16位的，但是外部有20条地址总线](../week3/x86_mode/README.md)，所以能够支持1MB的寻址范围。但是因为寄存器都是16位的，没有办法直接存储20位的地址。所以8086采用了两个寄存器一组的方式来存储它。将一个寄存器（称为段寄存器）的值左移4位然后加上第二个寄存器（称为偏移寄存器或者指针寄存器）的值，得到一个20位的地址。  

这种做法将整个内存空间分成了很多部分。从0地址开始，每16位的地址空间（也就是64KB）为一页，而高4位（从第17位到第20位）则代表页码。那么，我们的应用程序如果也按照这个大小来组织，如果每段程序内所有的跳转都不超出64KB的范围，则这段程序就可以毫无修改地被放到内存上任何一页上面。无非在执行的时候，将程序当中的地址（偏移量）加上段寄存器当中的值（需左移4位），就可以得到最终的内存地址。  

因此，这个时期的应用程序也对应地被分成很多个段，如text段，data段，bss段。然后每个段被分别加载到内存上的某个分页内，将各个分页的起始地址的高16位根据其中的程序段的用途被分别设置到诸如CS（代码段）、DS（数据段）、BS（堆栈段）等段寄存器当中，在需要用到地址的时候偏移量加上这些段寄存器里面的值，就可以保证程序的正常运行。  

此时程序不再需要放在连续且固定的内存位置了。而是变得可被重定位（relocatable）。并且整个程序可以使用更大的内存空间了。这也说明，至少早在8086的时代，可执行文件当中的偏移地址，就已经和最终内存上的地址不是一个东西了，因为它并没有记录某个段具体在第几页，第几页是在被加载到内存时动态决定的。

**到这里其实问题已经得到了回答。那就是代码在可执行文件当中的地址，原本就不一定等于实际内存当中的地址。**  

## 为什么代码地址总是从0x400000开始
对于一个系统来说，要让屏幕上每个点都可被编程，1MB的地址空间就相当不够看了（20位的地址总线，所能提供的寻址空间，是1MB）。

所以后来32位CPU就出来了。但是在最先出来的时候，各方面软硬件还跟不上，内存也贵，大部分程序也都是16位的。所以本着能省就省的精神（毕竟牙膏厂），早期的386SX只安排了24条地址总线。（24条地址总线的寻址范围是16MB）很多早期的主板以及BIOS也不支持16MB，毕竟那个时候主流内存条都是1MB，你插个4条也就是4MB。win95基本上也就是按照这个规格设计的。因此这个4MB也就自然而然成为了一个新旧时代的分水岭：win95以前的旧版程序基本上只能运行在这个4MB范围内，而如果一个程序能够从4MB这个位置开始执行，那么它基本上是一个新时代程序。

另外一方面，为了更好支持多任务，增加进程之间的隔离以及保护操作系统本身，32位CPU新增了保护模式。保护模式下，所有的进程都有自己的“虚拟”地址空间，从0开始的。因为32位提供4GB的寻址范围，这在当时被认为是非常大的一个范围，所以就没有必要再将程序当中的地址拆成两个部分了。这种模式被称为[flat模式](../week3/address_transfer/README.md)。在flat模式下，程序又变得可以使用绝对地址（只不过是虚拟地址）了。也就是编译的时候可以将地址全部固定下来，并不需要在加载的时候进行重定位（relocate）。

**为了将这种flat模式的程序和之前的程序进行区别，当时的程序员选择了0x400000（4MB）这个具有纪念意义的位置。但是这其实只是一种选择，并非必须。** 程序的入口在地址0x400000的附近，并不是一定要这样。在flat模式下，只要放得下，它可以在任何地方。

## elf入口并不是0x400000的原因

如果每次程序都加载到同样的位置，会使得程序的破解，或者修改，变得非常的容易。同时，随着程序的不断变大，动态库技术开始发展。因为动态库是动态被加载进内存的（加载顺序不一定），所以无法保证每次都能加载到固定位置。  

所以，之前在16位模式使用的可重定位技术就在一定程度上又回来了。程序当中的地址又变成了相对地址，需要累加某个寄存器里的值之后，才能得到最终（逻辑）地址。最开始这项技术只是用于动态库。但是很快就有人发现，如果将其用在主程序，那么就会使得很多内存修改器，就是类似什么锁血补丁啊，什么激活码生成器啊变得难以生效。所以，在这个时期之后的可执行文件当中的地址，又变回了相对地址的形式，而不是最终在内存当中的绝对（逻辑）地址。

**那么既然调入内存要重新计算地址，在可执行文件当中程序从0开始显然计算最为简单方便。**



















