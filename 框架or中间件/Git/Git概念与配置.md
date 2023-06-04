## 1. Git基本概念
<img src="D:\Project\IT notes\框架or中间件\Git\img\Git大致架构.png" style="width:700px;height:200px;" />
Git存在四个逻辑区域：
- 工作区`workspace`：工作区，就是你平时存放项目代码的地方
- 暂存区`index/stage`：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
- 本地仓库`repository`：仓库区，就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中`HEAD`指向最新放入仓库的版本
- 远程仓库`remote`：远程仓库，托管代码的服务器

<img src="D:\Project\IT notes\框架or中间件\Git\img\Git文件状态.png" style="width:700px;height:300px;" />
Git中的文件存在四个状态：
- `untracked`：未跟踪，此文件在文件夹中，但并没有加入到`git`库，不参与版本控制，通过`git add`状态变为`Staged`
- `unmodify`：文件已经入库，未修改，即本地仓库中的文件快照内容与工作区文件夹中完全一致。这种类型的文件有两种去处，如果它被修改，而变为`Modified`。如果使用`git rm`移出版本库，则成为`Untracked`文件
- `modified`：文件已修改，仅仅是修改，并没有进行其他的操作。这个文件也有两个去处，通过`git add`可进入暂存`staged`状态，使用`git checkout`则丢弃修改过，返回到`unmodify`状态, 这个`git checkout`即从库中取出文件, 覆盖当前修改
- `staged`：暂存状态，执行`git commit`则将修改同步到库中，这时库中的文件和本地文件又变为一致，文件为`Unmodify`状态.。执行`git reset HEAD filename`取消暂存，文件状态为`Modified`

<img src="D:\Project\IT notes\框架or中间件\Git\img\Git文件状态与命令.png" style="width:700px;height:400px;" />

## 2. Git主要命令
### 1. 新建代码库
```shell
# 初始化本地仓库，在当前目录下生成 .git 文件夹
git init
# 新建一个目录，将其初始化为Git代码库
git init [project-name]

# 默认在当前目录下创建和版本库名相同的文件夹并下载版本到该文件夹下
git clone <远程仓库的网址>

# 指定本地仓库的目录
git clone <远程仓库的网址> <本地目录>

# -b 指定要克隆的分支，默认是master分支
git clone <远程仓库的网址> -b <分支名称> <本地目录>
```

### 2. 查看文件状态
```shell
#查看指定文件状态
git status [filename]
#查看所有文件状态
git status
```

### 3. 工作区<-->暂存区
```shell
# 添加指定文件到暂存区
git add [file1] [file2] ...
# 添加指定目录到暂存区，包括子目录
git add [dir]
# 添加当前目录的所有文件到暂存区
git add .
#当我们需要删除暂存区或分支上的文件, 同时工作区也不需要这个文件了, 可以使用（⚠️）
git rm file_path
#当我们需要删除暂存区或分支上的文件, 但本地又需要使用, 这个时候直接push那边这个文件就没有，如果push之前重新add那么还是会有。
git rm --cached file_path
#直接加文件名   从暂存区将文件恢复到工作区，如果工作区已经有该文件，则会选择覆盖
#加了【分支名】 +文件名  则表示从分支名为所写的分支名中拉取文件 并覆盖工作区里的文件
git checkout

# 把指定的文件添加到暂存区中
git add <文件路径>

# 添加所有修改、已删除的文件到暂存区中
git add -u [<文件路径>]
git add --update [<文件路径>]

# 添加所有修改、已删除、新增的文件到暂存区中，省略 <文件路径> 即为当前目录
git add -A [<文件路径>]
git add --all [<文件路径>]

# 查看所有修改、已删除但没有提交的文件，进入一个子命令系统
git add -i [<文件路径>]
git add --interactive [<文件路径>]
```

### 4. 工作区<-->本地仓库
```shell
# 把暂存区中的文件提交到本地仓库，调用文本编辑器输入该次提交的描述信息
git commit

# 把暂存区中的文件提交到本地仓库中并添加描述信息
git commit -m "<提交的描述信息>"

# 把所有修改、已删除的文件提交到本地仓库中
# 不包括未被版本库跟踪的文件，等同于先调用了 "git add -u"
git commit -a -m "<提交的描述信息>"

# 修改上次提交的描述信息
git commit --amend

# 重置暂存区，但文件不受影响
# 相当于将用 "git add" 命令更新到暂存区的内容撤出暂存区，可以指定文件
# 没有指定 commit ID 则默认为当前 HEAD
git reset [<文件路径>]
git reset --mixed [<文件路径>]

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
git reset <commit ID>
git reset --mixed <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件未修改
# 相当于调用 "git reset --mixed" 命令后又做了一次 "git add"
git reset --soft <commit ID>

# 将 HEAD 的指向改变，撤销到指定的提交记录，文件也修改了
git reset --hard <commit ID>

# 生成一个新的提交来撤销某次提交
git revert <commit ID>
```

### 5. 本地仓库<-->远程仓库
```shell
# 取回远程仓库的变化，并与本地分支合并
# 首先会执行 `git fetch`，然后执行 `git merge`，把获取的分支的 HEAD 合并到当前分支
git pull
# 把本地仓库的分支推送到远程仓库的指定分支
git push <远程仓库的别名> <本地分支名>:<远程分支名>

# 删除指定的远程仓库的分支
git push <远程仓库的别名>:<远程分支名>
git push <远程仓库的别名> --delete <远程分支名>

# 列出已经存在的远程仓库
git remote

# 列出远程仓库的详细信息，在别名后面列出URL地址
git remote -v
git remote --verbose

# 添加远程仓库
git remote add <远程仓库的别名> <远程仓库的URL地址>

# 修改远程仓库的别名
git remote rename <原远程仓库的别名> <新的别名>

# 删除指定名称的远程仓库
git remote remove <远程仓库的别名>

# 修改远程仓库的 URL 地址
git remote set-url <远程仓库的别名> <新的远程仓库URL地址>

# 将远程仓库所有分支的最新版本全部取回到本地
git fetch <远程仓库的别名>

# 将远程仓库指定分支的最新版本取回到本地
git fetch <远程主机名> <分支名>
```

### 6. 分支
```shell
# 列出本地的所有分支，当前所在分支以 "*" 标出
git branch

# 列出本地的所有分支并显示最后一次提交，当前所在分支以 "*" 标出
git branch -v

# 创建新分支，新的分支基于上一次提交建立
git branch <分支名>

# 修改分支名称
# 如果不指定原分支名称则为当前所在分支
git branch -m [<原分支名称>] <新的分支名称>
# 强制修改分支名称
git branch -M [<原分支名称>] <新的分支名称>

# 删除指定的本地分支
git branch -d <分支名称>

# 强制删除指定的本地分支
git branch -D <分支名称>

# 切换到已存在的指定分支
git checkout <分支名称>

# 创建并切换到指定的分支，保留所有的提交记录
# 等同于 "git branch" 和 "git checkout" 两个命令合并
git checkout -b <分支名称>

# 创建并切换到指定的分支，删除所有的提交记录
git checkout --orphan <分支名称>

# 替换掉本地的改动，新增的文件和已经添加到暂存区的内容不受影响
git checkout <文件路径>

# 把指定的分支合并到当前所在的分支下
git merge <分支名称>
```

### 7. 其他命令
```shell
# 显示当前的Git配置
git config --list
# 编辑Git配置文件
git config -e [--global]
#初次commit之前，需要配置用户邮箱及用户名，使用以下命令：
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
#调出Git的帮助文档
git --help
#查看某个具体命令的帮助文档
git +命令 --help
#查看git的版本
git --version

# 打印所有的提交记录
git log

# 打印从第一次提交到指定的提交的记录
git log <commit ID>

# 打印指定数量的最新提交的记录
git log -<指定的数量>

# 重命名指定的文件或者文件夹
git mv <源文件/文件夹> <目标文件/文件夹>

# 移除跟踪指定的文件，并从本地仓库的文件夹中删除
git rm <文件路径>

# 移除跟踪指定的文件夹，并从本地仓库的文件夹中删除
git rm -r <文件夹路径>

# 移除跟踪指定的文件，在本地仓库的文件夹中保留该文件
git rm --cached

```


