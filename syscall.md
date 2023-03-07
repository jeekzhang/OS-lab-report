<font face="宋体">

&nbsp;
**<font size=12><p align="center">操作系统实验报告</p></font>**
&nbsp;
<font size=6><p align="center">lab2 【System calls】 </p></font>
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

#### Part A  System call trace

**编译准备：**

- Add `$U/_trace` to UPROGS in Makefile

  ​	![2022-10-11 15-05-10 的屏幕截图-1665967904969](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211124260-1211016667.png)

- Add a prototype for the system call to `user/user.h`

  ![2022-10-11 15-06-01 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211124648-562484890.png)

-  Add a stub to `user/usys.pl`

  ![2022-10-11 15-06-07 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211124961-1547363278.png)

- Add a syscall number to `kernel/syscall.h`

  ![2022-10-11 15-06-22 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211125266-2135944251.png)

- 编译测试：`make qemu`,成功运行

  ![2022-10-11 15-06-37 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211125570-1634131622.png)



**实现trace：**

- 实现思路：

  根据提示，是要在`kernel/sysproc.c`中添加一个函数 `sys_trace()`，该函数要记住传进来的参数，并在struct proc中新加一个变量用于储存这一参数。结合前面的功能描述，这个参数就是`mask`，也即`trace 32 grep hello README`中的`32`。

  查看`kernel/sysproc.c`中其他函数（如`sys_exit()`），发现其没有参数，但使用了`argint(0,&n)`来获取所需的n值，其用于获取第0个寄存器的信息并用n来储存。故`mask`也应该用同样的方法来读取。

  ![2022-10-17 09-20-01 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211125849-507583172.png)

  同时要在`struct proc`中添加`mask`变量

  ![2022-10-17 09-29-48 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211126082-2138269849.png)

  再把`mask`赋值给当前`proc`的`mask`，根据`sys_getpid()`，可用`myproc()->mask`来进行赋值。

- 编写函数：先读取0号寄存器的内容到`mask`中，再把`mask`赋给当前`proc`的`mask`

  ![2022-10-17 09-34-44 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211126393-1239611864.png)

**子项复制：**

- Modify `fork()` (see `kernel/proc.c`) to copy the trace mask from the parent to the child process.

  即把父进程的`mask`赋给子进程：

  ![2022-10-17 09-35-50 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211126696-1957029207.png)

**打印输出：**

-  在`syscall()`中添加相应的trace项：

  ![2022-10-17 09-36-18 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211126983-2127034321.png)

  ![2022-10-17 09-36-25 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211127259-1092877010.png)

- Add an array of syscall names to index into.

  用于打印输出

  ![2022-10-17 09-36-51 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211127553-142082053.png)

- 输出实现：

  首先，打印内容的结构是：`3: syscall read -> 1023`（3为pid，syscall为统一输出，read为系统调用的名字，1023为其返回值。

  pid直接用`myproc()->pid`即可获取

  系统调用的名字则通过上述的表中根据编号(`num`)获取，编号要与`mask`进行匹配，匹配成功才要进行输出，`mask`的第`num`位为1则需打印输出。匹配过程考虑位运算，将1左移`num`位，再`&`上`mask`，若当`mask`的第`num`位不为1时，结果为0。

  返回值调用`syscalls[num]()`后即得到，因p->trapframe->a0存储了该值，输出它即可。

  ![2022-10-17 09-37-08 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211127882-1709531831.png)



**测试：**

![2022-10-17 11-39-04 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211128167-154250474.png)

![2022-10-17 11-39-47 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211128450-1904439140.png)



#### Part B  Sysinfo

**编译准备：**

- 重复Part A的操作：

  ![2022-10-15 10-40-27 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211128720-1377883360.png)

  ![2022-10-15 10-59-08 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211128982-291730399.png)

  ![2022-10-15 11-00-07 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211129259-478177484.png)

  ![2022-10-17 10-20-13 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211129488-134308873.png)

- 其他准备：

  Predeclare the existence of `struct sysinfo`

  ![2022-10-17 10-53-30 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211129775-572152157.png)

  Add `Student ID` for print

  ![2022-10-17 11-00-51 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211130035-715981815.png)

**辅助函数：**

To collect the amount of free memory, add a function to `kernel/kalloc.c`

阅读 `kernel/kalloc.c`中的其他函数发现，有freelist可以使用，用一个while循环遍历该freelist，并让计数器count++，即可获取free的page数目，count*PGSIZE即为free memory。同时，涉及到临界区变量，该过程要加锁。

![2022-10-17 10-58-37 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211130319-1715634930.png)

To collect the number of processes, add a function to `kernel/proc.c`

copy `kernel/kalloc.c`中的其他函数的遍历方法，并置计数器count，当状态不为UNUSED即count++，该过程也要加锁。

![2022-10-17 10-59-20 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211130621-703214871.png)



 **实现sysinfo：**

- 实现思路：

  先创建一个`struct sysinfo`

  调用之前两个辅助函数获取`sysinfo`内的`freemem`和`nproc`值，

  再把学号赋值给`sid`并打印

  接下来进行`copyout`操作，将对应位置改为`info`即可，注意前面参数的获取也要有，`p`通过`myproc()`获取，`addr`从0号寄存器读入。

- 编写函数：注意要在`"defs.h"`添加辅助函数的定义

  ![2022-10-17 11-00-34 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211130939-329862905.png)

**测试：**

![2022-10-17 11-41-22 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211131215-1999790400.png)



<font size=5>二、问题回答：</font>

（1）System calls Part A部分，简述一下trace全流程.

`trace`先调用`user/trace.c`进行参数读入和预处理，再通过`entry(trace)`从用户态转到内核态，执行后续操作。在内核态中具体调用`sys_trace()`函数，该函数从0号寄存器中读入所谓的`mask`，并将`mask`放入`mypro()`的`mask`中。当进行`syscall()`时，就会根据`myproc()`的`mask`是否与编号`num`匹配来打印相应内容，完成`trace`。

（2）kernel/syscall.h是干什么的，如何起作用的？

通过宏定义，将系统调用函数的名字作为标识符替换为数字编号。通过编号转化后，就可以将系统调用名字的数据转为数据放入寄存器（a7）中，当syscall需要时便从寄存器中读取编号，从而执行对应函数。

（3）命令 “trace 32 grep hello README”中的trace字段是用户态下的还是实现的系统调用函数trace？

trace字段应是在用户态下的函数`user/trace.c`，通过内核调用`sys_trace()`来实现具体操作。



<font size=5>三、问题解决：</font>

【问题】在Part B的调用两个辅助函数获取`sysinfo`内的`freemem`和`nproc`值时，出现错误，显示函数没有定义。

![2022-10-15 14-39-38 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211131517-452992250.png)

如上图的getfreemem()函数，其在`kernel/kalloc.c`中实现。其定义按道理应添加在`kernel/kalloc.h`中，然后include该.h文件即可，但并没有该.h文件。经代码搜索后发现`kernel/kalloc.c`中函数的定义是在`defs.h`中声明的，类似于`user/user.h`，故应在 `"defs.h"`添加辅助函数的定义。![2022-10-18 08-51-34 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211131788-1621308451.png)

![2022-10-18 08-51-30 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026211132087-502196644.png)



<font size=5>四、实验感想：</font>

1. 通过这次实验具体函数的实现，理解了用户态和内核态的区别与关系，以及用户态是如何转到内核态的，这是在代码层面一直困扰我的问题。

2. 这次实验还有一个地方有点颠覆我的认知，就是内核函数获取参数不是像之前写的代码那样靠传参，而是直接从寄存器中读。

   

<font size=5>五、实验参考：</font>

1. [Lab: System calls (mit.edu)](https://pdos.csail.mit.edu/6.S081/2022/labs/syscall.html)
2. [kernel/syscall.c 的几个小函数简介](https://blog.csdn.net/NP_hard/article/details/121325896)
