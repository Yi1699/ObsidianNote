虚拟机开机，ifconfig查看ip地址
需提前安装ssh相关工具
使用命令`ssh root@192.168.1.108` 即可登录
root为用户，@后为IP地址

---
#### 使用ssh从服务器（如虚拟机）下载文件到本地
例如：(本地为windows，服务器为Linux)
先在需要下载的目的地址打开终端，后输入
`scp root@192.168.1.108:/home/lighthouse/file/test.c ./`
输入密码后即可将文件从服务器下载至本地

#### ssh上传至服务器
`scp D:\1\test.sql lighthouse@192.168.1.108:/home/lighthouse/file`
