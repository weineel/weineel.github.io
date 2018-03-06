---
title: 关于git reset 的总结
date: 2017.10.27 17:00:19
tags: git
categories: git
---

----
git reset／revert 就是程序员的后悔药。主要总结了reset，懂了reset自然就懂了revert(更安全的reset，后悔药的后悔药)，但是git reset 会有更干净的commit路径。
<!-- more -->

#### 常用格式
1. `git reset [option] [commit]` //重制指定提交，可以具体指定到文件，否则就是所有文件。
```
# 重置前一次提交的filename.java，会丢弃所有修改没有任何保留。
git reset --hard head^ filename.java
```
2. `git reset head [file list]` // 重制为当前提交，一般用于重制暂存区到工作区。
3. `git reset head^` // 重制到当前提交的前次提交
4. `git reset head~n` // 重制到当前提交的前n此提交，即在这次提交之后提交都删除, head^ 相当于head~1
5. `git reset [commit-hash/tag name]` // 可以指定提交的hash值或者tag名

#### 选项详解
options: 如果会把重置的内容放到哪个位置，哪个位置就不能有和重置内容相同的文件。
1. `--mixed` // 重置提交和暂存区，即所有被重置的提交(和暂存区)的修改都放到工作区, 默认选项。在可能情况下不会影响工作区原有内容。
2. `--soft` // 重置提交, 即被重置的提交的修改放到暂存区。在可能情况下不会影响暂存区和工作区原有内容。

3. `--hard` // 重置提交, 暂存区，工作区。将丢失所有被重置的修改。
4. `--merge` // 保留工作区，直接丢弃暂存区的修改，丢弃所有重置的提交  。。
5. `--keep` // 保留工作区，把暂存区的内容放到工作区(两个区中不能有相同的文件)，丢弃所有重置的提交。
