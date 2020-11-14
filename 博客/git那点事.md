# git那点事

我以前只会 git 的clone、remote、pull、add、commit、push、merge、reset、stash、这些基本操作，这里我通过[learngitbranching](https://learngitbranching.js.org/?locale=zh_CN)这个网站系统了学习了一下 git 的其他操作。通过本文，您可以学到：

1. git的常用工作流是怎样的？
2. git rebase 怎么操作？
3. git cherry-pick 怎么操作？
4. git revert 是什么？
5. git 相对引用是怎么回事？
6. git tag 怎么使用
7. git branch -f 和 git branch xx xxx

## 工作流

一般来说，git 工作流分为**git-flow工作流**和**rebase工作流**。

**git-flow工作流**是指，git里面维护了master、develop、hotfix、feature、release这些分支：新功能通过新建feature分支然后合并到develop分支，最后发布到master分支；热更通过新建hotfix分支然后合并到develop分支，最后发布到master分支等等。

**rebase工作流**是指，git里面只维护了master分支，所有其它的分支在完成后都需要 rebase master 分支，整理好提交信息之后再提交，这样 master 分支的提交信息就不杂乱。

注意：两种工作流**不是互斥**的，有的公司两种工作流都一起使用。

## git rebase

git rebase 的使用场景分为两种，一种是**交互式整理提交**；一种是**让git时间线更流畅**。

```bash
## 交互式整理提交
## 下面的命令会打开一个交互式vim，然后选择对之前的各个commit做一些操作，常用的操作是reword和squash
## HEAD~3 是git的相对引用，我们后面会讲
git rebase -i (HEAD~3)

## 让git时间线更流畅
## 下面的命令会把当前分支的提交作为新的提交加到master上面
git fetch
git rebase origin master
git rebase --continue
git rebase --abort
```

需要注意的是：

1. git rebase 针对的是没有 push 的 commit，所以对于 commit**不要轻易 push 到远程仓库**
2. git rebase 打开交互式 vim 之后，**使用 i 进行修改，使用 wq 退出**
3. git rebase 会改变 git 的历史提交，所以只适用于**只有你一个人维护**的分支上
4. git pull --rebase 相当于 git fetch 和 git rebase；git pull 相当于 git fetch 和 git merge
5. git rebase 可以**带第二个参数**，```git rebase xx xxx```表示在 xxx 分支上以 xx 分支为基准执行变基，并且切换到 xxx 分支。
6. git rebase 之后，在 master 分支可以 merge，也可以 rebase，也可以 cherry-pick 你的分支，具体**看喜好**。

## git cherry-pick

git cherry-pick 可以用来**选取提交节点**，使用方法很简单，命令如下：

```bash
git cherry-pick <提交号>
```

注意：

1. 有时 git cherry-pick 可以用来代替 git rebase 的部分功能；而且 git commit --amend 也可以用来合并然后修改上次的提交信息

## git revert

我们都知道，使用 git reset 可以把未 push 的 commit 进行回退，但是如果已经 push 了呢？使用 git revert 即可。这个命令**会创建一个新的提交，这个提交会删除上个提交的内容**。命令如下：

```bash
git revert HEAD
git push
```

## git 相对引用

我们经常会看到比如```HEAD^```、```HEAD^^```、```HEAD~3```这种形式的命令，这个就是**相对引用**。意思如下：

```
HEAD^: 表示当前HEAD上一个节点，如果有2个^就表示上上个节点，以此类推
HEAD~3: 表示当前HEAD向上移动多少个节点，不加数字时表示向上移动一个
```

注意：

1. ```git rebase -i (HEAD~3)```表示将之前3个节点进行 rebase。
2. ```git checkout master^2```表示回到 master 的第二个父节点（按提交的时间顺序），注意**这里的 ^ 后面接了数字，表示第几个父节点**。如果不接数字的话，表示提交时间最早的那个父节点。

## git tag

git tag 可以用来**打标签**，后面的提交号是可选的，如果没有则表示在当前HEAD上面打标签。

```bash
git tag v1 <提交号>
```

## git branch 高阶用法

我们都知道```git checkout -b xxx```是```git branch xxx```和```git checkout xxx```的缩写形式，于是git branch也有两个类似的高阶用法：

```bash
## 表示在 xxx 节点创建 xx 分支，但是不切换到这个分支
git branch xx xxx

## 表示将已经存在的 xx 分支强制移动到 xxx 节点，并且不切换到这个分支
git branch -f xx xxx
```
