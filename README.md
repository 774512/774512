# 遇到的问题及解决方法
1.书上源程序报错wait函数

![image](https://github.com/774512/774512/assets/148979339/2a18b473-f826-4194-afdc-46d6a59a0ac1)

解决方法：引入#include <wait.h>

2.system隐式声明

![image](https://github.com/774512/774512/assets/148979339/774bd953-e04f-4c14-993f-26b1524391fc)

解决方法：引入#include <stdlib.h>

3.无法引用pthread库

![image](https://github.com/774512/774512/assets/148979339/11179713-a6b6-4bd9-a890-9e3a159e121f)

手动编译，在指令最后加入 -lpthread
# 实验具体步骤
## 1.1进程相关编程问题
### 多运行几次程序观察结果

![image](https://github.com/774512/774512/assets/148979339/7c60fd1e-42f6-43d4-9abb-0366f5bff1d4)

结果分析：
  函数调用一次，返回两次。两次返回的区别是：子进程的返回值是0，父进程返回值为新子进程的进程ID。
  调用fork()函数以后，若未成功创建子进程，则函数返回值为1；若成功创建子进程，其中父进程的返回值为子进程的地址，子进程的返回值为0，余下代码由父子进程共同执行。
  对于子进程，child:pid值为0，子进程进入else (pid==0)语句以后，调用getpid()函数返回值为调动函数的进程的地址，即其自身的地址（child:pid1为子进程的地址）。
  对于父进程，parent:pid值为子进程的地址，进入else语句后，调用getpid()函数，其返回值为调用函数的进程的地址，即其自身的地址（parent:pid1为父进程的地址）。
  综上所述， Parent:pid的值应与child:pid1的值相同，都为子进程的地址； Parent:pid1的值为父进程的地址；Child:pid的值为0
### 去除 wait 后再观察结果

![image](https://github.com/774512/774512/assets/148979339/4014152a-eaa1-4f6f-b3ca-f867adb07b09)

结果分析
    - 这三次parent先输出，原因是注释掉了wait()函数,父进程不用等子进程执行完毕再执行
    - 其余特性与未注释掉wait()函数时一致
### 添加一个全局变量并在父进程和子进程中对这个变量做不同操作
- 设置了全局变量int my_int = 1 ,在子进程里面对其进行加操作，父进程里面对其进行减操作，输出相应的值和地址。
- 代码修改

![image](https://github.com/774512/774512/assets/148979339/243feb6d-df43-418b-98e6-23d76a1f36a7)

运行结果

![image](https://github.com/774512/774512/assets/148979339/fb75f790-6827-41e6-9003-674383926b17)

结果分析：
    - pid与pid1的分析结果与前边实验相同
    - 父子进程的my_int的地址都是相同的，这是因为子进程采用了虚拟内存技术，实际地址为基址+偏移量，子进程实际上只是把偏移量拷贝过来了放在了不同的基址中。而输出地址时输出的是变量位置在各自基址上的偏移量。
    - 父进程中my_int值为0，子进程中my_int值为2，原因是采用了写时复制技术，子进程在写时将虚拟空间中的堆栈等变成实际的物理空间，子进程的资源实际上是父进程的拷贝，二者实际对应不同的my_int。
###  在 return 前增加对全局变量的操作并输出结果
代码修改

![image](https://github.com/774512/774512/assets/148979339/145599ee-fcfa-4f7f-9702-bac09d0dbcd3)

运行结果

![image](https://github.com/774512/774512/assets/148979339/a0eb5ba1-8835-4ff0-9fa0-101ebaaf41da)

结果分析：
    - 我们可以发现父进程与子进程输出的my_int的值是不一样的并且输出各自的pid1+my_int(此处的my_int是父子进程各自进行一定操作后的my_int)
### 修改程序体会在子进程中调用 system 函数和在子进程中调用 exec 族函数
代码修改

![image](https://github.com/774512/774512/assets/148979339/924fcd80-867f-4202-a051-ee5ded2da09e)

![image](https://github.com/774512/774512/assets/148979339/635b98d3-55ca-4cdc-9a20-fe9479c71a03)

- 利用system函数调用外部函数程序运行的结果为

   ![image](https://github.com/774512/774512/assets/148979339/2f0769c9-5caf-48ae-9b52-b8cbe89f964a)

- 利用exec函数调用外部函数程序运行的结果为

   ![image](https://github.com/774512/774512/assets/148979339/b987f49d-e1d4-4256-9d2d-4e7252c3b09e)

结果分析：
    我们从运行结果里面可以发现，通过system调用外部函数后，进程的pid为child的pid1加一，而通过execl调用外部函数时进程的pid和child的pid1相等。这是因为，system中是一个fork和execl，相当于创建了一个子进程的子进程。而execl是直接将相应的外部函数加载到当前的地址上，覆盖(破坏)了当前进程的内容，但是进程的pid是不会变的。
## 1.2线程相关编程实验
### 1.21在进程中给一变量赋初值并成功创建两个线程

![image](https://github.com/774512/774512/assets/148979339/5dd47a62-4f15-4f4c-bbef-5e30f9daaf1c)

运行结果

![image](https://github.com/774512/774512/assets/148979339/e2c92a21-7933-4ac6-8ace-6e9969d4bc79)

结果分析：
    这个函数里用了一个全局变量s，执行函数是一个10000的累加，理想的运行结果应该是20000，我这里运行了3次这个程序，每次的结果都不同。 
    s++ 是有三个步的，读取s，s+1，写入s。
    在程序运行的某个时刻，th1携带myfunc执行s++，读取s,此时s=100,进行s+1， 与此同时th2也开始读取s，此时的s还是等于100, 这时th1，执行写入s=101，th2执行s++,写入s ，s=101. th2中的s 就会覆盖掉 th1中的s 。这样造成了结果的误差。
### 1.22控制互斥与同步
- 要解决两个线程的同步互斥问题，使得一个线程进入临界区的时候另外一个线程无法更改共享数据的值，，可以使用互斥锁（Mutex）来保护共享变量的访问。在每个线程对变量进行操作之前，先获取互斥锁，操作完成后再释放锁。
代码修改

![image](https://github.com/774512/774512/assets/148979339/64c168ab-a40d-4482-8c73-1cf35f0aedad)

运行结果：

![image](https://github.com/774512/774512/assets/148979339/e018c73d-ab26-424a-84cf-ab19151b7444)

三次运行结果一致，th1运行后 进行了加锁，th2这时候想要运行，就必须等待th1解锁之后才行
### 1.23调用 system 函数和调用 exec 族函数在线程中实现
代码修改

![image](https://github.com/774512/774512/assets/148979339/1c4b8f7e-fc77-4233-b35a-b88146c017c0)

- 调用system函数的结果如下

   ![image](https://github.com/774512/774512/assets/148979339/5bac9334-53eb-4844-ae49-d92bb33ec81a)

- 调用execl函数的结果如下

  ![image](https://github.com/774512/774512/assets/148979339/293a19bf-7583-42e1-84f7-8000f5891722)

结果分析：
    我们发现新线程相当于拥有了一个新的pid，而原来的线程拥有的pid和进程的pid是一样的。并且两个线程的tid是不一样的。当通过system调用外部函数时，相当于又新开了个进程，pid有所改变，执行完外部函数后返回继续执行原来的程序。但当通过execl调用外部函数时，是将整个外部函数覆盖了目前的进程，所以进程后边的部分不在执行。
## 自旋锁实验

代码

![image](https://github.com/774512/774512/assets/148979339/97692981-4099-4f02-b425-04dca9a54e61)

![image](https://github.com/774512/774512/assets/148979339/d29381c5-5a06-41cb-a268-c074343a47e2)

运行结果

![image](https://github.com/774512/774512/assets/148979339/7fd98fbc-c762-4272-9bfb-7471ca4197a3)

结果分析：
    









 






<!---
774512/774512 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
