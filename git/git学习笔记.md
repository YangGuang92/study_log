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

### 6. git常用命令

- 仓库

  ```bash
  # 在当前目录新建一个Git代码库
  $ git init
  
  # 新建一个目录，将其初始化为Git代码库
  $ git init [project-name]
  
  # 下载一个项目和它的整个代码历史
  $ git clone [url]
  ```

- 配置

  ```bash
  # 显示当前的Git配置
  $ git config --list
  
  # 编辑Git配置文件
  $ git config -e [--global]
  
  # 设置提交代码时的用户信息
  $ git config [--global] user.name "[name]"
  $ git config [--global] user.email "[email address]"
  ```

- 增加\删除文件

  ```bash
  # 添加指定文件到暂存区
  $ git add [file1] [file2] ...
  
  # 添加指定目录到暂存区，包括子目录
  $ git add [dir]
  
  # 添加当前目录的所有文件到暂存区
  $ git add .
  
  # 添加每个变化前，都会要求确认
  # 对于同一个文件的多处变化，可以实现分次提交
  $ git add -p
  
  # 删除工作区文件，并且将这次删除放入暂存区
  $ git rm [file1] [file2] ...
  
  # 停止追踪指定文件，但该文件会保留在工作区
  $ git rm --cached [file]
  
  # 改名文件，并且将这个改名放入暂存区
  $ git mv [file-original] [file-renamed]
  ```

- 代码提交

  ```bash
  # 提交暂存区到仓库区
  $ git commit -m [message]
  
  # 提交暂存区的指定文件到仓库区
  $ git commit [file1] [file2] ... -m [message]
  
  # 提交工作区自上次commit之后的变化，直接到仓库区
  $ git commit -a
  
  # 提交时显示所有diff信息
  $ git commit -v
  
  # 使用一次新的commit，替代上一次提交
  # 如果代码没有任何新变化，则用来改写上一次commit的提交信息
  $ git commit --amend -m [message]
  
  # 重做上一次commit，并包括指定文件的新变化
  $ git commit --amend [file1] [file2] ...
  ```

- 分支

  ```bash
  # 列出所有本地分支
  $ git branch
  
  # 列出所有远程分支
  $ git branch -r
  
  # 列出所有本地分支和远程分支
  $ git branch -a
  
  # 新建一个分支，但依然停留在当前分支
  $ git branch [branch-name]
  
  # 新建一个分支，并切换到该分支
  $ git checkout -b [branch]
  
  # 新建一个分支，指向指定commit
  $ git branch [branch] [commit]
  
  # 新建一个分支，与指定的远程分支建立追踪关系
  $ git branch --track [branch] [remote-branch]
  
  # 切换到指定分支，并更新工作区
  $ git checkout [branch-name]
  
  # 切换到上一个分支
  $ git checkout -
  
  # 建立追踪关系，在现有分支与指定的远程分支之间
  $ git branch --set-upstream [branch] [remote-branch]
  
  # 合并指定分支到当前分支
  $ git merge [branch]
  
  # 选择一个commit，合并进当前分支
  $ git cherry-pick [commit]
  
  # 删除分支
  $ git branch -d [branch-name]
  
  # 删除远程分支
  $ git push origin --delete [branch-name]
  $ git branch -dr [remote/branch]
  ```

- 标签

  ```bash
  # 列出所有tag
  $ git tag
  
  # 新建一个tag在当前commit
  $ git tag [tag]
  
  # 新建一个tag在指定commit
  $ git tag [tag] [commit]
  
  # 删除本地tag
  $ git tag -d [tag]
  
  # 删除远程tag
  $ git push origin :refs/tags/[tagName]
  
  # 查看tag信息
  $ git show [tag]
  
  # 提交指定tag
  $ git push [remote] [tag]
  
  # 提交所有tag
  $ git push [remote] --tags
  
  # 新建一个分支，指向某个tag
  $ git checkout -b [branch] [tag]
  ```

- 查看信息

  ```bash
  # 显示有变更的文件
  $ git status
  
  # 显示当前分支的版本历史
  $ git log
  
  # 显示commit历史，以及每次commit发生变更的文件
  $ git log --stat
  
  # 搜索提交历史，根据关键词
  $ git log -S [keyword]
  
  # 显示某个commit之后的所有变动，每个commit占据一行
  $ git log [tag] HEAD --pretty=format:%s
  
  # 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
  $ git log [tag] HEAD --grep feature
  
  # 显示某个文件的版本历史，包括文件改名
  $ git log --follow [file]
  $ git whatchanged [file]
  
  # 显示指定文件相关的每一次diff
  $ git log -p [file]
  
  # 显示过去5次提交
  $ git log -5 --pretty --oneline
  
  # 显示所有提交过的用户，按提交次数排序
  $ git shortlog -sn
  
  # 显示指定文件是什么人在什么时间修改过
  $ git blame [file]
  
  # 显示暂存区和工作区的差异
  $ git diff
  
  # 显示暂存区和上一个commit的差异
  $ git diff --cached [file]
  
  # 显示工作区与当前分支最新commit之间的差异
  $ git diff HEAD
  
  # 显示两次提交之间的差异
  $ git diff [first-branch]...[second-branch]
  
  # 显示今天你写了多少行代码
  $ git diff --shortstat "@{0 day ago}"
  
  # 显示某次提交的元数据和内容变化
  $ git show [commit]
  
  # 显示某次提交发生变化的文件
  $ git show --name-only [commit]
  
  # 显示某次提交时，某个文件的内容
  $ git show [commit]:[filename]
  
  # 显示当前分支的最近几次提交
  $ git reflog
  ```

- 远程同步

  ```bash
  # 下载远程仓库的所有变动
  $ git fetch [remote]
  
  # 显示所有远程仓库
  $ git remote -v
  
  # 显示某个远程仓库的信息
  $ git remote show [remote]
  
  # 增加一个新的远程仓库，并命名
  $ git remote add [shortname] [url]
  
  # 取回远程仓库的变化，并与本地分支合并
  $ git pull [remote] [branch]
  
  # 上传本地指定分支到远程仓库
  $ git push [remote] [branch]
  
  # 强行推送当前分支到远程仓库，即使有冲突
  $ git push [remote] --force
  
  # 推送所有分支到远程仓库
  $ git push [remote] --all
  ```

- 撤销（重要）

  ```bash
  # 恢复暂存区的指定文件到工作区
  $ git checkout [file]
  
  # 恢复某个commit的指定文件到暂存区和工作区
  $ git checkout [commit] [file]
  
  # 恢复暂存区的所有文件到工作区
  $ git checkout .
  
  # 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
  $ git reset [file]
  
  # 重置暂存区与工作区，与上一次commit保持一致
  $ git reset --hard
  
  # 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
  $ git reset [commit]
  
  # 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
  $ git reset --hard [commit]
  
  # 重置当前HEAD为指定commit，但保持暂存区和工作区不变
  $ git reset --keep [commit]
  
  # 新建一个commit，用来撤销指定commit
  # 后者的所有变化都将被前者抵消，并且应用到当前分支
  $ git revert [commit]
  
  暂时将未提交的变化移除，稍后再移入
  $ git stash
  $ git stash pop
  ```

- 其他

  ```bash
  # 生成一个可供发布的压缩包
  $ git archive
  ```

















### 7. 特殊场景命令及理解

- `git rabase`: 和merge一样，都是用来合并分支的，但是rebase可以把变成一条直线更加直观，其实现原理是：**你可以使用 `rebase` 命令将提交到某一分支上的所有修改都移至另一分支上，就好像“重新播放”一样。**

  

