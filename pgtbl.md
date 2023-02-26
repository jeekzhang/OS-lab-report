<font face="宋体">

&nbsp;
**<font size=12><p align="center">操作系统实验报告</p></font>**
&nbsp;
<font size=6><p align="center">lab3 【Page tables】 </p></font>
&nbsp;&nbsp;

<div align=center><img width = '200' height ='200' src ="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.huawenku.cn%2Fuploads%2F150601%2Fxiaohui.jpg&refer=http%3A%2F%2Fwww.huawenku.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1617160664&t=761914188dd5b3da1e91e3a495453919"/></div>

&nbsp;&nbsp;&nbsp;&nbsp;
<font size=5>



&nbsp;

<center>
学生姓名：<u>jeekzhang</u>
</center>




&nbsp;

<center>
学&ensp;号：<u>20307130xxx</u>
</center>



&nbsp;

<center>
专&ensp;业：<u>计算机科学与技术</u>
</center>



</font>



<font size=5>一、实验内容：</font>

#### Part A  Speed up system calls

- Perform the mapping in `proc_pagetable()` in `kernel/proc.c`

  **实现思路：**

  先查看 `proc_pagetable()` 的内部实现，发现其主要用到了`mappages()`这一函数，分别map了trampoline page和trapframe page。我们需要做的便是map一个usyscall page，与以上两项对比，需要改变的便是参数va, pa, perm。va即已在`memlayout.h`中定义的虚拟地址USYSCALL，perm即权限位，要<u>用户可访问+读</u>，便置为`PTE_R | PTE_U`。现在只剩pa了，即要映射到的物理地址，其中要存储了当前进程的usyscall，和trapframe相仿，故我们要在proc中放置该地址。最后再同上一项的操作一样，如果该项map失败，uvmunmap前面的page。

  **具体实现：**

  proc中添加usyscall*

  ![image-20221029154451232](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101312286-855048090.png)

  map usyscall page

  ![image-20221029155721182](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101312714-82037015.png)



- Allocate and initialize the page in `allocproc()`

  Allocate：模仿trapframe 

  ![image-20221029160041379](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101313152-906313682.png)page即可：

  

  Initialize：让usyscall中的pid等于当前进程的pid
  
  ![image-20221029160159531](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101313481-91433738.png)



- Free the page in `freeproc()`

  同样模仿即可

  ![image-20221029160434995](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101313781-409464850.png)

  比较坑的是，在freepagetable时还要记得把map的page都uvmunmap了，所以要添加USYSCALL项

  ![image-20221029160656570](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101320082-53268938.png)



**测试：**

![image-20221029161056845](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101314439-1863600586.png)



#### Part B  Print a page table

- Define vmprint() and insert `if(p->pid==1) vmprint(p->pagetable)` in exec.c just before the `return argc`

  Insert

  ![image-20221029161459459](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101314752-1565876347.png)

  Define

  第一行的打印较为简单，先打印"page table"，再用%p打印pagetable即可

  ![image-20221029161625408](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101315011-558424981.png)

  打印结果如图：

  ![image-20221029162145671](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101315336-279075671.png)

-  Print lines for each PTE

  **实现思路：**

  即要遍历pagetable的每一项PTE并打印其本身和对应的物理地址，需要注意的是xv6的页表类似一个三级树，父PTE对应的物理地址又是子pagetable的地址，故要用递归的方法以打印子pagetable。注意要置一个count来区分递归的深度，递归到2即截止，同时count也能作为打印..的标准。

  三级树示意图：

  ![22834193-d2d202f6da9bd8d7](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101315678-1063251973.png)

  查看freewalk的遍历方式：

  ![image-20221029162452472](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101315958-750947339.png)

  递归主函数，参数还包含了count，count<2则正常递归

  ![image-20221029163622932](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101316278-898456481.png)

  打点函数，根据不同的count打印不同的点

  ![image-20221029163738979](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101316562-1232486653.png)

  主函数：

  ![image-20221029163842325](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101316876-1008458611.png)



**测试：**

![image-20221101084657962](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101317160-803191270.png)



#### Part C  Detect which pages have been accessed

- Implementing `sys_pgaccess()` in `kernel/sysproc.c`

  先在 `pgaccess` 查看所需要读取的参数

  ![image-20221101080252406](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101317483-252318003.png)

  由于base和mask是地址类型，故用 `argaddr()` 读取，len是int类型，用 `argint()`读取。在定义时，为方便copycout()的使用，我们定义addr为mask地址，mask为一个需要置位的无符号数（即传递的值）。

  ![image-20221101080518622](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101317740-1264315080.png)

- Check arguments are right

  检查page数（len）是否合规：

  在用mask来标注那一页是否被访问时，我们用mask的那一位置1来说明，故page的最大页数取决于mask的位数，因为mask是无符号64位，故最大len为64，大于该值返回-1表示错误。

  ![image-20221101081402862](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101318067-1107724610.png)

  ![image-20221101081501542](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101318389-1761027118.png)

  检查地址是否越界：

  只需要检查最大页的地址是否大于最大地址即可（见上图）

- Do the scan and copy

  只需要查找每一页对应的PTE项，看看其PTE_A是否为1，是则把mask对应位置1，然后再把PTE的A位置0（clear操作）

  至于查找PTE项，调用walk()即可

  PTE_A需要在声明PTE_U等的位置声明一下，查表（见上图Figure3.2）可知是第6位，1L左移6位即可

  clear操作要把PTE的A位置0，相当于要与上第6位为0、其余位为1的无符号数，该数将PTE_A取反可得到

  最后把操作完的mask用copyout传到对应位置

  ![image-20221101082445181](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101318689-1401474861.png)

  ![image-20221101081943907](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101319013-2022108078.png)



**测试：**

![image-20221101084425837](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101319309-240511002.png)



<font size=5>二、问题回答：</font>

（1）在Part A 加速系统调用部分，除了getpid()系统调用函数，你还能想到哪些系统调用函数可以如此加速？

- 根据加速的原理，还可以加速的系统调用函数应该是要返回内核的某个参数或其地址，不需要再调用内核其他函数。对比user.h中的函数，发现有这样一个函数——uptime()，它返回的是发生中断的次数，返回值是trap.c中的ticks。



（2）虚拟内存有什么用处？

- 内存管理：为每个进程提供了较为一致的地址空间，可简化内存管理。由于页表的存在，操作系统进行内存分配时，不需要分配连续的物理页面，这简化了内存分配。
- 内存保护：每个进程有独立的内存空间，这保护了每个进程的地址空间不被其他进程破坏。同时，在PTE项中加入权限位，也保证了进程不去访问不该访问的page。

- 提高性能：在空间上，页表尤其是多级页表的存在使得虚拟内存和物理内存间的映射变得灵活，可减少碎片的产生，使物理内存能够被充分利用起来；在时间上，如Part A所做的操作，可加速一些只访问内核参数的函数。

  

（3）为什么现代操作系统采用多级页表？

- 在绝大多数情况下节省页表内存空间。这说起来有点违反常识，因为实际上对于一次va到pa的映射，只用一级页表，相当于只有一层索引信息，而多级页表则意味着多层索引信息。之所以说是多级页表可以节省页表内存空间，并不是以上面的说法来看待的。以上的说法之所以成立，是我们认为进程覆盖了整个虚拟地址空间，一级页表的页表项都被用到了。但在大多数情况下，进程远远用不到整个虚拟地址空间，某个一级页表项也不需要用到，在多级页表机制中其对应的所有二级页表也就不需要创建出来了，节省页表内存空间便是从此而来。打个比方，比如按一级页表机制有1024页表项，二级页表按32*32项算，如果只用到了一半，第二级页表则有一半先不需要创建出来。还有一个问题，就是为什么第一级页表不能也只创建一半。这是由页表的性质决定的，第一级页表得是连续的、固定的一段空间，所以不管页表项创建了多少个，这一段空间都得在那，不能做其他用途。

- 可以离散地存储页表，以此可缓解操作系统的碎片问题。正如第一个优点最后提及的那样，多级页表第一级后面的页表就不用是连续和固定的地址空间了。比如xv6里的机制是把父页表项映射出来的物理地址作为子页表的地址。

  

（4）简述Part C的detect流程。

- 首先，如果该page被access过，则PTE项的第六位即PTE_A会自动置1。然后，在detect之前，会检查参数是否正确，页面是否不超过最大值，最后页的地址是否越界。之后则会正式进行每一页的detect，先通过walk()找到正确的PTE项，未找到则返回-1，找到就将mask对应页号的位置1，再通过与上~PTE_A将PTE项的第六位置回0。至此，便完成了系统调用的access。



<font size=5>三、问题解决：</font>

【问题】在Part A完成后，make qemu后显示异常kerneltrap

![image-20221101085916657](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101319707-299273323.png)

发现在freeproc函数中调用了procfreepagetable函数，来free p的pagetable，很明显，我们map了一个usyscall page，需要在其中也free掉。

![image-20221029160656570](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221101101320082-53268938.png)



<font size=5>四、实验感想：</font>

1. 这次实验主要体现了两个字——“模仿”，无论是Part A的map模仿trapframe，还是Part B的遍历和递归模仿freewalk。在写一些大型的code时，我们很难从整体去厘清某个部分函数该如何实现，该调用哪些函数来实现，但可以从相仿的函数入手，看看这些函数是如何实现的，如何获取参数和调用其他函数。
2. 通过这次实验，我深刻地理解了页表的实现机制。相比于课上讲的映射方式，code实现起来就没那么简单了，要有创建、释放、遍历、转换等函数，还要有和proc其他结构配合使用的函数。此外，在实验中的遍历和递归也让我明白了多级页表的机制。



<font size=5>五、实验参考及git地址：</font>

1. [Lab: page tables (mit.edu)](https://pdos.csail.mit.edu/6.S081/2022/labs/pgtbl.html)
2. [第三章 页表 ](https://www.jianshu.com/p/56b905b8e0f4)
3. [MIT6.S081: OS lab](https://gitee.com/zhang-gike/MIT6.S081/tree/pgtbl/)

</font>