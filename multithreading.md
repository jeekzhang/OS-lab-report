<font face="宋体">

&nbsp;
**<font size=12><p align="center">操作系统实验报告</p></font>**
&nbsp;
<font size=6><p align="center">lab5 【Multithreading】 </p></font>
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

#### Part A  Uthread: switching between threads

1. 实现思路：

   根据实验要求，我们可以知道应该是要在三处进行代码的添加，分别是 `user/uthread.c`的 `thread_create()` 和 `thread_schedule()` 以及 `user/uthread_switch.S`中的 `thread_switch` 。在阅读了xv6 book后，得到了这样一个提示，就是内核的 `swtch` 做了上下文切换的工作，这与我们需要实现的线程切换十分相似。上下文通过结构体 `struct context`的形式来保存寄存器的内容。对于线程的切换，我们同样需要寄存器的保存与恢复，故我们应先声明一个 `struct context`，并在`struct thread`中添加一个`context`项。

   对于 `thread_schedule()`的添加内容，就是要调用`thread_switch`，但参数需要思考一下填什么，参考内核的`sched`函数，所以应该分别填入t和next_thread的context。填入时，注意指针的使用，同时函数参数的类型要保持一致。

   对于 `thread_switch`的添加内容，模仿`swtch.S`进行寄存器的保存和读取即可。

   对于 `thread_create()`的添加内容，参考`allocproc`，它对context的ra和rp进行了赋值。我们知道，ra是存的是PC的值，sp存的是栈顶的地址。具体到 `thread_create()`，就是要把func函数指针（地址）赋给ra；由于栈是由高到低增长，故栈顶是stack数组最后一位的地址。

   

2. 具体实现

   `user.h`中声明`context`

   ![image-20221129142747008](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105739037-894646513.png)

   `thread`中添加`context`项，并将`thread_switch`的参数类型改为`struct context *`

   ![image-20221129145306328](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105739468-1668477159.png)

   在`user/uthread_switch.S`中的 `thread_switch`进行寄存器的保存与恢复

   ![image-20221129143006753](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105739758-1477536251.png)

   `thread_create()` 和 `thread_schedule()` 的代码添加

   ![image-20221129143142407](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105740190-1251937779.png)

**测试：**

![image-20221129152101284](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105740595-367775124.png)

![image-20221129152115647](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105740874-605712903.png)



#### Part B Using threads

1. 测试单线程

   ```bash
   $ make ph
   $ ./ph 1
   ```

   输出结果：

   ![2022-10-11 18-45-21 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213210722-290655618.png)

   

2. 测试双线程

   ```bash
   $ make ph
   $ ./ph 2
   ```

   输出结果：

   ![2022-10-11 18-45-42 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213211055-772765807.png)

   put()添加key到哈希表中，get()从哈希表中获取key，结果说明有大量的key本应在put()时被放到哈希表中却未能正确放入。

   

3. 解决missing问题

   既然是多线程导致的put()问题，应该是在线程切换中导致某些key未能正确添加，故可以在put时加入lock的机制来防止这一情况的发生。

   ```c++
   pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;//初始化锁
   ...
   static
   void put(int key, int value)
   {
     pthread_mutex_lock(&lock);//上锁
     int i = key % NBUCKET;
     // is the key already present?
     struct entry *e = 0;
     for (e = table[i]; e != 0; e = e->next) {
       if (e->key == key)
         break;
     }
     if(e){
       // update the existing key.
       e->value = value;
     } else {
       // the new is new.
       insert(key, value, &table[i], table[i]);
       
     }
     pthread_mutex_unlock(&lock);//开锁
   }
   ...
   ```

   再进行测试`./ph 2`：

   ![2022-10-11 19-44-39 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213211316-1526573431.png)

   尝试`grade`测试：

   ```bash
   make grade
   ```

   ![2022-10-11 19-45-49 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213211621-1604258092.png)

   显示`ph_safe`可以通过，但`ph_fast`不行，故需要进行改进来提高性能

   

4. lock改进

   正如实验网站所说的，并不是所有的情况都需要上锁，或者说，不是所有的put()都应该上一把锁，只需要将可能冲突的情况用一把锁锁住即可。

   恰好这里的哈希函数求了一个i值，我们可以根据不同的i值来上不同的锁

   ```c++
   ...
   pthread_mutex_t lock[NBUCKET];
   
   static
   void put(int key, int value)
   {
     int i = key % NBUCKET;
     pthread_mutex_lock(&lock[i]);//上锁
     // is the key already present?
     struct entry *e = 0;
     for (e = table[i]; e != 0; e = e->next) {
       if (e->key == key)
         break;
     }
     if(e){
       // update the existing key.
       e->value = value;
     } else {
       // the new is new.
       insert(key, value, &table[i], table[i]);
       
     }
     pthread_mutex_unlock(&lock[i]);//开锁
   }
   
   int
   main(int argc, char *argv[])
   {
   ...
     for(int j=0;j<NBUCKET;j++)
         pthread_mutex_init(&lock[j],NULL);
   ...
   }
   
   ```

   进行测试`./ph 2`：

   ![2022-10-11 20-31-05 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213211942-1044413407.png)

   进行grade测试：

   ![2022-10-11 20-18-57 的屏幕截图](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213212312-2130551536.png)




#### Part C Barrier

1. 实现思路：

   根据代码提示，只需要在barrier函数中添加内容即可。首先，因为涉及到全局变量的改变，barrier应该上一个锁，分别在前后进行lock和unlock。然后bstate.nthread++，接下来判断该值是否达到了阈值，不是则进行睡眠等待；是则要开启新的一轮，bstate.nthread置零，bstate.round++，并唤醒睡眠的线程。

2. 具体实现：

   ![image-20221129150411106](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105741179-1066128341.png)

**测试：**

![image-20221129152648035](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105741474-1810966270.png)



<font size=5>二、问题回答&实验分析：</font>

##### Part A：

1. 为什么`thread_switch` 只需要保存/恢复`callee-save registers`？

   在swtch.S中`callee-save registers`的注明应该有点问题，ra和sp应该也是属于的，也就是说context的全部内容应该都是`callee-save registers`。callee-save，顾名思义，就是callee保存的寄存器，具体到这里就是 `thread_switch()` ，但该函数需要保存旧的、读取新的且未在其他地方保存寄存器内容，故需要对其保存/恢复。与之相对的是caller保存的寄存器（`callee-save registers`），后者是由调用程序自行负责保存和恢复，都会被保存在线程的堆栈上，故不需要额外的保存和恢复。

   ![image-20221129154243489](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105741755-1870546763.png)<img src="https://picx.zhimg.com/v2-6bf2eda179bcb277cdd3ede431c73c4b_r.jpg?source=1940ef5c" alt="img" style="zoom: 67%;" />

##### Part B：

1. 为什么两个线程会丢失 keys，但是一个线程不会？确定一种两个线程的执行序列，可以使得 key 丢失。

   我们先来看看put的机制：先获取当前key的hash值i，然后以此为头，查找表中是否已经有key，若是则更换；否则就要进行insert。

   insert操作将key插在来最初查找开始表头的前面。

   那么问题就来了，假如当key的next已经指向原head了，这时发生了线程切换，因为head还没来得及更换，新的key又指向了head，再把head替换为key，则原来的key就被丢弃了，这便导致了miss。

2. 可以使用哈希值i，来上不同的锁lock[i]的原因：

   相同的i值，说明会有相同的表头，会出现上述的情况，自然应该用一个锁

   但不同的i值，表头不同，即使出现上述情况，也不会导致key的丢失

3. 为什么不直接锁insert()？

   既然问题是出在insert()，为什么不可以直接锁insert()，最开始我也是这样做的，但后面想到了这样一种情况：

   如果我当前的put遍历到一半发生了切换，新线程是key与原来一样且都未添加，则此线程会将key添加到表头。再次切换后，原线程因为已经遍历过表头了，它会认为key还未添加，故又会把key加到表头，造成key的重复添加。

4. 各情况下吞吐率图

   | Theads | Code     | Puts（/s) | Gets(/s) |
   | ------ | -------- | --------- | -------- |
   | 1      | no_lock  | 22151     | 22104    |
   | 2      | no_lock  | 45720     | 36789    |
   | 1      | lock_old | 20854     | 22028    |
   | 2      | lock_old | 13160     | 41102    |
   | 1      | lock_new | 21641     | 21340    |
   | 2      | lock_new | 24980     | 41980    |

![download](https://img2022.cnblogs.com/blog/3018649/202210/3018649-20221026213212602-91647687.png)



<font size=5>三、问题解决：</font>

【问题】在Part C最开始实现时，我并没有在到达阈值时将bstate.nthread直接置0，而是在退出时bstate.nthread--，这便不能过。

![image-20221130103916395](https://img2023.cnblogs.com/blog/3018649/202211/3018649-20221130105742012-1393231646.png)

【分析】这正是Hits的问题所提到的，考虑这样的情况：如果达到阈值后，这一轮的线程还未退出，即bstate.nthread还没开始--。这时有一个新的线程来了，bstate.nthread此时>=nthread，会把bstate.round++，但这不是新的一轮完了。故会导致错误。

【解决】所以要让bstate.nthread和bstate.round同时发生变化，前者回到初值0，后者+1。



<font size=5>四、实验总结：</font>

1. 这次实验的代码因为有部分内容已经在之前的作业中做了，工作量有所减轻，但重新看这次，会对以前有些地方的实现有了一些新的理解。

2. 相比于书上和课上直接给到的知识，实操起来真的会有更大收获。比如课上以count为例讲临界区，因为是直接就告诉你是怎么回事了，只需要去理解这个过程就可以了，但实验中具体的代码就需要自己去寻找出错的条件，对lock的机制掌握自然也就更深入了。

3. 另外，这学期学的“计算机可视化”没想到在这里吞吐量的绘图发挥了用处，也算是某种意义上的学科交叉了。

   

<font size=5>五、实验参考及git地址：</font>

1. [Lab: Multithreading (mit.edu)](https://pdos.csail.mit.edu/6.S081/2022/labs/thread.html)
2. [第五章 调度 | xv6 中文文档 (gitbooks.io)](https://th0ar.gitbooks.io/xv6-chinese/content/content/chapter5.html)
3. [为什么要区分caller saved和callee saved registers? ](https://www.zhihu.com/question/453450905)
4. [MIT6.S081: OS lab of jeekzhang](https://gitee.com/zhang-gike/MIT6.S081/tree/multithreading/)