### 导读
在上一篇文章[\[前端漫谈\]一巴掌拍平Git中的各种概念](https://juejin.im/post/5c395796518825261e1f012a)中，描述了 Git 的一些概念，但是太过虚化，描述的都是一些概念和命令。这篇文章结合实际场景，主要描述我在项目实践中使用 `Git` 管理项目、团队协作的一些经验。包括  1）`merge` 和 `rebase` 使用的区别和选择；2）多人团队合作开发流程；3）标准化 `commit message`；4）`commit` 精细化管理等。这些都是为项目的健壮发展和代码的精细管理所流的泪累积出来的。

### 0x000 前言
由上一片文章[\[前端漫谈\]一巴掌拍平Git中的各种概念](https://juejin.im/post/5c395796518825261e1f012a)中，可以知道，`Git` 世界就像一个 宇宙，每一个 commit 都是一颗星球，而 `commitId` 就是星球的坐标，`branch` 是一条条的航线，穿过无数的 星球，`tag` 是航线上重要的星球，可能是供给站，可能是商业中心，而 `HEAD` 则是探索号飞船，不断向前探索。中间可能会有岔道，但是永远有一个真正的方向等待勇敢的船长。

![](https://user-gold-cdn.xitu.io/2019/12/5/16ed677489e062cd?w=526&h=1040&f=png&s=102569)

### 0x001 `merge` 还是 `rebase`

`merge` 还是 `rebase`，这是经久不衰的讨论点。但是这里我不去争论孰优孰略，我只说我在不同场景的实践。

#### 1. merge

我通常使用 `merge` 来将多个分支合并到当前分支，比如要发布的时候，将多个功能分支合并到带发布分支：

已知：`feat/A`、`feat/B`、`feat/C`，是从主分支新建的功能分支，`feat/B`和`feat/C`都修改了文件`1`。

- 新建待发布分支：
    ```
    # 从主分支新建分支 pub/191205
    $ git checkout -b pub/191205
    Switched to a new branch 'pub/191205'
    ```
- 合并`feat/A`到`pub/191205`：
    ```shell
    $ git merge feat/A
    Updating 53ab8fd..e443dd4
    Fast-forward
     featA | 1 +
     1 file changed, 1 insertion(+)
     create mode 100644 featA
    ```

`pub/191205` 和 `feat/A` 都是从主分支新建，所以 `pub/191205` 指向的 `commit` 是 `feat/A` 的祖先，当把 `feat/A` 合并到`pub/191205`的时候，会发生快速合并（Fast-forward）。不会新建一个合并节点（当然也可以通过`--no-ff(no-fast-forward)`来强制生成一个节点）：

```shell
# 查看 log
$ git log --oneline
e443dd4 (HEAD -> pub/191205, feat/A) feat: a
53ab8fd (master) chore: first commit
```

- 合并`feat/B`到`pub/191205`
    ```shell
    $ git merge feat/B
    # 进入 vim 填写合并信息
    Merge made by the 'recursive' strategy.
     1     | 2 +-
     featB | 1 +
     2 files changed, 2 insertions(+), 1 deletion(-)
     create mode 100644 featB
    ```

`feat/B`是从主分支新建的分支，`pub/191205`原本指向的也是`feat/B`的祖先，但是因为已经和`feat/A`合并了，所以`pub/191205`不再是`feat/B`的祖先。因此，`pub/191205`和`feat/B`的合并不再是快速合并（Fast-forward），而是`Merge made by the 'recursive' strategy.`。会产生一个新的节点：

```shell
$ git log --oneline
5d0ee9b (HEAD -> pub/191205) Merge branch 'feat/B' into pub/191205
d7773d6 (feat/B) feat: b
e443dd4 (feat/A) feat: a
53ab8fd (master) chore: first commit
```

- 合并`feat/C`到`pub/191205`
    ```shell
    $ git merge feat/C
    Auto-merging 1
    CONFLICT (content): Merge conflict in 1
    Automatic merge failed; fix conflicts and then commit the result.
    ```
`feat/C`和`fix/B`修改了相同文件，所以产生冲突，因此，会提示解决冲突。这时候查看状态，可以发现，处于`you have unmerged paths`状态：
    ```
    $ git status
    On branch pub/191205
    You have unmerged paths.
      (fix conflicts and run "git commit")
      (use "git merge --abort" to abort the merge)
    
    Changes to be committed:
    	new file:   featC
    
    Unmerged paths:
      (use "git add <file>..." to mark resolution)
    	both modified:   1
    ```
这时候可以执行`git merge --abort`放弃继续合并，恢复合并之前的状态。也可以解决冲突之后，执行`git merge`。这里选择解决冲突：
    ```
    # 解决冲突
    $ git commit
    # 进入 vim 编写 message
    [pub/191205 98d63aa] Merge branch 'feat/C' into pub/191205
    ```

`feat/C`是从主分支新建的分支，`pub/191205`原本指向的也是`feat/C`的祖先，但是因为已经和`feat/A`、`feat/B`合并了，所以`pub/191205`不再是`feat/C`的祖先。因此，`pub/191205`和`feat/C`的合并不再是快速合并（Fast-forward），会产生一个新的节点：
    ```
    $ git log --oneline
    98d63aa (HEAD -> pub/191205) Merge branch 'feat/C' into pub/191205
    5d0ee9b Merge branch 'feat/B' into pub/191205
    d7773d6 (feat/B) feat: b
    52dd922 (feat/C) feat: c
    e443dd4 (feat/A) feat: a
    53ab8fd (master) chore: first commit
    ```

历史如下：


![](https://user-gold-cdn.xitu.io/2019/12/5/16ed6948b2bf099f?w=1124&h=1002&f=png&s=112165)

#### 2. rebase
注：`rebase` 的功能很强大，这里先介绍和 `merge` 相对应的功能。

我通常用它来和主分支同步，比如一个新版本发布，主分支比我当前的功能分支超前，我使用`rebase`将当前分支和主分支“合并（变基）”。

已知：`feat/A`、`feat/B` 是从主分支新建，`feat/A`开发完成之后合并到主分支。`feat/B`继续开发，需要将`master`的功能合并到当前分支上，使用`merge`可以这么做：
- 切换到 feat/B
    ```shell
    $ git switch feat/B
    Switched to branch 'feat/B'
    ```
- 将 master 合并到 feat/B
    ```shell
    $ git merge master
    # 进入 vim 编写 message
    Merge made by the 'recursive' strategy.
     featA | 1 +
     1 file changed, 1 insertion(+)
     create mode 100644 featA
    ```
- 查看状态
    ```
    $ git log
    b4f178e (HEAD -> feat/B) Merge branch 'master' into feat/B
    d7773d6 feat: b
    e443dd4 (pub/191205, master, feat/A) feat: a
    53ab8fd chore: first commit
    ```
因为`master`合并了`feat/A`，因此不再是`feat/B`的祖先节点，不会进行快速合并（Fast-forward），会产生一个新的节点。历史如下

![](https://user-gold-cdn.xitu.io/2019/12/5/16ed691213b8e3c1?w=892&h=788&f=png&s=80873)

这么做是可以，但是我不喜欢这个合并产生的节点，所以我选择使用`rebase`：
- 恢复到合并`feat/B`之前
    ```
    $ git reset e443dd4 --hard
    HEAD is now at e443dd4 feat: a
    ```
- 使用`rebase`“合并（变基）”`master`
    ```
    $ git rebase master
    git rebase master
    First, rewinding head to replay your work on top of it...
    Applying: feat: b
    ```
- 查看历史：
    ```
    $ git log --oneline
    ef3450c (HEAD -> feat/B) feat: b
    e443dd4 (pub/191205, master, feat/A) feat: a
    53ab8fd chore: first commit
    ```
可以发现没有新的节点产生，但是`rebase`的操作过程并不只是不产生一个合并节点而已，它的中文翻译是`变基`，听起来很 Gay 的样子。但它的意思是“改变基础”。那改变的是什么基础呢？就是这个分支`checkout`出来的`commit`，原本`feat/B`是从`master`中`checkout`出来的，但是使用`git rebase master`之后，就会以`master`最新的节点作为`feat/B`分支的基础。就像`feat/B`上所有的`commit`都是基于最新的`master`提交的。

历史如下：

![](https://user-gold-cdn.xitu.io/2019/12/5/16ed6a0f442c3c24?w=898&h=780&f=png&s=86252)

由于`rebase`之后，`master`始终是`feat/B`的祖先节点，因此，之后将`feat/B`合并到`master`将执行`Fast-Farword`，不会产生冲突（如果有冲突，`rebase`的时候就需要解决了），也不会产生新节点。

#### 3. merge 还是 rebase

`merge`还是`rebase`，有人提倡不要使用`rebase`，应该`rebase`改变了历史（在上一小节中一直在改变分支的启始节点），有人提倡使用`merge`，保留完整的历史。

我是这么做的，在私有的分支上，我始终使用`rebase`将主分支的更新合并到私有的分支上（后面还有很多使用`rebase`的操作，都是在私有的分支，这里的私有的分支，指的是只有自己使用的分支，一旦分享出去，或者有人基于你的分支开发，那就不再是私有），而在将自己的分支合并到其他分支（主分支或者待发布分支），则使用`merge`。

- 切换到主分支：
    ```
    $ git switch mater
    Switched to branch 'master'
    ```
- 将`feat/B` 合并到主分支
    ```
    $ git merge feat/B
    Updating e443dd4..ef3450c
    Fast-forward
     1     | 2 +-
     featB | 1 +
     2 files changed, 2 insertions(+), 1 deletion(-)
     create mode 100644 featB
    ```

这样在长时间开发(`master`中间发布过`n`多版本)的`feat/B`就不会有无数乱七八糟的分支合并。而在`master`也不会存在`rebase`导致的历史变更后果。

历史如下：


![](https://user-gold-cdn.xitu.io/2019/12/5/16ed6b65459f8809?w=444&h=894&f=png&s=59133)

准则：**不要对在你的仓库外有副本的分支执行变基**。

如果你遵循这条金科玉律，就不会出差错。 否则，人民群众会仇恨你，你的朋友和家人也会嘲笑你，唾弃你。-- [3.6 Git 分支 - 变基 - 变基的风险](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)


### 0x002 多人合作开发

#### 1. 新功能开发

**开发方式**
新功能开发的时候从主分支新建新分支，所有该功能的开发工作都在这个分支上完成。如果主分支有新的发布，使用`rebase`同步主分支功能：

![](https://user-gold-cdn.xitu.io/2019/12/6/16edb1768e484ec4?w=446&h=992&f=png&s=73493)

**名称规范**
功能分支的命名方式是`feat/${name}_${featName}`，它的构成如下：
- 常量`feat`：表示这是一个功能分支
- 变量`name`：你的名字
- 变量`featName`：功能名字
好处是见名知意，一看就知道是功能分支，是谁负责，是什么功能


#### 2. bug 修复及其发布

**开发方式**
`bug`修复大体上和新功能的开发类似，但是`bug`修复一般时间短，立马上线。
`bug`修复从主分支新建新分支，所有的`bug`修复工作都在这个分支上完成。如果主分支有新的发布，使用`rebase`同步主分支功能（这个步骤其实和新功能开发一样）：

![](https://user-gold-cdn.xitu.io/2019/12/6/16edb1eb4c460d9f?w=460&h=958&f=png&s=73919)

**名称规范**
`bug`修复分支的命名方式是`hotfix/${name_${bugName}}`，它的构成如下：
- 常量`hotfix`：表示这是一个功能分支
- 变量`name`：你的名字
- 变量`bugName`：`bug`名字
好处是见名知意，一看就知道是`bug`修复分支，是谁负责，是什么`bug`

**bug 发布**
`bug`发布可以直接推送到待发布版本分支，比如`1.1.1`，然后`CodeReview`（如果有），然后合并主分支部署上线。

完整过程如下：

![](https://user-gold-cdn.xitu.io/2019/12/6/16edb209869268ef?w=448&h=1070&f=png&s=77502)

#### 2.5 stash

一般我们修复`bug`的时候都在开发新功能，也就是在`feat/*`上，这时候如何快速进入`bug`修复状态呢？可以保存当前代码，提交`commit`，但是这时候会有一些问题，比如，1）当前的代码并未完成，并不想提交；2）`commit`有钩子，比如`ESLint`，必须修复语法问题才能提交。

这时候就是使用`stash`了。`stash`可以将当前工作区和暂存区的内容暂时保存起来，之后再使用。

如下：
- 开发功能中
    ```
    $ echo "this is a feat" >> feat.txt
    ```
- 收到`bug`通知
    ```
    $ git stash
    Saved working directory and index state WIP on master: ef3450c feat: b
    
    $ git switch master
    $ git checkout -b hotfix/bugA
    Switched to a new branch 'hotfix/bugA'
    ```
- 修复`bug`之后
    ```
    $ git switch feat/A
    $ git stash pop
    On branch hotfix/bugA
    Changes to be committed:
      (use "git restore --staged <file>..." to unstage)
    	new file:   feat.txt
    
    Dropped refs/stash@{0} (32cf119fc1dcbe7088d1a12e290b868d6707526d)
    ```

`stash`命令有一整套完整的增删改查指令，可以查看[git-pro 7.3 Git 工具 - 储藏与清理](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%82%A8%E8%97%8F%E4%B8%8E%E6%B8%85%E7%90%86)了解更多。


#### 3. 新功能发布

新功能发布和`bug`发布有些不同，1）可能会有多个功能共同发布，需要提前合并，避免大冲突；2）可能有`bug`修复需要插队。3）可能需要等待后端发布，时间长。

因为（2），所有无法像`bug`发布那样直接推送到版本分支，等待发布。因为在真正发布之前，是无法知道准确版本的。

因为（1）、（3），所以需要提前合并，所以引入一个“日期分支”的概念，即以日期为分支名，比如`pub/191205`。

所以发布过程如下：

![](https://user-gold-cdn.xitu.io/2019/12/6/16edb4372a5e19d2?w=1218&h=1282&f=png&s=153896)


(其实我还画了一张贼复杂的图，把自己都恶心了，有空还是画个动态图吧（没空）)

![](https://user-gold-cdn.xitu.io/2019/12/6/16edb44bd94d1ba9?w=1204&h=1230&f=png&s=226744)

### 0x003 标准化 commit message
标准化 commit message 可以参考[阮一峰 - Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)。（阮一峰真的是写博客跨不过去的坎😢，写啥都可以引用他）

1. 安装`commitizen`
```
$ npm install -g commitizen
```
2. 在项目目录中初始化`commitizen`
```
$ commitizen init cz-conventional-changelog --save --save-exact
```
3. 使用`git cz`代替`git commit`，以下是我常用的类型：
```
$ git cz
feat:       一个新功能
fix:        一个 bug 修复
docs:       只改变文档
refactor:   改变代码但是不添加或者修复功能，我一般用于优化或者重构
test:       添加测试代码
chore:      其他改变
style:      样式更新
```

### 0x004 commit 精细化管理

首先是为什么？为什么要管理`commit`，`commit`有啥好管理的？

在以前，我觉得`git`是用来记录代码操作的，我对代码的任何操作都应该被记录下来，而且就像历史一样，是神圣不可侵犯的。通过`git`历史，我必须要可以知道我在某一刻做了什么，就算我在一个`commit`添加了一行代码，然后在后一个`commit`删除了它，我也必须可以从`log`中看出来。

所以我的`git`历史中充满了各种无效的`commit`，因为有时候真的不知道如何为命名。

但是后来，我就想通了，我使用`git`的目标是不是为了记录，而是为了项目的稳定发展。只要实现了这个目的，手段不是问题，更何况`git`只是一个工具，工具是用来用的，不是用来供奉的。让自己快乐快乐才叫做意义。

所谓的管理`commit`，就是对`commit`执行增、删、改、拆操作。会在后面的章节一一列出。而管理的目的，是为了让每一个`commit`都有存在的意义，让`Git`成为项目管理真正的后盾。

后面的例子将同时提供`SourceTree`的操作，命令式可以看上一篇文章[\[前端漫谈\]一巴掌拍平Git中的各种概念](https://juejin.im/post/5c395796518825261e1f012a)。

#### 1. 排序 commit
场景：完成登陆页面之后，提交一个`commit`，`message`是`feat(登陆): 完成登陆页面`。然后进入其他功能的开发，之后又回到登陆页面的开发。提交记录如下：

![](https://user-gold-cdn.xitu.io/2019/12/7/16edf03cfad7caa6?w=1238&h=334&f=png&s=82049)

我们有两个`feat(登陆)`或者多个相关的的`commit`，但是却分布于不同的地方，假设每一个`feat(登陆)`只会与前一个`feat(登陆)`有文件修改的交集，那么我们希望`feat(登陆)`相关的功能可以放在一起。如下：

![](https://user-gold-cdn.xitu.io/2019/12/7/16edf0547a4990b8?w=1022&h=330&f=png&s=79595)

如何实现：

![](https://user-gold-cdn.xitu.io/2019/12/7/16edf3a1a97da11d?w=1404&h=992&f=gif&s=2442320)


#### 2. 合并 commit
场景：完成登陆页面之后，提交一个`commit`，`message`是`feat(登陆): 完成登陆页面`。然后进入其他功能的开发，后来发现登陆有一个文案错误，继续修改，完成之后又提交一个`commit`，`message`为`feat(登陆): 修改文案`。提交记录如下：

![](https://user-gold-cdn.xitu.io/2019/12/7/16edf22b6231f2f2?w=1280&h=360&f=png&s=84521)

在我看来，`feat(登陆): 修改文案`这个`commit`的存在是不应该的，比如，1）如果有一天我们需要单独上“登陆”功能，还有可能被遗漏；2）单独占据一个`commit`可能只是为了修复一个符号问题，在回溯历史的时候有不必要的浪费。也就是我希望一个`commit`它是独立的，不依赖后续`commit`的存在。

所以我希望将这两个`commit`合并：

![](https://user-gold-cdn.xitu.io/2019/12/7/16edf38157688d94?w=1278&h=280&f=png&s=67757)

操作过程：

![](https://user-gold-cdn.xitu.io/2019/12/7/16edf38ae5ca01bd?w=1404&h=992&f=gif&s=4688159)

#### 3. 更新 commit

更新`commit`的场景有两个：
1. 更新`message`
    - 正好上面有一个不符合标准的`message`:
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf4714e5d03d0?w=1276&h=272&f=png&s=68011)
    - 我希望改为：
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf4b622541f2d?w=1278&h=252&f=png&s=67379)
    - 操作说明
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf4bb49a69508?w=1342&h=948&f=gif&s=1526061)
2. 更新、添加、删除
    - 1）有时候我们会通过修改某个变量来做一些测试，然后提交的时候突然发现忘记改回来；2）忘记或者误添加文件；3）忘记删除文件。这时候可以通过再创建一个`commit`再改回来，但是误添加的文件依旧会在历史中存在，占据一定的空间。我们可以根据上面的“合并”方式合并`commit`消除影响，也可以一步到位：
    - `feat(mine): 个人中心`提交中有一个`mime.html`文件，我希望删掉`bad line`；还有一个`mineBad.bad`这么一个看起来坏坏的文件，我希望删除它。
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf5aaf953830b?w=1770&h=1250&f=png&s=731638)    
    - 操作过程（略复杂）:
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf676a6ddd303?w=1342&h=948&f=gif&s=3526906)

#### 4. 增加/分离 commit

1. 增加一个`commit`的意义其实不大，在更新`commit`的过程中我们选择的是`更正上一次提交`，也就是`git commit --amend`，但是如果我们不选择，而是创建一个提交，其实就是增加一个`commit`了。
    - 我希望在`feat(mine): 完成个人中心`和`feat(main): 完成主页`中间添加一个`commit`，可以通过新建一个`commit`然后之后通过前面的`排序`手段来做到，也可以一步到位：
    
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf6bb25577ad6?w=1286&h=268&f=png&s=69965)
    - 操作过程（和前面差不多，只是不选择更正上一次提交）：
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf768b80ad4dc?w=1351&h=955&f=gif&s=4292963)

2. 分离`commit`的意义重大，有时候我们希望只发布一个功能，却发现这个功能的`commit`中包含我们不希望发布的另一个功能，可能是因为本来要放到两个`commit`的功能误添加到一个`commit`，
    - 我们有一个`feat(detail): 完成详情页`的`commit`，却不小心把`other`的功能给包含进去了，这时候我希望只发布`detail`页面，因此，对于`commit`的分离是必须的：

    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf82a488740e6?w=1770&h=1250&f=png&s=713635)
    
    - 操作过程
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf9ed42f92d38?w=1760&h=1244&f=gif&s=5030731)
#### 5. 删除 commit

当我们做了一次修改后来发现这个修改没有必要，就可以删除这个`commit`，但是不推荐，除非真的确认。
- 在`feat(detail): 完成详情页面`后面做了一个不需要的提交：
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf9be193a7187?w=1770&h=1250&f=png&s=744855)

- 删除步骤
    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edf9fdcc03a779?w=1760&h=1244&f=gif&s=2794303)

#### 7. cherry-pick

有时候，我们需要发布一个分支中的几个功能，比如我们在一次统一优化中修复了 5 个 bug，做了 5 个优化，但是其中几个并没有通过验证：

- `refactor/A` 分支中有  3 个commit，通过了 2（用 ok 标记） 个
    
    ![](https://user-gold-cdn.xitu.io/2019/12/9/16ee663fbbcd1c9e?w=1274&h=362&f=png&s=78515)

- `fix/A` 分支中有 3 个 commit，通过了 2（用 ok 标记） 个
    ![](https://user-gold-cdn.xitu.io/2019/12/9/16ee664c7d6475c3?w=1276&h=360&f=png&s=72370)

我们只能发布通过的 bug 修复和优化（标注了 ok 的），而这些修复和优化并不一定在哪个分支，是随机分布的：

![](https://user-gold-cdn.xitu.io/2019/12/9/16ee6655e6bc1f52?w=1270&h=374&f=png&s=86123)

在这种场景中，虽然可以用分支去处理，但是有点麻烦，这个时候 cherry-pick 是最好的工具。
- 操作过程
    
    ![](https://user-gold-cdn.xitu.io/2019/12/9/16ee6663319b8763?w=1760&h=1244&f=gif&s=3801658)

### 0x005 reflog
上面的很多操作都涉及到历史的操作，用普通的 revert 或者 reset 是无法消除影响的，只有在清楚这些命令的原理和本质的情况下才应该使用这些命令。但是对于这些操作也是有办法处理的，那就是 `reflog`：

在`git`中，所有的操作都会被记录下来，比如`切换分支`、`合并分支`等，可以使用 `reflog`查看这个记录，下面是`cherry-pick`例子产生的记录：
```
$ git reflog
# 执行 cherry-pick，一共 4 个 commit
b185e09 (HEAD -> pub/191206) HEAD@{0}: cherry-pick: feat(A): ok
dd67bf5 HEAD@{1}: cherry-pick: fix(A): ok
1d0237e HEAD@{2}: cherry-pick: feat(A): ok
51f808e HEAD@{3}: cherry-pick: refactor(A): ok
### 从 master 新建分支 pub/191206
a48cdd2 (master) HEAD@{4}: checkout: moving from master to pub/191206
```

![](https://user-gold-cdn.xitu.io/2019/12/7/16edfbea83843837?w=1288&h=658&f=png&s=148098)

如果我们撤销`cherry-pick`，可以执行以下命令：
```
$ git reset --hard HEAD@{4}
HEAD is now at a48cdd2 chore: 项目初始化
```
- 就没啦
![](https://user-gold-cdn.xitu.io/2019/12/7/16edfbf94bd6a838?w=1276&h=536&f=png&s=108387)
- 再次查看`reflog`，多了一条记录
    ```
    $ git reflog
    a48cdd2 (HEAD -> pub/191206, master) HEAD@{0}: reset: moving to HEAD@{4}
    b185e09 HEAD@{1}: cherry-pick: feat(A): ok
    dd67bf5 HEAD@{2}: cherry-pick: fix(A): ok
    1d0237e HEAD@{3}: cherry-pick: feat(A): ok
    51f808e HEAD@{4}: cherry-pick: refactor(A): ok
    a48cdd2 (HEAD -> pub/191206, master) HEAD@{5}: checkout: moving from master to pub/191206
    ```
- 撤销`cherry-pick`又后悔啦
    ```
    $ git reset --hard HEAD@{1}
    HEAD is now at b185e09 feat(A): ok
    ```
- 效果

    ![](https://user-gold-cdn.xitu.io/2019/12/7/16edfc1bb3507750?w=1278&h=668&f=png&s=149821)

- 又又后悔啦！！！滚

### 0x006 总结

1. 勿忘初心，砥砺前行。我们一开始使用`git`是为了更好的辅助项目，而不是让项目更加复杂，如果不使用这些方式可以让你的项目更加简单，那就不要用，为了使用`git`而使用`git`，不如不使用。

2. 要理解工具的原理，再去使用，不要盲目。使用上面的命令之前，务必了解这些命令或者操作背后发生了什么。


### 0x007 后记

- 我一直在寻找一种好的表达方式，从截图标注、绘图，到 `gif` 等，希望可以将文章讲的更加透彻。现在看来，可能还是 `gif` 比较好。

- 写一篇文章真的有点难啊，构思、布局、实验、总结，每一步都需要花很大的功夫，但是一篇精心总结的文章，对自己的帮助还是很大的，希望对各位也有帮助把。

### 0x008 资源

- [git pro](https://git-scm.com/book/zh/v2)
- [阮一峰 - Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
- [\[前端漫谈\]一巴掌拍平Git中的各种概念](https://juejin.im/post/5c395796518825261e1f012a)
- [SourceTree](https://www.sourcetreeapp.com/)：Git 可视化工具
- [ProcessOn](https://www.processon.com/)：绘图工具
- [截图](https://apps.apple.com/cn/app/%E6%88%AA%E5%9B%BE-jietu-%E5%BF%AB%E9%80%9F%E6%A0%87%E6%B3%A8-%E4%BE%BF%E6%8D%B7%E5%88%86%E4%BA%AB%E7%9A%84%E6%88%AA%E5%B1%8F%E5%B7%A5%E5%85%B7/id1059334054?mt=12)：截图和标注
- [GIF Brewery 3 by Gfycat](https://apps.apple.com/cn/app/gif-brewery-3-by-gfycat/id1081413713?mt=12)：GIF 录制和标注工具
- [IDEA](http://www.jetbrains.com/idea/)：IDE

### 0x009 带货

最近发现一个好玩的库，作者是个大佬啊--[基于 React 的现象级微场景编辑器](https://juejin.im/post/5db718b7f265da4d1e26cb9c)。

![](https://user-gold-cdn.xitu.io/2019/10/30/16e189057e12e6f8?imageslim)


---
