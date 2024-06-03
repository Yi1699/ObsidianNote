# Ext2文件系统设计

---
### 一、文件系统数据结构

##### 1.磁盘
创建一个名为Disk的文件用于模拟文件系统所在的磁盘，磁盘大小为$(4096 + 512 + 3)*512$ ,磁盘分为4611个块，每个块的块大小为512KB，第一个块为组描述符，第二个块为块位图，第三个块为索引结点位图，接下来512个块为索引结点表，剩余4096个块为数据块，用于存放数据。

##### 2.组描述符
```c
/*组描述符，占32字节*/
struct ext2_group_desc 
{
    char bg_volume_name[16];    //卷名
    __u16 bg_block_bitmap;      //保存块位图的块号
    __u16 bg_inode_bitmap;      //保存索引结点位图的块号
    __u16 bg_inode_table;       //索引结点表的起始块号
    __u16 bg_free_blocks_count; //本组空闲块的个数
    __u16 bg_free_inodes_count; //本组空闲索引结点的个数
    __u16 bg_used_dirs_count;   //本组目录的个数
    char bg_pad[4];             //填充
};
```
组描述符占32字节，用于存放 本组的各个信息，包括卷名，块位图的块号，索引结点位图的块号，索引结点表的起始块号，空闲块的个数，空闲索引结点的个数，目录的个数等，结构体中bg_pad用于填充，使组的大小满足8的倍数。

##### 3.索引结点
```c
/*索引结点，占64字节*/
struct ext2_inode
{
    __u16 i_mode;       //文件类型及访问权限
    __u16 i_blocks;     //文件的数据块个数
    __u32 i_size;       //大小(字节)
    __u64 i_atime;      //访问时间
    __u64 i_ctime;      //创建时间
    __u64 i_mtime;      //修改时间
    __u64 i_dtime;      //删除时间
    __u16 i_block[8];    //指向数据块的指针
    char i_pad[8];       //填充
};
```
一个索引结点即代表一个文件，索引结点存储文件的各种信息，包括文件类型、权限、数据块个数、大小等。

##### 4.目录项
```c
/*目录项*/
struct ext2_dir_entry
{
    __u16 inode;                            //索引节点号
    __u16 rec_len;                          //目录项长度
    __u8 name_len;                          //文件名长度
    __u8 file_type;                         //文件类型(1:普通文件，2:目录…)
    char name[NAME_LEN];  //文件名
};
```
目录作为一种特殊的文件，目录体中存放位于当前目录中所有文件的目录项，方便用户进行检索，目录项中存放文件的索引结点号，文件类型，文件名等相关信息。

##### 5.底层函数实现
```c
int initialize_disk();/*初始化文件系统*/
void initialize_memory();	 /*初始化文件系统的内存数据*/
void update_group_desc(); /*将内存中的组描述符更新到"硬盘".*/
int ext2_new_inode(__u16 mode, int dir_offset, const char* name); /*分配一个新的索引结点*/
int ext2_alloc_block(__u16 inode); /*分配一个新的数据块*/
void ext2_free_inode(__u16 inode); /*释放特定的索引结点*/
void ext2_free_block_bitmap(__u16 block_num); /*释放特定块号的数据块位图*/
void ext2_free_inode_bitmap(__u16 inode_num); /*释放指定的索引结点位图*/
void ext2_free_blocks(__u16 inode_num);	/*释放特定文件的所有数据块*/
int search_file_entry_off(const char* name, __u8 type, unsigned short temp_curr_dir);	/*在当前目录中查找文件*/
int test_fd(int number);/*检测文件打开ID(fd)是否有效*/
int find_block_addr(unsigned short node); /*给定索引结点号，返回索引结点在磁盘的偏移量*/
int find_inode();//找空闲结点并更新索引结点位图与组描述符
int find_block();//找空数据块，返回空数据块块号
const char* rw_change(__u16 mode);//将索引结点中的mode转换为rwx字符
__u16 type_to_rw(__u16 mode);//提取索引结点的用户权限代码
const char* view_type(__u8 type);//将文件类型代码转换为字符串
```
这些函数提供了文件系统的初始化等基础功能，用于完成文件系统的底层的一些基础操作，为用户层的函数提供服务。

##### 6.用户层函数实现
```c
/*用户调用接口*/
int mkdir(const char* name);//当前目录新建一个文件夹
int create(const char* name, __u16 mode);//当前目录新建文件，参数为文件名，文件类型代码，默认权限为rwx
int delete_file(const char* name, __u8 type, unsigned short temp_curr_dir);//当前目录下删除指定文件，参数为文件名，文件类型代码，以及当前目录索引结点号
int cd(const char* name);//进入某个目录，正确进入返回1，否则返回0或-1.
int attrib(const char* name, __u16 rw, __u8);//修改文件权限
int open(const char* filename, __u8 type);//打开文件
int close(int fd);//关闭文件
int read(int fd);//读文件
int write(int fd, const char* sources);//向指定文件写入字符串
void shell();//启动用户接口
void format();//重新建立文件系统,无参数
void ls();//展示当前目录所有文件
void login();//登录用户
void password();//修改密码
```
这些函数为用户对文件系统进行相关操作提供了函数接口，并提供了一个shell()函数用于识别用户输入的命令，并调用这些函数接口，完成相关操作。

##### 7.常驻内存的相关变量
```c
unsigned short fopen_table[16] ; /*文件打开表，最多可以同时打开16个文件*/
unsigned short last_alloc_inode; /*上次分配的索引结点号*/
unsigned short last_alloc_block; /*上次分配的数据块号*/
unsigned short current_dir;  /*当前目录(索引结点）*/
char current_path[256];   	 /*当前路径(字符串) */
struct ext2_group_desc group_desc;	 /*组描述符*/ 
const char* Path = "Disk";//磁盘名
char Username[256] = {"\0"};//用户名
char PassWord[256] = {"\0"};//用户密码
```
这些变量为文件系统在内存中存放的部分信息，包括文件打开表，当前目录等实时信息，以全局变量的形式定义于文件系统中。

### 二、主要功能实现
##### 1.文件系统初始化
（1）实现方法
先新建一个名为Disk的文件用于模拟Ext2文件系统的磁盘，并填充4611 * 512字节的0，代表磁盘容量。后更新组描述符，将第一个块分配给组描述符，并新建一个根目录，将第一个索引结点和第一个数据块分配给根目录，同时修改两个位图，并在根目录下写入.和..两个目录项。后还需要对内存区的变量进行初始化操作。
（2）实现代码
```c
//初始化文件系统
int initialize_disk()	 
{
    FILE * fil;
    fil = fopen(Path, "w+");
    fseek(fil, 0, SEEK_SET);
    __u32 init_array[128] = {0};
    for(int i = 0; i < 4611; i++) fwrite(&init_array, sizeof(init_array), 1, fil);//将磁盘写入0,大小为4611×512
    //更新并写入组描述符
    strcpy(group_desc.bg_volume_name, "Root");
    group_desc.bg_block_bitmap = 1;
    group_desc.bg_inode_bitmap = 2;
    group_desc.bg_inode_table = 3;
    group_desc.bg_free_blocks_count = 4095;//除去一个根目录
    group_desc.bg_free_inodes_count = 4095;
    group_desc.bg_used_dirs_count = 1;//根目录
    strcpy(group_desc.bg_pad, "000");
    fseek(fil, 0, SEEK_SET);
    fwrite(&group_desc, sizeof(ext2_group_desc), 1, fil);
    time_t curr;
    time(&curr);
    //根目录索引结点
    struct ext2_inode root;
    root.i_mode = 0x0207;//0000_0010_0000_0111默认为rwx;
    root.i_blocks = 1;
    root.i_size = 2 * sizeof(ext2_dir_entry);
    root.i_atime = curr;
    root.i_ctime = curr;
    root.i_mtime = curr;
    root.i_dtime = 0;
    for(int i = 0; i < 8; i++) root.i_block[i] = 0;
    fseek(fil, group_desc.bg_inode_table * BLOCK_SIZE, SEEK_SET);
    fwrite(&root, sizeof(ext2_inode), 1, fil);

    struct ext2_dir_entry dir_curr;//当前目录
    dir_curr.inode = 1;//索引结点从1开始计数，数据块从0开始计数
    dir_curr.rec_len = sizeof(ext2_dir_entry);
    dir_curr.name_len = NAME_LEN;
    dir_curr.file_type = 2;
    strcpy(dir_curr.name, ".");
    fseek(fil, 515 * BLOCK_SIZE, SEEK_SET);
    fwrite(&dir_curr, sizeof(ext2_dir_entry), 1, fil);

    struct ext2_dir_entry dir_back;//上级目录
    dir_back.inode = 1;
    dir_back.rec_len = sizeof(ext2_dir_entry);
    dir_back.name_len = NAME_LEN;
    dir_back.file_type = 2;
    strcpy(dir_back.name, "..");
    fseek(fil, 515 * BLOCK_SIZE + 1 * sizeof(ext2_dir_entry), SEEK_SET);
    fwrite(&dir_back, sizeof(ext2_dir_entry), 1, fil);

    fseek(fil, group_desc.bg_block_bitmap * BLOCK_SIZE, SEEK_SET);
    __u32 bitchange = 0x80000000;//修改位图第一位为1
    fwrite(&bitchange, sizeof(__u32), 1, fil);
    fseek(fil, group_desc.bg_inode_bitmap * BLOCK_SIZE, SEEK_SET);
    fwrite(&bitchange, sizeof(__u32), 1, fil);
    fclose(fil);
    return 0;
}

//初始化内存区变量
void initialize_memory()
{
    last_alloc_inode = 1; /*上次分配的索引结点号*/
    last_alloc_block = 0; /*上次分配的数据块号*/
    current_dir = 1;  /*当前目录(索引结点）*/
    for(int i = 0;i < 16; i++) fopen_table[i] = 0;
    strcpy(current_path, "Root/");//当前路径
}
```
##### 2.文件创建相关
##### 1.create()
（1）实现方法
创建文件的用户层函数为create()，参数为文件名，文件类型代码，根据提供的文件名与文件类型首先需要判断当前目录是否可写，不可写则不允许创建。后根据多级索引机制查找判断当前目录下该文件是否存在，存在则不可创建，否则继续查找（允许两个同名不同类型的文件存在），找重名过程中顺便找到空闲目录项的磁盘偏移量，用于后续索引结点的分配。
（2）实现代码
```c
//在当前目录创建一个文件，参数为文件名，文件类型代码
int create(const char* name, __u16 mode)
{
    __u16 modetype = mode >> 8;//提取文件类型代码，除去文件权限
    if(modetype < 0 || modetype > 7)
    { 
        printf("Type Error!");
        return -1;
    }
    char Name[NAME_LEN];
    int name_len = strlen(name);
    if(name_len >= NAME_LEN)
    {//大于最大长度，取前NAME_LEN位作为文件名
        name_len = NAME_LEN;
        for(int i = 0; i < NAME_LEN - 1; i++) Name[i] = name[i];
        Name[NAME_LEN - 1] = '\0';
    }
    else strcpy(Name, name);
    FILE *fil = fopen(Path, "r+");
    fseek(fil, find_block_addr(current_dir) , SEEK_SET);
    ext2_inode currblock;
    fread(&currblock, sizeof(ext2_inode), 1, fil);//读出当前目录的索引结点
    __u16 rw = type_to_rw(currblock.i_mode);
    __u16 a = 0x0002;//0010
    if((rw & a) == 0)
    {//当前目录没有写权限
        printf("Do not have enough right!\n");
        return -1;
    }
    int isNameSame = 0;
    int isFull = 1;
    int new_off = 0;//找到的空闲目录项的磁盘偏移地址
    for(int i = 0; i < 8; i++)
    {//对当前目录下的每个目录项进行遍历，判断有无文件重名，且寻找空闲目录项
        ext2_dir_entry curr_dir_fil;
        if(isNameSame == 1) break;
        if(i <= 5)
        {//直接索引
            for(int j = 0; j < BLOCK_SIZE / sizeof(ext2_dir_entry); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(ext2_dir_entry), SEEK_SET);
                fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                if(curr_dir_fil.inode == 0)
                {//找到空闲目录项
                    new_off = (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(ext2_dir_entry);
                    isFull = 0;
                    break;
                }
                else
                {//判断是否重名，且文件类型相同
                    if(curr_dir_fil.file_type == modetype && strcmp(curr_dir_fil.name, name) == 0)
                    {
                        isNameSame = 1;
                        return isNameSame;
                    }
                }
            }
        }
        else if(i == 6)
        {//一级索引
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                for(int k = 0; k < BLOCK_SIZE / sizeof(ext2_dir_entry); k++)
                {
                    fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                    if(curr_dir_fil.inode == 0)
                    {
                        new_off = (515 + first_block) * BLOCK_SIZE + k * sizeof(ext2_dir_entry);
                        isFull = 0;
                        break;
                    }
                    else
                    {
                        if(curr_dir_fil.file_type == modetype && strcmp(curr_dir_fil.name, name) == 0)
                        {
                            isNameSame = 1;
                            return isNameSame;
                        }
                    }
                }
                if(new_off > 0) break;
            }
        }
        else if(i == 7)
        {
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {//遍历一级索引块中的每一个二级索引块地址
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                for(int k = 0; k < BLOCK_SIZE / sizeof(__u16); k++)
                {//遍历二级索引块中的每个数据块地址
                    __u16 second_block;//二级索引块号
                    fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(__u16), SEEK_SET);
                    fread(&second_block, sizeof(__u16), 1, fil);
                    fseek(fil, (515 + second_block) * BLOCK_SIZE, SEEK_SET);
                    for(int g = 0; g < BLOCK_SIZE / sizeof(ext2_dir_entry); g++)
                    {//对数据块中的每个目录项进行遍历
                        fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                        if(curr_dir_fil.inode == 0)
                        {
                            new_off = (515 + second_block) * BLOCK_SIZE + g * sizeof(ext2_dir_entry);
                            isFull = 0;
                            break;
                        }
                        else
                        {
                            if(curr_dir_fil.file_type == modetype && strcmp(curr_dir_fil.name, name) == 0)
                            {
                                isNameSame = 1;
                                return isNameSame;
                            }
                        }
                    }
                    if(new_off > 0) break;
                }
                if(new_off > 0) break;
            }
        }
        if(new_off > 0) break;
    }
    if(isFull == 1) printf("Error!Can not add more diractory!");//目录项已满
    int new_node_ok = ext2_new_inode(mode, new_off, name);//分配一个新结点
    if(new_node_ok == 1)
    {
        printf("New file %s has been created!You can use it now.\n", Name);
        return 0;
    }

    return isNameSame;
}
```
（3）结果演示
![[Pasted image 20231129111419.png]]
创建多个文件，文件正常创建，ls命令可以显示各个文件相关信息
![[Pasted image 20231129111502.png]]
若出现重名文件，则无法创建
![[Pasted image 20231129111534.png]]
##### 2.ext2_new_inode()
（1）实现方法
底层函数ext2_new_inode()，用于分配索引结点，参数为文件类型，找到的空目录项地址，与文件名。根据提供的文件名与类型，根据不同情况进行判断，如果为目录文件，则在分配索引结点后还需要分配一个新数据块，并新建两个目录项.和..；若为文件则只需要分配索引结点。分配索引结点需要调用find_inode()函数找到空闲的索引结点号再进行分配，在分配索引结点后，需要在当前目录下找到的空闲目录项的地址新建一个该文件的目录项。
（2）实现代码
```c
//分配一个新的索引结点,参数为文件类型，找到的空目录项地址，文件名
int ext2_new_inode(__u16 mode, int dir_offset, const char* name) 
{
    time_t curr;
    time(&curr);//获得当前时间
    ext2_inode new_node;//待分配索引结点
    ext2_dir_entry new_node_dir;//待分配目录项
    __u8 modetype = (__u8)(mode >> 8);
    if(modetype == 2)
    {//类型为目录文件
        new_node.i_mode = mode;
        new_node.i_blocks = 0;
        new_node.i_size = 0;
        new_node.i_atime = curr;
        new_node.i_ctime = curr;
        new_node.i_mtime = curr;
        new_node.i_dtime = 0;
        for(int i = 0; i < 8; i++)
        {
            new_node.i_block[i] = 0;
            new_node.i_pad[i] = 0x00;
        }
        int Inode_find = find_inode();//找到的空闲索引结点号
        if(Inode_find == -1) return -1;
        else new_node_dir.inode = (__u16)Inode_find;
        
        new_node_dir.rec_len = sizeof(ext2_dir_entry);
        new_node_dir.name_len = strlen(name);
        new_node_dir.file_type = modetype;
        strcpy(new_node_dir.name, name);

        FILE * fil = fopen(Path, "r+");

        //在当前目录中插入新增目录项
        fseek(fil, dir_offset, SEEK_SET);
        fwrite(&new_node_dir, sizeof(ext2_dir_entry), 1, fil);

        //创建两个目录项
        ext2_dir_entry dir_curr;//当前目录
        dir_curr.inode = Inode_find;
        dir_curr.rec_len = sizeof(ext2_dir_entry);
        dir_curr.name_len = NAME_LEN;
        dir_curr.file_type = modetype;
        strcpy(dir_curr.name, ".");

        ext2_dir_entry dir_back;//上级目录
        dir_back.inode = current_dir;
        dir_back.rec_len = sizeof(ext2_dir_entry);
        dir_back.name_len = NAME_LEN;
        dir_back.file_type = modetype;
        strcpy(dir_back.name, "..");

        //找到空数据块，并将两个目录项插入该数据块
        int new_block = find_block();
        fseek(fil, (515 + new_block) * BLOCK_SIZE, SEEK_SET);
        fwrite(&dir_curr, sizeof(ext2_dir_entry), 1, fil);
        fwrite(&dir_back, sizeof(ext2_dir_entry), 1, fil);

        //插入新的索引结点
        new_node.i_blocks = 1;
        new_node.i_size = 2 * sizeof(ext2_dir_entry);
        new_node.i_block[0] = new_block;
        fseek(fil, find_block_addr(Inode_find), SEEK_SET);
        fwrite(&new_node, sizeof(ext2_inode), 1, fil);
        fclose(fil);
        return 1;
    }
    else
    {//文件类型非目录
        new_node.i_mode = mode;
        new_node.i_blocks = 0;
        new_node.i_size = 0;
        new_node.i_atime = curr;
        new_node.i_ctime = curr;
        new_node.i_mtime = curr;
        new_node.i_dtime = 0;
        for(int i = 0; i < 8; i++)
        {
            new_node.i_block[i] = 0;
            new_node.i_pad[i] = 0xff;
        }
        int Inode_find = find_inode();
        if(Inode_find == -1) return -1;
        else new_node_dir.inode = (__u16)Inode_find;

        new_node_dir.rec_len = sizeof(ext2_dir_entry);
        new_node_dir.name_len = strlen(name);
        new_node_dir.file_type = modetype;
        strcpy(new_node_dir.name, name);
        FILE * fil = fopen(Path, "r+");
        //当前目录写入新的目录项
        fseek(fil, dir_offset, SEEK_SET);
        fwrite(&new_node_dir, sizeof(ext2_dir_entry), 1, fil);
        //找到空闲索引结点，并插入新的索引结点
        fseek(fil, find_block_addr(Inode_find), SEEK_SET);
        fwrite(&new_node, sizeof(ext2_inode), 1, fil);
        fclose(fil);
        return 1;
    }
    return 0;
}
```
##### 3.find_inode()和find_block()
（1）实现方法
底层函数find_inode()和find_block()用于寻找空闲索引结点和空闲块。以寻找空闲结点为例，先将整张图分成每块4字节的128小块，从上次分配的结点号所在的小块开始，遍历整个位图，循环判断是否有空结点，读取位图时，每次读出四字节大小的__u32类型，利用按位与运算判断哪一位空闲，找到空闲号，用按位或将该位置1，并返回结点号。
（2）实现代码
```c
//找空闲索引结点并更新索引结点位图与组描述符
int find_inode()
{
    __u32 inode_bitmap[128];
    FILE *fp = fopen(Path, "r+");
    fseek(fp, group_desc.bg_inode_bitmap * BLOCK_SIZE, SEEK_SET); // inode 位图
    fread(inode_bitmap, BLOCK_SIZE, 1, fp);     // inode_bitmap保存索引节点位图
    int count = 0;
    int i = last_alloc_inode / 32;//从上次分配的索引结点处开始读出判断
    while(count < BLOCK_SIZE)
    {
        if(i == 127) i = 0;//循环判断是否有空索引
        __u32 j = 0x80000000, k = inode_bitmap[i];
        for (int m = 0; m < 32; m++)
        {
            count++;
            if ((k & j) == 0)//代表该位空闲
            {
                inode_bitmap[i] = inode_bitmap[i] | j;
                group_desc.bg_free_inodes_count -= 1;
                fseek(fp, 0, SEEK_SET);
                fwrite(&group_desc, sizeof(ext2_group_desc), 1, fp);
                fseek(fp, group_desc.bg_inode_bitmap * BLOCK_SIZE, SEEK_SET);
                fwrite(&inode_bitmap, BLOCK_SIZE, 1, fp);//更新inode位图
                fclose(fp);
                last_alloc_inode = i * 32 + m + 1;
                return i * 32 + m + 1;//返回索引节点号，结点号起始为1，故需加1
            }
            else    j = j >> 1;//右移一位
        }
    }
    fclose(fp);
    return -1;
}

//寻找空数据块,返回空数据块的块号
int find_block()
{
    __u32 block_bitmap[128];
    FILE *fp = fopen(Path, "r+");
    fseek(fp, group_desc.bg_block_bitmap * BLOCK_SIZE, SEEK_SET);
    fread(&block_bitmap, BLOCK_SIZE, 1, fp); // block_bitmap保存块位图
    int count = 0;
    int i = last_alloc_block / 32;//自上次分配的数据块开始找
    while(count < BLOCK_SIZE)
    {
        if(i == 127) i = 0;//循环判断是否有空数据块，到头了返回起始避免溢出
        __u32 j = 0x80000000, k = block_bitmap[i];
        for (int m = 0; m < 32; m++)
        {
            count++;
            if ((k & j) == 0)
            {
                block_bitmap[i] = block_bitmap[i] | j;
                group_desc.bg_free_blocks_count -= 1; //块数减 1
                fseek(fp, 0, SEEK_SET);
                fwrite(&group_desc, sizeof(ext2_group_desc), 1, fp);
                fseek(fp, group_desc.bg_block_bitmap * BLOCK_SIZE, SEEK_SET);
                fwrite(&block_bitmap, BLOCK_SIZE, 1, fp);//更新空闲块位图
                fclose(fp);
                last_alloc_block = i * 32 + m;
                return i * 32 + m;//返回块号
            }
            else    j = j >> 1;
        }
    }
    fclose(fp);
    return -1;
}
```
##### 4.ext2_alloc_block()
（1）实现方法
底层函数ext2_alloc_block()，用于分配一个新的数据块，实现时利用多级索引机制进行分配，若为直接索引，则找到的数据块直接放入索引指针，若刚为一级索引，则需要再找一块数据块用于存放一级索引地址，然后将找到的新数据块号放入该块中，二级索引同理。
（2）实现代码
```c
//分配一个新的数据块，返回数据块块号，参数为当前索引结点号
int ext2_alloc_block(__u16 inode)
{
    ext2_inode curr;//当前文件索引结点
    __u16 new_block = find_block();//找到的数据块块号
    FILE *fil = fopen(Path, "r+");
    fseek(fil, find_block_addr(inode), SEEK_SET);
    fread(&curr, sizeof(ext2_inode), 1, fil);
    if(curr.i_blocks >= 6 + ENTRY_NUM + ENTRY_NUM * ENTRY_NUM)
    {//数据块不够
        printf("The file can't be larger! Allocation failed!");
        fclose(fil);
        return -1;
    }
    if(curr.i_blocks < 6)
    {//直接索引
        curr.i_block[curr.i_blocks] = new_block;
        curr.i_blocks++;
    }
    else if(curr.i_blocks - 6 < ENTRY_NUM)
    {//一级索引
        if(curr.i_blocks == 6)
        {
            __u16 first_dir = find_block();//数据块用于存放第一级索引地址
            curr.i_block[6] = first_dir;
        }
        fseek(fil, (515 + curr.i_block[6]) * BLOCK_SIZE + (curr.i_blocks - 6) * sizeof(__u16), SEEK_SET);
        fwrite(&new_block, sizeof(__u16), 1, fil);
        curr.i_blocks++;
    }
    else
    {//二级索引
        int jud = curr.i_blocks - 6 - ENTRY_NUM;
        if(jud == 0)
        {//一级索引已满
            __u16 fir_dir = find_block();
            curr.i_block[7] = fir_dir;
        }
        __u16 sec_dir;
        if(jud % ENTRY_NUM == 0)
        {//二级索引已满
            sec_dir = find_block();
            fseek(fil, (515 + curr.i_block[7]) * BLOCK_SIZE + (jud / ENTRY_NUM) * sizeof(__u16), SEEK_SET);
            fwrite(&sec_dir, sizeof(__u16), 1, fil);
        }
        fseek(fil, (515 + curr.i_block[7]) * BLOCK_SIZE + (jud / ENTRY_NUM) * sizeof(__u16), SEEK_SET);
        fread(&sec_dir, sizeof(__u16), 1, fil);
        fseek(fil, (515 + sec_dir) * BLOCK_SIZE + (jud % ENTRY_NUM) * sizeof(__u16), SEEK_SET);
        fwrite(&new_block, sizeof(__u16), 1, fil);
        curr.i_blocks++;
    }
    fseek(fil, find_block_addr(inode), SEEK_SET);
    fwrite(&curr, sizeof(ext2_inode), 1, fil);
    fclose(fil);
    return new_block;
}
```
##### 5.用户接口shell()
（1）实现方法
读取用户输入命令，并识别命令，调用相关的功能，对于不同的命令，需要输入不同参数以实现相关功能，对于文件的操作部分，由于文件允许同名，因此需要输入文件类型代码加以区分。
（2）实现代码
```c
void shell()/*启动用户接口*/
{
    char Command[16];
    while(1)
    {
        printf("\n%s@Disk: %s ", Username, current_path);
        scanf("%s", Command);
        if(strcmp(Command, "create") == 0)
        {
            char filename[256];
            int ftype;
            scanf("%s", filename);
            printf("Choose the file's type: 0:其他 1:普通文件 2:目录 3:字符设备 4:块设备 5:管道 6:套接字 7:符号指针\n");
            scanf("%d", &ftype);
            __u16 mode = 0x0007;
            switch (ftype)
            {//默认的文件类型与权限
                case 0: break;
                case 1: mode = 0x0106; break;
                case 2: mode = 0x0206; break;
                case 3: mode = 0x0306; break;
                case 4: mode = 0x0406; break;
                case 5: mode = 0x0506; break;
                case 6: mode = 0x0606; break;
                case 7: mode = 0x0706; break;
                default: mode = 0x0f06; break;
            }
            int jud = create(filename, mode);
            if(jud == 1) printf("\nError!The file named%s had been used!Please change another one!\n", filename);
        }
        else if(strcmp(Command, "ls") == 0)
        {
            printf("\n");
            ls();
        }
        else if(strcmp(Command, "mkdir") == 0)
        {
            char filename[256];
            scanf("%s", filename);
            int jud = mkdir(filename);
            if(jud == 1) printf("\nError!The file named %s had been used!Please change another one!\n", filename);
        }
        else if(strcmp(Command, "cd") == 0)
        {
            char filename[256];
            scanf("%s", filename);
            if(strlen(filename) > NAME_LEN) printf("Do not find the directory named \"%s\"\n", filename);
            else
            {
                int jud = cd(filename);
                if(jud == 0) printf("Do not find the directory named \"%s\"\n", filename);
            }
        }
        else if(strcmp(Command, "delete") == 0)
        {
            char filename[256];
            __u8 type;
            scanf("%s%d", filename, &type);
            int jud = delete_file(filename, type, current_dir);
            if(jud == 1) printf("Successfully delete!\n");
        }
        else if(strcmp(Command, "open") == 0)
        {
            char filename[256];
            __u8 type;
            scanf("%s%d", filename, &type);
            if(strlen(filename) > NAME_LEN) printf("Do not find the directory named \"%s\"\n", filename);
            else
            {
                int jud = open(filename, type);
                if(jud >= 0) printf("Open the file successfully\n");
            }
        }
        else if(strcmp(Command, "close") == 0)
        {
            char filename[256];
            __u8 type;
            scanf("%s%d", filename, &type);
            if(strlen(filename) > NAME_LEN) printf("Do not find the directory named \"%s\"\n", filename);
            else
            {
                int find_entry_addr = search_file_entry_off(filename, type, current_dir);
                ext2_dir_entry find_entry;
                FILE* fil;
                fil = fopen(Path, "r+");
                fseek(fil, find_entry_addr, SEEK_SET);
                fread(&find_entry, sizeof(ext2_dir_entry), 1, fil);
                fclose(fil);
                int fd = -1;
                for(int i = 0; i < 8; i++)
                {
                    if(fopen_table[i] == find_entry.inode) fd = i;
                }
                if(close(fd) > 0) printf("Close the file successfully\n");
            }
        }
        else if(strcmp(Command, "write") == 0)
        {
            char filename[256];
            __u8 type;
            scanf("%s%d", filename, &type);
            getchar();
            if(strlen(filename) > NAME_LEN) printf("Do not find the directory named \"%s\"\n", filename);
            else if(type == 2) printf("Can't write in a directory\n");
            else
            {
                int find_entry_addr = search_file_entry_off(filename, type, current_dir);
                ext2_dir_entry find_entry;
                int fd = -1;
                FILE* fil;
                fil = fopen(Path, "r+");
                fseek(fil, find_entry_addr, SEEK_SET);
                fread(&find_entry, sizeof(ext2_dir_entry), 1, fil);
                for(int i = 0; i < 8; i++)
                {
                    if(fopen_table[i] == find_entry.inode) fd = i;
                }
                if(fd < 0) fd = open(filename, type);
                if(fd >= 0)
                {
                    char *str;
                    int size = 1;
                    int len = 0;
                    char ch;
                    str = (char*) malloc(size); // 分配初始内存
                    printf("Input string you want to write into the file until you press \"Enter\":\n");
                    while ((ch = getchar()) != '\n') 
                    { // 逐个读取字符，直到读取换行符为止
                        if (len >= size) { // 如果当前字符串长度超过了已分配内存的大小，则重新分配更多的内存
                            size *= 2;
                            str = (char*)realloc(str, size);
                        }
                        *(str + len) = ch; // 将读取的字符存储到字符串中
                        len++;
                    }
                    *(str + len) = '\0'; // 在字符串末尾添加 '\0'，表示字符串结束
                    if(write(fd, str) > 0) printf("\nSuccessfully write!\n");
                    free(str);
                    close(fd);
                }
                fclose(fil);
            }
        }
        else if(strcmp(Command, "read") == 0)
        {
            char filename[256];
            __u8 type;
            scanf("%s%d", filename, &type);
            if(strlen(filename) > NAME_LEN) printf("Do not find the directory named \"%s\"\n", filename);
            else if(type == 2) printf("Can't read in a directory\n");
            else
            {
                int find_entry_addr = search_file_entry_off(filename, type, current_dir);
                ext2_dir_entry find_entry;
                int fd = -1;
                FILE* fil;
                fil = fopen(Path, "r+");
                fseek(fil, find_entry_addr, SEEK_SET);
                fread(&find_entry, sizeof(ext2_dir_entry), 1, fil);
                for(int i = 0; i < 8; i++)
                {
                    if(fopen_table[i] == find_entry.inode) fd = i;
                }
                if(fd < 0) fd = open(filename, type);
                if(fd >= 0)
                {
                    if(read(fd) > 0) printf("\n\nRead finished!\n");
                }
                close(fd);
                fclose(fil);
            }
        }
        else if(strcmp(Command, "format") == 0)
        {
            printf("Formating the file system...\n");
            format();
        }
        else if(strcmp(Command, "login") == 0)
        {
            login();
        }
        else if(strcmp(Command, "password") == 0)
        {
            password();
        }
        else if(strcmp(Command, "attrib") == 0)
        {
            char filename[256];
            __u8 type;
            __u16 rw;
            scanf("%s%d%u", filename, &type, &rw);
            if(strlen(filename) > NAME_LEN) printf("Do not find the directory named \"%s\"\n", filename);
            else
            {
                int jud = attrib(filename, rw,type);
                if(jud >= 0) printf("Successfully changed the rwx!\n");
            }
        }
        else if(strcmp(Command, "quit") == 0)
        {
            return ;
        }
        else
        {
            printf("Command not found!\n");
        }
    }
}
```
（3）结果演示
![[Pasted image 20231129111716.png]]
执行quit命令则退出shell与该文件系统，重新启动文件则shell自动启动等待用户输入命令
##### 6.ls()
（1）实现方法
遍历当前目录下的所有目录项，将每个目录项的信息进行输出
（2）实现代码
```c
void ls()
{
    printf("-----------------------------------------------------------------------------------------------------------------------------\n");
    printf("%-24s%-16s%-28s%-28s%-28s%-8s\n","Name","Type","Creation","Modification","Last access","Rights");
    FILE * fil = fopen(Path, "r+");
    fseek(fil, find_block_addr(current_dir), SEEK_SET);
    ext2_inode currblock;
    int count = 0;
    fread(&currblock, sizeof(ext2_inode), 1, fil);
    for(int i = 0; i < 8; i++)
    {//对每个目录项进行遍历
        if(i >= currblock.i_blocks) break;
        ext2_dir_entry curr_dir_fil;//每个目录项
        if(i <= 5)
        {//直接索引
            for(int j = 0; j < BLOCK_SIZE / sizeof(ext2_dir_entry); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(ext2_dir_entry), SEEK_SET);
                fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                if(curr_dir_fil.inode != 0)
                {//目录项存在
                    ext2_inode file_inode;//文件索引结点
                    fseek(fil, find_block_addr(curr_dir_fil.inode), SEEK_SET);
                    fread(&file_inode, sizeof(ext2_inode), 1, fil);
                    //将三个时间转化成字符串并删去末尾的换行符
                    time_t cre = file_inode.i_ctime;
                    time_t modi = file_inode.i_mtime;
                    time_t las = file_inode.i_atime;
                    char* temp = ctime(&cre);
                    char* ct = (char*)malloc(strlen(temp) + 2);
                    strcpy(ct, temp);
                    if(ct[strlen(ct) - 1] == '\n') ct[strlen(ct) - 1] = '\0';
                    temp = ctime(&modi);
                    char* mt = (char*)malloc(strlen(temp) + 2);
                    strcpy(mt, temp);
                    if(mt[strlen(mt) - 1] == '\n') mt[strlen(mt) - 1] = '\0';
                    temp = ctime(&las);
                    char* at = (char*)malloc(strlen(temp) + 2);
                    strcpy(at, temp);
                    if(at[strlen(at) - 1] == '\n') at[strlen(at) - 1] = '\0';

                    printf("%-24s%-16s%-28s%-28s%-28s%-8s\n",curr_dir_fil.name, view_type(curr_dir_fil.file_type), ct, mt, at, rw_change(file_inode.i_mode));
                    count++;
                }
            }
        }
        else if(i == 6)
        {//一级索引
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                for(int k = 0; k < BLOCK_SIZE / sizeof(ext2_dir_entry); k++)
                {
                    fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(ext2_dir_entry), SEEK_SET);
                    fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                    if(curr_dir_fil.inode != 0)
                    {
                        ext2_inode file_inode;
                        fseek(fil, find_block_addr(curr_dir_fil.inode), SEEK_SET);
                        fread(&file_inode, sizeof(ext2_inode), 1, fil);
                        time_t cre = file_inode.i_ctime;
                        time_t modi = file_inode.i_mtime;
                        time_t las = file_inode.i_atime;
                        char* ct = ctime(&cre);
                        if(ct[strlen(ct) - 1] == '\n') ct[strlen(ct) - 1] = '\0';
                        char* mt = ctime(&modi);
                        if(mt[strlen(mt) - 1] == '\n') mt[strlen(mt) - 1] = '\0';
                        char* at = ctime(&las);
                        if(at[strlen(at) - 1] == '\n') at[strlen(at) - 1] = '\0';
                        printf("%-24s%-16s%-28s%-28s%-28s%-8s\n",curr_dir_fil.name, view_type(curr_dir_fil.file_type), ct, mt, at, rw_change(file_inode.i_mode));
                        count++;
                    }
                }
            }
        }
        else if(i == 7)
        {//二级索引
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {//遍历一级索引块中的每一个二级索引块地址
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                for(int k = 0; k < BLOCK_SIZE / sizeof(__u16); k++)
                {//遍历二级索引块中的每个数据块地址
                    __u16 second_block;//二级索引块号
                    fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(__u16), SEEK_SET);
                    fread(&second_block, sizeof(__u16), 1, fil);
                    
                    for(int g = 0; g < BLOCK_SIZE / sizeof(ext2_dir_entry); g++)
                    {//对数据块中的每个目录项进行遍历
                        fseek(fil, (515 + second_block) * BLOCK_SIZE + sizeof(ext2_dir_entry), SEEK_SET);
                        fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                        if(curr_dir_fil.inode != 0)
                        {
                            ext2_inode file_inode;
                            fseek(fil, find_block_addr(curr_dir_fil.inode), SEEK_SET);
                            fread(&file_inode, sizeof(ext2_inode), 1, fil);
                            time_t cre = file_inode.i_ctime;
                            time_t modi = file_inode.i_mtime;
                            time_t las = file_inode.i_atime;
                            char* ct = ctime(&cre);
                            if(ct[strlen(ct) - 1] == '\n') ct[strlen(ct) - 1] = '\0';
                            char* mt = ctime(&modi);
                            if(mt[strlen(mt) - 1] == '\n') mt[strlen(mt) - 1] = '\0';
                            char* at = ctime(&las);
                            if(at[strlen(at) - 1] == '\n') at[strlen(at) - 1] = '\0';
                            printf("%-24s%-16s%-28s%-28s%-28s%-8s\n",curr_dir_fil.name, view_type(curr_dir_fil.file_type), ct, mt, at, rw_change(file_inode.i_mode));
                            count++;
                        }
                    }
                }
            }
        }
    }
    fclose(fil);
}
```
##### 7.进入目录相关
##### 1.cd()
（1）实现方法
用户层函数，根据输入的文件名查找文件目录项的磁盘偏移量，并读出该目录项，首先需要读出该目录的索引结点，判断其是否具有读权限，若有则改变当前目录为该目录，并修改当前路径，添加该目录名。需要注意，如果目录为.或..，当进入的是根目录时，都不进行改变，若执行`cd ..`，需要删除当前路径的最后一条目录名。
（2）实现代码
```c
//进入某个目录，正确进入返回1，否则返回0或-1.
int cd(const char* name)
{
    __u8 type = 0x02;//目录类型
    int find_entry_addr = search_file_entry_off(name, type, current_dir);//该目录项在磁盘的偏移量
    ext2_dir_entry find_entry;
    FILE* fil;
    fil = fopen(Path, "r+");
    fseek(fil, find_entry_addr, SEEK_SET);
    fread(&find_entry, sizeof(ext2_dir_entry), 1, fil);
    ext2_inode find_node;//该目录索引结点
    fseek(fil, find_block_addr(find_entry.inode), SEEK_SET);
    fread(&find_node, sizeof(ext2_inode), 1, fil);
    __u16 rw = type_to_rw(find_node.i_mode);
    fclose(fil);
    if(find_entry_addr == -1)
    {//未找到
        return 0;
    }
    if(rw < 4 && rw >= 0)
    {//没有读权限，无法进入
        printf("Do not have enough right!");
        return -1;
    }
    else
    {
        if(strcmp(find_entry.name, ".") == 0) return 1;
        else if(strcmp(find_entry.name, "..") == 0)
        {//回到上一目录
            if(strcmp(current_path, "Root/") == 0) return 1;//根目录无法再往前进
            int p = strlen(current_path);
            current_dir = find_entry.inode;
            for(int i = 1; i <= 256; i++)
            {//删去当前路径的该目录名，回到上一个目录
                if(current_path[p - i] == '/' && i != 1) return 1;
                else current_path[p - i] = '\0';
            }
            return 1;
        }
        else
        {//当前路径加上该目录
            strcat(current_path, name);
            strcat(current_path, "/");
            current_dir = find_entry.inode;
            return 1;
        }
    }
    return 0;
}
```
（3）结果演示
![[Pasted image 20231129114850.png]]
若目录a有权限，则进入成功
否则，修改权限为0，则无法进入
![[Pasted image 20231129115048.png]]
![[Pasted image 20231129115129.png]]
另外，可以通过cd命令返回上一级目录
##### 2.search_file_entry_off()
（1）实现方法
该函数根据所给的文件名寻找当前目录的文件，返回该目录项的磁盘偏移量，参数为文件名，文件类型，当前目录索引结点号。实现时根据多级索引机制遍历当前目录。
（2）实现代码
```c
//根据文件名找到当前目录的文件,返回该目录项的磁盘偏移量,参数为文件名，文件类型，当前目录索引结点号
int search_file_entry_off(const char* name, __u8 type, unsigned short temp_curr_dir)
{
    FILE *fil = fopen(Path, "r+");
    fseek(fil, find_block_addr(temp_curr_dir) , SEEK_SET);
    ext2_inode currblock;
    ext2_dir_entry curr_dir_fil;//对每个目录项进行遍历
    fread(&currblock, sizeof(ext2_inode), 1, fil);//读出当前目录的索引结点
    for(int i = 0; i < 8; i++)
    {
        if(i >= currblock.i_blocks) break;
        if(i <= 5)
        {//直接索引
            for(int j = 0; j < BLOCK_SIZE / sizeof(ext2_dir_entry); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(ext2_dir_entry), SEEK_SET);
                fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                if(curr_dir_fil.inode != 0 && strcmp(curr_dir_fil.name, name) == 0 && curr_dir_fil.file_type == type)
                {//文件存在且类型和文件名相同
                    fclose(fil);
                    return (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(ext2_dir_entry);
                }
            }
        }
        else if(i == 6)
        {//一级索引
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                for(int k = 0; k < BLOCK_SIZE / sizeof(ext2_dir_entry); k++)
                {
                    fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(ext2_dir_entry), SEEK_SET);
                    fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                    if(curr_dir_fil.inode != 0 && strcmp(curr_dir_fil.name, name) == 0 && curr_dir_fil.file_type == type)
                    {
                        fclose(fil);
                        return (515 + first_block) * BLOCK_SIZE + k * sizeof(ext2_dir_entry);
                    }
                }
            }
        }
        else if(i == 7)
        {
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {//遍历一级索引块中的每一个二级索引块地址
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                for(int k = 0; k < BLOCK_SIZE / sizeof(__u16); k++)
                {//遍历二级索引块中的每个数据块地址
                    __u16 second_block;//二级索引块号
                    fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(__u16), SEEK_SET);
                    fread(&second_block, sizeof(__u16), 1, fil);
                    for(int g = 0; g < BLOCK_SIZE / sizeof(ext2_dir_entry); g++)
                    {//对数据块中的每个目录项进行遍历
                        fseek(fil, (515 + second_block) * BLOCK_SIZE + g * sizeof(ext2_dir_entry), SEEK_SET);
                        fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                        if(curr_dir_fil.inode != 0 && strcmp(curr_dir_fil.name, name) == 0 && curr_dir_fil.file_type == type)
                        {
                            fclose(fil);
                            return (515 + second_block) * BLOCK_SIZE + g * sizeof(ext2_dir_entry);
                        }
                    }
                }
            }
        }
    }
    fclose(fil);
    return -1;
}
```
##### 8.删除文件相关
##### 1.delete_file()
（1）实现方法
用户层函数，删除指定的文件，首先找到该文件的索引结点，判断当前目录和该文件是否有写权限，没有则无法删除，后根据不同类型执行不同操作，如果删除的是目录，则需要递归调用该函数，删除该目录下所有文件；若为文件则只需要删除该文件。删除时，需要把所有分配的数据块进行回收，并将当前目录下的该目录项删除。
（2）实现代码
```c
//删除指定的文件，参数为文件名，文件类型代码，当前目录索引结点号
int delete_file(const char* name, __u8 type, unsigned short temp_curr_dir)
{
    int find_entry_addr = search_file_entry_off(name, type, temp_curr_dir);
    if(find_entry_addr == -1) return 0;
    ext2_dir_entry find_entry;
    FILE* fp = fopen(Path, "r+");
    fseek(fp, find_entry_addr, SEEK_SET);
    fread(&find_entry, sizeof(ext2_dir_entry), 1, fp);
    fclose(fp);

    if(find_entry.inode == 0) return 0;
    else
    {//找到该文件
        FILE *fil = fopen(Path, "r+");
        fseek(fil, find_block_addr(temp_curr_dir), SEEK_SET);
        ext2_inode temp_curr_block;
        fread(&temp_curr_block, sizeof(ext2_inode), 1, fil);//当前目录的索引结点
        __u16 rwt = type_to_rw(temp_curr_block.i_mode);
        __u16 a = 0x0002;//0010
        fseek(fil, find_block_addr(find_entry.inode) , SEEK_SET);
        ext2_inode currblock;
        fread(&currblock, sizeof(ext2_inode), 1, fil);//读出找到文件的索引结点
        __u16 rwc = type_to_rw(temp_curr_block.i_mode);
        if((rwt & a) == 0 || (rwc & a) == 0)
        {//当前目录和该文件没有写权限，无法删除
            printf("Do not have enough right!\n");
            return -1;
        }
        __u32 zero[128] = {0};
        if(find_entry.file_type == 2)
        {//若删除的为目录，递归删除该目录下的所有文件
            ext2_dir_entry curr_dir_fil;//对每个目录项进行遍历
            for(int i = 0; i < 8; i++)
            {
                if(currblock.i_block[i] == 0) continue;
                if(i <= 5)
                {//直接索引
                    for(int j = 0; j < BLOCK_SIZE / sizeof(ext2_dir_entry); j++)
                    {
                        fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(ext2_dir_entry), SEEK_SET);
                        fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                        if(i == 0 && (j == 1 || j == 0))
                        {//如果是目录项前两个，即.和..目录，则不直接对其进行删除
                            continue;
                        }
                        else if(curr_dir_fil.inode != 0)
                        {//递归删除所有文件
                            delete_file(curr_dir_fil.name, curr_dir_fil.file_type, find_entry.inode);
                        }
                    }
                    ext2_free_block_bitmap(currblock.i_block[i]);//将该数据块回收
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
                else if(i == 6)
                {//一级索引
                    __u16 first_block;//一级索引块号
                    for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
                    {
                        fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                        fread(&first_block, sizeof(__u16), 1, fil);
                        for(int k = 0; k < BLOCK_SIZE / sizeof(ext2_dir_entry); k++)
                        {
                            fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(ext2_dir_entry), SEEK_SET);
                            fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                            if(curr_dir_fil.inode != 0)
                            {
                                delete_file(curr_dir_fil.name, curr_dir_fil.file_type, find_entry.inode);
                            }
                        }
                        ext2_free_block_bitmap(first_block);//删除存放目录项的所有数据块
                        fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                        fwrite(&zero, BLOCK_SIZE, 1, fil);
                        group_desc.bg_free_blocks_count++;
                    }
                    ext2_free_block_bitmap(currblock.i_block[i]);//删除存一级索引的数据块
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
                else if(i == 7)
                {//二级索引
                    __u16 first_block;//一级索引块号
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
                    {//遍历一级索引块中的每一个二级索引块地址
                        fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                        fread(&first_block, sizeof(__u16), 1, fil);
                        fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                        for(int k = 0; k < BLOCK_SIZE / sizeof(__u16); k++)
                        {//遍历二级索引块中的每个数据块地址
                            __u16 second_block;//二级索引块号
                            fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(__u16), SEEK_SET);
                            fread(&second_block, sizeof(__u16), 1, fil);
                            fseek(fil, (515 + second_block) * BLOCK_SIZE, SEEK_SET);
                            for(int g = 0; g < BLOCK_SIZE / sizeof(ext2_dir_entry); g++)
                            {//对数据块中的每个目录项进行遍历
                                fread(&curr_dir_fil, sizeof(ext2_dir_entry), 1, fil);
                                if(curr_dir_fil.inode != 0)
                                {
                                    delete_file(curr_dir_fil.name, curr_dir_fil.file_type, find_entry.inode);
                                }
                            }
                            ext2_free_block_bitmap(second_block);
                            fseek(fil, (515 + second_block) * BLOCK_SIZE, SEEK_SET);
                            fwrite(&zero, BLOCK_SIZE, 1, fil);
                            group_desc.bg_free_blocks_count++;
                        }
                        ext2_free_block_bitmap(first_block);
                        fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                        fwrite(&zero, BLOCK_SIZE, 1, fil);
                        group_desc.bg_free_blocks_count++;
                    }
                    ext2_free_block_bitmap(currblock.i_block[i]);
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
            }
            ext2_free_inode(find_entry.inode);//释放该文件索引结点，并删除当前目录下该文件的目录项号
            ext2_dir_entry ze = {0, 0, 0, 0, "\0"};
            fseek(fil, find_entry_addr, SEEK_SET);
            fwrite(&ze, sizeof(ext2_dir_entry), 1, fil);
            group_desc.bg_free_inodes_count++;
        }
        else
        {//删除的是普通文件
            for(int i = 0; i < 8; i++)
            {
                if(currblock.i_block[i] == 0) continue;
                if(i <= 5)
                {
                    __u16 block_num = currblock.i_block[i];//数据块号
                    ext2_free_block_bitmap(block_num);
                    fseek(fil, (515 + block_num) * BLOCK_SIZE, SEEK_SET);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
                else if(i == 6)
                {
                    __u16 first_block;//一级索引块号
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
                    {
                        fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                        fread(&first_block, sizeof(__u16), 1, fil);
                        ext2_free_block_bitmap(first_block);
                        fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                        fwrite(&zero, BLOCK_SIZE, 1, fil);
                        group_desc.bg_free_blocks_count++;
                    }
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    ext2_free_block_bitmap(currblock.i_block[i]);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
                else if(i == 7)
                {
                    __u16 first_block;//一级索引块号
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
                    {//遍历一级索引块中的每一个二级索引块地址
                        fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                        fread(&first_block, sizeof(__u16), 1, fil);
                        fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                        for(int k = 0; k < BLOCK_SIZE / sizeof(__u16); k++)
                        {//遍历二级索引块中的每个数据块地址
                            __u16 second_block;//二级索引块号
                            fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(__u16), SEEK_SET);
                            fread(&second_block, sizeof(__u16), 1, fil);
                            ext2_free_block_bitmap(second_block);//二级索引块中的数据块地址，更新索引位图
                            fseek(fil, (515 + second_block) * BLOCK_SIZE, SEEK_SET);
                            fwrite(&zero, BLOCK_SIZE, 1, fil);
                            group_desc.bg_free_blocks_count++;
                        }
                        fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                        ext2_free_block_bitmap(first_block);
                        fwrite(&zero, BLOCK_SIZE, 1, fil);
                        group_desc.bg_free_blocks_count++;
                    }
                    fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
                    ext2_free_block_bitmap(currblock.i_block[i]);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
            }
            ext2_free_inode(find_entry.inode);
            ext2_dir_entry ze = {0, 0, 0, 0 ,"\0"};
            fseek(fil, find_entry_addr, SEEK_SET);
            fwrite(&ze, sizeof(ext2_dir_entry), 1, fil);
            group_desc.bg_free_inodes_count++;
        }
        time_t curr;
        time(&curr);
        temp_curr_block.i_mtime = curr;
        temp_curr_block.i_atime = curr;
        fseek(fil, find_block_addr(temp_curr_dir), SEEK_SET);
        fwrite(&temp_curr_block, sizeof(ext2_inode), 1, fil);//当前目录的索引结点
        fclose(fil);
        update_group_desc();
        return 1;
    }
    return 0;
}
```
（3）结果演示
![[Pasted image 20231129120504.png]]
如图所示，在上一个命令条件下，a目录中含有多个子文件，在根目录下删除a，其所有子文件均被删除。
![[Pasted image 20231129120837.png]]
当当前目录d没有写权限时，可以看到，无法删除d目录内的文件。
##### 2.ext2_free_block_bitmap()和ext2_free_inode_bitmap()函数
（1）实现方法
底层函数，以释放数据块为例，根据所给的数据块号，定义一个偏移相关位数的32位二进制数，并利用按位与操作，将该为置为0，更新数据块位图。
（2）实现代码
```c
//释放指定块号的数据块位图，参数为要释放的数据块块号
void ext2_free_block_bitmap(__u16 block_num)
{
    __u32 block_bitmap[128];
    FILE *fp = fopen(Path, "r+");
    fseek(fp, group_desc.bg_block_bitmap * BLOCK_SIZE, SEEK_SET);
    fread(&block_bitmap, BLOCK_SIZE, 1, fp); // block_bitmap保存块位图
    int index = block_num / 32;//数组号数
    int offset = block_num % 32;//存放位图的数组位偏移
    __u32 temp = 0x7fff;//0111_1111_1111_1111
    temp = temp >> offset;//右移偏移位数
    temp = ~temp;//取反并进行按位与，用于将该位置0
    block_bitmap[index] = block_bitmap[index] & temp;
    fseek(fp, group_desc.bg_block_bitmap * BLOCK_SIZE, SEEK_SET);
    fwrite(&block_bitmap, BLOCK_SIZE, 1, fp);
    fclose(fp);
}

//释放指定的索引结点位图
void ext2_free_inode_bitmap(__u16 inode_num)
{
    __u32 block_bitmap[128];
    FILE *fp = fopen(Path, "r+");
    fseek(fp, group_desc.bg_inode_bitmap * BLOCK_SIZE, SEEK_SET);
    fread(&block_bitmap, BLOCK_SIZE, 1, fp); // inode_bitmap保存块位图
    int index = inode_num / 32;//数组号数
    int offset = inode_num % 32;//存放位图的数组位偏移
    __u32 temp = 0x8000;
    temp = temp >> offset;//右移偏移位数
    temp = ~temp;
    block_bitmap[index] = block_bitmap[index] & temp;
    fseek(fp, group_desc.bg_inode_bitmap * BLOCK_SIZE, SEEK_SET);
    fwrite(&block_bitmap, BLOCK_SIZE, 1, fp);
    fclose(fp);
}
```
##### 3.ext2_free_blocks()函数
（1）实现方法
底层函数，释放所给索引结点的所有数据块，根据多级索引的机制，逐级释放，此函数仅针对普通文件。
（2）实现代码
```c
//释放特定文件的所有数据块，参数为索引结点号
void ext2_free_blocks(__u16 inode_num)
{
    FILE *fil = fopen(Path, "r+");
    fseek(fil, find_block_addr(inode_num) , SEEK_SET);
    ext2_inode currblock;
    fread(&currblock, sizeof(ext2_inode), 1, fil);//读出文件的索引结点
    __u32 zero[128] = {0};
    for(int i = 0; i < 8; i++)
    {
        if(currblock.i_block[i] == 0) continue;
        if(i <= 5)
        {
            __u16 block_num = currblock.i_block[i];//数据块号
            ext2_free_block_bitmap(block_num);
            fseek(fil, (515 + block_num) * BLOCK_SIZE, SEEK_SET);
            fwrite(&zero, BLOCK_SIZE, 1, fil);
            group_desc.bg_free_blocks_count++;
        }
        else if(i == 6)
        {
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                ext2_free_block_bitmap(first_block);
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                fwrite(&zero, BLOCK_SIZE, 1, fil);
                group_desc.bg_free_blocks_count++;
            }
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            ext2_free_block_bitmap(currblock.i_block[i]);
            fwrite(&zero, BLOCK_SIZE, 1, fil);
            group_desc.bg_free_blocks_count++;
        }
        else if(i == 7)
        {
            __u16 first_block;//一级索引块号
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            for(int j = 0; j < BLOCK_SIZE / sizeof(__u16); j++)
            {//遍历一级索引块中的每一个二级索引块地址
                fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE + j * sizeof(__u16), SEEK_SET);
                fread(&first_block, sizeof(__u16), 1, fil);
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                for(int k = 0; k < BLOCK_SIZE / sizeof(__u16); k++)
                {//遍历二级索引块中的每个数据块地址
                    __u16 second_block;//二级索引块号
                    fseek(fil, (515 + first_block) * BLOCK_SIZE + k * sizeof(__u16), SEEK_SET);
                    fread(&second_block, sizeof(__u16), 1, fil);
                    ext2_free_block_bitmap(second_block);//二级索引块中的数据块地址，更新索引位图
                    fseek(fil, (515 + second_block) * BLOCK_SIZE, SEEK_SET);
                    fwrite(&zero, BLOCK_SIZE, 1, fil);
                    group_desc.bg_free_blocks_count++;
                }
                fseek(fil, (515 + first_block) * BLOCK_SIZE, SEEK_SET);
                ext2_free_block_bitmap(first_block);
                fwrite(&zero, BLOCK_SIZE, 1, fil);
                group_desc.bg_free_blocks_count++;
            }
            fseek(fil, (515 + currblock.i_block[i]) * BLOCK_SIZE, SEEK_SET);
            ext2_free_block_bitmap(currblock.i_block[i]);
            fwrite(&zero, BLOCK_SIZE, 1, fil);
            group_desc.bg_free_blocks_count++;
        }
    }
    ext2_dir_entry ze = {0, 0, 0, 0 ,"\0"};
    group_desc.bg_free_inodes_count++;
    currblock.i_blocks = 0;
    for(int i =  0; i < 8; i++) currblock.i_block[i] = 0;
    fseek(fil, find_block_addr(inode_num), SEEK_SET);
    fwrite(&currblock, sizeof(ext2_inode), 1, fil);
    fclose(fil);
    update_group_desc();
}
```
##### 9.打开关闭文件
##### 1.open()
（1）实现方法
用户层函数，根据给定的文件名，先找到该文件，判断有无读写权限，无权限不允许打开，有则将其加入文件打开表，如果文件打开表已满则无法加入，否则返回找到的打开表空项的索引地址。
（2）实现代码
```c
//打开文件，成功返回文件打开表所在位置,否则返回-1
int open(const char* filename, __u8 type)
{
    int find_entry_addr = search_file_entry_off(filename, type, current_dir);
    ext2_dir_entry find_entry;
    FILE* fil;
    fil = fopen(Path, "r+");
    fseek(fil, find_entry_addr, SEEK_SET);
    fread(&find_entry, sizeof(ext2_dir_entry), 1, fil);
    ext2_inode find_node;
    fseek(fil, find_block_addr(find_entry.inode), SEEK_SET);
    fread(&find_node, sizeof(ext2_inode), 1, fil);
    __u16 rw = type_to_rw(find_node.i_mode);
    fclose(fil);
    __u16 a = 0x0006;//0110
    if(find_entry_addr == -1)
    {//未找到该文件
        printf("Don not find this file\n");
        return -1;
    }
    if((a & rw) == 0)
    {//没有读写权限
        printf("Do not have enough right!\n");
        return -1;
    }
    else
    {
        for(int i = 0; i < 16; i++)
        {
            if(fopen_table[i] == 0)
            {//遍历文件打开表，找空项存入索引结点号并返回可用文件打开ID
                fopen_table[i] = find_entry.inode;
                return i;
            }
        }
        printf("The number of opened files reaches the limit\n");
    }
    return -1;
}
```
##### 2.close()
（1）实现方法
用户层函数，文件打开表对应位置置0，并修改文件访问时间。
（2）实现代码
```c
//关闭文件
int close(int fd)
{
    if(test_fd(fd) <= 0)
    {//fd不可用
        return -1;
    }
    else
    {
        int offinode = find_block_addr(fopen_table[fd]);
        ext2_inode finode;
        FILE *fil = fopen(Path, "r+");
        fseek(fil, offinode, SEEK_SET);
        fread(&finode, sizeof(ext2_inode), 1, fil);
        time_t curr;
        time(&curr);
        finode.i_atime = curr;//修改访问时间
        fseek(fil, offinode, SEEK_SET);
        fwrite(&finode, sizeof(ext2_inode), 1, fil);
        fopen_table[fd] = 0;
        fclose(fil);
    }
    return 1;
}
```
（3）结果演示
![[Pasted image 20231129121211.png]]
可以看到，当文件有读写权限时，可以被open，若文件没有读写权限，则open失败。
##### 10.读写相关操作
##### 1.write()
（1）实现方法
用户层函数，写操作，执行写操作前必须保证文件已打开，否则无法执行，打开后，需要判断该文件是否有写权限，没有则退出，有写权限，则将所给的字符串写入该文件，直到遇到空字符，写过程中，需要判断是否用完了一块数据块，用完一块则调用分配数据块函数进行分配，该函数写之前默认对该文件进行覆盖清空。
（2）实现代码
```c
//对文件进行写操作，参数为文件打开ID，要写入的字符串sources
int write(int fd, const char* sources)
{
    if(test_fd(fd) <= 0) return -1;
    else
    {
        int offinode = find_block_addr(fopen_table[fd]);
        ext2_inode finode;
        FILE *fil = fopen(Path, "r+");
        fseek(fil, offinode, SEEK_SET);
        fread(&finode, sizeof(ext2_inode), 1, fil);
        ext2_free_blocks(fopen_table[fd]);
        __u16 rw = type_to_rw(finode.i_mode);
        __u16 a = 0x0002;//0010
        if((rw & a) == 0)
        {//没有写权限
            printf("Do not have enough right!\n");
            return -1;
        }
        int i = 0;
        int block_num = finode.i_block[0];
        __u32 filesize = 0;
        while(sources[i] != '\0')
        {//直到空字符
            if(filesize % BLOCK_SIZE == 0)
            {//写满一块
                fclose(fil);
                block_num = ext2_alloc_block(fopen_table[fd]);//分配一块新的数据块
                fil = fopen(Path, "r+");
                fseek(fil, offinode, SEEK_SET);
                fread(&finode, sizeof(ext2_inode), 1, fil);
            }
            fseek(fil, (515 + block_num) * BLOCK_SIZE + (i % BLOCK_SIZE), SEEK_SET);
            fwrite(&sources[i], 1, 1, fil);
            i++;
            filesize++;
        }
        time_t curr;
        time(&curr);
        finode.i_mtime = curr;
        finode.i_size = filesize;//更新索引结点的文件大小与修改时间
        fseek(fil, offinode, SEEK_SET);
        fwrite(&finode, sizeof(ext2_inode), 1, fil);
        fclose(fil);
    }
    return 1;
}
```
（3）结果演示
![[Pasted image 20231129121659.png]]
当文件没有写权限时无法写入，有则可以，向文件写入三千三百个字符成功，已启用一级索引。
##### 2.read()
（1）实现方法
用户层函数，同样需要文件已打开，函数从所给文件中读出所有字符，由于事先未知字符串长度，因此逐字符进行读出，并需要根据多级索引机制，切换读写头所在地址。
（2）实现代码
```c
//从文件中读出所有字符，参数为fd
int read(int fd)
{
    if(test_fd(fd) <= 0) return -1;
    else
    {
        int offinode = find_block_addr(fopen_table[fd]);
        ext2_inode finode;
        FILE *fil = fopen(Path, "r+");
        fseek(fil, offinode, SEEK_SET);
        fread(&finode, sizeof(ext2_inode), 1, fil);
        int block_num = finode.i_block[0];
        __u32 filesize = finode.i_size;
        char word;
        for(int i = 0; i < filesize; i++)
        {//以文件大小作为结束标志，从头读到尾并逐字符输出
            if(i < BLOCK_SIZE * 6)
            {//直接索引
                if(i % BLOCK_SIZE == 0)
                {//每读完一块，更改块号
                    block_num = finode.i_block[i / BLOCK_SIZE];
                }
                fseek(fil, (515 + block_num) * BLOCK_SIZE + i % BLOCK_SIZE, SEEK_SET);
                fread(&word, 1, 1, fil);
                printf("%c", word);
            }
            else if(i - BLOCK_SIZE * 6 < ENTRY_NUM * BLOCK_SIZE)
            {//一级索引
                if(i % BLOCK_SIZE == 0)
                {//每读完一块，从一级索引块中读出数据块地址
                    __u16 first_inode;
                    fseek(fil, (515 + finode.i_block[6]) * BLOCK_SIZE + (i - BLOCK_SIZE * 6) / BLOCK_SIZE * sizeof(__u16), SEEK_SET);
                    fread(&first_inode, sizeof(__u16), 1, fil);
                    block_num = first_inode;
                }
                fseek(fil, (515 + block_num) * BLOCK_SIZE + i % BLOCK_SIZE, SEEK_SET);
                fread(&word, 1, 1, fil);
                printf("%c", word);
            }
            else
            {//二级索引
                if(i % BLOCK_SIZE == 0)
                {
                    __u16 inode_num;
                    int remain = (i - BLOCK_SIZE * 6 - BLOCK_SIZE * ENTRY_NUM);
                    fseek(fil, (515 + finode.i_block[7]) * BLOCK_SIZE + remain / (ENTRY_NUM * BLOCK_SIZE) * sizeof(__u16), SEEK_SET);
                    fread(&inode_num, sizeof(__u16), 1, fil);
                    fseek(fil, (515 + inode_num) * BLOCK_SIZE + (remain % (ENTRY_NUM * BLOCK_SIZE)) / BLOCK_SIZE * sizeof(__u16), SEEK_SET);
                    fread(&inode_num, sizeof(__u16), 1, fil);
                    block_num = inode_num;
                }
                fseek(fil, (515 + block_num) * BLOCK_SIZE + i % BLOCK_SIZE, SEEK_SET);
                fread(&word, 1, 1, fil);
                printf("%c", word);
            }
        }
        fclose(fil);
    }
    return 1;
}
```
（3）结果演示
![[Pasted image 20231129121849.png]]
读出结果与写入结果相同，证明读写正常，且多级索引工作正常。
##### 11.用户登录与密码
##### 1.login()
（1）实现方法
用户层函数，实现用户的登录功能，如果无密码，则只需要输入用户名，若有密码，则另需要输入密码，并判断输入是否正确。
（2）实现代码
```c
//登录用户
void login()
{
    printf("Login to the file system...\nUsername:");
    scanf("%s", Username);
    if(strlen(PassWord) > 0)
    {
        while(1)
        {
            char inputpsw[256];
            printf("Password:");
            scanf("%s", inputpsw);
            if(strcmp(inputpsw, PassWord) == 0)
            {
                printf("Login successfully!\n");
                return;
            }
            else
            {
                printf("Password error!\n");
            }
        }
    }
    printf("Login successfully!\n");
}
```
（3）结果演示
![[Pasted image 20231129110724.png]]
初始没有密码，可以直接登录，后设置一个密码，再登录时需要输入密码，输入错误则无法登录。
##### 2.password()
（1）实现方法
用户层函数，修改用户密码，如果密码不存在，直接新建一个密码；若已有密码，则需要先输入旧密码，再修改新密码。
（2）实现代码
```c
//修改登录密码
void password()
{
    if(strlen(PassWord) > 0)
    {
        while(1)
        {
            char inputpsw[256];
            printf("Old password:");
            scanf("%s", inputpsw);
            if(strcmp(inputpsw, PassWord) == 0) break;
            else
            {
                printf("\nPassword error!\n");
                return;
            }
        }
    }
    printf("New password:");
    scanf("%s",PassWord);
    printf("Password change successfully!\n");
}
```
（3）结果演示
![[Pasted image 20231129110837.png]]
修改时需要输入旧密码，密码修改成功
##### 12.修改权限
（1）实现方法
用户层函数，根据所给的权限进行对该文件权限修改。
（2）实现代码
```c
//修改文件权限，参数为文件名，要更改的读写权限代码，文件类型
int attrib(const char* name, __u16 rw, __u8 type)
{
    int find_entry_addr = search_file_entry_off(name, type, current_dir);
    if(find_entry_addr == -1)
    {
        printf("Don not find this file\n");
        return -1;
    }
    if(rw > 7)
    {//非法的权限代码
        printf("Illegal setting!\n");
        return -1;
    }
    ext2_dir_entry find_entry;
    FILE* fil;
    fil = fopen(Path, "r+");
    fseek(fil, find_entry_addr, SEEK_SET);
    fread(&find_entry, sizeof(ext2_dir_entry), 1, fil);
    ext2_inode find_node;
    fseek(fil, find_block_addr(find_entry.inode), SEEK_SET);
    fread(&find_node, sizeof(ext2_inode), 1, fil);
    __u16 a = 0xfff0 & find_node.i_mode;//提取i_mode中的类型代码
    find_node.i_mode = a | rw;//类型代码与修改的权限代码按位或，得到修改后的mode代码
    fseek(fil, find_block_addr(find_entry.inode), SEEK_SET);
    fwrite(&find_node, sizeof(ext2_inode), 1, fil);
    fclose(fil);
    return 0;
}
```
（3）结果演示
![[Pasted image 20231129122026.png]]
当文件f没有读权限时打开该文件失败，使用attrib修改f的权限为可读，则打开成功。
