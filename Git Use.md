### 创建ssh key

```
ssh-keygen -t rsa -C "1337936093@qq.com"
```
创建好在user文件夹.ssh目录
id_rsa.pub 即为key

##### 设置用户名，邮箱
```
git config --global user.name "xxxx"

git config --global user.email "xxxx"
```

### 登录Github设置添加key

---
# 常用操作

### 1.创建版本库
可新建文件夹或在已有文件夹下执行
````
git init
````
#### 1.添加文件
```
git add xxxx
//添加文件到仓库，可一次添加多个后一次性提交commit
```
---
##### git add 文件
- 方法一 git add 添加多个文件，文件之间以空格隔开
`git add file1 file2 file3`

- 方法二 多次git add
```
git add file1
git add file2
git add file2
```

- 方法三 添加指定目录下的文件
config目录下及子目录下所有文件，home目录下的所有.php文件
```
git config/*
git home/*.php
```

- 方法四 git add . 添加所有的文件， 或者 git add --all 添加所有的文件
```
git add .
git add --all
```


##### git add 文件夹
`git add 文件夹名`

---
#### 2.提交到仓库
```
git commmit -m "xxxxx"
//xxxxx写提交说明
```
#### 3.查看仓库状态
```
git status
```
或
```
git diff xxxxx
//xxxxx为需要查看修改历史的文件
```
#### 4.版本回退
```
git log
//可查看具体修改历史，加上--pretty=oneline参数可减少记录行数
```

```
git reflog
//查看命令历史
```

```
git reset --hard HEAD^
//回到上一个版本,HEAD~100为回到前100个版本
```
或
```
git reset --hard commit_id
//commit_id从历史中寻找
```
#### 5.撤销修改
##### -1.撤销工作区修改
```
git checkout -- xxxxxx
//xxxxx为文件名，执行后撤销该文件在工作区的所有修改
```
或
```
git restore xxxx
//丢弃xxxx在工作区的修改
```
##### -2.撤销暂存区修改

即已经add进暂存区
```
git reset HEAD xxxx
//将暂存区的修改退回工作区，即工作区的修改仍存在
//后需执行上一条指令
```
或
```
git restore --staged xxxx
//将暂存区修改重新放回工作区
```
##### -3.已提交到仓库
参考版本回退
#### 6.删除文件
（针对已提交文件）
##### -1.文件管理器中已删除
可先
```
git status 
查看修改状态
```
然后
```
$ git rm xxxx
$ git commit -m "******"
//删除未修改文件，即与版本库相同
```
或
```
git rm -f
//删除工作区和暂存区文件，即删除文件与版本库文件不同
```

#### 7.本地仓库上传github
```
git remote add origin git@github.com:Yi1699/Repository.git
//origin 为远程库的默认命名
```
登入github创建仓库后，将本地仓库与远程仓库相关联, Repository.git为仓库名称,Yi1699为github账号名称。

```
git push -u origin master
//第一次将本地仓库的当前分支master推送到远程仓库
```
在后续推送时可用如下命令
```
git push origin master
//将master分支的最新修改推送至github
```
注意：当本地仓库落后远程库时，需要先
`git pull origin master`
后再进行`add`和`commit`操作，并使用`git push`推送至远程库


#### --删除远程库
“删除”指解除本地和远程的仓库绑定关系，并非真正删除远程库
```
$ git remote -v
//查看远程库信息

$ git remote rm origin
//origin为远程库名字
```

#### 8.克隆github远程仓库
登录github创建好仓库后
在本地执行下述命令Repository.git 为仓库名
```
git clone git@github.com:Yi1699/Repository.git
```
此为ssh协议克隆，可用https等其他协议（速度慢）

#### 9.显示分支图
```
git log --all --decorate --oneline --graph
//巧记"a dog=git log --all --decorate --oneline --graph
```