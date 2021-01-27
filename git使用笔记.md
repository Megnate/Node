# 解决中文乱码

在 **git bash**空白处**右键**，选择**Options**，选择**Text**，然后选择locale中的**zh_CN**，编码选择**UTF-8**

# 创建git仓库并操作

创建新的目录：**mkdir** 目录名

这个也是一个**Linux**命令

显示工作目录：**pwd**

这个也是一个**Linux**命令

在新创建的目录下建立git仓库，转入新目录：**cd** 目录名

**创建：**git  init

**添加文件：**git add 完整文件名（完整的意思是带后缀）

**添加一个文件夹中的所有文件：**git add .   注意有一个点

**对这一次操作进行注释：**git commit -m '注释的文字内容'

**查看是否有其他文件正在提交：**git status

**查看修改内容：**git diff 完整文件名

**查看修改的历史记录：**git log   退出按Q

**回退上一个版本：**git reset --hard HEAD^

**回退上n个版本：**git reset --hard HEAD~n

**查看当前目录中的文件：**ls            这是一个**Linux**命令

**查看每一次提交的版本号：**git reflog            便于返回指定的版本

**返回最新的版本：**git reset --hard 版本号

**打印文件内容：**cat 完整文件名

**撤销还未提交的操作：**git checkout -- 完整文件名（文件名前面有空格）

**删除某一个文件：**git rm 完整文件名
	如果只是   rm 完整文件名   只是在这个文件夹中删除了该文件，但是没有在git 文件库中删除该文件，在GitHub中还是存在该文件

**在文件库中彻底删除：**只需要再次 git commit -m'注释'   再次提交就可以了
	如果没有提交，那么也是可以通过  git checkout -- 完整文件名  来恢复

# 登录git bash

**告知git名字：**git config --global user.name "名字"

**告知git邮箱：**git config --global user.email "邮箱"

# 连接GitHub

**创建SSH Key：**ssh -keygen -t rsa -C"1327358992@qq.com"
	里面有两个文件，id_rsa是私密钥匙，id_rsa.pub是公开的
	在网页上登录github.com/settings   找到SSH，添加新的SSH Key，title是随便写的，内容是id_ras.pub中复制的内容

**连接自己的Github网站：**git remote add origin https://github.com/Megnate/My-first-knowledge-graph-.git      **(一定要记得最后的.git)**

**将本体库内容推送到远程：**git push -u origin master
	接着就是各种输入用户名和密码的操作

此时再看GitHub中，已经和本地库是一样的

所以，接下来只要本体进行了提交，就可以**push**到GitHub上：**git push origin master**

# 从GitHub中提取出文件

在本体库中克隆出一个远程库：**git clone GitHub网址.git**       **(一定要记得最后的.git)**

# 创建新的时间分支

现在所有的时间分支都是**master**这个主分支，之前所有的版本退回命令都是基于这个分支的

**创建新的分支并切换：**git checkout -b 分支名字     此时已经切换到该分支中了

**创建分支：**git branch 分支名字

**切换分支：**git checkout 分支名字

在新的分支中的操作，在**master主分支**中是不会出现的，就是说需要让两个**分支合并：**git merge 分支名字   这个操作是在**master**分支上

合并完成之后就可以**删除该分支了：**git branch -d 分支名字
	这个是快速删除，分支中的更改版本都被删除掉了

**可以使用这个：**git merge --no-ff -m "注释" 分支名字，此时之前的版本信息都还存在

# 禁止重复输账号密码

git config --global credential.help

git pull

再输入一遍就可以了

# 出现错误

fatal: 'origin' does not appear to be a git repository fatal: Could not read from remote repository.

git remote add origin  + 库的SSH的名字

然后再提交就可以了

# 最常用的操作

提交本地文件到git：

git add 完整文件名

git commit -m'注释这次的操作'

git push origin master

建立分支并合并分支：

git checkout -b 分支名

git checkout master

git merge 分支名字

从GitHub中下载到本地：

git clone github网址.git

或者是：git pull origin master

**连接自己的Github网站：**git remote add origin https://github.com/Megnate/My-first-knowledge-graph-.git      **(一定要记得最后的.git)**

```git
git remote add origin git@github.com:Megnate/Node.gi
这个是看github网页中的SSH代码
```

# 在VS中使用git

![image-20201216210851619](C:\Users\梅桂楠\AppData\Roaming\Typora\typora-user-images\image-20201216210851619.png)

点击这个，顺序操作即可与自己的Github链接，创建新的仓库

![image-20201216210955513](C:\Users\梅桂楠\AppData\Roaming\Typora\typora-user-images\image-20201216210955513.png)

可以使用**git bash**来操作GitHub，也可以使用VS自带的选择按钮来操作GitHub

在**全部提交**选项处还可以下拉选择**全部提交并推送**

# push或者pull出现错误

进入了一个

Please enter a commit message to explain why this merge is necessary, # especially if it merges an updated upstream into a topic branch. # # Lines starting with ‘#’ will be ignored, and an empty message aborts # the commit.

这样的界面

按字母 **i** 进入insert模式

然后按“**Esc**”

输入   :wq

这个输入你可能看不到，但是可以输入

# GitHub

github上删除文件后与本地同步

抓取并合并远程仓库全部内容`git pull origin master`

再推送本地仓库就可以了`git push origin master`