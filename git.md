转自廖雪峰的网站：
http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000

--------------------------------------
1. 自定义git - global config
--------------------------------------
 git config -- Get and set repository or global options

$ git config --global user.name "Your
 Name"
$ git config --global user.email "email@example.com"
$ git config --global color.ui true




--------------------------------------
2. 创建版本库 repository
--------------------------------------
1) git init -- Create
 an empty Git repository or reinitialize an existing one

2) git add file -- Add
 file contents to the index

3) git commit -m "messages..." -- Record changes to the repository

Summary:


初始化一个Git仓库，使用git init命令。

添加文件到Git仓库，分两步：


第一步，使用命令git add <file>，注意，可反复多次使用，添加多个文件；


第二步，使用命令git commit，完成。


--------------------------------------
3. 版本控制-回退，管理，撤销，删除
--------------------------------------
1) git status -- Show the working tree status
   git diff -- Show changes between commits, commit and working
 tree, etc


Sumamry:








要随时掌握工作区的状态，使用git status命令。如果git status告诉你有文件被修改过，用git
 diff可以查看修改内容。


2) 版本回退 git reset
git log -- show commit logs
git log --pretty=oneline
在Git中，用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。
退回上一个版本：
git reset --hard HEAD^
git reset --hard commitid 
版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了。

Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是改变HEAD指向，然后顺便把工作区的文件更新了。

git reflog 
Git提供了一个命令git
 reflog用来记录你的每一次命令



Summary:


HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git
 reset --hard commit_id。


穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。


要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。



3) 工作区和暂存区
Working directory 
repository - .git 


工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

Git的版本库里存了很多东西，其中最重要的就是称为stage（或者叫index）的暂存区，还有Git为我们自动创建的第一个分支master，以及指向master的一个指针叫HEAD。



把文件往Git版本库里添加的时候，是分两步执行的：

第一步是用git add把文件添加进去，实际上就是把文件修改添加到暂存区；

第二步是用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支。

因为我们创建Git版本库时，Git自动为我们创建了唯一一个master分支，所以，现在，git
 commit就是往master分支上提交更改。
 

4) 管理修改
为什么Git比其他版本控制系统设计得优秀，因为Git跟踪并管理的是修改，而非文件。
提交后，用git
 diff HEAD -- readme.txt命令可以查看工作区和版本库里面最新版本的区别
每次修改，如果不add到暂存区，那就不会加入到commit中。

5) 撤销修改
git
 checkout -- file可以丢弃工作区的修改
git checkout -- <file> to discard changes in working directory

git
 reset HEAD file可以把暂存区的修改撤销掉（unstage），重新放回工作区
git reset HEAD <file> to unstage
git
 reset命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本。

Summary:


场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，版本回退，不过前提是没有推送到远程库。


6) 删除文件
git rm file, then git cmmit

summary:
命令git
 rm用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

--------------------------------------
4. 远程仓库
--------------------------------------
1) 本地Git仓库和GitHub仓库之间的传输前设置
a) 创建SSH Key。 
ssh-keygen
 -t rsa -C "youremail@example.com"
b) 登陆GitHub，打开“Account settings”，“SSH
 Keys”页面，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容.

2) 添加远程库
a) 登陆GitHub，找到“Create a new repo”按钮，创建一个新的仓库
b) git remote
 add origin git@github.com:githubaccount/reponame.git
添加后，远程库的名字就是origin，这是Git默认的叫法，也可以改成别的，但是origin这个名字一看就知道是远程库。
c) git
 push -u origin master
把本地库的内容推送到远程，用git
 push命令，实际上是把当前分支master推送到远程。
由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。
d)  git
 push origin master

Summary:


要关联一个远程库，使用命令git remote add origin git@server-name:path/repo-name.git；

关联后，使用命令git push -u origin master第一次推送master分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令git push origin master推送最新修改。



3) 从远程库克隆

git clone git@github.com:account/reponame.git

GitHub给出的地址不止一个，还可以用https://github.com/account/reponame.git这样的地址。实际上，Git支持多种协议，默认的git://使用ssh，但也可以使用https等其他协议。


summary:


要克隆一个仓库，首先必须知道仓库的地址，然后使用git clone命令克隆。

Git支持多种协议，包括https，但通过ssh支持的原生git协议速度最快。




--------------------------------------
5. 分支管理
--------------------------------------

1) 创建与合并分支


主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。
a) master分支



一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点：



b) 创建新的分支，例如dev时，Git新建了一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev上：



Git创建一个分支很快，因为除了增加一个dev指针，改改HEAD的指向，工作区的文件都没有任何变化！

c) 现在开始，对工作区的修改和提交就是针对dev分支了，比如新提交一次后，dev指针往前移动一步，而master指针不变：





d) 可以把dev合并到master上。最简单的方法，就是直接把master指向dev的当前提交，就完成了合并：




所以Git合并分支也很快！就改改指针，工作区内容也不变！



e) 合并完分支后，甚至可以删除dev分支。删除dev分支就是把dev指针给删掉，删掉后，我们就剩下了一条master分支：





COMMANDS:

i) git checkout -b dev

git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：
$ git branch dev$ git checkout dev



ii) git branch

git branch命令会列出所有分支，当前分支前面会标一个*号



iii) 把dev分支的工作成果合并到master分支上

   git checkout master
   git merge dev

git merge命令用于合并指定分支到当前分支。


iiii) git
 branch -d dev  删除分支

Summary:


Git鼓励大量使用分支：

查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>

创建+切换分支：git checkout -b <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>


2) 解决冲突

用带参数的git log也可以看到分支的合并情况：
git log --graph --pretty=oneline --abbrev-commit
Summary:



当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

用git log --graph命令可以看到分支合并图。



3) 分支管理策略


合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast
 forward合并就看不出来曾经做过合并。



4) bug 分支  git stash

Git还提供了stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作。

git stash
git stash list

恢复stash内容:
a) 一是用git
 stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；
b)  另一种方式是用git
 stash pop，恢复的同时把stash内容也删了。

summary:


修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git
 stash pop，回到工作现场。


4) feature 分支
Summary:


开发一个新feature，最好新建一个分支；

如果要丢弃一个没有被合并过的分支，可以通过git branch -D <name>强行删除。


5) 多人协作


多人协作的工作模式通常是这样：


首先，可以试图用git push origin branch-name推送自己的修改；


如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；


如果合并有冲突，则解决冲突，并在本地提交；


没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！

如果git pull提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令git
 branch --set-upstream branch-name origin/branch-name。
Summary:


查看远程库信息，使用git remote -v；


本地新建的分支如果不推送到远程，对其他人就是不可见的；


从本地推送分支，使用git push origin branch-name，如果推送失败，先用git
 pull抓取远程的新提交；


在本地创建和远程分支对应的分支，使用git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；


建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；


从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。




--------------------------------------

6. 标签管理

--------------------------------------



发布一个版本时，我们通常先在版本库中打一个标签，这样，就唯一确定了打标签时刻的版本。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照。

Git的标签虽然是版本库的快照，但其实它就是指向某个commit的指针，所以，创建和删除标签都是瞬间完成的。



1) 创建标签

git tag <name>就可以打一个新标签

git tag查看所有标签

git tag <name> <commit id>

git show <tagname>查看标签信息



Summary:


命令git tag <name>用于新建一个标签，默认为HEAD，也可以指定一个commit
 id；


git tag -a <tagname> -m "blablabla..."可以指定标签信息；


git tag -s <tagname> -m "blablabla..."可以用PGP签名标签；


命令git tag可以查看所有标签。


2) 操作标签

a) 标签删除

git tag -d tagname
b) 推送某个标签到远程

git push origin <tagname>

c) 一次性推送全部尚未推送到远程的本地标签

git push origin --tags
d) 标签已经推送到远程，要删除远程标签: 先从本地删除 -> 然后，从远程删除


git tag -d tagname
git push origin :refs/tags/tagname


Summary:

命令git
 push origin <tagname>可以推送一个本地标签；

命令git push origin
 --tags可以推送全部未推送过的本地标签；

命令git tag -d <tagname>可以删除一个本地标签；

命令git push origin :refs/tags/<tagname>可以删除一个远程标签



--------------------------------------
7. 自定义git
--------------------------------------

1) 忽略特殊文件
在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。



忽略文件的原则是：

- 忽略操作系统自动生成的文件，比如缩略图等；

- 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的.class文件；

- 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。


2) 配置别名

git config --global alias.st status



3） 搭建git 服务器


a) 安装git：
$ sudo apt-get install git

b) 创建一个git用户，用来运行git服务：
$ sudo adduser git

c) 创建证书登录：

收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到/home/git/.ssh/authorized_keys文件里，一行一个。

d) 初始化Git仓库：

先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令：
$ sudo git init --bare sample.git

Git就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的Git仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的Git仓库通常都以.git结尾。然后，把owner改为git：
$ sudo chown -R git:git sample.git

e) 禁用shell登录：

出于安全考虑，第二步创建的git用户不允许登录shell，这可以通过编辑/etc/passwd文件完成。找到类似下面的一行：
git:x:1001:1001:,,,:/home/git:/bin/bash

改为：
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell

这样，git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git用户指定的git-shell每次一登录就自动退出。

f) 克隆远程仓库：

现在，可以通过git clone命令克隆远程仓库了，在各自的电脑上运行：
$ git clone git@server:/srv/sample.gitCloning into 'sample'...warning: You appear to have cloned an empty repository.

---------------------

本文来自 Note_Book_11 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/shawnx11/article/details/50061293?utm_source=copy 
