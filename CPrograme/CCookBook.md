# C语言笔记

### Linux下将不同线程绑定到不同的CPU上运行

cpu_set_t结构体是一个掩码数组，一共有1024位，每一位都可以对应一个cpu核心, 每个bit表示进程是否要与某个CPU核绑定

用到的函数和宏:

```C
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
```

实例1:

```C
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
```

运行结果:

```Shell
-> % ./a.out 
Set returned by pthread_getaffinity_np() contained:
    CPU 0
    CPU 1
```

实例2:

```C
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
```



### Linux下将不同进程绑定到不同的CPU上运行

1. 命令行


2. 系统接口


参考:

https://www.cnblogs.com/x_wukong/p/5924298.html
https://blog.csdn.net/guotianqing/article/details/80958281
https://www.jianshu.com/p/e2059724d22e
https://www.jianshu.com/p/f59d7df06432

### 线程调度优先级设置

1. 线程调度的三种策略:

    **SCHED_OTHER**: Linux默认的分时调度策略, 所有的线程的优先级别都是0, 线程的调度是通过分时来完成的; 简单地说，如果系统使用这种调度策略，程序将无法设置线程的优先级。请注意, 这种调度策略也是抢占式的, 当高优先级的线程准备运行的时候, 当前线程将被抢占并进入等待队列. 这种调度策略仅仅决定线程在可运行线程队列中的具有相同优先级的线程的运行次序.

    **SCHED_FIFO**: 它是一种实时的先进先出调用策略, 且只能在超级用户下运行. 这种调用策略仅仅被使用于优先级大于0的线程. 它意味着，使用SCHED_FIFO的可运行线程将一直抢占使用SCHED_OTHER的运行线程. 此外SCHED_FIFO是一个非分时的简单调度策略, 当一个线程变成可运行状态, 它将被追加到对应优先级队列的尾部((POSIX 1003.1). 当所有高优先级的线程终止或者阻塞时, 它将被运行. 对于相同优先级别的线程, 按照简单的先进先运行的规则运行. 我们考虑一种很坏的情况, 如果有若干相同优先级的线程等待执行, 然而最早执行的线程无终止或者阻塞动作, 那么其他线程是无法执行的, 除非当前线程调用如pthread_yield之类的函数, 所以在使用SCHED_FIFO的时候要小心处理相同级别线程的动作.

    **SCHED_RR**: 鉴于SCHED_FIFO调度策略的一些缺点, SCHED_RR对SCHED_FIFO做出了一些增强功能. 从实质上看, 它还是SCHED_FIFO调用策略. 它使用最大运行时间来限制当前进程的运行, 当运行时间大于等于最大运行时间的时候, 当前线程将被切换并放置于相同优先级队列的最后. 这样做的好处是其他具有相同级别的线程能在“自私“线程下执行.

2. 获取线程可以设置的最高和最低优先级:

    **int sched_get_priority_max(int policy)**;
    
    **int sched_get_priority_min(int policy)**;
    
    policy可以为SCHED_OTHER, SCHED_FIFO, SCHED_RR; 对于 SCHED_OTHER 策略, sched_priority只能为0; 对于SCHED_FIFO, SCHED_RR策略, sched_priority从1到99.

3. 获取和设置线程的优先级:

    ```C
    struct sched_param
    {
        int __sched_priority; // 所要设定的线程优先级
    };
    ```
    
    该结构体仅仅包含一个成员变量sched_priority, 指明所要设置的静态线程优先级
    
    param.sched_priority = 51;

    **int pthread_attr_setschedparam(pthread_attr_t \*attr, const struct sched_param \*param)**;
    
    **int pthread_attr_getschedparam(const pthread_attr_t \*attr, struct sched_param \*param)**;

4. 获取和设置线程的调度策略:

    **int pthread_attr_setschedpolicy(pthread_attr_t \*attr, int policy)**;
    
    **int pthread_attr_getschedpolicy(pthread_attr_t \*attr, int policy)**;

5. 继承调度属性:

    **int pthread_attr_setinheritsched(pthread_attr_t \*attr, int inheritsched)**;
    
    **int pthread_attr_getinheritsched(pthread_attr_t \*attr, int *inheritsched)**;  

    手动设置了调度策略或优先级时, 必须显示的设置线程调度策略的inheritsched属性, 因为pthread没有为inheritsched设置默认值. 所以在改变了调度策略或优先级时必须总是设置该属性.

    第一个函数中inheritsched的取值为: PTHREAD_INHERIT_SCHED或者PTHREAD_EXPLICIT_SCHED.
    前者为继承创建线程的调度策略和优先级, 后者指定不继承调度策略和优先级, 而是使用自己设置的调度策略和优先级; 无论何时, 当你需要控制一个线程的调度策略或优先级时, 必须将inheritsched属性设置为PTHREAD_EXPLICIT_SCHED.
    
6. 总结:

    前面这种方式是静态改变策略和优先级
    
    调度策略和优先级是分开来描述的. 前者使用预定义的SCHED_RR, SCHED_FIFO, SCHED_OTHER, 后者是通过结果体struct sched_param给出的
    
    这些设置调度策略和优先级的函数操作的对象是线程的属性pthread_attr_t, 而不是直接来操作线程的调度策略和优先级的. 函数的第一个参数都是pthread_attr_t

7. 直接设置正在运行的线程的调度策略和优先级(动态设置线程的调度策略和优先级):

    **int pthread_setschedparam(pthread_t thread, int policy, const struct sched_param \*param)**;
    
    **int pthread_getschedparam(pthread_t thread, int \*policy, struct sched_param \*param)**; 
    
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













