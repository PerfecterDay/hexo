---
title: git 常用命令
date: 2018-10-30 21:52:10
tags: git
category: git
---
<img src="/pics/git-frame.jpg" width="50%" height="50%" />

## git 常用命令
0. git clean: 删除工作区中未跟踪（不在版本库也未索引的）的文件
1. git add:将文件添加到暂存区
2. git ls-files --satge:查看暂存区的文件
3. git rm --cached filename:从暂存区和工作区删除某个文件，提交后，版本库中会删除这个文件
4. git checkout head filename:从暂存区恢复工作区中的文件，通常是想放弃对工作区中的修改或者误删了工作区文件
5. git reset filename:从版本库中恢复某个文件，当你删除了工作区中的文件，又删除了暂存区中的文件，想恢复文件可以用这个命令
6. git commit:将暂存区中的文件提交到版本库
7. git commit --amend：将本次提交与前一次的提交合并为一次提交
8.  git checkout branchname:切换分支
9.  git checkout -b branchname:切换分支,如果分支名不存在则基于当前分支新建一个分支
10. git reset filename:从版本库中还原某个文件到暂存区，当想撤销某次git add filename操作时使用，不加filename时，重置整个暂存区，工作区内容不受影响
11. git reset:恢复暂存区文件为版本库状态,工作区文件不会改变
12. git reset --hard head~1:用上一次提交的版本库状态恢复暂存区和工作区
13. git log: 查看提交历史记录
14. git reflog:查看引用历史记录
15. git reset ref:将版本库恢复到某次提交状态，适用于往前reset后又想往后reset
16. git stash:将当前工作区中的修改保存起来,并用版本库中的内容恢复暂存区和工作区，当开发到一半时，需要处理另一个bug时使用
17. git stash pop:将保存的内容恢复到工作区

## git 分支
0. git checkout -b 本地分支名x origin/远程分支名x : 在本地新建分支x，并自动切换到该本地分支x
1. git branch -a:查看所有分支
2. git branch &lt;BranchName&gt;:创建 BranchName 分支
3. git checkout &lt;BranchName&gt;:切换到 BranchName 分支
4. git checkout -b &lt;BranchName&gt;:在当前分支基础上新建 BranchName 分支，并切换到 BranchName 分支
5. git brnch -d &lt;BranchName&gt;:删除 BranchName 分支
6. git merge &lt;BranchName&gt;: 将 BranchName 合并到当前分支
7. git rebase &lt;targetBranch&gt; &lt;sourceBranch&gt;: Rebase 实际上就是取出 sourceBranch 的一系列的提交记录，“复制”它们，然后在targetBranch 的后边逐个的放下去，创造更线性的提交历史。sourceBranch 省略时，表示将当前分支 rebase到 targetBranch。

## git远程命令
1. git clone: 从远程库克隆
2. git remote :显示远程主机
3. git remote add &lt;主机名&gt; &lt;网址&gt;:添加远程主机
4. git remote rm &lt;主机名&gt;：删除远程主机
5. git remote rename &lt;原主机名&gt; &lt;新主机名&gt;：远程主机的改名
6. git fetch &lt;远程主机名&gt;：取回远程主机所有更新
8. git fetch origin 远程分支名x:本地分支名x : 在本地新建分支x，但是不会自动切换到该本地分支x，需要手动checkout
9. git fetch &lt;远程主机名&gt; &lt;分支名&gt;：取回远程主机指定分支的更新
10. git pull &lt;远程主机名&gt; &lt;远程分支名&gt;:&lt;本地分支名&gt;：取回远程主机某个分支的更新，再与本地的指定分支合并
11. git pull &lt;远程主机名&gt; &lt;远程分支名&gt;：取回远程主机某个分支的更新，再与本地的当前分支合并
12. git branch --set-upstream &lt;本地分支名&gt; &lt;远程主机名&gt;/&lt;远程分支名&gt; ：手动建立追踪关系
13. git pull origin : 如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名
14. git pull :如果当前分支只有一个追踪分支，连远程主机名都可以省略
15.  git push remote localbranch:remotebranch : 将本地localbranch分支推送到remote的remotebranch分支上
16. git push remote localbranch : 将本地localbranch分支推送到remote的同名分支上
17. git push remote :remotebranch :删除remote的remotebranch分支
18. git push : 如果当前分支只有一个追踪分支，那么主机名都可以省略

## git高级命令
<img src="/pics/merge.png" width="30%" height="30%" />
图上的情况，并不是移动分支指针就能解决问题的，它需要一种合并策略。首先，我们需要明确的是谁和谁的合并，是 2，3 与 4，5，6的合并吗？说到分支，我们总会联想到线，就会认为是线的合并。其实不是的，真实合并的是 3 和 6。因为每一次提交都包含了项目完整的快照，即合并只是 tree 与 tree 的合并。

我们可以先想一个简单的算法。用来比较3和6。但是我们还需要一个比较的标准，如果只是3和6比较，那么3与6相比，添加了一个文件，也可以说成是6与3比删除了一个文件，这无法确切表示当前的冲突状态。因此我们选取他们的两个分支的分歧点（merge base）作为参考点，进行比较。

比较时，相对于 merge base（提交1）进行比较。

首先把1、3、6中所有的文件做一个列表，然后依次遍历这个列表中的文件。现在我们拿列表中的一个文件进行举例，把在提交1、3、6中的该文件分别称为版本1、版本3、版本6。

1. 版本1、版本3、版本6的 sha-1 值完全相同，这种情况表明没有冲突
2. 版本3或6至少一个与版本1状态相同（指的是sha-1值相同或都不存在），这种情况可以自动合并。比如1中存在一个文件，在3中没有对该文件进行修改，而6中删除了这个文件，则以6为准就可以了
3. 版本3或版本6都与版本1的状态不同，情况复杂一些，自动合并策略很难生效，需要手动解决。我们来看一下这种状态的定义。

冲突状态定义：
* 1 and 3: DELETED BY THEM;(them在feature分支)
* 1 and 6: DELETED BY US;(us表示我们在master分支)
* 3 and 6: BOTH_ADDED;
* 1 and 3 and 6: BOTH_MODIFIED
我们拿第一种情况举例，文件有两种状态 1 和 3，1 表示该文件存在于 commit 1（也就是MERGEBASE），3 表示该文件在 commit 3 （master 分支）中被修改了，没有 6，也就是该文件在 commit 6（feature 分支）被删除了，总结来说这种状态就是 DELETEDBY_THEM。

可以再看一下第四种情况，文件有三种状态 1、3、6，1 表示 commit 1（MERGEBASE）中存在，3 表示 commit 3（master 分支）进行了修改，6 表示（feature 分支）也进行了修改，总结来说就是 BOTHMODIFIED（双方修改）。

遇到不可自动合并冲突时，git会将这些状态写入到暂存区。与我们讨论不同的是，git使用1，2，3标记文件，1表示文件的base版本，2表示当前的分支的版本，3表示要合并分支的版本。
git merge   合并时也会有 fast-forward 模式。
1. git merge &lt;分支名&gt;： 将指定分支合并到所在分支
