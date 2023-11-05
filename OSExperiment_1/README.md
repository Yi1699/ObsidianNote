# 一.环境搭建
### 1).华为云服务器搭建
按照系统环境搭建指导书的说明，在华为云服务器搭建好OpenEuler鲲鹏服务器，通过命令`ssh root@120.46.184.168`，并输入服务器密码后，成功连接至云服务器。
执行环境指导书中的命令，可以显示云服务器的各项信息（指导书中gcc命令应为`gcc --version`）。安装winSCP，新建站点登录远程主机，便可以使用图形界面管理文件。

#### 搭建成功界面
![[Pasted image 20231022164548.png]]
- 客户机通过ssh成功连接服务器
![[Pasted image 20231022164717.png]]
### 2).本地OpenEuler虚拟机系统安装
按照教程安装即可。
root用户密码的数字需要从主键盘输入！

---
# 二.实验
### 1.1 进程
#### 1.1.1
- 代码实现
打开虚拟机，在vim编辑器中写如下代码：
![[Pasted image 20231018175311.png]]
- 运行结果：
![[Pasted image 20231018175431.png]]
删除wait()后：
![[Pasted image 20231018193046.png]]
若将wait()移至else语句开头处：
![[Pasted image 20231018193346.png]]
- 结果分析
以图一为例，父进程中的pid为fork()返回值，返回值为子进程的pid值，故进入最后一个判断语句，pid1为父进程自身pid值。而子进程中fork()返回值为0，故进入第二个判断语句，pid1为子进程自身pid值。由于两个进程运行由系统调度，故两个进程的执行顺序并不确定，可能产生图示的交错输出结果。此处删除wait()后没有明显区别。
若将wait()移至前面，则父进程会等待子进程执行结束后再继续运行。

#### 1.1.2
- 代码实现
增加value全局变量，在子进程中执行+2，父进程中执行-2
![[Pasted image 20231018210941.png]]
- 输出结果
![[Pasted image 20231018210920.png]]
- 结果分析
两个进程同一变量的地址输出是一样的，但value数值不一样，因为子进程创建时对父进程进行了一份完全复制，包括父进程中变量的虚拟地址号，但是两个进程使用不同的物理地址，即程序中输出的一样的地址实际对应不同的物理地址空间，故value实际存在于两个不同的虚拟地址中，在不同的进程中操作也会导致value的值不同。

#### 1.1.3
- 代码实现
![[Pasted image 20231018212106.png]]
- 输出结果
![[Pasted image 20231018212047.png]]
- 结果分析
return前对全局变量操作，仍然是在当前进程中，各自进程中操作的是各自进程的value值，所以父进程中对value依次执行减2，加1的操作，最终输出结果为-1，而子进程中依次执行加2，加1的操作，最终结果为3

#### 1.1.5
- 代码实现
1. 调用`system()`函数
![[Pasted image 20231019224353.png]]
2. 调用`exec()`族函数
![[Pasted image 20231020193013.png]]
3. `system call`程序代码
![[Pasted image 20231019230430.png]]
- 输出结果
1. 调用`system()`
![[Pasted image 20231019224606.png]]
2. 调用`exec()`族函数
![[Pasted image 20231020193048.png]]
- 结果分析
父进程的pid值为3822，父进程中用fork()创建了一个子进程，子进程pid为3823，在子进程中执行system()内语句，语句中调用./system_call的可执行文件，而system_call创建了一个新进程，并输出进程的pid值，故输出的pid值为其新创建的进程的pid值，为3824，该语句并不影响原进程的执行，原进程仍会继续输出自身pid值，即3823。
若调用exec族函数，在原进程内部执行`system_call`可执行文件，则原进程被exec函数所调用的程序所替代，所以不会继续执行子进程该语句的下一条语句，而原进程的pid等值并不发生改变，只有进程内部则被替换，故`system_call`输出的pid值与原子进程的pid值一样。

### 1.2线程
#### 1.2.1
线程编译指令
`gcc thread.c -o thread -lpthread`
- 代码实现
![[Pasted image 20231020202241.png]]
- 输出结果
![[Pasted image 20231020202405.png]]
- 结果说明
实验创建了两个线程，分别对value的值进行+100和-100的操作，循环100000次，由于程序多线程运行，当两个线程同时对全局变量value的值进行写入操作时，不能保证一个线程写入的value值是另一个线程修改后的，可能造成出错。

### 1.2.2
- 代码实现
```c
em_t s;
int main()
{
        sem_init(&s, 0, 1);
        pthread_t pid1, pid2;
        int res;
        res = pthread_create(&pid1, NULL, thread_add, NULL);
        if(res){
                printf("create failed!\n");
                return 0;
        }else printf("thread1 create success!\n");
        res = pthread_create(&pid2, NULL, thread_sub, NULL);
        if(res){
                printf("create failed!\n");
                return 0;
        }else printf("thread2 create success!\n");
        pthread_join(pid1, NULL);
        pthread_join(pid2, NULL);
        printf("value = %d\n", value);
        return 0;
}
void* thread_add()
{
        for(int i = 0; i < 100000; i++)
        {
                sem_wait(&s);
                value +=100;
                sem_post(&s);
        }
        return 0;
}
void* thread_sub()
{
        for(int i = 0; i < 100000; i++)
        {
                sem_wait(&s);
                value -= 100;
                sem_post(&s);
        }
        return 0;
}
```

- 输出结果
![[Pasted image 20231020205740.png]]
- 结果解释
程序使用了全局变量s作为信号量，使用sem_wait()实现P操作，使用sem_post()实现V操作，在线程进入临界区修改共享变量时执行P操作，退出时执行V操作，这两个函数是原子语句。

### 1.2.3
- 代码实现
1. system()函数
![[Pasted image 20231022113114.png]]
2. exec()族
![[Pasted image 20231022115104.png]]
- 输出结果
![[Pasted image 20231022113032.png]]
2. exec()族
![[Pasted image 20231022115004.png]]

- 结果解释
在父进程中创建两个线程，在线程中输出自身的tid值和pid值，每个线程都有自己的一份tid，故两者输出不同的tid值，而由于这两个线程都由同一进程创建，故调用getpid函数后，得到的是创建这两个线程的进程的pid值，在两个线程中分别执行system_call可执行文件，每个system_call程序都创建一个进程，输出自身进程pid值。而system调用并不影响原进程和线程的执行，故会执行到线程函数返回。
而当system调用换为exec族函数后，执行system_call调用，原进程的内容将完全被更换为system_call中的程序，故在system_call中输出的pid值与原进程的pid值完全一致。在进程被替换后，进程创建的所有线程也被销毁，因此执行该语句的线程无法执行后续内容。从输出中可以见到，线程2刚创建完甚至还没来得及输出其tid值，线程1便已经执行了exec族函数使所有线程被销毁，此时线程2已经没有继续执行其后续语句的机会了。

### 1.3 自旋锁
- 代码实现

![[Pasted image 20231022151051.png]]
![[Pasted image 20231022151248.png]]
- 输出结果
1. 线程未上锁
![[Pasted image 20231022150804.png]]
![[Pasted image 20231022150918.png]]
2. 线程上锁
![[Pasted image 20231022151201.png]]
- 结果分析
程序定义了一个全局变量作为共享变量，后分别创建两个线程对共享变量进行+1操作，假如没有在操作共享变量前加锁，那么在大量的操作下，两个线程有可能同时对变量进行操作，导致一个线程的操作被另一个操作覆盖，如输出结果1。
此程序定义了结构体作为自旋锁标志位，通过上锁函数令自旋锁在未上锁时上锁并操作共享变量，而当标志位已被置1上锁则进入自旋状态，直到标志被置0解锁才退出自旋，继续执行程序。