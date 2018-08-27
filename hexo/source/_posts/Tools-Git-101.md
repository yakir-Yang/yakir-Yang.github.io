---
title: Git 深度揭秘
date: 2018-08-27 14:25:06
tags:
toc: true
categories: Tools
---

本教程分为以下三部分，帮助大家进行 Daily Work，以及深度揭秘 Git

- **常用 Git 使用套路**

- **Git 底层实现简**

- **进阶 Git 使用套路**

<!-- more -->


## Git 目录结构

> Remote Repository  VS  Local Repository



### WorkSpace

```
➜  ~ git clone ssh://git@localhost:2222/git-server/repos/git101.git
Cloning into ‘git101'...
Warning: Permanently added '[localhost]:2222' (ECDSA) to the list of known hosts.
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.

➜  ~/git101 git:(master) tree
.
├── README.md
└── tools
    └── helloguys.sh

1 directory, 2 files

➜  ~/git101 git:(master) git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```



### Repo

- Local Git Repository

- ```
  ➜  ~/git101 git:(master) tree .git -L 1
  .git
  ├── HEAD
  ├── config
  ├── description
  ├── hooks
  ├── index
  ├── info
  ├── logs
  ├── objects
  ├── packed-refs
  └── refs

  ➜  ~/git101 git:(master) git branch -a
  * master
    remotes/origin/HEAD -> origin/master
    remotes/origin/master
  ```

- Remote Server Git Repository

- ```
  ➜  git101.git git:(master) tree -L 1
  .
  ├── HEAD
  ├── config
  ├── description
  ├── hooks
  ├── info
  ├── objects
  ├── packed-refs
  └── refs

  ➜  git101.git git:(master) git branch -a
  * master
  ```



### 相互关系

![Git目录结构-相互关系](/images/tools/git101/Git目录结构-相互关系.png)

- Workspace  工作区
- Git Index / Stage 缓冲区
- Local Git Repository 本地仓库
- Remote Server Git Repository 服务器仓库



---



## 常用 Git 使用套路

> Git Daily Work

常用 Git 使用套路

![image-20180827115740479](/images/tools/git101/Git常用套路.png)

- 常用场景一：本地获取服务器最新的代码，然后进行开发，本地制作提交（Commit）后，直接上传至服务器。
- 常用场景二：本地基于早前的代码，然后进行开发，本地制作提交（Commit）后。需要与服务器最新代码进行合并（Merge），解决冲突（Conflict）后，才能上传至服务器。



### 常用 Git 使用套路 (1)

![image-20180827120014106](/images/tools/git101/Git常用套路1.png)

```
# 1. 下载项目
~$ git clone https://github.com/yakir-Yang/any101

~$ cd any101

~/any101$ git:(master) git log

# 2. 本地修改源码
~/any101$ git:(master) cp -r /tmp/dockerfile/ .

# 3. 添加改动到 Git 缓冲区
~/any101$ git:(master) git add dockerfile README.md

~/any101$ git:(master) git status

# 4. 生成提交（Commit）
~/any101$ git:(master) git commit -s

~/any101$ git:(master) git log

# 5. 推送至服务器
~/any101$ git:(master) git push origin
```



### 常用 Git 使用套路 (2.1)

![image-20180827120110912](/images/tools/git101/Git常用套路2.1.png)

```
# 1. 本地修改源码
~/any101$ git:(master) cp -r /tmp/openssl/ .

# 2. 添加改动到 Git 缓冲区
~/any101$ git:(master) git add openssl README.md

# 3. 生成 Commit 提交
~/any101$ git:(master) git commit -s

# 4. 第一次尝试提交失败，提示需更新本地代码
~/any101$ git:(master) git push

# 5. 将服务器 Repository 重新镜像到本地
~/any101$ git:(master) git fetch origin

# 6. 查看本地相对服务器有哪些提交（Commit）
~/any101$ git:(master) git log origin/master..master

# 7. 解决冲突（Conflicts），将本地的 Commit 合并到最新代码的尾部
~/any101$ git:(master) git rebase -i origin/master

~/any101$ git:(master) git status

# 8. 成功更新服务器
~/any101$ git:(master) git push
```



### 常用 Git 使用套路 (2.2)

![image-20180827120351347](/images/tools/git101/Git常用套路2.2.png)

```
# 1. 生成 Commit 提交
~/any101$ git:(master) git commit -s

# 2. 第一次尝试提交失败，提示需更新本地代码
~/any101$ git:(master) git push

# 3. 将服务器 Repository 重新镜像到本地
~/any101$ git:(master) git fetch origin

# 4. 将本地分支重命名为 openssl
~/any101$ git:(master) git branch -M openssl

# 5. 将镜像的远程 master 分支本地化
~/any101$ git:(openssl) git checkout origin/master -b master

# 6. 解决冲突（Conflicts），将本地 openssl 分支改动合并到本地 master 分支
~/any101$ git:(master) git merge —no-ff openssl

~/any101$ git:(master) git status

# 7. 成功更新服务器
~/any101$ git:(master) git push
```



### 常用 Git 使用套路 - Conflicts

```
~/any101$ git:(master) git merge --no-ff openssl
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.

~/any101 git:(master) cat README.md
# any101

101 Tutorial for technologies:
 - golang

<<<<<<< HEAD
 - dockerfile
   - ubuntu kernel debpkg
=======
 - openssl
   - cheat sheet style guide
>>>>>>> openssl

~/any101 git:(master) git add README.md

~/any101 git:(master) git status

~/any101 git:(master) git commit
```

**“<<<< HEAD” 与 “====” 之间的内容：是 master 分支已经存在的内容。**

**“====” 与 “>>>> openssl” 之间的内容：是 openssl 分支想要新增的内容。**

**我们在处理 master 分支已经存在的内容时候，必须谨慎小心，因为这是 ”别人的代码”。**



### 常用 Git 使用套路 (2) - Rebase Vs Merge

- Merge: 利于定位 Feature 研发历程

![image-20180827120542764](/images/tools/git101/Git常用套路2-Rebase-VS-Merge.png)

- Rebase: 项目主线干净清晰

![image-20180827120648125](/images/tools/git101/Git常用套路2-Rebase-VS-Merge-1.png)



---



## Git 底层实现简要

> Git Commands 是围绕什么在转？

```
➜  ~/git101 git:(master) tree
.
├── README.md
└── tools
    └── helloguys.sh

➜  ~/git101 git:(master) git ls-tree -r -t 4ab074c98ebe
100644 blob 4a2c3884a095f7b7646e3ed71fdd601d0b896a97	README.md
040000 tree 5ff9cceb595b910225d606506a2f3c0c4bc0fc91	tools
100644 blob 1df8f7baf078d111fcb312d251660b60e68b9d9e	tools/helloguys.sh
```

**Git 底层类似文件系统，也是一种 Key-Value 型数据库，整体关系可以利用树来表示。**

**Git 有三种类型文件：commit、tree、blob**



![image-20180827120800841](/images/tools/git101/Git底层实现简要.png)



### Git 底层实现简要（对象存储）

```
➜  ~/git101 git:(master) tree .git/objects
.git/objects
├── info
└── pack
    ├── pack-21c478e192543d82ff2f325b24798dfa2f6f60c7.idx
    └── pack-21c478e192543d82ff2f325b24798dfa2f6f60c7.pack


➜  ~/git-server/repos/git101.git git:(master) tree objects
objects
├── 1d
│   └── f8f7baf078d111fcb312d251660b60e68b9d9e
├── 45
│   └── 763aa525b40b27b7f3ca75b00f74663cefabcf
├── 4a
│   ├── 2c3884a095f7b7646e3ed71fdd601d0b896a97
│   └── b074c98ebe7ea82934025937106bfd4ecf56f2
├── 5f
│   └── f9cceb595b910225d606506a2f3c0c4bc0fc91
├── info
└── pack
```



### Git 底层实现简要（快照）

```
➜  ~/git101 git:(master) tree .git/objects
.git/objects
├── info
└── pack
    ├── pack-21c478e192543d82ff2f325b24798dfa2f6f60c7.idx
    └── pack-21c478e192543d82ff2f325b24798dfa2f6f60c7.pack

2 directories, 2 files

➜  ~/git101 git:(master) git verify-pack -v .git/objects/pack/pack-*.idx
4ab074c98ebe7ea82934025937106bfd4ecf56f2 commit 223 145 12.
45763aa525b40b27b7f3ca75b00f74663cefabcf tree   69 79 157
4a2c3884a095f7b7646e3ed71fdd601d0b896a97 blob   42 49 236
5ff9cceb595b910225d606506a2f3c0c4bc0fc91 tree   40 51 285
1df8f7baf078d111fcb312d251660b60e68b9d9e blob   33 43 336
non delta: 5 objects
.git/objects/pack/pack-21c478e192543d82ff2f325b24798dfa2f6f60c7.pack: ok

➜  ~/git101 git:(master) git cat-file -p 45763a
100644 blob 4a2c3884a095f7b7646e3ed71fdd601d0b896a97	README.md
040000 tree 5ff9cceb595b910225d606506a2f3c0c4bc0fc91	tools

➜  ~/git101 git:(master) git cat-file -p 4a2c388
# Git 101

This is a test git repository.
```



### Git 底层实现简要（Branch / Tag）

```
➜  ~/git101 git:(master) git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/release-v1.0

➜  ~/git101 git:(master) git tag
v0.1

➜  ~/git101 git:(master) cat .git/packed-refs
# pack-refs with: peeled fully-peeled
4ab074c98ebe7ea82934025937106bfd4ecf56f2 refs/remotes/origin/master
4ab074c98ebe7ea82934025937106bfd4ecf56f2 refs/remotes/origin/release-v1.0
4ab074c98ebe7ea82934025937106bfd4ecf56f2 refs/tags/v0.1

➜  ~/git101 git:(master) tree .git/refs
.git/refs
├── heads
│   └── master # 本地 master 分支，文件内容就是 Commit ID
├── remotes
│   └── origin
│       └── HEAD # 服务器默认分支 (ref: refs/remotes/origin/master)
└── tags
```

**Branch & Tag 都是通过指向 Commit ID 来表示。每个 Commit IDs 记录了 Parent Commit ID，因此通过单个 Commit ID 就可以推算出所有 Commit History，既是 Branch& Tag 需要实现的功能。**



---



## 进阶 Git 使用套路

> 手抖出错了，如果快速恢复？ (Q&A)
>
> 1. **如何生成补丁？**
> 2. **如何打补丁？**
> 3. **如何管理本地修改？**
> 4. **如何编辑 Commit ？**
> 5. **如何查看 log 修改记录？**
> 6. **如何查看两个版本之间的差异？**
> 7. **如何回退代码到指定版本？**
> 8. **如何在同一个 Git Repository， 添加多个 Remote Repository ？**
> 9. **如何合并分支？**
> 10. **如何推送服务器？**
> 11. **手抖敲了 git reset --hard，如何找回本地提交？**
> 12. **主分支有新的 Bug，但是之前没有，如何快速定位是谁干的？**
> 13. **开源项目本地化后，如何进行跟踪管理？**

<br>



**1. 如何生成补丁？**

```
git format-patch [commit-id]

git format-patch -4 [commit-id]

git format-patch [start-commit-id]^..[end-commit-id]
```

<br>



**2. 如何打补丁？**

```
git cherry-pick [commit-id]

git cherry-pick [from-commit-id]^..[end-commit-id]

git am [patch-file]

git apply [patch-file or diff-file]

patch -p1 < [path-file or diff-file]
```

<br>



**3. 如何管理本地修改？**

```
git status

git stash

git stash pop

git clean -df

git reset HEAD .

git checkout .
```

<br>




**4. 如何编辑 Commit**

```
git commit -s

git commit --amend

git commit --amend --author="Kuankuan Yang <yakir.yang@qq.com>”

> git config --global core.editor "vim"
>
> git config --global user.name "Kuankuan Yang"
>
> git config --global user.email "yakir.yang@qq.com"
>
> git rebase -i [commit-id]
```

<br>



**5. 如何查看 log 修改记录？**

```
git log [commit-id]

git log -p [commit-id]

git log --stat [commit-id]

git log --one-line [commit-id]

git show [commit-id]
```

<br>



**6. 如何查看两个版本之间的差异？**

```
git log [old-branchname]..[new-branchname]

git diff [old-branchname]..[new-branchname]

git log [old-commitid]..[new-commitid]

git diff [old-commitid]..[new-commitid]
```

<br>



**7. 如何回退代码到指定版本？**

```
git checkout [commit-id] -- .

git checkout [commit-id] -- [directory-path or file-path]

git reset [commit-id]

git reset --hard [commit-id]

git revert [commit-id]
```

<br>



**8. 如何在同一个 Git Repo 里添加多个 Remote Repository ？**

```
git remote add [what-you-like] [remote-repo-git-address]

> git remote -v
>
> git branch -a
>
> git branch -vv
>
> git branch -u [remote]/[remote-branchname]
```

<br>



**9. 如何合并分支？**

```
git rebase -i [local-branchname]

git rebase -i [remote]/[remote-branchname]

git merge --no-ff [local-branchname]

git merge --no-ff [remote]/[remoet-branchname]
```

<br>



**10. 如何推送服务器？**

```
git push

git push origin [local-branchname]:[remote-branchname]

git push origin [local-tagname / local-branchname]

git push origin :[remote-branchname or remote-tagname]

git push -f

>  git branch -vv
>
> git branch -u [remote]/[remote-branchname]
```

<br>



**11. 手抖敲了 git reset --hard，如何找回本地提交？**

```
git reflog

> git log [reflog-id]
>
> git checkout [reflog-id] -b [new-branchname]
```

<br>



**12. 主分支有新的 Bug，但是之前没有，如何快速定位是谁干的？**

```
git bisect start

git bisect bad [bad-commit-id]

git bisect good [good-commit-id]

```
*"git bisect" 将会使用二分法，不断切换代码版本。你需要做的就是 Keep building and testing，然后告诉 "git bisect" whether it is good or bad。二分法到最后，"git bisect" 就会打印出最早引入 bug 的 commit id 是谁。*

<br>



**13. 开源项目本地化后，如何进行跟踪管理？**

> - 尽量使用开源社区已经 Released 的 Branch or Tag 代码做为 Code Base。
> - 如果开源社区有新 Feature 正是你所要，尽量消化社区的代码改动，然后以 Cherry-Pick 的方式，抓取有限的补丁移植到本地。
> - 尽量不要升级项目的 Code Base，因为有极大可能引入很多未知 Bug。
> - 如果一定要升级项目的 Code Base，最简单方式是将私有化的修改全部制作成补丁，然后一个一个打到新的 Code Base 上，类似常用套路 2.1 的做法。
> - 巧用 Commit Message，将私有修改与社区修改进行区分。为 Code Base 之后的每个 Commit Message 添加合适的前缀  **[PROJECT_CODE] / BACKPORT / HACK / TEST / WIP / HACK**
> - 公司允许的情况下，尽量向社区贡献补丁，反哺社区  :-D
