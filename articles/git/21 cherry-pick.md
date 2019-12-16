### cherry-pick
`cherry-pick`用来将一些`commit`应用到当前分支

### 初始化项目
```
$ git init
Initialized empty Git repository in ...

$ echo 1 >> 1.txt && git add . && git commit -m 'first commit'
[master (root-commit) aa34396] first commit
 1 file changed, 1 insertion(+)
 create mode 100644 1.txt
```

以上，初始化了一个本地仓库，并提交了一个`commit`，接着新建一个分支，在这个分支之上再提交两个`commit`
```
$ echo 2 >> 2.txt && git add . && git commit -m 'second commit'
[newBranch d7edd4f] second commit
 1 file changed, 1 insertion(+)
 create mode 100644 2.txt

 $ echo 3 >> 3.txt && git add . && git commit -m 'third commit'
[newBranch 0fb599e] third commit
 1 file changed, 1 insertion(+)
 create mode 100644 3.txt
```

### 应用指定`commit`
此时，`master`有一个`commit`：
```
$ git log master
commit aa34396d000eb6b162693b47d59dab5465648832 (HEAD -> master)
Author: lyxxxx <lyxxxx@yeah.net>
Date:   Thu Nov 21 23:19:52 2019 +0800

    first commit
```

而`newBranch`有三个`commit`
```
commit 0fb599ea3c296f295f8ed5c7f687acd2e9521007 (newBranch)
Author: lyxxxx <lyxxxx@yeah.net>
Date:   Thu Nov 21 23:22:08 2019 +0800

    third commit

commit d7edd4f453034762d87052e7f75f0b13608dfded
Author: lyxxxx <lyxxxx@yeah.net>
Date:   Thu Nov 21 23:21:49 2019 +0800

    second commit

commit aa34396d000eb6b162693b47d59dab5465648832 (HEAD -> master)
Author: lyxxxx <lyxxxx@yeah.net>
Date:   Thu Nov 21 23:19:52 2019 +0800

    first commit
```
我们希望吧`third commit`合并到`master`，但是不合并`second commit`。使用`merge`会合并所有的`commit`，显然不行，所以这里需要用`cherry-pick`
```
$ git switch master
Switched to branch 'master'

$ git cherry-pick 0fb599ea3c296f295f8ed5c7f687acd2e9521007
[master 8a18b61] third commit
 Date: Thu Nov 21 23:22:08 2019 +0800
 1 file changed, 1 insertion(+)
 create mode 100644 3.txt
```
查看提交历史
```
$ git log
commit 8a18b61dc68db3d6b3a524d9faeb5e0a21a85c59 (HEAD -> master)
Author: lyxxxx <lyxxxx@yeah.net>
Date:   Thu Nov 21 23:22:08 2019 +0800

    third commit

commit aa34396d000eb6b162693b47d59dab5465648832
Author: lyxxxx <lyxxxx@yeah.net>
Date:   Thu Nov 21 23:19:52 2019 +0800

    first commit
```