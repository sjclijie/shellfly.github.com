---
layout: post
title: "git history"
date: 2013-01-22 00:08
comments: true
categories: memo
tags: git
---

### 修改上一次commit
`git commit --amend`,如果上次提交后没有做任何修改，运行这个命令可以修改上次提交的message，如果你发现少提交了一个文件，或者有其他小的改动
觉得应该放在上一个commit里面，就可以在commit最后加上`--amend` 参数，来更新上次commit。

### 查看历史记录 git log vs git reflog
平时大家一般都习惯使用 `git log` 查看当前commit记录，但是默认只显示出当前的最新状态，如果一个commit被其他命令删除，这里是就看不见了。
`git reflog` 则记录了目前为止的所有动作，包括经过的所有commit。要想恢复一个被删除的commit,`git reflog` 是不二之选。`git reflog`
其实就是`git log -g --abbrev-commit --pretty=oneline`的一个alias。

### 撤销commit git revert vs git reset
<!-- more -->
`git revert <commit>` 会创建一个新的commit,这个commit会撤销命令里给出的commit的所有操作，并且不会修改原有commit历史记录。要撤销最近一次commit最好的办法就是使用`git revert`。当然也可
以指定很多commit 这些commit会被一一撤销，如果有中间出现冲突，需要像`git merge`那样手动解决冲突。

`git reset (--soft | --mixed | --hard | --merge | --keep) [-q] [<commit>]` 恢复到指定的commit，会修改commit的历史记录,可以回溯到任一commit,就像时光倒流一样，这也是版本控制的好处所在。

* `--soft`
  只是把当前HEAD指向commit，并不改变index和work tree里的文件，所有和commit不同的文件都会变成“changes to be commited”
  
* `--mixed`
  默认选项，除了把HEAD指向commit外，还会更新index文件，但不影响work tree里的文件。
    
* `--hard`
  把HEAD指向commit，并且更新index和work tree里的文件，如果想彻底删除一个commit，可以选择这个命令
