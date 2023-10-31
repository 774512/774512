![test_1 24](https://github.com/774512/774512/assets/148979339/eba19a6e-b6d3-433d-a756-5a9b1df760c4)![image](https://github.com/774512/774512/assets/148979339/92c7e7c1-1aef-488c-911e-8445929f3671)# 遇到的问题及解决方法
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
  - 进行三次运行，从每次的结果我们可以发现子进程里面的pid1与父进程的pid相同，这是因为父进程经过fork返回的是子进程的pid，而子进程getpid()后获得自己的pid，二者当然相同
  - 子进程经过fork后获得的pid是0
  - 父进程gitpid()获得自己的pid，与子进程的不同
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
    - 我们从运行结果里面可以发现，通过system调用外部函数后，进程的pid为child的pid1加一，而通过execl调用外部函数时进程的pid和child的pid1相等。这是因为，system中是一个fork和execl，相当于创建了一个子进程的子进程。而execl是直接将相应的外部函数加载到当前的地址上，覆盖(破坏)了当前进程的内容，但是进程的pid是不会变的。
## 1.2线程相关编程实验
### 1.21在进程中给一变量赋初值并成功创建两个线程
![image](https://github.com/774512/774512/assets/148979339/5dd47a62-4f15-4f4c-bbef-5e30f9daaf1c)
运行结果
![image](https://github.com/774512/774512/assets/148979339/e2c92a21-7933-4ac6-8ace-6e9969d4bc79)
结果分析：
    - 这个函数里用了一个全局变量s，执行函数是一个10000的累加，理想的运行结果应该是20000，我这里运行了3次这个程序，每次的结果都不同。 
    - s++ 是有三个步的，读取s，s+1，写入s。
    - 在程序运行的某个时刻，th1携带myfunc执行s++，读取s,此时s=100,进行s+1， 与此同时th2也开始读取s，此时的s还是等于100, 这时th1，执行写入s=101，th2执行s++,写入s ，s=101. th2中的s 就会覆盖掉 th1中的s 。这样造成了结果的误差。
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





 






<!---
774512/774512 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
