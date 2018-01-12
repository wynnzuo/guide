# GIT管理规范

## 引言

使用Git过程中，必须通过创建分支进行开发，坚决禁止在主干分支上直接开发。review的同事有责任检查其他同事是否遵循分支规范。git是个非常好用的版本工具，不但可以在linux下环境使用，还可以在windows下使用。我们的整个代码工程需要使用这个来管理，我们自己的一些联系也可以很方便的使用它去管理，节省了很多代码维护的成本。

团队开发中，遵循一个合理、清晰的Git使用流程，是非常重要的。

否则，每个人都提交一堆杂乱无章的commit，项目很快就会变得难以协调和维护。

下面是ThoughtBot 的Git使用规范流程。
## 使用Git流程规范
![ThoughtBot 的Git使用规范流程](http://pic.cr173.com/up/2015-10/2015101908561045784.png)

### 代码库分类
根据代码库分布的位置及作用，分为以下几类：

**主库** (Trunk Repository)：位于服务端，所有开发的代码最终都要合到主库。

**个人代码库**（You GitHub Repository）：从主库fork出来，位于服务端。每个人自已开发的代码，由本地的git库push到每个人自己的个人代码库（服务端），再由个人代码库（服务端）合入主加。

**个人工作库** (Local Repository)：位于每个开发人员的开发机器，从个人代码库（服务端）clone到本地。每个开发人员开发的代码，先commit到个人工作库，再由个人工作库push到个人代码库（服务端）。

### 人员角色分类
这里说的角色，都是人员在主库上的角色。基于简化的原则，人员分为三类：

**Owner**：拥有主库的所有权限。

**Committer**：具有将开发人员的合并需求（MR）合入主库的权限。基于安全考虑，我们设置为只能通过MR的方式将代码合入主库，而不能直接push到主库。

**Developer**：只能从自己的个人代码库（服务端）提交合并代码的请求（MR），是否能够合入，由Committer进行审核。

### 基本流程

在主库已经存在的情况下，日常操作流程如下：

- 开发人员从主库fork出自己的个人代码库。
- 开发人员将自己的个人代码库clone到本地，即个人工作库。
- 开发人员在开发了新分支代码后（包括新增和修改），先将代码commit到自己的个人工作库，再由个人工作库push到个人代码库。
- 开发人员提交从个人工作库到主库的MR，Committer审核后，决定是否将MR合入主库。
- 每个开发人员从主库pull最新代码到个人工作库。

## 分支管理规范

 通常每个应用或者是二方库的代码将包括 master、develop、release、hotfix、feature分支，release、hotfix 分支的命名规则分别为：release-*，hotfix-*。feature分支的命名可以使用除master，develop，release-*，hotfix-*之外的任何名称。

- master 固定分支 (生产分支，tag做版本号)
- develop 固定分支 (开发分支,上线前合并代码,code review)
- feature 临时分支 (功能分支，每个项目一个分支)
- release 临时分支 (预发布分支)
- fixbug 临时分支 (修复BUG分支)

![分支管理](http://img.blog.csdn.net/20140917111805739?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWliaXNvZnQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### master分支
master和develop分支都是主分支，主分支是所有开发活动的核心分支。所有的开发活动产生的输出物最终都会反映到主分支的代码中。

![主分支](http://static.oschina.net/uploads/img/201302/25142843_6BPt.png)

master分支上存放的应该是随时可供在生产环境中部署的代码（Production Ready state）。当开发活动告一段落，产生了一份新的可供部署的代码时，master分支上的代码会被更新。同时，每一次更新，都有对应的版本号标签（TAG）。

把最稳定的代码放在 master 分支上（相当于 SVN 的 trunk 分支），我们不要直接在 master 分支上提交代码，只能在该分支上进行代码合并操作，例如将其它分支的代码合并到 master 分支上。

### develop分支
 develop分支是保存当前最新开发成果的分支。通常这个分支上的代码也是可进行每日夜间发布的代码（Nightly build）。因此这个分支有时也可以被称作“integration branch”。

 当develop分支上的代码已实现了软件需求说明书中所有的功能，通过了所有的测试后，并且代码已经足够稳定时，就可以将所有的开发成果合并回master分支了。对于master分支上的新提交的代码建议都打上一个新的版本号标签（TAG），供后续代码跟踪使用。

 我们日常开发中的 代码需要从 master 分支拉一条 develop 分支出来，该分支所有人都能访问，但一般情况下，我们也不会直接在该分支上提交代码，代码同样是从其它分支合并到 develop 分支上去。

 ### feature分支
使用规范：

- 可以从develop分支发起feature分支
- 代码必须合并回develop分支
- feature分支的命名可以使用除master，develop，release-*，hotfix-*之外的任何名称


feature分支（有时也可以被叫做“topic分支”）通常是在开发一项新的软件功能的时候使用，这个分支上的代码变更最终合并回develop分支或者干脆被抛弃掉（例如实验性且效果不好的代码变更）。

一般而言，feature分支代码可以保存在开发者自己的代码库中而不强制提交到主代码库里。

![image](http://img.blog.csdn.net/20140917111827267?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWliaXNvZnQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### release分支
使用规范：

- 可以从develop分支派生
- 必须合并回develop分支和master分支
- 分支命名惯例：release-*

`release`分支是为发布新的产品版本而设计的。在这个分支上的代码允许做小的缺陷修正、准备发布版本所需的各项说明信息（版本号、发布时间、编译时间等等）。通过在release分支上进行这些工作可以让develop分支空闲出来以接受新的feature分支上的代码提交，进入新的软件开发迭代周期。

当develop分支上的代码已经包含了所有即将发布的版本中所计划包含的软件功能，并且已通过所有测试时，我们就可以考虑准备创建`release`分支了。而**所有在当前即将发布的版本之外的业务需求一定要确保不能混到`release`分支之内（避免由此引入一些不可控的系统缺陷）**。

成功的派生了`release分支`，并被赋予版本号之后，develop分支就可以为“下一个版本”服务了。所谓的“下一个版本”是在当前即将发布的版本之后发布的版本。版本号的命名可以依据项目定义的版本号命名规则进行。


待上线完成后，将 `release` 分支上的代码同时合并到 `develop` 分支与 `master` 分支，并在 `master` 分支上打一个 `tag`，例如 `v1.2.0`。
### hotfix分支
使用规范：

- 可以从master分支派生
- 必须合并回master分支和develop分支
- 分支命名惯例：`hotfix-*`

除了是计划外创建的以外，hotfix分支与release分支十分相似：都可以产生一个新的可供在生产环境部署的软件版本。当生产环境中的软件遇到了异常情况或者发现了严重到必须立即修复的软件缺陷的时候，就需要从master分支上指定的TAG版本派生`hotfix`分支来组织代码的紧急修复工作。

![hotfix分支](http://img.blog.csdn.net/20140917111851338?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWliaXNvZnQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


当生产环境发现 `bug` 时，我们需要从对应的 `tag` 上（例如 `v1.2.0`）拉出一条 hotfix 分支（例如 `hotfix1.2.1`），并在该分支上做 bug 修复。待 bug 完全修复后，需将 `hotfix` 分支上的代码同时合并到 `develop` 分支与 `master` 分支。最后，在 master 分支打`tag`（例如 `v1.2.1`）。

## 管理代码规范
1. 上传内容：**保证GIT上保存的是“干净”的代码，不得有编译后再次生成的代码**，如Java字节码文件和JSP生成文件，也不能有IDE生成文件；
2. 上传注释：**必须加简要的注释，注释的内容应包含开发的模块名称以及功能描述**；
> 功能提交：[模块名称]功能描述，如：[用户模块]用户列表增加手机号字段显示；<br/>
> Bug Fix：[模块名称]Bug-编号：Bug描述，如：[用户模块]Bug-1203：用户创建保存失败已修复；

3. 上传质量：**提交和合并到分支上的代码尽量保证是自己测试通过的代码**，以免影响别的项目/同事；

## 版本号规范
格式为：`x.y.z`，其中，`x` 用于有**重大重构时**才会升级，`y` 用于**有新的特性发**布时才会升级，`z` 用于**修改了某个 bug** 后才会升级。

> 参见：https://semver.org/lang/zh-CN/

# 一个完整的步骤

根据上面的规范，整理出一个完整的步骤。

![ThoughtBot 的Git使用规范流程](http://pic.cr173.com/up/2015-10/2015101908561045784.png)

主仓库管理员工作：
1. 创建主库（`Trunk Repository`)
2. 填写`milestone`，可以用于录入月度开发计划**模块信息**。比如：2017年11月开发计划。
3. 填写`Issues`。月度开发计划**模块的子项**
4. 创建主分支 `master`、`develop`

开发者工作：
1. `fork` 主仓库到 个人远程仓库
2. `git clone `到本地工作站。
3. `git checkout -b 本地分支名x origin/远程分支名x` ，获取远程`develop`
4. 新功能开发时，`git checkout -b 新功能分支 develop`,创建新的功能分支
5. 功能完成提交代码，` git add 文件` 把文件提交到暂存区，`git commit -m '提交注释'`
6. 从主仓库`pull` `develop`分支代码。参见 **`开发者个人仓库与主库同步`** 章节
7. 合并 新功能分支到本地`develop`分支。

```
git checkout develop
git merge --no-ff 新功能分支名
```
8. 推送到个人运程仓库
9. 在个人远程仓库发起 `pull request`请求,等待合并新功能代码到主仓库 `develop`分支。

Committer管理员工作：
1. 合并开发者的`pull request`
2. 打`release`分支测试
3. 测试通过后，合并入`master`分支，并打`tag`
4. 线上有bug时，从`master`分支的`tag` 拉出一条 `hotfix` 分支,开发者修复后需将 hotfix 分支上的代码同时合并到 `develop` 分支与 `master` 分支。最后，在 `master` 分支打`tag`

# 附录
## 创建issues
### issues介绍
GitHub 的 issue 功能，对个人而言，就如同 TODO list 。

你可以把所有想要在下一步完成的工作，如 feature 添加、bug 修复等，都写成一个个的 issue ，放在上面。既可以作为提醒，也可以统一管理。

另外，每一次 commit 都可以选择性的与某个 issue **关联**。比如在 message 中添加 #n，就可以与第 n 个 issue 进行**关联**。

```
commit message title, #1
```
这个提交会作为一个 comment ，出现在编号为1的 issue 记录中。

如果添加
- close #n
- closes #n
- closed #n
- fix #n
- fixes #n
- fixed #n
- resolve #n
- resolves #n
- resolved #n

比如

```
commit message title, fix #n
```

则可以自动关闭第 n 个 issue 。

issue 可以有额外的属性：

- Labels，标签。包括 enhancement、bug、invalid 等，表示 issue 的类型，解决的方式。除了自带的以外，也可以去自定义。

- Milestone，里程碑。几经修改后，它现在已经与git tag和Github release区分开来，仅仅作为issue的一个集合。通常用来表示项目的一个阶段，比如demo、release等，保护达成这些阶段需要解决的问题。有时候，也会与版本计划重合，比如v1.0、v2.0等。issue不能设置截止时间，但是milestone可以。
> `milestone`: 可以用于录入月度开发计划。比如：2017年11月开发计划。<br/>
> `issue`: 用于录入月度开发计划的功能小项。

- Assignee，责任人。指定这个 issue 由谁负责来解决。

充分利用这些功能，让每一个 commit 的意义更加明确，可以起到了良好的**过程管理**作用，使得这个 Git 库的项目进度更加显然。而且，这也是项目后期，写文档的绝佳素材。

对团队而言，这就是一个**协作系统**。

GitHub 的 issue ，就是一个轻量级**协作系统**。它的 comment 支持 GitHub Flavored Markdown，可以进行内容丰富的交流。Git 本身就是分布式的代码版本控制软件，是为了程序员的协作而设计的。而 issue 的 Assignee 功能，就是这个在线协作系统的核心。

### issue作用
- 整理思路，快速开发代码
- 方便后续出现线上问题，快速定位
- 有类似功能开发时，方便别人借鉴，和自己快速回忆
- 相互学习

### 创建Issues流程
- 创建Issues
- 找有相关开发经验同事评审
- 影响范围较大的Issues必须拉上大家一起评审


### issues规范
- 需求概述
- 难点，解决方案
- 概要设计
- 详细设计


## 开发者个人仓库与主库同步

### 为fork的库配置原始远程库
这些命令在linux下使用，同样在wondows下的git bash也适用。如果你已经配置过原始远程库的路径，可以跳过这一部分，执行获取原始仓库分支和对应的提交。
为了与原始仓库同步，首先需要在Git配置一个远程指向上层仓库upstream repository。
1. 打开终端
2. 首先在终端中配置原仓库的位置。进入项目目录，执行如下命令：查看你的远程仓库的路径：

```
$ git remote -v
```
3. 配置原仓库的路径 ：

```
$ git remote add upstream 远程仓库地址
```

4. 再次查看远程目录的位置：

```
$ git remote -v
```

### 与远程原始库同步
1. 打开终端.
2. 改变当前工作目录到本地的仓库。
3. 获取原始仓库分支和对应的提交，分支`master`的提交会保存到本地分支，`upstream/master` ：

```
$ git fetch upstream
```

4. 切换到你fork仓库本地的master分支：

```
$ git checkout master
```

5. 把原始upstream/master的改变合并到你本地的master分支。这会使你fork的分支master 与上层仓库upstream repository同步，而不会丢失你本地所做的改变：

```
$ git merge upstream/master
```

6. 把自己账户下的远程仓库同步到自己的本地仓库，即推送自己的本地仓库到自己的远程仓库：

```
$ git push
```

到此，本地仓库和自己远程仓库都已同步到原始仓库，并且保留了你自己所做的修改。