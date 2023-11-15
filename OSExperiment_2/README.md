# 一、进程软中断通信

- 代码实现
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <signal.h>
int flag = 0;
void inter_handler(int Signal) {
    flag = 0;
    printf("\n%d stop test!\n", Signal);
    return;
}
void waiting()
{
    while(flag);
    return;
}
int main() {
    pid_t pid1=-1, pid2=-1;
    signal(3, inter_handler);
    while (pid1 == -1) pid1 = fork();
    if (pid1 > 0) {
        while (pid2 == -1) pid2 = fork();
        if (pid2 > 0) {
            // TODO: 父进程
            flag = 1;
            sleep(5);
            kill(pid1, 16);
            wait(0);
            kill(pid2, 17);
            wait(0);
            printf("\nParent process is killed!!\n");
            exit(0);
        } 
        else {
            // TODO: 子进程 2
            signal(3, SIG_IGN);
            flag = 1;
            signal(17, inter_handler);
	        waiting();
            printf("\nChild process2 is killed by parent!!\n");
            return 0;
        }
    } 
    else {
        // TODO：子进程 1
        signal(3, SIG_IGN);
        flag = 1;
        signal(16, inter_handler);
	    waiting();
        printf("\nChild process1 is killed by parent!!\n");
        return 0;
    }
    return 0;
}
```

加入alarm闹钟中断后：
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <signal.h>
int flag = 0;
void inter_handler(int Signal) {
    flag = 0;
    printf("\n%d stop test!\n", Signal);
    return;
}
void waiting()
{
    while(flag);
    return;
}
int main() {
    pid_t pid1=-1, pid2=-1;
    signal(3, inter_handler);
    signal(SIGALRM, inter_handler);
    while (pid1 == -1) pid1 = fork();
    if (pid1 > 0) {
        while (pid2 == -1) pid2 = fork();
        if (pid2 > 0) {
            // TODO: 父进程
            flag = 1;
	    int clk = alarm(5);
	    while(flag);
            kill(pid1, 16);
            wait(0);
            kill(pid2, 17);
            wait(0);
            printf("\nParent process is killed!!\n");
            exit(0);
        } 
        else {
            // TODO: 子进程 2
            signal(3, SIG_IGN);
            flag = 1;
            signal(17, inter_handler);
	    waiting();
            printf("\nChild process2 is killed by parent!!\n");
            return 0;
        }
    } 
    else {
        // TODO：子进程 1
        signal(3, SIG_IGN);
        flag = 1;
        signal(16, inter_handler);
	waiting();
        printf("\nChild process1 is killed by parent!!\n");
        return 0;
    }
    return 0;
}
```

- 运行结果

1. 5s内按下`ctrl+\` 
![[Pasted image 20231106231038.png]]
2. 5s后自动结束进程
![[Pasted image 20231106231120.png]]

3. 闹钟中断
5s后自动退出：
![[Pasted image 20231111174951.png]]

- 结果分析：
	- 为了确保子进程接收到信号，在子进程中需要先将标志位置1，然后调用signal命令等待父进程发来的信号，因为父进程中的signal信号为所有进程共同使用，因此在此之前需要调用signal中的SIG_IGN屏蔽掉我们从键盘中输入的信号，保证其接收到的信号来自父进程。在signal后调用waiting函数，进入自旋状态，等待父进程的信号，当标志位恢复0后退出自旋状态，结束进程。
	
	- 最初认为的结果：按下`ctrl+\`后，父进程，进程1，进程2先后结束；若等待5s后，不等按下该键，其自动结束。
	
	- 实际结果：与预想中结果一致。父进程先接收到信号3，子进程1接收到信号16后终止，子进程2接收到信号17后终止，最后父进程被杀死。现象产生原因：5s内按下，此时父进程接收到了信号3，于是向两个子进程分别发送信号16和17，在子进程均终止后，父进程最后结束，程序终止。若5s未按下，父进程未收到信号，便往后继续执行向两个子进程传送信号的命令，子进程在收到信号后终止，父进程也在所有进程执行完毕后终止。
	
	- 改为闹钟中断后：5s内按下`Ctrl + \`与之前相同，不同的是5s后自动退出时，父进程接收到了信号14，说明接收到闹钟信号，结束自旋状态，向子进程发送16和17信号，使各个进程结束。
	- kill命令使用了两次，分别向子进程1和2发送16和17信号，执行后子进程接收到来自父进程的信号并结束。
	- 进程调用return函数和exit函数都可以主动退出，而kill是强制退出。主动退出比较好，如果用kill命令杀死某个子进程而其父进程没有调用wait函数等待，则该子进程为处于僵死状态导致资源占用，并且kill语句可能导致部分语句未执行而退出，程序运行不稳定；而主动退出可以等待必要的语句执行结束后退出，程序更加稳定。

---
# 二、管道通信

- 代码实现
```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <signal.h>
#include <stdio.h>
int pid1,pid2; // 定义两个进程变量
int main( ) 
{
    int fd[2];
    char InPipe[5000]; // 定义读缓冲区
    char c1='1', c2='2';
    pipe(fd); // 创建管道
    while((pid1 = fork( )) == -1); // 如果进程 1 创建不成功,则空循环
    if(pid1 == 0) { // 如果子进程 1 创建成功,pid1 为进程号
        lockf(fd[1], 1, 0); // 锁定管道
        for(int i = 0; i < 2000; i++)
        {
            write(fd[1], &c1, 1);
        }
         // 分 2000 次每次向管道写入字符’1’
        sleep(5); // 等待读进程读出数据
        lockf(fd[1], 0, 0); // 解除管道的锁定
        exit(0); // 结束进程 1
    }
    else {
        while((pid2 = fork()) == -1); // 若进程 2 创建不成功,则空循环
        if(pid2 == 0) {
            lockf(fd[1],1,0);
            for(int i = 0; i < 2000; i++)
            {
                write(fd[1], &c2, 1);
            } // 分 2000 次每次向管道写入字符’2’
            sleep(5);
            lockf(fd[1],0,0);
            exit(0);
        }
        else {
            wait(0); // 等待子进程 1 结束
            wait(0); // 等待子进程 2 结束
            lockf(fd[0], 1, 0);
            read(fd[0], InPipe, 4000); // 从管道中读出 4000 个字符
            InPipe[4000] = '\0'; // 加字符串结束符
            lockf(fd[0], 0, 0);
            printf("%s\n",InPipe); // 显示读出的数据
            exit(0); // 父进程结束
        }
    }
}

```

- 运行结果
![[Pasted image 20231112161146.png]]

若不加锁控制同步互斥：
![[Pasted image 20231112163139.png]]

- 结果分析：
	- 每个进程各自有不同的用户地址空间,任何一个进程的全局变量在另一个进程中都看不到，所以进程之间要交换数据必须通过内核,在内核中开辟一块缓冲区,进程A把数据从用户空间拷到内核缓冲区,进程B再从内核缓冲区把数据读走,内核提供的这种机制称为进程间通信。
	
	- 代码中创建了两个子进程，并定义了一个匿名管道进行多个进程之间的通信。子进程1和子进程2分别向管道中写入2000个‘1’和‘2’，在进程1对管道执行写入操作前，进程1对管道进行了上锁，然后向管道中写入2000个'1'，并等待进程读出数据，后解除管道的锁定状态。进程2在写入数据前也会对管道上锁，实现方法与进程1相同，等两个进程结束后，父进程对管道的读出端上锁，并从管道中读出4000个字符，并输出，可以看到结果是输出2000个‘1’后输出2000个'2'。当然由于进程的执行顺序不确定，可能会出现进程2比进程1先执行的情况，此时便会先输出2000个‘2’再输出2000个'1'。
	
	- 若未进行同步互斥操作，两个进程对管道的写入会存在竞争关系，因此输出结果中的1和2顺序不固定，1和2交替输出。

---
# 三、内存的分配与回收

- 代码实现
```c
#include <stdio.h>
#include <stdlib.h>

#define PROCESS_NAME_LEN 32     /*进程名长度*/
#define MIN_SLICE 10            /*最小碎片的大小*/
#define DEFAULT_MEM_SIZE 1024   /*内存大小*/
#define DEFAULT_MEM_START 0     /*起始位置*/
/* 内存分配算法 */
#define MA_FF 1
#define MA_BF 2
#define MA_WF 3

/*描述每一个空闲块的数据结构*/
struct free_block_type{
    int size;
    int start_addr;
    struct free_block_type *next;
};

/*每个进程分配到的内存块的描述*/
struct allocated_block{
    int pid;
    int size;
    int start_addr;
    char process_name[PROCESS_NAME_LEN];
    struct allocated_block *next;
};

int mem_size=DEFAULT_MEM_SIZE;  /*内存大小*/
int ma_algorithm = MA_FF;       /*当前分配算法*/
static int pid = 0;             /*初始pid*/
int flag = 0;                   /*内存设置标志位*/
int num_free = 0;
/*指向内存中空闲块链表的首指针*/
struct free_block_type *free_block = NULL;

/*进程分配内存块链表的首指针*/
struct allocated_block *allocated_block_head = NULL;

/*设置内存大小标志*/
/*初始化空闲块，默认为一块，可以指定大小及起始地址*/
struct free_block_type* init_free_block(int mem_size){
    struct free_block_type *fb;

    fb=(struct free_block_type *)malloc(sizeof(struct free_block_type));
    if(fb==NULL){
        printf("No mem\n");
        return NULL;
    }
    fb->size = mem_size;
    fb->start_addr = DEFAULT_MEM_START;
    fb->next = NULL;
    num_free = 1;
    return fb;
};

void rearrange(int algorithm);
void rearrange_FF();
void rearrange_BF();
void rearrange_WF();
int free_mem(struct allocated_block *ab);
int dispose(struct allocated_block *free_ab);
int allocate_mem(struct allocated_block *ab);

/*显示菜单*/
void display_menu(){
    printf("\n");
    printf("1 - Set memory size (default=%d)\n", DEFAULT_MEM_SIZE);
    printf("2 - Select memory allocation algorithm\n");
    printf("3 - New process \n");
    printf("4 - Terminate a process \n");
    printf("5 - Display memory usage \n");
    printf("0 - Exit\n");
}

/*设置内存的大小*/
int set_mem_size(){
    int size;
    if(flag!=0){  //防止重复设置
        printf("Cannot set memory size again\n");
        return 0;
    }
    printf("Total memory size =");
    scanf("%d", &size);
    if(size>0) {
        mem_size = size;
        free_block->size = mem_size;
    }
    flag=1;
    return 1;
}

/* 设置当前的分配算法 */
void set_algorithm(){
    int algorithm;
    printf("\t1 - First Fit\n");
    printf("\t2 - Best Fit\n");
    printf("\t3 - Worst Fit\n");
    scanf("%d", &algorithm);
    if(algorithm>=1 && algorithm <=3)
    {
        ma_algorithm=algorithm;
        //按指定算法重新排列空闲区链表
        rearrange(ma_algorithm); 
    }
}

/*按指定的算法整理内存空闲块链表*/
void rearrange(int algorithm){
    switch(algorithm){
        case MA_FF:  rearrange_FF(); break;
        case MA_BF:  rearrange_BF(); break;
        case MA_WF: rearrange_WF(); break;
    }
}

//首次适应算法
void rearrange_FF()
{//采用选择排序方法将空闲链表按地址从小到大排序
    //min_addr->next为最小结点
    struct free_block_type* min_addr = free_block;
    //frt用于标志已完成排序的结点，pref为frt前序结点，pre为mov前序结点，mov用于遍历找地址最小结点
    struct free_block_type* frt, *pref, *pre, *mov;
    if(!free_block) return;
    else if(!free_block->next) return;
    pref = free_block; frt = free_block;pre = free_block;mov = free_block->next;
    while(frt->next)
    {
        min_addr = frt;
        while(mov && min_addr->next)
        {
            if(mov->start_addr < min_addr->next->start_addr)
            {
                min_addr = pre;
                mov = mov->next;
                pre = pre->next;
            }
            else
            {
                pre = pre->next;
                mov = mov->next;
            }
        }
        if(frt->start_addr < min_addr->next->start_addr)
        {
            if(frt != free_block) pref = frt;
            frt = frt->next;
            pre = frt;
            if(frt->next) mov = frt->next;
            continue;
        }
        mov = min_addr->next;
        min_addr->next = mov->next;
        mov->next = frt;
        if(frt == free_block)
        {
            free_block = mov;
            pref = free_block;
        }
        
        else pref->next = mov;
        mov = frt->next;
        pre = frt;
    }
}

//最优适应算法
void rearrange_BF()
{
    struct free_block_type* min_size = free_block;
    struct free_block_type *frt, *pref, *pre, *mov;
    if(!free_block) return;
    else if(!free_block->next) return;
    pref = free_block; frt = free_block;pre = free_block;mov = free_block->next;
    while(frt->next)
    {
        min_size = frt;
        while(mov&& min_size->next)
        {
            if(mov->size < min_size->next->size)
            {
                min_size = pre;
                mov = mov->next;
                pre = pre->next;
            }
            else
            {
                pre = pre->next;
                mov = mov->next;
            }
        }
        if(frt->size < min_size->next->size)
        {
            if(frt != free_block) pref = frt;
            frt = frt->next;
            pre = frt;
            if(frt->next) mov = frt->next;
            continue;
        }
        mov = min_size->next;
        min_size->next = mov->next;
        mov->next = frt;
        if(frt == free_block)
        {
            free_block = mov;
            pref = free_block;
        }
        else pref->next = mov;
        mov = frt->next;
        pre = frt;
    }
}

//最差适应算法
void rearrange_WF()
{
    struct free_block_type* max_size = free_block;
    struct free_block_type *frt, *pref, *pre, *mov;
    if(!free_block) return;
    else if(!free_block->next) return;
    pref = free_block; frt = free_block;pre = free_block;mov = free_block->next;
    while(frt->next)
    {
        max_size = frt;
        while(mov && max_size->next)
        {
            if(mov->size > max_size->next->size)
            {
                max_size = pre;
                mov = mov->next;
                pre = pre->next;
            }
            else
            {
                pre = pre->next;
                mov = mov->next;
            }
        }
        if(frt->size > max_size->next->size)
        {
            if(frt != free_block) pref = frt;
            frt = frt->next;
            pre = frt;
            if(frt->next) mov = frt->next;
            continue;
        }
        mov = max_size->next;
        max_size->next = mov->next;
        mov->next = frt;
        if(frt == free_block)
        {
            free_block = mov;
            pref = free_block;
        }
        else pref->next = mov;
        mov = frt->next;
        pre = frt;
    }
}


/*创建新的进程，主要是获取内存的申请数量*/
int new_process(){
    struct allocated_block *ab;
    int size;
    int ret;
    ab=(struct allocated_block *)malloc(sizeof(struct allocated_block));
    if(!ab) exit(-5);
    ab->next = NULL;
    pid++;
    sprintf(ab->process_name, "PROCESS-%02d", pid);
    ab->pid = pid;    
    printf("Memory for %s:", ab->process_name);
    scanf("%d", &size);
    if(size>0) ab->size = size;
    ret = allocate_mem(ab);  /* 从空闲区分配内存，ret==1表示分配ok*/
    /*如果此时allocated_block_head尚未赋值，则赋值*/
    if((ret==1) &&(allocated_block_head == NULL)){ 
        allocated_block_head=ab;
        return 1;
    }
    /*分配成功，将该已分配块的描述插入已分配链表*/
    else if (ret == 1) {
        ab->next = allocated_block_head;
        allocated_block_head = ab;
        return 2;
    }
    else if(ret == -1){ /*分配不成功*/
        printf("Allocation fail\n");
        free(ab);
        return -1;       
     }
    return 3;
}

/*分配内存模块*/
int allocate_mem(struct allocated_block *ab){
    struct free_block_type *fbt, *pre;
    int request_size=ab->size;
    fbt = free_block;
    pre = free_block;
    int isAllocated = -1;
    while(fbt)
    {
        if(fbt->size >= request_size && fbt->size - request_size <= MIN_SLICE)
        {//当找到的空闲区满足条件且剩余空间较小，则全部将其分配
            ab->start_addr = fbt->start_addr;
            ab->size = fbt->size;
            if(fbt == free_block) free_block = fbt->next;
            else
            {
                pre->next = fbt->next;
                if(fbt) free(fbt);
                num_free--;
            }
            isAllocated = 1;
            break;
        }
        else if(fbt->size >= request_size && fbt->size - request_size > MIN_SLICE)
        {//找到的空闲区满足条件而剩余空间较大，则分割后进行分配
            ab->start_addr = fbt->start_addr;
            fbt->size = fbt->size - request_size;
            fbt->start_addr = fbt->start_addr + request_size;
            isAllocated = 1;
            break;
        }
        if(fbt == free_block) fbt = fbt->next;
        else
        {
            fbt = fbt->next;
            pre = pre->next;
        }
    }
    if(isAllocated == -1)
    {   //若未找到合适的空闲区，判断剩余空间总和是否可用，可用则进行紧缩
        int free_sum = 0;
        fbt = free_block;
        while(fbt)
        {
            free_sum += fbt->size;
            fbt=fbt->next;
        }
        if(free_sum >= request_size)
        {//剩余空间总和满足需求，进行紧缩
            int start_temp = 0;
            struct allocated_block* p = allocated_block_head;
            int alloc_sum = 0;
            while(p)
            {//将分配好的进程全部移至内存前面
                p->start_addr = start_temp;
                start_temp = p->start_addr + p->size;
                alloc_sum += p->size;
                p = p->next;
            }
            fbt = free_block->next;
            pre = free_block;
            while(pre) 
            {//删除整个空闲分区链
                if(pre) free(pre);
                pre = fbt;
                if(fbt) fbt = fbt->next;//避免fbt为空
            }
            //剩余空间合并成一个大空闲分区
            free_block = (struct free_block_type*)malloc(sizeof(struct free_block_type));
            free_block->next = NULL;
            free_block->size = mem_size - start_temp;
            free_block->start_addr = start_temp;
            num_free = 1;
            isAllocated = allocate_mem(ab);
        }
        if(num_free == 0) free_block = NULL;
    }

    if(isAllocated == 1)
    {//分配好后按照相应算法排序
        rearrange(ma_algorithm);
    }
    return isAllocated;
}

struct allocated_block* find_process(int pid)
{
    struct allocated_block * s = allocated_block_head;
    while(s)
    {
        if(s->pid == pid) return s;
        else s = s->next;
    }
    return NULL;
}

/*删除进程，归还分配的存储空间，并删除描述该进程内存分配的节点*/
void kill_process(){
    struct allocated_block *ab;
    int pid;
    printf("Kill Process, pid=");
    scanf("%d", &pid);
    ab=find_process(pid);
    if(ab!=NULL){
        free_mem(ab); /*释放ab所表示的分配区*/
        dispose(ab);  /*释放ab数据结构节点*/
    }
}

/*将ab所表示的已分配区归还，并进行可能的合并*/
int free_mem(struct allocated_block *ab){
    struct free_block_type *fbt, *pre;
    fbt = (struct free_block_type*)malloc(sizeof(struct free_block_type));
    if(!fbt) return -1;
    //新结点插入到表头
    fbt->next = free_block;
    free_block = fbt;
    fbt->size = ab->size;
    fbt->start_addr = ab->start_addr;
    num_free++;
    rearrange_FF();//按地址大小有序排列
    pre = free_block;
    while(pre)
    {//遍历一遍链表，进行可能的合并
        if(pre->next != fbt && fbt != free_block)
        {
            pre = pre->next;
            continue;
        }
        if(fbt->next)
        {
            if(pre->start_addr + pre->size == fbt->start_addr && fbt->start_addr + fbt->size == fbt->next->start_addr)
            {//前后均与新加入结点相邻，则将三个分区合并
                pre->size = pre->size + fbt->size + fbt->next->size;
                pre->next = fbt->next->next;
                free(fbt->next);
                free(fbt);
                num_free -= 2;
                break;
            }
            else if(fbt->next->start_addr == fbt->size + fbt->start_addr)
            {//只有新节点后面的分区相邻，进行合并
                fbt->size = fbt->size + fbt->next->size;
                struct free_block_type *temp = fbt->next;
                fbt->next = fbt->next->next;
                free(temp);
                num_free--;
                break;
            }
        }
        if(pre->start_addr + pre->size == fbt->start_addr)
        {//只有前面的分区与新节点相邻，进行合并
            if(fbt->next) {if(fbt->start_addr + fbt->size == fbt->next->start_addr) break;}
            pre->size = pre->size + fbt->size;
            pre->next = fbt->next;
            free(fbt);
            num_free--;
            break;
        }
        break;
    }
    rearrange(ma_algorithm);//按相应算法排列
    return 1;
}

/*释放ab数据结构节点*/
int dispose(struct allocated_block *free_ab){
    struct allocated_block *pre, *ab;
    if(free_ab == allocated_block_head) { /*如果要释放第一个节点*/
        allocated_block_head = allocated_block_head->next;
        free(free_ab);
        return 1;
    }
    pre = allocated_block_head;  
    ab = allocated_block_head->next;
    while(ab!=free_ab){
        pre = ab;
        ab = ab->next;
    }
    pre->next = ab->next;
    free(ab);
    return 2;
}

/* 显示当前内存的使用情况，包括空闲区的情况和已经分配的情况 */
int display_mem_usage(){
    struct free_block_type *fbt=free_block;
    struct allocated_block *ab=allocated_block_head;

    printf("----------------------------------------------------------\n");

    /* 显示已分配区 */
    printf("Used Memory:\n");
    printf("%10s %20s %10s %10s\n", "PID", "ProcessName", "start_addr", " size");
    while(ab!=NULL){
        printf("%10d %20s %10d %10d\n", ab->pid, ab->process_name, ab->start_addr, ab->size);
        ab=ab->next;
    }
    if(fbt==NULL){
        printf("\nNo more memory can be used!\n");
        printf("----------------------------------------------------------\n");
        return(-1);
    } 
    /* 显示空闲区 */
    printf("\nFree Memory:\n");
    printf("%20s %20s\n", "      start_addr", "       size");
    while(fbt!=NULL){
        printf("%20d %20d\n", fbt->start_addr, fbt->size);
        fbt=fbt->next;
    }    

    printf("----------------------------------------------------------\n");
    return 0;
}

void do_exit()
{
    struct allocated_block* s = allocated_block_head;
    struct allocated_block* p = s; 
    while(s)
    {
        if(p) s = p->next;
        free(p);
        if(s) p = s;
    }
    struct free_block_type* sf = free_block;
    struct free_block_type* pf = sf; 
    while(sf)
    {
        if(pf) sf = pf->next;
        free(pf);
        if(sf) pf = sf;
    }
}

int main(){
    char choice;
    pid=0;
    free_block = init_free_block(mem_size); //初始化空闲区
    while(1) {
        display_menu();	//显示菜单
        fflush(stdin);
        choice=getchar();	//获取用户输入
        switch(choice){
            case '1': set_mem_size(); break; 	//设置内存大小
            case '2': set_algorithm(); flag=1; break;//设置算法
            case '3': new_process(); flag=1; break;//创建新进程
            case '4': kill_process(); flag=1;   break;//删除进程
            case '5': display_mem_usage(); flag=1; break;	//显示内存使用
            case '0': do_exit(); exit(0);	//释放链表并退出
            default: break;
        }
    } 
}
```

- 运行结果

1. 首次适应
![[Pasted image 20231114224916.png]]

申请多个进程后如下图所示
![[Pasted image 20231114224958.png]]

杀死1进程再杀死3进程后
![[Pasted image 20231114231601.png]]

此时申请一个大小为50的进程：
![[Pasted image 20231114225316.png]]

再申请一个200的进程
![[Pasted image 20231114225349.png]]

再申请一个150的进程
![[Pasted image 20231114225413.png]]

原本的空闲分区只有一个50和一个100的两个分区块，此时申请一个150大小的进程，若直接进行分配显然无法满足，但可以看见输出的结果中，该进程成功申请到了一个150的内存空间，并且之前的进程的内存起始地址都有所改变，这是因为算法中进行了紧缩操作，将所有的空闲分区移动至内存后面部分，并合并成一个大分区，此时便产生了一个150的空闲分区，可以分配给该进程。

最后把所有进程杀死
![[Pasted image 20231114225439.png]]
与初始大小相同

2. 最优适应
![[Pasted image 20231114225641.png]]

新建多个进程，同首次适应
![[Pasted image 20231114224958.png]]

依次把1号和3号进程杀死后
![[Pasted image 20231114225813.png]]

新建一个250的进程后
![[Pasted image 20231114225932.png]]

此时若新建一个大小为50的进程，则结果如下：
![[Pasted image 20231114230059.png]]

全部进程杀死后同最初的内存大小
 
3. 最差适应
![[Pasted image 20231114230204.png]]

新建多个进程同首次适应，此时所有内存已被分配结束
依次删除1号和3号进程后
![[Pasted image 20231114230315.png]]

此时新建一个大小为50的进程，结果如下：
![[Pasted image 20231114230349.png]]
再分配一个大小200的进程后
![[Pasted image 20231114230452.png]]

此时再请求分配一个大小为150的进程
![[Pasted image 20231114230522.png]]

所有进程杀死后内存空间大小同最初大小

- 结果分析
	- 最初的空闲区只有一个大小为内存大小的结点，随着进程的分配和释放将会出现多个孔，因此产生多个空闲区结点。
	
	- 对于首次适应算法，每次进程被杀死后都对空闲区链表进行排序，按照地址从小到大排序，分配时对链表进行遍历，将找到的第一个合适的空间进行分配，如结果中显示，当有两个大小为100和300 的空间时，申请50的进程优先分配在最先找到的结点，由于链表已经按地址从小到大排序，最先找到的结点即为地址最小的结点。
	
	- 对于最优适应算法，每次进程被杀死后将空闲区链表按照内存从小到大排序，如在结果中，有两个空闲区的大小分别为50和100，此时若分配大小为50的进程，必然优先分配在大小为50的空闲区结点中，不论这个结点的内存地址在后面或前面，由于链表按照内存从小到大排序，因此此时首次找到的合适的结点即为最小满足的空闲区结点。
	
	- 对于最差适应算法，与最优适应算法相反，将空闲区链表保持在按照内存的从大到小排序，如在结果中，空闲区链表只有两个大小分别为300和100的空闲区结点，分配大小为50的进程时，优先分配在大小为300的结点中，尽管这个结点在内存中的位置较为靠后，分配完后此时剩余空间较大，因此需要进行分割，原来的结点大小变为250，由于链表按照内存从大到小排序，因此此时首次找到的合适的结点即为最大可满足的空闲区结点。
	
	- 内存在分配时需要从头到尾遍历空闲区链表，由于空闲区链表已经按照要求的算法进行有序排列，因此遍历找到的第一个可满足结点即符合要求。在找到符合要求的空闲区结点后，需要比较该结点空间与申请空间的大小，若分配后剩余空间较小，即小于规定的剩余最小碎片大小（MIN_SLICE），则将其全部分配；若剩余空间较大，则对内存空间进行分割，原结点大小变为分配后的大小。若遍历完一遍后未找到合适的空闲区，则还需要判断剩余的所有空闲区大小总和是否满足申请进程的所需空间要求，若剩余的空闲区大小之和满足条件，则进行紧缩操作，然后进行分配；若和不满足，则返回-1。如果已经分配完毕，则需要对空闲区链表进行对应的算法排序。
	
	- 紧缩操作时，先将所有的已分配进程移动至内存前面部分，按照地址从小到大重新分配每个进程的起始地址。然后删除整个空闲链表，将剩余内存空间合并为一个大的空闲分区结点，再进行下一步的分配操作。
	
	- 若需要删除某个进程，需要根据指定的PID号找到对应的进程结点并返回，此时需要对该进程申请的空间进行回收。回收时先新建一个空闲分区结点，将进程释放的分区存入该结点内，而后需要进行可能的合并操作，即将几个内存相邻的分区合并成一个大分区。首先需要先将空闲区链表按照地址的大小进程有序的排列，此时可以调用首次适应算法执行该命令。然后对空闲区链表进行遍历，找到相邻的分区，因为此时只插入一个结点，在结点插入前该链表的所有空闲区必然已经没有相邻空间存在，因此只需要判断该结点是否与其前后两个分区相邻。此时需要分为三种情况，若只与前面的分区相邻，则将前面结点链接至原结点的下一个结点，并修改前面结点的大小后删除原结点；若只与后面的分区相邻，则将原结点大小改为原结点大小与后一个结点大小之和后，将原结点链接至后一个结点的后一个结点，并删除原来的后一个结点；若原结点空间与前后两个结点都相邻，那么可以将前一个结点的大小修改成三个结点大小之和，再将前一个结点链接到原结点的下一个结点的下一个结点，并将原结点和其下一个结点删除。完成可能的合并操作后，需要按照相应的算法对空闲区链表进行排序以满足后续分配要求。
	