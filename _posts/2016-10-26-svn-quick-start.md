---
layout: post
title: SVN快速使用总结
---

本文总结了完成一个需求开发所需知道的SVN基本指令。

# 一般开发流程
- [获取代码(更新代码)](#获取代码更新代码)
- [新建分支](#新建分支) 
- [查看分支](#查看分支)
- [提交分支(解决冲突)](#提交分支解决冲突)
- [合并分支](#合并分支)
- [删除分支](#删除分支)

# 各环节SVN基本命令
## 获取代码(更新代码)
第一次从仓库里获取代码

```sh
$ svn checkout [your_repository_url]
```

如果已有本地代码，则需要**更新代码**，确保在最新代码上开发

```sh
$ cd [your_local_codes_path]
$ svn update
```

## 新建分支
SVN仓库中，trunk分支一般存放通过测试的最新代码，因此不应该在该分支下修改代码。合理的做法是：从trunk分支切出一个feature分支，然后在该分支下修改代码，直到测试通过后合并到trunk分支。

### 建立分支

```sh
$ svn copy [your_trunk_url] [your_feature_branch_url] -m [your_log]
```

> `-m` 填写建立分支的日志

### 切换分支
从trunk分支切换到feature分支

```sh
$ svn switch [your_feature_branch_url]
```

## 查看分支
切换分支后就可以进行代码修改。修改一段时间后，如果想了解当前分支文件的状态，可以通过以下命令:

```sh
$ svn status
```

该命令显示的格式为：**状态** **文件**，常见的状态有：

|字符|状态|说明|
|:---:|:---:|---|
|A|添加|与上一版相比增加的文件|
|C|冲突|该文件冲突|
|D|删除|该文件已从仓库删除，以后SVN不再跟踪版本|
|M|修改|该文件被修改过|
|S|处于其他分支|当前分支的子路径处于其他分支|
|?|未纳入版本管理|通常是新增文件，SVN还没跟踪该文件的版本。可以使用`svn add`把文件加入SVN，此时再运行`svn status`时，文件的状态就显示为A|
|!|文件缺失|SVN找不到该文件。一般出现在没有使用SVN命令删除文件的情况。当需要从仓库删除某文件时，应该使用`svn delete`，这样文件的状态就变为D，提交以后该文件的版本就不再被跟踪|

## 提交分支(解决冲突)
开发完成后向远程分支提交代码

```sh
$ svn commit -m [your_log]
```

一般情况下代码就提交到远程分支了，但是如果有人和你修改了同一段代码，并且先提交到了远程分支，就会导致本次提交失败，此时需要先解决冲突再提交代码。

### 解决冲突
更新代码，显示冲突

```sh
$ svn update
$ Conflict discovered in [your file]
  Select: (p) postpone, (df) diff-full, (e) edit,
  (mc) mine-conflict, (tf) theirs-conflict,
  (s) show all options:
```

|符号|说明|
|:-:|-|
|p|标记冲突，暂不处理|
|df|显示所有冲突|
|e|编辑冲突|
|mc|冲突以本地文件为准|
|tf|冲突以远程仓库为准|
|s|显示所有选项|

一般先输入`df`命令看冲突是否严重，如果不严重则通过`e`直接编辑，编辑页面通常为
```sh
<<<<<<< .mine
[your_version]
=======
[their_version]
>>>>>>> [version]
```
在`<<<<<<< .mine`和`>>>>>>> [version]`之间解决冲突，然后保存。回到Select界面，此时会多出一个`(r) resolve`的命令。输入`r`通知SVN已解决冲突。

### 使用postpone解决冲突
如果冲突很严重，需要和提交者讨论解决，可以输入`p`标记，此时输入`svn status`显示：
 
```sh
 C [your_file]
 ? [your_file].working
 ? [your_file].merge-left.[version]
 ? [your_file].merge-right.[version]
```

|文件|说明|
|:---:|----|
|[your_file]|所有冲突标记在该文件|
|[your_file].working|当前工作副本|
|[your_file].merge-left.[version]|产生冲突前基础版本|
|[your_file].merge-right.[version]|仓库里的最新版本|

用以下命令解决冲突
```sh
$ svn resolve --accept [base | working | mine-conflict | theirs-conflict | mine-full | theirs-full] [conflicting file] 
```

|参数|说明|
|---|---|
|base|将[your_file].merge-left.[version]做为最终结果|
|working|把[your_file]解决冲突后的结果做为最终结果|
|mine-conflict|将[your_file].working做为最终结果|
|theirs-conflict|将[your_file].merge-right.[version]做为最终结果|
|mine-full|将所有[your_file].working做为最终结果|
|theirs-full|将所有[your_file].merge-right.[version]做为最终结果|

解决冲突后，文件状态变为M，这时再向仓库提交代码即可。

## 合并分支
feature分支通过测试后就可以合并到trunk分支。首先切换到trunk分支，然后执行以下命令

```sh
$ svn merge [your_feature_branch_url]
```

> merge还具有回滚的功能：`svn merge -r old:new .`。注意不要少最后一个点，这表示把new版本会滚到old版本

顺利的话，feature分支就合并到trunk分支了，但是如果有别人和你修改了同一段代码并且提交到trunk分支就可能再次出现冲突。同样先解决冲突再提交。

## 删除分支
完成功能开发，合并到trunk后，删除feature分支

```sh
$ svn delete [your_feature_branch_url] -m [your_log]
```
# 结束
以上就是开发过程中常用的SVN命令，当然SVN的命令是非常丰富的，想要更强大的功能可以通过`svn help`来进一步学习。






