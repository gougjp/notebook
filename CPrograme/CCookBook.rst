C语言笔记
============

Linux下将不同线程绑定到不同的CPU上运行
---------------

cpu_set_t结构体是一个掩码数组，一共有1024位，每一位都可以对应一个cpu核心, 每个bit表示进程是否要与某个CPU核绑定

用到的函数和宏:

.. code-block:: c

    // 定义CPU集合的变量
    cpu_set_t mask;
    // 初始化set集，将set设置为空
    void CPU_ZERO(cpu_set_t *set);
    // 将某个CPU加入到集合
    void CPU_SET(int cpu, cpu_set_t *set);
    // 将某个CPU从CPU集中移出
    void CPU_CLR(int cpu, cpu_set_t *set);
    // 判断某个CPU是否已在CPU集中设置了
    int CPU_ISSET (int cpu, const cpu_set_t *set);
    // 将线程绑定到CPU
    int pthread_setaffinity_np(pthread_t thread, size_t cpusetsize, const cpu_set_t *cpuset);
    // 获取线程绑定CPU的信息
    int pthread_getaffinity_np(pthread_t thread, size_t cpusetsize, cpu_set_t *cpuset);

实例1:

.. code-block:: c

    #define _GNU_SOURCE
    #include <pthread.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <errno.h>

    #define handle_error_en(en, msg) \
            do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

    int
    main(int argc, char *argv[])
    {
        int s, j;
        cpu_set_t cpuset;
        pthread_t thread;

        thread = pthread_self();

        /* Set affinity mask to include CPUs 0 to 7 */

        CPU_ZERO(&cpuset);
        for (j = 0; j < 8; j++)
            CPU_SET(j, &cpuset);

        s = pthread_setaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
        if (s != 0)
            handle_error_en(s, "pthread_setaffinity_np");

        /* Check the actual affinity mask assigned to the thread */

        s = pthread_getaffinity_np(thread, sizeof(cpu_set_t), &cpuset);
        if (s != 0)
            handle_error_en(s, "pthread_getaffinity_np");

        printf("Set returned by pthread_getaffinity_np() contained:\n");
        for (j = 0; j < CPU_SETSIZE; j++)
            if (CPU_ISSET(j, &cpuset))
                printf("    CPU %d\n", j);

        exit(EXIT_SUCCESS);
    }

运行结果:

::

    -> % ./a.out 
    Set returned by pthread_getaffinity_np() contained:
        CPU 0
        CPU 1

实例2:

.. code-block:: c

    //-------------------------------------------------------------------------------------------
    /** @ingroup group_lte_source_auxlib_sys
     *
     *  @param[in]   coreNum Core Id
     *
     *  @return  0 if SUCCESS
     *
     *  @description
     *  This function binds a thread to a core
     *
    **/
    //-------------------------------------------------------------------------------------------
    int sys_affinity_bind(int coreNum)
    {
        cpu_set_t cpuset;
        int i, rc;

        /* set main thread affinity mask to CPU1 */

         CPU_ZERO(&cpuset);
         CPU_SET(coreNum, &cpuset);

         rc = pthread_setaffinity_np(pthread_self(), sizeof(cpu_set_t), &cpuset);
         if (rc)
         {
             perror("pthread_setaffinity_np failed");
             print_err("pthread_setaffinity_np failed: %d", rc);
         }

         /* check the actual affinity mask assigned to the thread */

         CPU_ZERO(&cpuset);

         rc = pthread_getaffinity_np(pthread_self(), sizeof(cpu_set_t), &cpuset);

         if (rc)
         {
             perror("pthread_getaffinity_np failed");
             print_err("pthread_getaffinity_np failed: %d", rc);
         }

         print_dbg("set sys affinity: ");
         for (i = 0; i < CPU_SETSIZE; i++)
             if (CPU_ISSET(i, &cpuset))
                 print_dbg("    CPU %d\n", i);

         if (!CPU_ISSET(coreNum, &cpuset))
         {
             print_err("affinity failed");
         }

         /**
            A new thread created by pthread_create(3) inherits a copy of its
            creator's CPU affinity mask. */

        return rc;
    }

Linux下将不同进程绑定到不同的CPU上运行
---------------

1. 命令行

使用命令**cat /proc/cpuinfo**查看CPU信息:

- ``processor``: 指明第几个cpu处理器

- ``cpu cores``: 指明每个处理器的核心数

也可以使用系统调用sysconf获取cpu核心数: 

.. code-block:: c

    #include <unistd.h>

    int sysconf(_SC_NPROCESSORS_CONF);/* 返回系统可以使用的核数，但是其值会包括系统中禁用的核的数目，因 此该值并不代表当前系统中可用的核数 */
    int sysconf(_SC_NPROCESSORS_ONLN);/* 返回值真正的代表了系统当前可用的核数 */

    /* 以下两个函数与上述类似 */
    #include <sys/sysinfo.h>

    int get_nprocs_conf (void);/* 可用核数 */
    int get_nprocs (void);/* 真正的反映了当前可用核数 */

使用命令**taskset**命令指令将进程绑定到CPU:

- 获取进程pid

::

    -> % ps
      PID TTY          TIME CMD
     2683 pts/1    00:00:00 zsh
     2726 pts/1    00:00:00 dgram_servr
     2930 pts/1    00:00:00 ps

- 查看进程当前运行在哪个cpu上

::

    -> % taskset -p 2726
    pid 2726's current affinity mask: 3

显示的十进制数字3转换为2进制为最低两个是1, 每个1对应一个cpu, 所以进程运行在2个cpu上

- 指定进程运行在cpu1上

::

    -> % taskset -pc 1 2726
    pid 2726's current affinity list: 0,1
    pid 2726's new affinity list: 1

注意, cpu的标号是从0开始的, 所以cpu1表示第二个cpu(第一个cpu的标号是0)

至此, 就把应用程序绑定到了cpu1上运行, 查看如下:

::

    -> % taskset -p 2726
    pid 2726's current affinity mask: 2

- 启动程序时绑定cpu

::

    #启动时绑定到第二个cpu
    -> % taskset -c 1 ./dgram_servr&
    [1] 3011

    #查看确认绑定情况
    -> % taskset -p 3011
    pid 3011's current affinity mask: 2

2. 系统接口

sched_setaffinity可以将某个进程绑定到一个特定的CPU

.. code-block:: c

    #define _GNU_SOURCE             /* See feature_test_macros(7) */
    #include <sched.h>

    /* 设置进程号为pid的进程运行在mask所设定的CPU上
     * 第二个参数cpusetsize是mask所指定的数的长度
     * 通常设定为sizeof(cpu_set_t)

     * 如果pid的值为0,则表示指定的是当前进程 
     */
    int sched_setaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);

    int sched_getaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);/* 获得pid所指示的进程的CPU位掩码,并将该掩码返回到mask所指向的结构中 */

实例:

.. code-block:: c

    #include<stdlib.h>
    #include<stdio.h>
    #include<sys/types.h>
    #include<sys/sysinfo.h>
    #include<unistd.h>

    #define __USE_GNU
    #include<sched.h>
    #include<ctype.h>
    #include<string.h>
    #include<pthread.h>
    #define THREAD_MAX_NUM 200  //1个CPU内的最多进程数

    int num=0;  //cpu中核数
    void* threadFun(void* arg)  //arg  传递线程标号（自己定义）
    {
             cpu_set_t mask;  //CPU核的集合
             cpu_set_t get;   //获取在集合中的CPU
             int *a = (int *)arg; 
             int i;

             printf("the thread is:%d\n",*a);  //显示是第几个线程
             CPU_ZERO(&mask);    //置空
             CPU_SET(*a,&mask);   //设置亲和力值
             if (sched_setaffinity(0, sizeof(mask), &mask) == -1)//设置线程CPU亲和力
             {
                       printf("warning: could not set CPU affinity, continuing...\n");
             }

               CPU_ZERO(&get);
               if (sched_getaffinity(0, sizeof(get), &get) == -1)//获取线程CPU亲和力
               {
                        printf("warning: cound not get thread affinity, continuing...\n");
               }
               for (i = 0; i < num; i++)
               {
                        if (CPU_ISSET(i, &get))//判断线程与哪个CPU有亲和力
                        {
                                 printf("this thread %d is running processor : %d\n", i,i);
                        }
               }

             return NULL;
    }

    int main(int argc, char* argv[])
    {
             int tid[THREAD_MAX_NUM];
             int i;
             pthread_t thread[THREAD_MAX_NUM];

             num = sysconf(_SC_NPROCESSORS_CONF);  //获取核数
             if (num > THREAD_MAX_NUM) {
                printf("num of cores[%d] is bigger than THREAD_MAX_NUM[%d]!\n", num, THREAD_MAX_NUM);
                return -1;
             }
             printf("system has %i processor(s). \n", num);

             for(i=0;i<num;i++)
             {
                       tid[i] = i;  //每个线程必须有个tid[i]
                       pthread_create(&thread[i],NULL,threadFun,(void*)&tid[i]);
             }
             for(i=0; i< num; i++)
             {
                       pthread_join(thread[i],NULL);//等待所有的线程结束，线程为死循环所以CTRL+C结束
             }
             return 0;
    }

运行结果:

::

    -> % ./a.out
    system has 2 processor(s). 
    the thread is:0
    the thread is:1
    this thread 0 is running processor : 0
    this thread 1 is running processor : 1

参考:

https://www.cnblogs.com/x_wukong/p/5924298.html
https://blog.csdn.net/guotianqing/article/details/80958281
https://www.jianshu.com/p/e2059724d22e
https://www.jianshu.com/p/f59d7df06432

线程调度优先级设置
--------------

1. 线程调度的三种策略:

``SCHED_OTHER``: Linux默认的分时调度策略, 所有的线程的优先级别都是0, 线程的调度是通过分时来完成的; 简单地说，如果系统使用这种调度策略，程序将无法设置线程的优先级。请注意, 这种调度策略也是抢占式的, 当高优先级的线程准备运行的时候, 当前线程将被抢占并进入等待队列. 这种调度策略仅仅决定线程在可运行线程队列中的具有相同优先级的线程的运行次序.

``SCHED_FIFO``: 它是一种实时的先进先出调用策略, 且只能在超级用户下运行. 这种调用策略仅仅被使用于优先级大于0的线程. 它意味着，使用SCHED_FIFO的可运行线程将一直抢占使用SCHED_OTHER的运行线程. 此外SCHED_FIFO是一个非分时的简单调度策略, 当一个线程变成可运行状态, 它将被追加到对应优先级队列的尾部((POSIX 1003.1). 当所有高优先级的线程终止或者阻塞时, 它将被运行. 对于相同优先级别的线程, 按照简单的先进先运行的规则运行. 我们考虑一种很坏的情况, 如果有若干相同优先级的线程等待执行, 然而最早执行的线程无终止或者阻塞动作, 那么其他线程是无法执行的, 除非当前线程调用如pthread_yield之类的函数, 所以在使用SCHED_FIFO的时候要小心处理相同级别线程的动作.

``SCHED_RR``: 鉴于SCHED_FIFO调度策略的一些缺点, SCHED_RR对SCHED_FIFO做出了一些增强功能. 从实质上看, 它还是SCHED_FIFO调用策略. 它使用最大运行时间来限制当前进程的运行, 当运行时间大于等于最大运行时间的时候, 当前线程将被切换并放置于相同优先级队列的最后. 这样做的好处是其他具有相同级别的线程能在“自私“线程下执行.


2. 获取线程可以设置的最高和最低优先级:

``int sched_get_priority_max(int policy)``;

``int sched_get_priority_min(int policy)``;

policy可以为SCHED_OTHER, SCHED_FIFO, SCHED_RR; 对于 SCHED_OTHER 策略, sched_priority只能为0; 对于SCHED_FIFO, SCHED_RR策略, sched_priority从1到99.


3. 获取和设置线程的优先级:

.. code-block:: c

    struct sched_param
    {
        int __sched_priority; // 所要设定的线程优先级
    };

该结构体仅仅包含一个成员变量sched_priority, 指明所要设置的静态线程优先级

param.sched_priority = 51;

``int pthread_attr_setschedparam(pthread_attr_t \*attr, const struct sched_param \*param)``;

``int pthread_attr_getschedparam(const pthread_attr_t \*attr, struct sched_param \*param)``;


4. 获取和设置线程的调度策略:

``int pthread_attr_setschedpolicy(pthread_attr_t \*attr, int policy)``;

``int pthread_attr_getschedpolicy(pthread_attr_t \*attr, int policy)``;


5. 继承调度属性:

``int pthread_attr_setinheritsched(pthread_attr_t \*attr, int inheritsched)``;

``int pthread_attr_getinheritsched(pthread_attr_t \*attr, int *inheritsched)``;  

手动设置了调度策略或优先级时, 必须显示的设置线程调度策略的inheritsched属性, 因为pthread没有为inheritsched设置默认值. 所以在改变了调度策略或优先级时必须总是设置该属性.

第一个函数中inheritsched的取值为: PTHREAD_INHERIT_SCHED或者PTHREAD_EXPLICIT_SCHED.

前者为继承创建线程的调度策略和优先级, 后者指定不继承调度策略和优先级, 而是使用自己设置的调度策略和优先级; 无论何时, 当你需要控制一个线程的调度策略或优先级时, 必须将inheritsched属性设置为PTHREAD_EXPLICIT_SCHED.

6. 总结:

前面这种方式是静态改变策略和优先级

调度策略和优先级是分开来描述的. 前者使用预定义的SCHED_RR, SCHED_FIFO, SCHED_OTHER, 后者是通过结果体struct sched_param给出的

这些设置调度策略和优先级的函数操作的对象是线程的属性pthread_attr_t, 而不是直接来操作线程的调度策略和优先级的. 函数的第一个参数都是pthread_attr_t


7. 直接设置正在运行的线程的调度策略和优先级(动态设置线程的调度策略和优先级):

``int pthread_setschedparam(pthread_t thread, int policy, const struct sched_param \*param)``;

``int pthread_getschedparam(pthread_t thread, int \*policy, struct sched_param \*param)``; 

失败条件: 

    pthread_setschedparam：
        
        thread参数所指向的线程不存在

    pthread_getschedparam: 

        参数policy或同参数policy关联的调度参数之一无效
        
        参数policy或调度参数之一的值不被支持
        
        调用线程没有适当的权限来设置指定线程的调度参数或策略
        
        参数thread指向的线程不存在
        
        实现不允许应用程序将参数改动为特定的值
        
当pthread_setschedparam函数的参数policy==SCHED_RR或者SCHED_FIFO时, 程序必须要在超级用户下运行

pthread_setschedparam 函数改变在运行线程的调度策略和优先级肯定就不用调用函数来设置inheritsched属性了
    

参考:

https://www.cnblogs.com/eleclsc/p/10523608.html

信号量
--------------------

信号量(Semaphore): 有时被称为信号灯, 是在多线程环境下使用的一种设施, 是可以用来保证两个或多个关键代码段不被并发调用. 在进入一个关键代码段之前, 线程必须获取一个信号量: 一旦该关键代码段完成了, 那么该线程必须释放信号量. 其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量.

类似计数器, 常用在多线程同步任务上, 信号量可以在当前线程某个任务完成后, 通知别的线程, 再进行别的任务.

信号量是在多线程环境中共享资源的计数器. 对信号量的基本操作无非有三个: 对信号量的增加, 然后阻塞线程等待, 直到信号量不为空才返回: 然后就是对信号量的减少.

分类:

- 二值信号量: 信号量的值只有0和1, 这和互斥量很类似, 若资源被锁住, 信号量的值为0; 若资源可用, 则信号量的值为1.

- 计数信号量: 信号量的值在0到一个大于1的限制值之间, 该计数表示可用的资源的个数.

信号量在创建时需要设置一个初始值, 表示同时可以有几个任务可以访问该信号量保护的共享资源, 初始值为1就变成互斥锁Mutex, 即同时只能有一个任务可以访问信号量保护的共享资源.

函数使用:

首先需要include \<semaphore.h\>这个库

**int sem_init(sem_t \*sem, int pshared, unsigned int value)**: 创建信号量, sem是要初始化的信号量; pshared表示此信号量是在进程间共享还是线程间共享, 如果其值为0, 就表示信号量是当前进程的局部信号量, 否则信号量就可以在多个进程间共享; value是信号量的初始值; 返回值success为0, failure为-1.

**int sem_wait(sem_t \*sem)**: 等待信号量, 如果信号量的值大于0, 将信号量的值减1, 立即返回. 如果信号量的值为0, 则线程阻塞; 返回值success为0, failure为-1. 当信号的计数为零的时候, sem_wait将休眠挂起当前调用线程, 直到信号量计数不为零, 在sem_wait返回后信号量计数将自动减1.

**int sem_post(sem_t \*sem)**: 释放信号量, 让信号量的值加1; 返回值success为0, failure为-1. 解除信号量等待限制, 让信号量计数加1, 该函数会立即返回不等待

**int sem_destroy(sem_t \*sem)**: 其中sem是要销毁的信号量, 只有用sem_init初始化的信号量才能用sem_destroy销毁.

**int sem_trywait(sem_t \*sem)**: 是一个立即返回函数, 不会因为任何事情阻塞, 根据其返回值得到不同的信息. 如果返回值为0, 说明信号量在该函数调用之前大于0, 但是调用之后会被该函数自动减1. 至于调用之后是否为零则不得而知了. 如果返回值为EAGAIN说明信号量计数为0.

**int sem_getvalue(sem_t * sem, int * sval)**: 获得当前信号量计数的值

在编程中, 信号量最常用的方式就是一个线程A使用sem_wait阻塞, 因为此时信号量计数为0, 直到另外一个线程B发出信号post后, 信号量计数加1, 此时, 线程A得到了信号, 信号量的计数为1不为空, 所以就从sem_wait返回了, 然后信号量的计数又减1变为零.

在使用信号量之前, 我们必须初始化信号. 第三个参数通常设置为零, 初始化信号的计数为0, 这样第一次使用sem_wait的时候会因为信号计数为0而等待, 直到在其他地方信号量post了才返回. 除非你明白你在干什么, 否则不要将第三个参数设置为大于0的数

第二个参数是用在进程之间的数据共享标志, 如果仅仅使用在当前进程中, 设置为0. 如果要在多个进程之间使用该信号, 设置为非零. 但是在Linux线程中, 暂时还不支持进程之间的信号共享, 所以第二个参数说了半天等于白说, 必须设置为0, 否则将返回ENOSYS错误.

参考:

https://www.cnblogs.com/hnrainll/archive/2011/04/20/2022487.html

互斥锁
------------------

posix下抽象了一个锁类型的结构: ptread_mutex_t, 通过对该结构的操作, 来判断资源是否可以访问. 顾名思义, 加锁(lock)后, 别人就无法打开, 只有当锁没有关闭(unlock)的时候才能访问资源.

即对象互斥锁的概念, 来保证共享数据操作的完整性. 每个对象都对应于一个可称为"互斥锁"的标记, 这个标记用来保证在任一时刻, 只能有一个线程访问该对象.

使用互斥锁(互斥)可以使线程按顺序执行. 通常, 互斥锁通过确保一次只有一个线程执行代码的临界段来同步多个线程. 互斥锁还可以保护单线程代码.

要更改缺省的互斥锁属性, 可以对属性对象进行声明和初始化. 通常, 互斥锁属性会设置在应用程序开头的某个位置, 以便可以快速查找和轻松修改.

1. 锁的创建

**int pthread_mutex_init(pthread_mutex_t \*restrict mutex, const pthread_mutexattr_t \*restrict attr)**

以动态方式创建互斥锁, 参数attr指定了新建互斥锁的属性, 如果参数attr为NULL, 则使用默认的互斥锁属性, 默认属性为快速互斥锁. 互斥锁的属性在创建锁的时候指定, 在LinuxThreads实现中仅有一个锁类型属性, 不同的锁类型在试图对一个已经被锁定的互斥锁加锁时表现不同.

**pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER**

用宏PTHREAD_MUTEX_INITIALIZER来静态的初始化锁, 采用这种方式比较容易理解, 互斥锁是pthread_mutex_t的结构体, 而这个宏是一个结构常量.
    
2. 锁的属性

**int pthread_mutexattr_init(pthread_mutexattr_t \*attr)**

初始化锁的属性, 然后可以调用其他的属性设置方法来设置其属性.

**int pthread_mutexattr_getpshared(const pthread_mutexattr_t \*attr, int \*pshared)**
**int pthread_mutexattr_setpshared(pthread_mutexattr_t \*attr, int pshared)**

获取和设置互斥锁的范围, 可以指定是该进程与其他进程的同步还是同一进程内不同的线程之间的同步. PTHREAD_PROCESS_PRIVATE表示进程内使用锁, PTHREAD_PROCESS_SHARE表示进程间使用锁, 默认是进程内使用锁.

**int pthread_mutexattr_gettype(const pthread_mutexattr_t \*attr, int \*type)**
**int pthread_mutexattr_settype(pthread_mutexattr_t \*attr, int type)**

获取和设置互斥锁的类型:
PTHREAD_MUTEX_TIMED_NP, 这个是缺省值, 也就是普通锁. 当一个线程加锁以后, 其余请求锁的线程将形成一个等待队列, 并在解锁后按优先级获得锁. 这种锁策略保证了资源分配的公平性.
PTHREAD_MUTEX_RECURSIVE_NP, 嵌套锁, 允许同一个线程对同一个锁成功获得多次, 并通过多次unlock解锁. 如果是不同线程请求, 则在加锁线程解锁时重新竞争.
PTHREAD_MUTEX_ERRORCHECK_NP, 检错锁, 如果同一个线程请求同一个锁, 则返回EDEADLK, 否则与PTHREAD_MUTEX_TIMED_NP类型动作相同. 这样就保证当不允许多次加锁时不会出现最简单情况下的死锁.
PTHREAD_MUTEX_ADAPTIVE_NP, 适应锁, 动作最简单的锁类型, 仅等待解锁后重新竞争.
    
3. 其他锁操作

**int pthread_mutex_lock(pthread_mutex_t \*mutex)**  加锁, 
**int pthread_mutex_trylock(pthread_mutex_t \*mutex)**  测试加锁, 语义与pthread_mutex_lock类似，不同的是在锁已经被占据时返回EBUSY而不是挂起等待
**int pthread_mutex_unlock(pthread_mutex_t \*mutex)**  解锁

不论哪种类型的锁, 都不可能被两个不同的线程同时得到, 而必须等待解锁. 对于普通锁和适应锁类型, 解锁者可以是同进程内任何线程; 而检错锁则必须由加锁者解锁才有效, 否则返回EPERM. 对于嵌套锁, 文档和实现要求必须由加锁者解锁, 但实验结果表明并没有这种限制, 这个不同目前还没有得到解释. 在同一进程中的线程, 如果加锁后没有解锁, 则任何其他线程都无法再获得锁.

4. 锁的释放

**int pthread_mutex_destroy(pthread_mutex_t \*mutex)**

可以释放锁占用的资源，但这有一个前提上锁当前是没有被锁的状态

5. 死锁

死锁主要发生在有多个依赖锁存在时, 会在一个线程试图以与另一个线程相反顺序锁住互斥量时发生. 如何避免死锁是使用互斥量应该格外注意的东西:

对共享资源操作前一定要获得锁

完成操作以后一定要释放锁

尽量短时间地占用锁

如果有多锁, 如获得顺序是ABC连环扣, 释放顺序也应该是ABC

线程错误返回时应该释放它所获得的锁

6. 锁使用举例

.. code-block:: c

    #include <pthread.h>
    #include <stdio.h>
     
    pthread_mutex_t mutex ;
    void *print_msg(void *arg){
            int i=0;
            pthread_mutex_lock(&mutex);
            for(i=0;i<15;i++){
                    printf("output : %d\n",i);
                    usleep(100);
            }
            pthread_mutex_unlock(&mutex);
    }
    int main(int argc,char** argv){
            pthread_t id1;
            pthread_t id2;
            pthread_mutex_init(&mutex,NULL);
            pthread_create(&id1,NULL,print_msg,NULL);
            pthread_create(&id2,NULL,print_msg,NULL);
            pthread_join(id1,NULL);
            pthread_join(id2,NULL);
            pthread_mutex_destroy(&mutex);
            return 1;
    }
    
参考
https://blog.csdn.net/happylzs2008/article/details/89067028

typedef用法总结
-------------------

参考
http://c.biancheng.net/view/298.html


