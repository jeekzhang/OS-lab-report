<font face="宋体">

&nbsp;
**<font size=12><p align="center">操作系统实验报告</p></font>**
&nbsp;
<font size=6><p align="center">lab6 【File system】 </p></font>
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


&nbsp;



</font>

</font>

<font size=5>一、实验内容：</font>

#### Part A  Large files

1. Change the definition of `NDIRECT` and add the definition of `NNINDIRECT`. Change the declaration of `addrs[]` in `struct inode` in `file.h`. Make sure that `struct inode` and `struct dinode` have the same number of elements in their `addrs[]` arrays.

   除此之外，记得在计算MAXFILE时把NNINDIRECT加上。

   ![image-20221211132154787](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154944939-47187317.png)

   ![image-20221211132334882](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154945438-2045385795.png)

2. Run `make clean` which forces make to rebuild fs.img

   ![2022-12-09 18-06-59 的屏幕截图](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154945703-466493029.png)

3. Modify `bmap()`

- 实现思路：根据实验要求，我们要构造的就是一个二级的映射，因为一级的映射已经有了（NINDIRECT），所以可以以一级为基础再套一级即可。同样，先将bn减去NINDIRECT，判断是否在NNINDIRECT的范围内。把地址取到NNINDIRECT对应的那块，没有（为0）则进行分配。因为是两级，所以NINDIRECT中的bn相当于有两个，第一个（cn）可由bn除得到，第二个（dn）可由bn除余得到。然后通过以上的cn和dn迭代获取addr的步骤得到最后的addr。

- 具体实现

  ![image-20221211132755504](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154945975-279295573.png)

**测试：**

![image-20221211134519704](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154946287-1181872735.png)

![image-20221211135058861](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154946557-269873636.png)



#### Part B Symbolic links

1. 编译准备

-  Create a new system call number for symlink, add an entry to user/usys.pl, user/user.h, and implement an empty sys_symlink in kernel/sysfile.c.

  ![image-20221211135451656](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154946883-636911691.png)

  ![image-20221211135513899](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154947181-648104964.png)

  ![image-20221211135527263](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154947488-742459270.png)

  ![2022-12-09 22-42-49 的屏幕截图](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154947779-2093026928.png)

  ![image-20221211135644530](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154948073-2095922114.png)

  ![image-20221211135706984](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154948360-1926995092.png)

- Add a new file type (`T_SYMLINK`) to kernel/stat.h to represent a symbolic link.

  ![image-20221211140207826](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154948667-409887109.png)

- Add a new flag to kernel/fcntl.h, (`O_NOFOLLOW`), that can be used with the `open` system call. 

  为保证不发生位冲突，将`O_NOFOLLOW`设为0x010

  ![image-20221211140326382](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154948953-1794460842.png)

- Add symlinktest to the Makefile and run it

  ![2022-12-09 22-34-25 的屏幕截图](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154949266-599907649.png)

  ![2022-12-09 22-48-51 的屏幕截图](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154949547-478172315.png)

  成功编译

2. Implement the `symlink(target, path)` system call to create a new symbolic link at path that refers to target. Note that target does not need to exist for the system call to succeed. You will need to choose somewhere to store the target path of a symbolic link

- 实现思路：简而言之，就是要在path去存一下target。一个简单的想法就是path下创建一个T_SYMLINK的inode，在inode里面加一个char[]的内容用来放target。这里面有诸多细节需要注意：首先放target到inode中，因为是char[]的复制，不能直接用“=”，strcpy也无法使用，所以回归最基本的办法，将一个一个char进行拷贝，以'\0'作为结尾判断，但要注意inode中sympath拿出来后要进行初始化，因为无法知道原来里面放的是啥。然后是creat有lock操作，结束了要unlock。

- 具体实现：

  ![image-20221211140935869](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154949853-100438551.png)

  ![image-20221211140742102](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154950135-166149181.png)

3. Modify the `open` system call to handle the case where the path refers to a symbolic link. 

- 实现思路：open操作的实现已经有了，我们要做的就是找到正确的path，确定了这一点后，我们便把代码写在open功能的前面。先做一个位判断和格式判断，设一个count后便进入循环，循环终止条件有：读到不是T_SYNLINK、count大于10。把inode里的sympath读出来到path中，以此完成一个循环。还有一些lock的细节，因为inode会因为path的不同而变化，故与inode有关的锁也会变，所以在inode变化后要把旧锁unlock，lock新锁。

- 具体实现：

  ![image-20221211143407534](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154950456-239945326.png)



**测试：**

![image-20221211143903737](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154950735-1581187075.png)



#### make grade

![image-20221211144809593](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154951118-2095528019.png)



<font size=5>二、问题解决：</font>

【问题】在Part A完成了Hints后，测试bigfile后显示wrote 267 blocks。

【分析】相比于最开始的版本少了1个block，再对比正确的65803 blocks可知应该是最后那个二级映射的block没有计算进去。

【解决】详细检查代码后发现，计算MAXFILE时没有把NNINDIRECT加上，这点确实在Hints里面没有提，最开始写的时候也没有注意到。加上后，再跑就能正常运行了。



【问题】在Part B进行测试时，显示FAILURE: Failed to open 1。

![image-20221211151259906](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154951410-133222501.png)

【分析】先尝试去看看path对不对，在open里添加一行printf代码。

![image-20221211151546381](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154951679-2035912561.png)

结果发现path就不对了，后面多了一些奇怪的东西，再看看在inode的sympath里面有没有。

![image-20221211151711178](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154951939-323978909.png)

![image-20221211151736660](https://img2023.cnblogs.com/blog/3018649/202212/3018649-20221211154952329-1177155884.png)

结果还是有这些东西，所以是在sys_symlink传值的时候出现的问题，传进来的target固然没有问题，只有可能就是sympath没有初始化。

【解决】在sys_symlink传值之前对sympath进行初始化，都初始化为'\0'。之后再进行测试，便能正常运行了。



<font size=5>三、实验总结：</font>

1. 这个实验除了在做文件系统，还用到了之前的很多内容，对知识进行了综合的考察。比如涉及到的锁的操作，并且还是以inode为参数的hash锁，这让我在PatrB的open函数中花了很长时间才反应过来。

2. 感觉自己还是过于依赖Hints了，一到Hints没有的内容，就不知道怎么做了。只盯着Hints有的内容，明明MAXFILE的define就在NDIRECT后面，却没有发现并做相应的修改。

3. 这就是本学期最后一次实验了，通过这些实验，我对操作系统的一些理论的内容通过实践有了更深的理解，对一个操作系统的搭建也有了较为完整的认识。除此之外，我还学会了linux下许多基本的指令以及git自己的项目上传branch的操作。

   

<font size=5>五、实验参考及git地址：</font>

1. [Lab: file system (mit.edu)](https://pdos.csail.mit.edu/6.S081/2022/labs/fs.html)
2. [MIT6.S081: OS lab of jeekzhang](https://gitee.com/zhang-gike/MIT6.S081/tree/fs/)