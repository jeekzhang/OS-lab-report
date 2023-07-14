<font face="宋体">

&nbsp;
**<font size=12><p align="center">操作系统实验报告</p></font>**
&nbsp;
<font size=6><p align="center">lab3 【Traps】 </p></font>
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




<center>
专&ensp;业：<u>计算机科学与技术</u>
</center>
&nbsp;

</font>

<font size=5>一、实验内容：</font>

#### Part A  RISC-V assembly

见问题回答



#### Part B  Backtrace

- Add the prototype for your `backtrace()` to `kernel/defs.h` so that you can invoke `backtrace` in `sys_sleep`.

  ![image-20221115141837075](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174601550-328029798.png)

  ![image-20221115141904029](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174601920-1804513272.png)




- Add the following function to `kernel/riscv.h`, and call this function in `backtrace` to read the current frame pointer. `r_fp()` uses [in-line assembly](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html) to read `s0`.

  ![image-20221115142233886](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174602712-883983309.png)

  在backtrace中获取s0的值，并用于读取第一个所需的返回地址：

  ![2022-11-14 01-18-03 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174603048-955236729.png)

  ![2022-11-14 01-18-19 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174603336-621089472.png)

- 循环，向上遍历帧指针

  **实现思路：**

  正如第一个地址的操作，每次都是打印fp-8所指向的内容。故可将打印写作循环里，并调整fp为上一个的帧指针，即fp-16所指向的内容。同时，通过 `PGROUNDDOWN(fp)`和 `PGROUNDUP(fp)`来作为循环终止的判断条件。

  **具体实现：**

  ![image-20221115143520282](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174603601-1323670298.png)



**测试：**

![2022-11-14 01-18-52 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174603881-1708849353.png)

![2022-11-14 01-23-56 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174604132-88024643.png)



#### Part C  Alarm

##### test0: invoke handler

- 准备工作

  Modify the Makefile to cause `alarmtest.c` to be compiled as an xv6 user program.

  ![image-20221115144600189](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174604445-103161767.png)

  The right declarations to put in `user/user.h` 

  ![image-20221115144757238](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174604718-1250755671.png)

  Update user/usys.pl (which generates user/usys.S), kernel/syscall.h, and kernel/syscall.c to allow `alarmtest` to invoke the sigalarm and sigreturn system calls.

  ![image-20221115145003360](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174604993-635365899.png)

  ![image-20221115145131302](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174605266-1299587399.png)

  ![image-20221115145153894](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174605552-1360271728.png)

  ![image-20221115145211517](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174605816-380756373.png)

- For now,  `sys_sigreturn` should just return zero.

  ![image-20221115145653447](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174606081-1480808717.png)

- `sys_sigalarm()` should store the alarm interval and the pointer to the handler function in new fields in the `proc` structure (in `kernel/proc.h`) . And keep track of how many ticks have passed since the last call (or are left until the next call) to a process's alarm handler; add a new field in `struct proc` for this too.

  ![image-20221115145825673](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174606345-970455126.png)

  ![image-20221115145731807](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174606627-191375164.png)

- Initialize `proc` fields in `allocproc()` in `proc.c`.

  ![image-20221115153140054](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174606869-841183008.png)

  

- Every tick, the hardware clock forces an interrupt, which is handled in `usertrap()` in `kernel/trap.c` in  `if(which_dev == 2) ...`. And modify `usertrap()` so that when a process's alarm interval expires, the user process executes the handler function.

  **实现思路：**

  首先，时间中断处要进行ticks的++，如何判断ticks是否和interval相同，若是则需执行handler函数。执行该函数的具体做法便是将handler的地址赋给程序计数器，即寄存器epc。要注意前面给的sigalarm(0,0)是停止周期调用，其实n==0的意义不大，ticks会恒大于等于0，故要判断interval是否为0，不是才要做上述操作。

  **具体实现：**

  ![image-20221115150713311](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174607139-175485046.png)

  


**测试：**

![image-20221115151051544](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174607416-523380367.png)

可见test0正常通过，但test1不行



##### test1/test2()/test3(): resume interrupted code

- Save and restore registers

  **实现思路：**

  类似于trapframe的构造，再构造一个save_trapframe用于保存之前的所有寄存器。像trapframe，故需要在`allocproc()/freeproc()`中进行alloc和free。当需要做程序跳转时，将当前的trapframe的内容放到save_trapframe中。

  **具体实现：**

  ![image-20221115153650820](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174607674-1264196483.png)

  ![image-20221115153705388](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174607968-1611457461.png)

  ![image-20221115153855607](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174610818-1566240168.png)

  

-  `sigreturn` should: 

  1. correctly return to the interrupted user code.
  2. prevent re-entrant calls to the handler----if a handler hasn't returned yet, the kernel shouldn't call it again.
  3. return a0.

  **实现思路：**

  借助于之前的save_trapframe，直接将其内容给trapframe就行了。因为epc也在其中，程序计数器会自动跟着做，同时a0也可以轻易的读到。至于保证handler执行时不被调用，只需要把置0这一步放到sigreturn中即可，因为执行的条件是要ticks==interval，只要不置0，handler就不会再被执行。

  ![image-20221115154033038](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174608523-149414547.png)

  

**测试：**

![image-20221115154724704](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174608789-614268869.png)



#### Make grade

添加answers-traps.txt和time.txt，运行make grade

![image-20221115155843423](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174609065-1180618732.png)



<font size=5>二、问题回答：</font>

（1） 函数的参数包含在哪些寄存器中？例如在main对printf的调用中，哪个寄存器保存13？

![image-20221115123957828](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174609352-1559209900.png)

从图中可以看出，函数的参数包含在a0~a7寄存器中。在main对printf的调用中，保存13的是a2寄存器。

```asm
  printf("%d %d\n", f(8)+1, 13);
  24:	4635                	li	a2,13
```



（2） Main的汇编代码中对函数f的调用在哪里？对g的调用在哪里？（Hint：编译器可能内联函数）

编译器进行了函数内联，直接将f(8)+1的值计算了出来，即12

```asm
  26:	45b1                	li	a1,12
```



（3） 函数printf位于哪个地址？

0x65a

```asm
  34:	62a080e7          	jalr	1578(ra) # 65a <printf>
```

![image-20221115130315510](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174609704-719528700.png)



（4） 在jalr到main中的printf之后，寄存器ra中存储的值是？

ra寄存器保存的是发生函数调用后应该返回的地址值，调用printf后应该返回执行的地址为0x38，故ra存储的即是0x38。



（5） 运行以下代码：

`unsigned int i = 0x00646c72;`
`printf("H%x Wo%s", 57616, &i);`

输出是什么？输出取决于 RISC-V是 little-endian的。如果 RISC-V 是 big-endian，怎样设置来产生相同的输出？是否需要更改i和57616为不同的值？

![image-20221115130503620](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174610028-271486488.png)

如果 RISC-V 是 big-endian：

57616不需要更改，因为它是作为一个整体的进制转换进行输出的

但i需要更改为726c6400



（6） 在下面的代码中，会打印出什么？（注意：答案不是特定值）为什么会发生这种情况？
`printf("x=%d y=%d", 3);`

![image-20221115130743717](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174610274-552420454.png)

先看正常的代码，`printf("x=%d y=%d",3,3);`，具体是将两个3分别放在a1和a2寄存器中，printf再去读寄存器里的值。但如果后面一个3缺省，但a2寄存器仍然可读取，所以y读的便是a2寄存器里的值，因为a2寄存器没有给定值，所以会打印一个不确定的值，当时a2寄存器是什么就打印什么。



<font size=5>三、问题解决：</font>

【问题1】在Backtrace实验中，最开始对于提示中“指向**返回地址加上8的地址”理解有误，导致输出的保存地址不正确

	【解决】画了一张简图（图太丑，就不放了）来理解，指针与指向具体内容的关系。文字说明就是，s0放的是fp的地址，该地址-8即是返回地址（打印内容）的地址，-16就是上一个fp的地址。

【问题2】完成实验进行make grade后，发现usertests失败，显示丢失了一些free page，应该是有些东西在结束后未被free掉。

![2022-11-14 22-02-40 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174610532-176857661.png)

	【解决】经过漫长的排查，发现是save_trapframe与trapframe之间的拷贝造成的。最开始，是直接将两个进行复制来拷贝。后来才想起来这两个是指针，只拷贝指针的话，确实指针之前指向的内容没被释放掉。

![image-20221115153855607](https://img2022.cnblogs.com/blog/3018649/202211/3018649-20221116174610818-1566240168.png)

此处改之前是`p->save_trapframe = p->trapframe`



<font size=5>四、实验感想：</font>

1. 这个实验因为第一周事情比较多，导致拖到第二周已经忘了之前是咋理解的。alarm实验不亏是hard，虽然做完了回想一下也没有那么多复杂的代码，但写的时候就有种拆东墙补西墙的感觉。先写test0的时候很顺手，而且看着后面的提示也没有很多。结果后面的测试大概就是test1能过了test3不能过，test3能过了test1又出问题，最后以为成功了，usertests出问题。
2. 这次实验的提示相比于之前的保姆式提示，略过了一些具体实现的介绍，比如添加一个新的.c文件到makefile里以及后面的一系列操作，还比如要保存和恢复寄存器并没有告诉你使用什么样的数据结构，这正是实验逐步深入的体现。
3. 通过具体的函数跳转并返回这一机制的实验，我捡回了很多关于计算机组成的知识，也对这些知识有了更深刻的理解，比如程序计数器的执行原理、寄存器的使用等等。



<font size=5>五、实验参考及git地址：</font>

1. [Lab: Traps (mit.edu)](https://pdos.csail.mit.edu/6.S081/2022/labs/traps.html)
2. [C/C++语言中 指针复制与指针赋值](https://blog.csdn.net/m0_37251750/article/details/98171337)
3. [jeekzhang/MIT6.S081](https://gitee.com/zhang-gike/MIT6.S081/tree/traps/)

</font>