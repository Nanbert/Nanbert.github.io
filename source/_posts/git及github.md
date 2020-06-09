---
title: git及github
date: 2020-06-09 08:21:42
subtitle:
categories:
tags:
cover:
---
1.git预设置
　`git config --global user.name "xxx"`
　`git config --global user.email "xx.com"`
　`git config --list`检查已有配置信息
　`git config --global core.editor vim`配置默认编辑器
　`git config --global alias.comi commit`命令别名
　`git config --global credential.helper cache`设置无密码推送
2.git 初始化
　`git init`初始化仓库，并建立.git子目录
3.`git status`
　-s:简单展现，A表示新添加到缓存区,左边M表示文件被修改并放入缓存区,右边M表示文件被修改并没有放入缓存区,??问号表示未被跟踪的文件
4.`git add <files>`
　跟踪文件并加入暂存区
5.`git diff`显示更为详细的变化
　默认比较工作目录与缓存区间的差异
　`--cached`比较缓存区与已提交的差异
　HEAD查看所有改动
　--stat:显示摘要
6.`git commit`
　注释行里会有最后一次运行git status的输出
　-v:会将diff输出加入到注释行
　-m:将提交信息与命令放在同一行
  -a:可以将所有跟踪文件跳过add步骤直接commit
  --amend:上次提交后发现忘了几个文件,可以add之后再加上此选项提交将会覆盖上次提交。
7.`git rm <files>`
　移除缓存区文件并且工作目录也一并移除
　--cached:只删除缓存区,保留工作目录
8.`git mv`移动文件
　等价于'mv file1 file2;git rm file1;git add README'
9.`git log`显示提交记录
　-p:显示每次提交的差异
  -(n):显示最近n次提交
  --since,--after:仅显示指定时间之后的提交
  --until,--before:仅显示指定时间之前的提交
  --author:仅显示指定作者相关的提交
  --committer:仅显示指定提交者相关的提交
  --grep:仅显示含指定关键字的提交
  -S:仅显示添加或移除了某个关键字的提交
  --stat:简略信息
  --graph:显示ASCLL图形表示分支的合并历史
  --relative-date:使用较短的相对时间显示
  --abbrev-commit:仅显示SHA-1的前几个字符,而非所有的40个字符
  --name-status:显示增删改的文件清单
  --name-only:仅在提交信息后显示以修改的文件清单
  --pretty=format:"<格式>":指定格式输出
  |选项|说明|
  |:-:|:-:|
  |%H|提交对象(commit)的完整哈希字串|
  |%h|提交对象的简短哈希字串|
  |%T|树对象的完整哈希字串|
  |%t|树对象的简短哈希字串|
  |%P|父对象的完整哈希字串|
  |%p|父对象的简短哈希字串|
  |%an|作者的名字|
  |%ae|作者的邮件|
  |%ad|作者修订日期|
  |%ar|作者修订日期,按多久以前的方式显示|
  |%cn|提交者的名字|
  |%ce|提交者的电子邮件|
  |%cd|提交日期|
  |%cr|提交日期,按多久以前的方式显示|
  |%s|提交说明|
10.`git reset HEAD <file>`:撤销某个暂存文件
11.`git checkout -- [file]`:撤销某个工作目录下文件的修改,危险的命令,你的修改将不会保存
12.`git remote`远程仓库的使用
　-v:显示简写对应的url
　`show origin`:展示origin的具体信息
　`rename o1 o2`:重命名某个远程
　`rm xx`:移除某个远程
13.`git push <remote-name> <branch-name>`
　`--tags`:推送所有标签
　使用git push <远程主机名(origin)> <本地分支名>:<远程分支名>
14.
　git fetch:会抓取数据到本地数据库，但不会自动合并并修改当前工作
  git clone:会自动将其添加为远程仓库并默认以'origin'缩写,并自动跟踪远程master
  git pull:主区数据并自动尝试合并到当前所在分支
15.git tag
　-a:增加个标签,可以在后面加个校验和,为指定的补打标签
　-m:在命令行增加说明
　-d:删除某个标签
　必须显示地推送标签至远程库'git push origin v1.5',当某个标签被删除或信息改变'git push origin  :refs/tags/v1.42'
///////**************************************************///
/***分支管理***/
`git clone -b <分支名> <仓库地址>`克隆指定分支
1.`git branch xxx`创建分支,不加任何xxx会显示所有分支
2.`git log --oneline --decorate`查看各个分支当前所指对象
3.`git checkout xxx`切换分支,切换分支会改变工作目录里的文件
4.`git log --oneline --decorate --graph --all`查看分叉历史
5.`git merge xxbranch`合并指定分支到当前分支,如果当前分支可以沿着一条线走下去则会有'fast-forward提示'
6.`git checkout -b serverfix origin/serverfix`跟踪远程库其他分支等价于`git checkout --track orighin/serverfix`
7.`git push origin --delete xxx` 删除远程分支
8.git branch 选项
 -d xxx: 删除分支
 -D xxx:强制删除某个未合并的分支
 -v:显示每个分支最后的提交
 --no-merged:查看所有包含未合并工作的分支
9. 
