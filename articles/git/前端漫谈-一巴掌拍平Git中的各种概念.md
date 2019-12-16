---
title: '[前端漫谈]一巴掌拍平Git中的各种概念'
date: 2019-01-10 15:17:43
tags:
---


### 0x000 导读
讲`git`的文章很多，但是大部分都是一个套路，讲概念，讲命令，太多的概念和命令总是让人有一种稀里糊涂的感觉，讲的很对，但似乎没能讲透，没有醍醐灌顶的感觉，大概是我的悟性差吧。所以这几天一直在做各种`git`的实验，并且阅读了一些博客、文档、资料，综合下来整理出了这么一篇文章。注意：

- 本篇文章旨在说明我对`git`的理解，只是一家之言，聊以分享。
- 本片文章不是一篇命令教程，或者概念说明，需要一定的`git`使用经验和踩坑经验。

### 0x001 存档和读档
- 存档

    其实吧，版本就是存档，就是游戏中的存档，我们不断的推进项目，完成一个又一个任务，然后中途不断的存档，就形成了版本迭代。而版本多了，自然就需要管理了，自然就有了版本管理系统。

    在游戏中，存档可以手动存档，也可以到指定点存档，也可以自动定场景存档。在游戏中，存档之后形成的东西叫做档案，而在`git`中，叫做`commit`。我们可以使用`git add`+`git commit`完成一个档案的创建，或者说版本的创建。

    一个`commit`拥有许多的属性，比如`ID`、`Message`、`Date`、`Author:`等等，这些信息都有助于我们了解这个版本，就像游戏中的存档会以关卡名/图片之类的信息来提示你这个存档所代表的进度，比如使用`git log`可以获取以下信息：

    ```
    commit 4963bab37862245f85c0754f7759858346ddfcbb (HEAD -> master)
    Author: **********
    Date:   Thu Jan 10 15:22:12 2019 +0800

        版本H

    commit 1a4268dc09c467cbae08dcdaadad808d393509df
    Author: **********
    Date:   Thu Jan 10 15:25:01 2019 +0800

        版本G

    commit 931b3c9e056adf2178b16910d30769503d60d8c3
    Author: **********
    Date:   Thu Jan 10 15:24:50 2019 +0800

        版本F
    ....
    ```

- 读档

    既然有存档，那就有读档。游戏中直接选择一个档案就行了，那`git`中呢？如果有可视化操作工具，那我们直接点击也是可以的，但现在不使用工具，而使用命令行，该如何呢。读取一个存档说白了在`git`中就是读取一个`commit`而已，所以我们可以使用`git checkout`和`git reset`两个命令来做到，那如何指定版本呢？前面提到的`commit`属性中的`ID`可以帮我们实现这个目标。

    - 环境说明：我在仓库中`git commit`了8个，每个`commit`都添加了一个从`a-h`的文件，并在`commit`信息中添加版本标记
        ![2019-01-10-18-04-08](https://user-gold-cdn.xitu.io/2019/1/12/1683ffe18c41627f?w=820&h=882&f=png&s=111493)
    - 使用`git checkout`切到版本A，可以发现，此时只有文件a
        ```
        $ git checkout 401e1b65a1c8f20ad7295985b2f9296d7f237c5b # 版本A 的 commitID
        Note: checking out '401e1b65a1c8f20ad7295985b2f9296d7f237c5b'.

        ...

        HEAD is now at 401e1b6 版本A

        $ ls
        README.md       a.txt
        ```
    - 使用`git reset`切换到版本G，可以发现，此时有了`a-g`几个文件了
        ```
        $ git reset 1a4268dc09c467cbae08dcdaadad808d393509df # 版本G 的 commitID
        Unstaged changes after reset:
        D       b.txt
        D       c.txt
        D       d.txt
        D       e.txt
        D       f.txt
        D       g.txt

        $ git stash
        Saved working directory and index state WIP on (no branch): 1a4268d 版本G

        l$ ls
        README.md       a.txt           b.txt           c.txt           d.txt           e.txt           f.txt           g.txt
        ```
- 总结：
    我们通过`commit`的`ID`属性配合其他命令来达到了任意读档的目的，可以在各种版本中随意穿梭，快乐的很啊。而读档的姿势其实还有很多，但不外乎是对`commit`操作方式的不同，在`git`中，我觉得**commit 才是构成整个版本管理的基本栗子**。每一个`commit`都是独立的个体，虽然和其他`commit`存在着关联，但是依旧是独立的，而我们在`commit`构成节点序列中来回移动和操作，就可以达到所有对版本管理的目的。
    
### 0x002 别名系统
在上一个章节中，我们已经可以依靠一些命令和`commit ID`做到在版本中自由的穿梭，但是却带来一个问题，那就是`commit ID`的记忆难度。`commit ID`是`hash`值，尽管`git`支持只提供前几位就能匹配到`hash`，并且也提供了`commit message`来说明`commit`，但是依旧存在`commit`的辨识和记忆问题。而这个问题，可以通过`别名系统`来解决。

所谓的`别名系统`，其实是我自己归纳的概念，指的其实就是`HEAD`、`branch`、`tag`这三个东西。在我看来，这三个东西都是一样的东西，都是别名，也就是标记一个`commit`的东西而已，只是在行为表现上有一些区别。

1. `HEAD`

    一个仓库只有一个`HEAD`，指向你当前所在的`commit`。如果你创建了一个`commit`，`HEAD`将会指向这个新的`commit`。也可以通过命令，强制`HEAD`指向某个`commit`，比如`reset`、`checkout`。也就是不论你在哪个`commit`之上，那么`HEAD`就在哪儿，或者说，其实你能在哪个`commit`，是通过修改`HEAD`指向的`commit`实现的。

    - 通过修改`HEAD`在各个版本之间旋转跳跃

        ![2019-01-11-13-20-54](https://user-gold-cdn.xitu.io/2019/1/12/1683ffe18ba4d719?w=641&h=580&f=png&s=157868)



2.  `branch`

    一开始我觉得这个东西才是`git`的核心，因为创建项目的时候，我们就处于`master`分支之上，并且我们在工作中，往往也是基于分支工作的。但是后来发现，分支在本质上毫无意义，并不需要真的基于`branch`去工作，基于`commit`就行了。而`branch`只是提供了一个方式来管理这些commit。`branch`和`HEAD`相同点是随着新的`commit`的创建，`branch`指向的`commit`会不断更新，当然前提是你需要在这个`branch`所在的`commit`上创建新的`commit`。而`branch`和`HEAD`的不同点在于`HEAD`只能有一个，`branch`可以有多个。
    
    实验一：用`branch`来实现切换版本
    - 目前的库情况
        ```
        $ git log --pretty=oneline
        1a4268dc09c467cbae08dcdaadad808d393509df (HEAD) 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a 版本C
        129526058361af7b59b22dd059f15c7e288b9862 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b 版本A
        ```
    - 为版本`A-G`分别创建一个分支
        ```
        $ git checkout 1a4268dc09c467cbae08dcdaadad808d393509df -b G # 版本G 的 commitID
        Switched to a new branch 'G'
        $ git checkout 931b3c9e056adf2178b16910d30769503d60d8c3 -b F # 版本F 的 commitID
        Switched to a new branch 'F'
        $ git checkout 071d00a07c5f04bee1cf9a9548c4953914dfcc6b -b E # 版本E 的 commitID
        Switched to a new branch 'E'
        $ git checkout 0caa2b71d1b03d2dacc7c2c92699466b632da185 -b D # 版本D 的 commitID
        Switched to a new branch 'D'
        $ git checkout 7855905df12bccfd38aad8f5b3b95904dc44233a -b C # 版本C 的 commitID
        Switched to a new branch 'C'
        $ git checkout 129526058361af7b59b22dd059f15c7e288b9862 -b B # 版本B 的 commitID
        Switched to a new branch 'B'
        $ git checkout 401e1b65a1c8f20ad7295985b2f9296d7f237c5b -b A # 版本A 的 commitID
        Switched to a new branch 'A'

        $ git log --pretty=oneline
        1a4268dc09c467cbae08dcdaadad808d393509df (HEAD -> G) 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (A) 版本A
        ```
    - 接下来就可以换一种方式在版本之间跳跃了，并且不需要记住或者查询冗长的`commit ID`
        ```
        $ git checkout A
        Switched to branch 'A'
        $ git checkout B
        Switched to branch 'B'
        $ git checkout C
        Switched to branch 'C'
        $ git checkout E
        Switched to branch 'E'
        $ git checkout F
        Switched to branch 'F'
        $ git checkout G
        Switched to branch 'G'
        ```
    实验二：分支跟随新的`commit`
    - 当前库的情况，注意：这里的`HEAD -> G`表示`HEAD`指向了`branch G`，而`branch G`指向了`版本G`
        ```
        $ git log --pretty=oneline
        1a4268dc09c467cbae08dcdaadad808d393509df (HEAD -> G) 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (A) 版本A
        ```
    - 添加一个文件，创建一个`commit`
        ```
        $ echo 'h'> h.txt
        $ git add h.txt
        $ git commit -m '版本H'
        [G d346d27] 版本H
        1 file changed, 1 insertion(+)
        create mode 100644 h.txt
        ```
    - 此时查看`log`，可以看到`HEAD`和`G`都指向了`版本H`，就是所谓的`branch`跟着`commit`动，但是它真的是跟着`commit`动吗？
        ```
        $ git log --pretty=oneline
        d346d279a5073b57ef86f5e7865f52f8286e34cd (HEAD -> G) 版本H
        1a4268dc09c467cbae08dcdaadad808d393509df 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (A) 版本A
        ```
    实验三：分支跟着啥动

    - 将`HEAD`指向`版本G`的`commit`，而不是分支`G`，也就是使用`git checkout commitID`，而不是使用`git checkout branchName`，可以看到，此时`HEAD`不指向`G`，而是`HEAD`和`G`同时指向了`版本H`的`commit`。
        ```
        $ git checkout d346d279a5073b57ef86f5e7865f52f8286e34cd # 版本 H 的 commitID
        $ git log --pretty=oneline
        d346d279a5073b57ef86f5e7865f52f8286e34cd (HEAD, G) 版本H
        1a4268dc09c467cbae08dcdaadad808d393509df 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (A) 版本A
        ```
    - 继续创建一个`commit`，可以看到，这个时候分支`G`不再跟着`commit`移动了，所以，只有在`HEAD`指向`branch`的时候，`branch`才会向前移动，也就是只要`HEAD`来到`branch`身边，`branch`就会跟着`HEAD`跑。
        ```
        $ echo 'i'> i.txt
        $ git add i.txt
        $ git commit -m "版本I"
        [detached HEAD 2e836eb] 版本I
        1 file changed, 1 insertion(+)
        create mode 100644 i.txt
        $ git log --pretty=oneline
        2e836ebb00b722a29a99101b1c6f7b276aeed033 (HEAD) 版本I
        d346d279a5073b57ef86f5e7865f52f8286e34cd (G) 版本H
        1a4268dc09c467cbae08dcdaadad808d393509df 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (A) 版本A
        ```

3. `tag`

    `tag`是比较特殊的一个别名类型，他无法移动，或者说不推荐移动。一旦一个`tag`和指向某个`coimmit`，就不希望它移动，因为`tag`就是用来标记这个`commit`的，他是一个孤独而忠诚的守望者，而不像`branch`，花间游龙似的浪子。

    - 现在库的情况
        ```
        $ git log --pretty=oneline
        1a4268dc09c467cbae08dcdaadad808d393509df (HEAD, G) 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (A) 版本A
        ```
    - 为每个版本添加一个`tag`，为了区别分支名，统统加了个`T`
        ```
        $ git tag TA A
        $ git tag TB B
        $ git tag TC C
        $ git tag TD D
        $ git tag TE E
        $ git tag TF F
        $ git tag TG G

        $ git log --pretty=oneline
        1a4268dc09c467cbae08dcdaadad808d393509df (HEAD, tag: G, G) 版本G
        931b3c9e056adf2178b16910d30769503d60d8c3 (tag: TF, F) 版本F
        071d00a07c5f04bee1cf9a9548c4953914dfcc6b (tag: TE, E) 版本E
        0caa2b71d1b03d2dacc7c2c92699466b632da185 (tag: TD, D) 版本D
        7855905df12bccfd38aad8f5b3b95904dc44233a (tag: TC, C) 版本C
        129526058361af7b59b22dd059f15c7e288b9862 (tag: TB, B) 版本B
        401e1b65a1c8f20ad7295985b2f9296d7f237c5b (tag: TA, A) 版本A
        ```
    - 现在又多了一种旋转跳跃的方式了
        ```
        $ git checkout TA
        Previous HEAD position was 1a4268d 版本G
        HEAD is now at 401e1b6 版本A
        $ git checkout TB
        Previous HEAD position was 401e1b6 版本A
        HEAD is now at 1295260 版本B
        $ git checkout TC
        Previous HEAD position was 1295260 版本B
        HEAD is now at 7855905 版本C
        $ git checkout TD
        Previous HEAD position was 7855905 版本C
        HEAD is now at 0caa2b7 版本D
        $ git checkout TE
        Previous HEAD position was 0caa2b7 版本D
        HEAD is now at 071d00a 版本E
        $ git checkout TF
        Previous HEAD position was 071d00a 版本E
        HEAD is now at 931b3c9 版本F
        $ git checkout TG
        Previous HEAD position was 931b3c9 版本F
        HEAD is now at 1a4268d 版本G
        ```
    - 总结
    所以，不管是`HEAD`、`tag`、`branch`，都是一种别名，除了行为表现上的差别，没有太大的不同，特别是`branch`和`tag`,不过都只是提供了一种管理`commit`的方式。

### 0x002 分叉
在上一章节中，我们揭开了别名系统的红盖头，这一章，我们就开始探索一下分叉的神秘。

和游戏中的存档一样，有时候一个游戏有许多的选择，这些选择指向了不同的结果。而作为游戏玩家，我们希望能够走完所有的选择，以探索更多的游戏乐趣。所以我们会在做选择的时候存档，而当我们走完一个选择，就会读取这个存档，继续往另一个选择探索。这个时候，就产生了两个不同的剧情走向，这就是分叉。

在`git`中，其实我们可以有无数的选择，每一个`commit`可以创建无数的`commit`，就会引申出无数的可能。

- 我们遇到了一个抉择，所以需要创建版本，暂时称为`版本X`吧
    ```
    $ git log --pretty=oneline
    2caeda3e3555f1d134212c2bc6b262026c295743 (HEAD) 版本X
    ....
    ```
- 然后我们选择了走`Y`，并且沿着`Y1`一直走到`Y3`，这是尽头
    ```
    $ git log --pretty=oneline
    d2e0375ba096c36fcd96e75d142128d3e07d767e (HEAD) 版本Y3
    4ca86274b4f7473ac6295100f08d9f3424cf6b8d 版本Y2
    fcff93ee95aa2be3418fc73db4c9ce9ed3201790 版本Y1
    2caeda3e3555f1d134212c2bc6b262026c295743 版本X
    ...
    ```
- 接着我们返回`X`，并选择另一个选择`Z`，从`Z1`走到`Z3`
    ```
    $ git checkout 2caeda3e3555f1d134212c2bc6b262026c295743 # 切到`版本X`
    $ git log --pretty=oneline
    16ffb118853798475d38f75bb9c157cc4d0c39cd (HEAD) 版本Z3
    0ca53ec8f01cc417d20ce8f5f9399501b696a077 版本Z2
    b4a71198961d3e86af6324b4cdeee6e3a2a08ac2 版本Z1
    2caeda3e3555f1d134212c2bc6b262026c295743 版本X
    ...
    ```
- 总结

可以看到，我们顺着两个选择一直往下发展，在这发展的过程中，我们完全没有用到`tag`和`branch`，也就是为了印证**commit 是构成 git 世界的基本栗子**这一说明。

从`git log`中，我们看不到了`Y`走向，那`Y`真的消失了吗？不是的，我们依旧可以通过`Y`的`commit ID`来寻回`Y`的记录。当然为了方便在`YZ`中切换，我们可以使用`branch`来标记一下`YZ`两个走向，这样就形成了`YZ`两个`branch`了，也就是分叉！

那那些没有被`branch`或者`tag`标记的`commit`呢？他们会消失吗？会，也不会。不会是因为不被标记的`commit`将变成`dangling commit`，我称之为游离的`commit`，是`git`中最孤独的存在，只要我们知道`commitID`，就会可唤回它。但是很大的可能是我们永远不会记得这么一个不被引用的`commit`，所以我呼吁，善待每一个`commit`。会是因为还是可能会被回收的，看[这里，git 也有 gc](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E7%BB%B4%E6%8A%A4%E4%B8%8E%E6%95%B0%E6%8D%AE%E6%81%A2%E5%A4%8D)。

### 0x003 合并

和游戏的存档不同的是，`git`中的版本可以合并，也就是说我可以在分支`Y`中做完任务`Y1`、`Y2`、`Y3`，然后分支`Z`中完成任务`Z1`、`Z2`、`Z3`，然后合并这两个分支，结果回到了`X`，但是却完成了`Y1-y3`、`Z1-Z3`，并拿到了神器`Y`和`Z`，这让`boss`怎么活？

- 实验一：使用`merge`合并`commit`

    - 创建`版本O`
        ```
        $ echo O >> o.txt
        $ git add o.txt
        $ git commit -m '版本O'
        [detached HEAD 478fa6d] 版本O
        1 file changed, 1 insertion(+)
        create mode 100644 o.txt
        ```
    - 基于`版本O`创建`版本P1`
        ```
        $ echo P >>p1.txt
        $ git add p1.txt
        $ git commit -m '版本P1'
        [detached HEAD a3ab178] 版本P1
        1 file changed, 1 insertion(+)
        create mode 100644 p1.txt
        $ git log --pretty=oneline
        a3ab178f950163f9eb5b7e857226bb616517d0d7 (HEAD) 版本P1
        478fa6d05021bbf7380ab69689c5683a84a394a4 版本O
        ```
    - 基于`版本O`创建`版本P2`
        ```
        $ git checkout 478fa6d05021bbf7380ab69689c5683a84a394a4 # 版本O 的 commitID
        $ echo p2 >> p2.txt
        $ git add p2.txt
        $ git commit -m '版本P2'
        [detached HEAD cbccf52] 版本P2
        1 file changed, 1 insertion(+)
        create mode 100644 p2.txt
        $ git log --pretty=oneline
        cbccf5206d3225fd3fab7c944573042d199bc82b (HEAD) 版本P2
        478fa6d05021bbf7380ab69689c5683a84a394a4 版本O
        ```
    - 合并`版本P1`到`版本P2`
        ```
        $ git merge a3ab178f950163f9eb5b7e857226bb616517d0d7 # 版本P1 的 commitID
        $ git log --pretty=oneline
        656a5426db088a1cbeba1cadc2c6cddd3ae9df1f (HEAD) Merge commit 'a3ab178f950163f9eb5b7e857226bb616517d0d7' into HEAD
        cbccf5206d3225fd3fab7c944573042d199bc82b 版本P2
        a3ab178f950163f9eb5b7e857226bb616517d0d7 版本P1
        478fa6d05021bbf7380ab69689c5683a84a394a4 版本O
        ```

- 实验三：使用`rebase`合并

    切换到`版本P2`，在`版本P2`中使用`rebase`
    ```
    $ git checkout cbccf5206d3225fd3fab7c944573042d199bc82b # 版本P2 的 commitID
    ....
    HEAD is now at cbccf52 版本P2
    $ git rebase a3ab178f950163f9eb5b7e857226bb616517d0d7 # 版本P1 的 commitID
    First, rewinding head to replay your work on top of it...
    Applying: 版本P2
    $ git log --pretty=oneline
    3bd7820a805e70ecd2d4791f7158b3eab69d5456 (HEAD) 版本P2
    a3ab178f950163f9eb5b7e857226bb616517d0d7 版本P1
    478fa6d05021bbf7380ab69689c5683a84a394a4 版本O
    ```

- 实验四：使用`cherry-pick`合并

    - 切换到`版本O`，新建`版本P3`
        ```
        $ echo 'p3'>> p3.txt
        $ git add p3.txt
        $ git commig -m '版本P3'
        git: 'commig' is not a git command. See 'git --help'.

        The most similar command is
                commit
        $ git commit -m '版本P3'
        [detached HEAD ae09e94] 版本P3
        1 file changed, 1 insertion(+)
        create mode 100644 p3.txt
        $ git log --pretty=oneline
        ae09e94fdab55dd69f4cf7ca2f3658c20026c441 (HEAD) 版本P3
        478fa6d05021bbf7380ab69689c5683a84a394a4 版本O
        ```
    - 切换到`版本P2`中使用`cherry-pick`合并`版本P3`的东西
        ```
        $ git checkout 3bd7820a805e70ecd2d4791f7158b3eab69d5456 # 版本P2 的commitID
        ...
        HEAD is now at 3bd7820 版本P2
        $ git cherry-pick ae09e94fdab55dd69f4cf7ca2f3658c20026c441 # 版本P3 的 commitID
        [detached HEAD f9dfba2] 版本P3
        Date: Sat Jan 12 11:35:27 2019 +0800
        1 file changed, 1 insertion(+)
        create mode 100644 p3.txt
        $ git log --pretty=oneline
        f9dfba2fcc22cd35c7e654b2ab1c17501124fb02 (HEAD) 版本P3
        3bd7820a805e70ecd2d4791f7158b3eab69d5456 版本P2
        a3ab178f950163f9eb5b7e857226bb616517d0d7 版本P1
        478fa6d05021bbf7380ab69689c5683a84a394a4 版本O
        ```


- 注意：合并中的冲突解决

    合并的过程中可能会出现冲突，比如同时拿到神器`P1`、`P2`，但是在`P1`中卖掉了`O`之前拿到的装备`S`，而在`P1`中则为`S`镶上了宝石，那么合并之后要怎么处理？是卖掉`S`?还是保留镶宝石的`S`？还是镶了宝石再卖掉？深井冰啊！我不要面子的啊...

    所以这里就涉及到了合并的冲突解决，这里不再赘述，不是我要讲的内容。

### 0x004 变基
这个名词让人想入菲菲啊，每次项目新成员加入，总是会提醒他们注意要变基....

这里不去说`merge`和`rebase`的爱恨情仇，只说一些`rebase`的操作，用`rebase`来整理`commit`。

上面说到**commit 是构成 git 世界的基本栗子**，所以，我们需要掌握一些`栗子`的操作方式

- 查看`commit`，可以使用`git log`，如果需要寻回忘记的`commit`，可以使用`reflog`来尝试看看是否能够找到
    ```
    $ git log --pretty=oneline
    68de3f74c6a4799a54047efe084336d5452f6bb0 (HEAD -> X) Merge branches 'Y' and 'Z' into X
    16ffb118853798475d38f75bb9c157cc4d0c39cd (Z) 版本Z3
    ...
    $ git reflog
    23e799e (HEAD) HEAD@{0}: rebase -i (pick): 版本Z
    01a10d6 HEAD@{1}: rebase -i (squash): 版本Y
    a15dd72 HEAD@{2}: rebase -i (squash): # This is a combination of 2 commits.
    b6f2ea3 HEAD@{3}: rebase -i (start): checkout 100468330c7819173760938d9e6d4b02f37ba001
    f4c4ccc HEAD@{4}: rebase -i (abort): updating HEAD
    ...
    ```
- 创建

    创建使用`git add`+`git commit`就行了
    ```
    $ echo error > error.txt
    $ git add error.txt
    $ git commit -m '一个错误的版本'
    [X bc90774] 一个错误的版本
    1 file changed, 1 insertion(+)
    create mode 100644 error.txt
    $ git log --pretty=oneline
    bc907749174ee0db222de16d60126c39fa3096fa (HEAD -> X) 一个错误的版本
    68de3f74c6a4799a54047efe084336d5452f6bb0 Merge branches 'Y' and 'Z' into X
    ...
    ```
- 更新上一个`commit`，直接使用`git commit --amend`
    ```
    $ echo error2 >> error.txt
    $ git add error.txt
    $ git commit --amend
    // 这里将打开一个vim窗口
    一个错误的版本

    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.
    #
    # Date:      Fri Jan 11 17:21:18 2019 +0800
    #
    # On branch X
    # Changes to be committed:
    #       new file:   error.txt
    #
    // 保存退出之后输出
    [X d5c4487] 一个错误的版本
    Date: Fri Jan 11 17:21:18 2019 +0800
    1 file changed, 2 insertions(+)
    create mode 100644 error.txt
    ```

- 要更新历史中的`commit`也是可以做到的，例如需要在版本X中加入文件`x1.txt`，要使用交互式模式
    ```
    git rebase -i 2e836ebb00b722a29a99101b1c6f7b276aeed033 # 指向 版本X 的前一个 commit
    ```
    此时将打开一个交互式窗口
    ```
    pick 913b571 版本X
    pick 0eca5e3 版本Y1
    pick 33a9ca3 版本Y2
    pick b95b3ca 版本Y3
    pick 839c481 版本Z1
    pick 6fb6cb3 版本Z2
    pick c28d3e0 版本Z3
    ...
    ```
    将`版本X`前面的`pick`改为`e`或者`edit`，保存，然后退出，这个时候，仓库将会回到`版本X`的状态，并输出
    ```
    Stopped at 913b571...  版本X
    You can amend the commit now, with

    git commit --amend

    Once you are satisfied with your changes, run

    git rebase --continue
    ```
    添加文件`x1`
    ```
    $ echo x1 > x1.txt
    $ git add x*
    $ git commit --amend
    // 打开交互式窗口可以修改 commit message
    $ git rebase --comtinue
    Successfully rebased and updated detached HEAD.
    ```
    此时又回会到原本的版本并且多出了文件`x1`，就像是在`版本X`中就已经加入一样

- 插入一个新的`commit`,上面的栗子中不使用`--amend`，就会在`X`和`Y1`之间插入一个新的`commit`
    ```
    $ git rebase -i 2e836ebb00b722a29a99101b1c6f7b276aeed033
    // 交互式窗口，吧`pick`改为`e`
    $ echo x2 > x2.txt
    $ git add x2.txt
    $ git commit -m '插入一个版本X2'
    [detached HEAD 1b4821f] 插入一个版本X2
    1 file changed, 1 insertion(+)
    create mode 100644 x2.txt
    $ git rebase --continue
    Successfully rebased and updated detached HEAD.
    $ git log --pretty=oneline
    30a57a7ed0bd2b04ee61ba14b59d60b416b4c22f (HEAD) 版本Z3
    4b0069b1b87f98aa90b792b97011ef08845ffd7b 版本Z2
    cc1d571328d765019b2e21a33a926de973152263 版本Z1
    595e928fed812c7de0e6b9732beeff6489dd8a3d 版本Y3
    4456b102fca02d12b51ae1a6397d84d5f298233f 版本Y2
    b6f2ea31b3ea5ec78b384e9b1669ac2c7063c339 版本Y1
    1b4821fcb5e296e282ffe9a87791ec1a94281a2f 插入一个版本X2
    100468330c7819173760938d9e6d4b02f37ba001 版本X
    ```

- 删除

    删除一个分支可以使用交互式`rebase`，命令：`git rebase -i commitID`，这里的`commitID`必须是你要删除的`commit`的前一个`commit`。
    ```
    $ git rebase -i 68de3f74c6a4799a54047efe084336d5452f6bb0
    ```
    此时将会打开一个`vim`窗口
    ```
    pick bf2c542 版本Y1
    pick 588feec 版本Y2
    pick 1b2ae37 版本Y3
    pick 38f7cf3 版本Z1
    pick 080e442 版本Z2
    pick 206a7ae 版本Z3
    pick 6b01f70 一个错误的版本

    # Rebase 2caeda3..6b01f70 onto 2caeda3 (7 commands)
    #
    # Commands:
    # p, pick = use commit
    # r, reword = use commit, but edit the commit message
    # e, edit = use commit, but stop for amending
    # s, squash = use commit, but meld into previous commit
    # f, fixup = like "squash", but discard this commit's log message
    # x, exec = run command (the rest of the line) using shell
    # d, drop = remove commit
    ...
    ```
    要删除`一个错误的版本`，将前面的`pick`改为`d`或者`drop`
    ```
    d 6b01f70 一个错误的版本
    ```
    保存退出，输出
    ```
    Successfully rebased and updated refs/heads/X.
    ```

- 合并多个`commit`，比如合并`Z1-Z3`，打开交互式窗口之后，将`Z2`、`Z3`的`pick`改为`s`
    ```
    $ git rebase -i 100468330c7819173760938d9e6d4b02f37ba001
    // 打开了交互式窗口
    pick bf2c542 版本Y1
    pick 588feec 版本Y2
    pick 1b2ae37 版本Y3
    pick 38f7cf3 版本Z1
    s 080e442 版本Z2
    s 206a7ae 版本Z3
    ```
    保存退出以后，又打开交互式窗口，显示要合并的`commit`的`message`，这里可以修改`commit`。
    ``` 
    # This is a combination of 3 commits.
    # This is the 1st commit message:

    版本Z1

    # This is the commit message #2:

    版本Z2

    # This is the commit message #3:

    版本Z3
    ```
    这里修改为`Z`，保存，退出，输出，可以看到，`Z1-Z3`消失了，取而代之的是`Z`，对`Y1-Y3`做操作
    ```
    detached HEAD f4c4ccc] 版本Z
    Date: Fri Jan 11 16:27:00 2019 +0800
    1 file changed, 3 insertions(+)
    create mode 100644 z.txt
    Successfully rebased and updated detached HEAD.
    $ git log --pretty=oneline
    f4c4ccc548052b884013c9b133db85e3e730a829 (HEAD) 版本Z
    595e928fed812c7de0e6b9732beeff6489dd8a3d 版本Y3
    4456b102fca02d12b51ae1a6397d84d5f298233f 版本Y2
    b6f2ea31b3ea5ec78b384e9b1669ac2c7063c339 版本Y1

    $ git rebase -i 100468330c7819173760938d9e6d4b02f37ba001
    [detached HEAD 01a10d6] 版本Y
    Date: Fri Jan 11 16:24:37 2019 +0800
    1 file changed, 3 insertions(+)
    create mode 100644 y.txt
    Successfully rebased and updated detached HEAD.
    $ git rebase --continue
    ```
- 重新排序`commit`顺序，比如重排`版本Y`和`版本Z`，交换一下顺序就好了
    ```
    $ git log --pretty=oneline
    23e799e9d6445f6f2fea50cf9599f7c407b521ee (HEAD) 版本Z
    01a10d6b62bea2f8391e25d193020a456c5b301f 版本Y
    $ git rebase -i 1b4821fcb5e296e282ffe9a87791ec1a94281a2f
    ```
    这时候打开交互式窗口，显示
    ```
    pick a1942a3 版本Y
    pick eeabc6c 版本Z
    ```
    将它交换顺序，保存，退出
    ```
    pick eeabc6c 版本Z
    pick a1942a3 版本Y
    ```
    查看结果
    ```
    Successfully rebased and updated detached HEAD.
    $ git log --pretty=oneline
    a1942a31048eabf70210034b0f2d9e2540e88064 (HEAD) 版本Y
    eeabc6cf5a54ea84ba1ac675a8bb80eae7237ffa 版本Z
    ```

### 0x005 总结

这是一篇比较乱七八糟的文章，不从传统出发，尝试用自己的思想去理解`git`这一神奇的工具。以前我觉得`git`是`命运石之门`，我们在不同的时间线(分支)上跳跃，所有的事件都必须且只能发生在时间线上。但是现在我觉得`git`是`无限的可能性的集合`，一个事件可以引申出无限的可能性。而我们要做的是用工具（branch、tag、reset、rebase、merge....）来组织这些可能性，从而构成一个有序的、向前发展的历史，引导整个历史的发展，构建可以走向未来的工程。

### 0x006 资料
- [git pro 官方文档](https://git-scm.com/book/zh/v2)
- [git 仿真沙盒](https://github.com/pcottle/learnGitBranching)
- [git 教程 阮一峰](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)
- [git 教程 菜鸟](http://www.runoob.com/git/git-tutorial.html)
