---
layout: post
title: Git Class Notes
excerpt: "Notes of getting started with Git based on CodeSchool courses"
modified: 2015-04-12
tags: [git, tool]
comments: true
---

{% include _toc.html %}

使用Git也有一段时间了，但一直没有系统的学习过。随着使用频率的增加，觉得还是这还是有必要的。于是趁着春节假期，在CodeSchool[^1]上把Git Path [^2]的四门课学完了。边学边练边记，最后总结成此文。

[^1]: <https://www.codeschool.com/>
[^2]: <https://www.codeschool.com/paths/git>


# Configuration

## Environment
添加基本配置:
{% highlight bash %}
git config --global user.name "Wei Ye"
git config --global user.email "vejuhust@gmail.com"
git config --global color.ui auto
git config --global color.ui true
{% endhighlight %}

添加高级配置:
{% highlight bash %}
git config --global core.editor vim
git config --global core.autocrlf input     # Mac/Linux
#git config --global core.autocrlf true     # Windows
#git config --global core.autocrlf false    # if use Windows only
git config --global help.autocorrect 1
git config --global merge.tool vimdiff
git config --global push.default simple
git config --global pull.rebase true
git config --global rerere.enabled true
{% endhighlight %}

删除配置:
{% highlight bash %}
git config --local --unset core.autocrlf input
git config --global --unset help.autocorrect 1
{% endhighlight %}

查看配置:
{% highlight bash %}
git config --global --list
{% endhighlight %}

批量修改配置:
{% highlight bash %}
git config --global --edit
{% endhighlight %}

亦可直接修改配置文件:

* System: `vim /etc/gitconfig`
* User: `vim ~/.gitconfig`
* Repository: `vim .git/config`


## Aliases
添加别名:
{% highlight bash %}
git config --global alias.la        "log --pretty=format:'%h %ad - [%an] %s' --graph"
git config --global alias.lg        "log --graph --decorate --pretty=oneline --abbrev-commit --all"
git config --global alias.s         "status -s"
git config --global alias.st        "status"
git config --global alias.sr        "remote show origin"
git config --global alias.br        "branch"
git config --global alias.bra       "branch -a"
git config --global alias.co        "checkout"
git config --global alias.ci        "commit"
git config --global alias.cia       "commit -a"
git config --global alias.d         "diff"
git config --global alias.ds        "diff --staged"
git config --global alias.pushall   "push --recurse-submodules=on-demand"
{% endhighlight %}

使用别名:
{% highlight bash %}
git la
git st
{% endhighlight %}

其他方面和环境参数一致。



# Basic Operation

## Get Started
获取帮助:
{% highlight bash %}
git help
git help config
{% endhighlight %}

初始化本地仓库:
{% highlight bash %}
git init
{% endhighlight %}

查看文件状态:
{% highlight bash %}
git status
{% endhighlight %}

添加修改:
{% highlight bash %}
git add README.md
git add --update    # 添加全部tracked状态的文件
git add --all       # 添加全部文件
git add .           # 添加当前目录
git add *.txt       # 添加当前目录中全部txt文件
git add '*.txt'     # 添加当前及子目录中全部txt文件
{% endhighlight %}

提交修改:
{% highlight bash %}
git commit -m "Add README.md"
git commit -a -m "Modify readme.md" # Add changes from all tracked files
{% endhighlight %}

:star2: **Advice**: commit message 文字应使用一般现在时。
{: .notice}

查看当前修改:
{% highlight bash %}
git diff            # 查看自上次commit以来unstaged的修改
git diff --staged   # 查看staged的修改
{% endhighlight %}


## Ignore Files
隐式忽略特定文件:
{% highlight bash %}
vim .git/info/exclude
{% endhighlight %}

显式忽略特定文件:
{% highlight bash %}
vim .gitignore
{% endhighlight %}

列出所有被忽略的文件:
{% highlight bash %}
git ls-files --other --ignored --exclude-standard
{% endhighlight %}

删除并停止追踪:
{% highlight bash %}
git rm README.txt
{% endhighlight %}

仅停止追踪:
{% highlight bash %}
git rm --cached file.log
{% endhighlight %}

清理本地未被追踪的文件:
{% highlight bash %}
git clean -nd   # 仅列出将删除的文件、目录
git clean -fd   # 强制删除
git clean -Xnd  # 仅删除被ignored的文件、目录
git clean -xnd  # 删除untracked及被ignored的文件、目录
{% endhighlight %}



# Undo

:warning: **Warning**: 不要尝试撤销/修改已经push的commit。
{: .notice}

撤销文件级修改:
{% highlight bash %}
git reset readme.md         # 与 git reset HEAD readme.md 相同
git reset HEAD readme.md    # 取消staged状态，即Unstage
git checkout -- readme.md   # 撤销单个文件的当前修改，"--"用于区分文件和branch，不可省
{% endhighlight %}

撤销commit级修改:
{% highlight bash %}
git reset HEAD              # 撤销当前所有staged操作
git reset --hard HEAD       # 撤销当前所有staged操作，并还原文件
git reset --soft HEAD^      # 撤销上一次commit，并将文件置于staging状态
git reset --hard HEAD^      # 撤销上一次commit，并还原文件
git reset --hard HEAD^^     # 撤销前两次commit，并还原文件
{% endhighlight %}

commit后补上文件:
{% highlight bash %}
git add todo.txt
git commit --amend -m "Modify readme.md & add todo.txt"
{% endhighlight %}



# Branch

查看当前所有分支:
{% highlight bash %}
git branch      # 本地
git branch -r   # 远程
git branch -a   # 本地及远程
{% endhighlight %}

创建新分支:
{% highlight bash %}
git branch cat
git branch dog 123abc   # 创建新分支并添加指定commit
{% endhighlight %}

切换分支:
{% highlight bash %}
git checkout cat
git checkout master
{% endhighlight %}

创建并切换到新分支:
{% highlight bash %}
git checkout -b dog
{% endhighlight %}

删除分支:
{% highlight bash %}
git branch -d cat   # 要求改动被merged
git branch -D cart  # 强制删除
{% endhighlight %}



# Tag

列出所有的tag:
{% highlight bash %}
git tag
git tag -l -n10     # 并显示最多10行的comment
{% endhighlight %}

回滚到指定tag:
{% highlight bash %}
git checkout v0.0.1
{% endhighlight %}

将commit添加为tag:
{% highlight bash %}
git tag -a v0.0.3 -m "version 0.0.3"            # 当前commit
git tag -a v0.0.2 -m "version 0.0.2" 5f216e3    # 指定commit
{% endhighlight %}

删除tag:
{% highlight bash %}
git tag -d v0
{% endhighlight %}

发布tag:
{% highlight bash %}
git push --tags
{% endhighlight %}



# Publish

从本地添加远程仓库:
{% highlight bash %}
git remote add origin git@github.com:vejuhust/learn-git.git
{% endhighlight %}

从本地移除远程仓库:
{% highlight bash %}
git remote rm origin
{% endhighlight %}

查看远程仓库:
{% highlight bash %}
git remote -v
{% endhighlight %}

push到远程仓库:
{% highlight bash %}
git push -u origin master   # -u将其设为默认，下次仅需git push
{% endhighlight %}

将本地staging分支push到bingint上的master分支:
{% highlight bash %}
git push bingint staging:master
{% endhighlight %}

从远程仓库下载更新:
{% highlight bash %}
git fetch
{% endhighlight %}

从远程仓库进行pull:
{% highlight bash %}
git pull
git push --rebase   # 避免无意义的merge commit
{% endhighlight %}

:clipboard: **Info**: `git pull`等价于`git fetch && git merge origin/master`。
{: .notice}

克隆远程仓库:
{% highlight bash %}
git clone git@github.com:vejuhust/learn-git.git
git clone git@github.com:vejuhust/learn-git.git demo
{% endhighlight %}

:clipboard: **Info**: `git clone`等价于下载仓库，然后`git remote add origin`，最后check out主分支。
{: .notice}

创建远程仓库:
{% highlight bash %}
git checkout -b cart
git push origin cart
{% endhighlight %}

删除远程仓库:
{% highlight bash %}
git push origin :cart
git branch -d cart
{% endhighlight %}

本地清理已删除的远程仓库:
{% highlight bash %}
git remote prune origin
{% endhighlight %}

对比本地和远程分支状态:
{% highlight bash %}
git remote show origin
{% endhighlight %}



# Merge

递归合并(recursive)，禁用fast-forward策略:
{% highlight bash %}
git merge --no-ff dog
{% endhighlight %}

将cat分支合并入master:
{% highlight bash %}
git checkout master
git merge cat
{% endhighlight %}

合并指定commit:
{% highlight bash %}
git merge 123abc
{% endhighlight %}

:star2: **Advice**: 多人协作时，应当先同步代码再发布，即，先`git pull`再`git push`。
{: .notice}

出现冲突时需要编辑冲突部分:
{% highlight bash %}
git pull
git mergetool
git commit -a
git push
{% endhighlight %}



# Rebase

:clipboard: **Info**: `git rebase`会先缓存本地独有修改，重现远程修改，再重现缓存区的所有修改
{: .notice}

对本地分支进行rebase:
{% highlight bash %}
git checkout admin
git rebase master
git checkout master
git merge admin
{% endhighlight %}

处理rebase冲突:
{% highlight bash %}
git rebase --continue
git rebase --skip
git rebase --abort
{% endhighlight %}

交互式rebase:
{% highlight bash %}
git rebase -i HEAD~3    # rebase当前HEAD前的最近三个commit
{% endhighlight %}

* rebase按行执行，修改行的顺序可以改变rebase顺序
* `p, pick` - 选择此commit
* `r, reword` - 选择并修改commit说明
* `e, edit` - 选择并通过amend方式修改commit
* `s, squash` - 与上一行commit合并
* `f, fixup` - 类似squash，但不保留该commit的message
* `x, exec` - 用shell执行此行剩余部分的命令



# History

## Check History

查看指定文件的历史:
{% highlight bash %}
git log --follow README.md
{% endhighlight %}

优化历史输出格式:
{% highlight bash %}
git log --pretty=format:"%h %ad - %s [%an]" # 自定义输出格式
git log --pretty=oneline    # 每个commit输出为一行
git log --oneline           # 每个commit输出为一行，SHA1缩减为前6个字符
git log --patch             # patch风格输出
git log --oneline -p        # 简约的patch风格输出
git log --oneline --stat    # 显示文件变化状况
git log --oneline --graph   # 字符图形化显示，突显branch变化
{% endhighlight %}

只查看一段时间内的记录:
{% highlight bash %}
git log --until=1.minute.ago
git log --since=1.hour.ago
git log --since=1.month.ago --until=2.weeks.ago
git log --since=2013-07-15 --until=2015-01-15
{% endhighlight %}

查看指定commit:
{% highlight bash %}
git log --patch --max-count=1 5f216e3
git show 5f216e3
{% endhighlight %}

查看具体到行的修改:
{% highlight bash %}
git diff                    # 与 git diff HEAD 相同
git diff HEAD               # 上次commit后出现的修改
git diff HEAD^              # 对比上上次commit后至今的修改
git diff HEAD~5             # 对比5次之前commit后至今的修改
git diff HEAD^..HEAD        # 比对出上次commit所进行的修改
git diff 123456..abcdefg    # 比对出123456到abcdefg之间所进行的全部修改
git diff master admin       # 对比不同branch
{% endhighlight %}

显示每一行的最后修改情况:
{% highlight bash %}
git blame index.html --date short
{% endhighlight %}


## Purge Histroy

可以通过清理记录来移除不想公开的部分，主要适用情况——

* 版权纠纷
* 体积过大的二进制文件
* 修改为尚未公开的commit

:star2: **Advice**: 操作之前先进行本地备份，例如`git clone learn learn-backup`。
{: .notice}

借助`git filter-branch`按commit逐条执行command，若中途command执行失败则停止:
{% highlight bash %}
# 按commit逐条执行command，然后重新commit
git filter-branch --tree-filter <command>

# 删除repo根目录下的passwords.txt
git filter-branch --tree-filter 'rm -fr passwords.txt'

# 删除所有目录中的.mp4
git filter-branch --tree-filter 'find . -name "*.mp4" -exec rm {} \;'

# 删除所有branch的repo根目录下的passwords.txt
git filter-branch --tree-filter 'rm -fr passwords.txt' -- --all

# 删除当前branch的repo根目录下的passwords.txt
git filter-branch --tree-filter 'rm -fr passwords.txt' -- HEAD

# 按commit逐条在staging上执行command，避免重新commit
git filter-branch --index-filter <command>

# 停止追踪repo根目录下的passwords.txt
git filter-branch --index-filter 'git rm --cached --ignore-unmatch passwords.txt'

# 强制第二次执行 git filter-branch 命令
git filter-branch -f --index-filter <command>

# 删去因为清洗而变空的commit
git filter-branch -f --prune-empty -- --all
git filter-branch --index-filter 'git rm --cached --ignore-unmatch passwords.txt' --prune-empty -- --all
{% endhighlight %}


## Reflog

查看本地reflog:
{% highlight bash %}
git reflog
git log --walk-reflogs  # 显示细节
{% endhighlight %}

恢复到指定误删commit:
{% highlight bash %}
git reset --hard HEAD@{1}
{% endhighlight %}

恢复指定误删commit到新分支:
{% highlight bash %}
git branch aviary HEAD@{1}
{% endhighlight %}



# Stashing

暂存当前修改，并恢复到上一次commit内容:
{% highlight bash %}
git stash                           # 与 git stash save 相同
git stash save
git stash save "hotfix"             # 添加message信息
git stash save --keep-index         # 仅stash处于unstaged状态的修改
git stash save --include-untracked  # 包含untracked的文件
{% endhighlight %}

查看被stash的内容:
{% highlight bash %}
git stash list              # 查看所有stash
git stash list --stat       # 查看所有stash的文件状态，接受类 git log 的参数
git stash show              # 与 git stash show stash@{0} 相同
git stash show stash@{0}    # 查看指定的stash，接受类 git log 的参数
{% endhighlight %}

恢复被stash的内容:
{% highlight bash %}
git stash apply                 # 与 git stash apply stash@{0} 相同
git stash apply stash@{0}       # 恢复到指定的stash
git stash branch dev stash@{0}  # 创建新的branch，并pop到此
{% endhighlight %}

删除stash:
{% highlight bash %}
git stash drop              # 与 git stash drop stash@{0} 相同
git stash drop stash@{0}    # 删除指定的stash
git stash clear             # 删除全部stash
{% endhighlight %}

恢复并删除被stash的内容，若apply遇到冲突则不会drop:
{% highlight bash %}
git stash pop               # 与 git stash apply & git stash drop 相同
{% endhighlight %}

:star2: **Advice**: 若出现冲突，可以先`git commit`或`reset`，再`apply`。
{: .notice}



# Submodule

在仓库中添加submodule:
{% highlight bash %}
git submodule add git@github.com:vejuhust/shared.git
git commit -m "Add shared submodule"
git push
{% endhighlight %}

更新submodule内的文件:
{% highlight bash %}
cd shared               # Step 1: 更新submodule本身
git checkout master     # 此步不可缺少，因HEAD默认在no branch上
git commit -a -m "Update shared."
git push
cd ..                   # Step 2: 更新所母仓库
git commit -a -m "Update shared."
git push    
git push --recurse-submodules=check     # 检查是否忘记push submodule
git push --recurse-submodules=on-demand # 或者，将母仓库和submodule均push
{% endhighlight %}

对有submodule的仓库进行克隆:
{% highlight bash %}
git clone git@github.com:vejuhust/learn-git.git
git submodule init
git submodule update
{% endhighlight %}

对有submodule的仓库进行pull:
{% highlight bash %}
git pull
git submodule update
{% endhighlight %}



# Teamwork Features

## .gitattributes
通过属性，可以对个别文件或目录定义不同的合并策略:
{% highlight bash linenos %}
*       text=auto
*.html  text
*.css   text
*.bat   text eol=crlf
*.sh    text eol=lf
*.jpg   binary
*.png   binary
{% endhighlight %}


## git cherry-pick
从其他分支复制commit:
{% highlight bash %}
git cherry-pick 123abc
git cherry-pick -x 123abc                   # 在message中添加来源commit的SHA信息
git cherry-pick --signoff 123abc            # 在message中添加进行操作人员的信息
git cherry-pick --edit 123abc               # 修改message
git cherry-pick --no-commit 123abc 456def   # 一次多个commit，需要自行commit
{% endhighlight %}

