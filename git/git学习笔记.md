# git学习笔记

### 1. git和svn的区别

- **svn**:是集中版本控制系统，所有的版本都在服务器上，用户本地的版本只有之前同步的版本，用户看不到历史版本，也不能在不同分支进行工作，服务器一出现问题，所有数据就会丢失
- **git:**分布式版本控制系统，所有的版本信息全部同步到本地，可以在本地查看素有版本历史，也可以离线本地提交，联网时push到对应的服务器上，只要一个用户的设备没有问题，就可以恢复所有的数据

### 2. git 安装和配置

> 注意：windows卸载git时，需要先清一下跟git有关的环境变量

windows下鼠标右键会有三个跟git有关的选项

- **Git Bash:** Linux风格的命令行，使用的最多，最推荐
- **Git CMD:** windows风格的命令行
- **Git GUI:** 图形化界面git,不建议初学者使用

### 3. git常用的配置

查看所有git配置：`git config -l`

查看git系统配置：` git config --system --list`

查看git本地配置：`git config --global --list`

**git相关配置文件位置**

- 系统级的配置：`C:\Program Files\Git\etc\gitconfig`
- 本地全局配置：`C:\Users\YANGGUANG\.gitconfig`

**当你安装git之后，首先需要做的事情是设置你的用户名和email地址**，这样每次提交git都会使用该信息，它被永远嵌入到你的提交中

```bash
git config --global user.name "yangguang"
git config --global user.email "yg962160627@sina.com"
```

如果你想要在某个项目中使用其他的用户和邮箱，你可以在这个项目中使用该命令但是不要加`--global`，总之`--global`是本地全局配置

### 4. git工作原理(核心)

**git一共有四个工作区域：**

- 工作目录（本地）：就是平时存放代码的地方
- 暂存区（本地）：用于临时存放你的改动，事实上就是一个文件，保存即将提交到文件列表信息
- 资源库（本地）：就是安全存放数据的位置，这里面有你提交的所有版本的数据，其中HEAD指向最新放入仓库的版本
- git仓库（远程）：远程托管代码的服务器

**四个区域的转换关系如下：**

```
正向：【工作目录】--(git add files)-->【暂存区】--(git commit)-->【本地资源库】--(git push)-->【远程仓库】

反向：【远程仓库】--(git pull)-->【本地资源库】--(git reset)-->【暂存区】--(git checkout)-->【工作目录】
```

### 5. git 基本操作操作

**文件的四种状态：**

- `untracked`: 未跟踪，该文件未提交到git库中，不能参与版本控制，通过`git add `状态会变成`staged`
- `staged`: 暂存状态，通过`git commmit`将修改同步到库中，这时文件的状态是`unmodify`，执行`git reset HEAD filename` 取消暂存，回到`unstracked`
- `unmodify`: 文件已经入库，未修改，即版本库中的文件快照内容于文件夹中的完全一致，这种文件有两种去处，如果被修改，就是`modified`状态，如果使用`git rm`移出版本库，就是`untracked`状态
- `modified`：文件已经修改，仅仅只是修改，并未进行其他操作，这种文件有两种去处，一种是通过`git add`加入暂存区变成`staged`状态，一种是通过`git checkout`丢弃修改，返回`unmodify`状态，也可以通过`git checkout`从git仓库中取出该文件(也就是该文件以前提交的版本)，覆盖当前修改

```bash
#查看git 装填
git status

# git add . 提交所有文件到暂存区
# git commit -m 提交暂存区的信息到本地仓库 -m 提交信息
# git push 推送到远程仓库
```

**忽略文件：**

 在主目录下创建一个`.gitignore`文件，用来进行忽略文件

```bash
*.txt #忽略所有txt文件
!lib.txt #但是lib.txt文件除外
.idea/   #忽略.idea目录下的所有文件
doc/*.txt #忽略doc目录下的所有txt文件
```

