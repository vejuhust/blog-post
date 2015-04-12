
==================
Config

# 基本配置
git config --global user.name "Wei Ye"
git config --global user.email "vejuhust@gmail.com"
git config --global color.ui auto
git config --global color.ui true

# 高级配置
git config --global core.editor vim
git config --global core.autocrlf input     # Mac/Linux
#git config --global core.autocrlf true     # Windows
#git config --global core.autocrlf false    # if use Windows only
git config --global help.autocorrect 1
git config --global merge.tool vimdiff  # Linux
#git config --global merge.tool opendiff # Mac
git config --global push.default simple
git config --global pull.rebase true
git config --global rerere.enabled true

# 取消配置
git config --local --unset core.autocrlf input
git config --global --unset help.autocorrect 1

# 查看所有配置
git config --global --list

# 修改所有配置
git config --global --edit
* System: vim /etc/gitconfig
* User: vim ~/.gitconfig
* Repository: vim .git/config


# 创建alias
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


# 使用alias
git la
git st


==================
Basic

# 获取帮助
git help
git help config

# 初始化本地repo
git init

# 查看文件状态
git status

# 添加修改
git add README.md
git add --update    # 添加全部tracked状态的文件
git add --all       # 添加全部文件
git add .           # 添加当前目录
git add *.txt       # 添加当前目录中全部txt文件
git add '*.txt'     # 添加当前及子目录中全部txt文件

# 提交修改(文字用一般现在时)
git commit -m "Add README.md"
git commit -a -m "Modify readme.md" # Add changes from all tracked files

# 查看当前修改
git diff            # 查看自上次commit以来unstaged的修改
git diff --staged   # 查看staged的修改

# 隐式忽略
vim .git/info/exclude

# 显式忽略
vim .gitignore

# 列出所有被忽略的文件
git ls-files --other --ignored --exclude-standard

# 删除并停止追踪
git rm README.txt

# 仅停止追踪
git rm --cached file.log

# 清理本地untracked的文件
git clean -nd   # 仅列出将删除的文件、目录
git clean -fd   # 强制删除
git clean -Xnd  # 仅删除被ignored的文件、目录
git clean -xnd  # 删除untracked及被ignored的文件、目录


==================
Undo

# 不要尝试撤销/修改已经push的commit

# 撤销文件级修改
git reset readme.md         # 与 git reset HEAD readme.md 相同
git reset HEAD readme.md    # 取消staged状态，即Unstage
git checkout -- readme.md   # 撤销单个文件的当前修改，"--"用于区分文件和branch，不可省

# 撤销commit级修改
git reset HEAD              # 撤销当前所有staged操作
git reset --hard HEAD       # 撤销当前所有staged操作，并还原文件
git reset --soft HEAD^      # 撤销上一次commit，并将文件置于staging状态
git reset --hard HEAD^      # 撤销上一次commit，并还原文件
git reset --hard HEAD^^     # 撤销前两次commit，并还原文件

# commit补刀方法
git add todo.txt
git commit --amend -m "Modify readme.md & add todo.txt"


==================
Branch

# 查看当前branch
git branch

# 创建新branch
git branch cat
git branch dog 123abc   # 创建新branch并添加指定commit

# 切换branch
git checkout cat
git checkout master

# 创建并切换到新branch
git checkout -b dog

# 删除branch
git branch -d cat   # 要求改动被merged
git branch -D cart  # 强制删除


# 列出所有的tag
git tag
git tag -l -n10     # 并显示最多10行的comment

# 回滚到指定tag (kinda like rollback)
git checkout v0.0.1

# 将commit添加为tag (name & description)
git tag -a v0.0.3 -m "version 0.0.3"            # 当前commit
git tag -a v0.0.2 -m "version 0.0.2" 5f216e3    # 指定commit

# 删除commit
git tag -d v0

# 发布新添加的tag
git push --tags


==================
Publish

# 从本地添加远程repo
git remote add origin git@github.com:vejuhust/learn-git.git

# 从本地移除远程repo
git remote rm origin

# 查看远程repo
git remote -v

# push到远程repo
git push -u origin master   # -u将其设为默认，下次仅需git push

# 将本地staging分支push到bingint上的master分支
git push bingint staging:master

# 从远程repo下载更新
git fetch

# 从远程repo进行pull - git fetch & git merge origin/master
git pull
git push --rebase   # 避免无意义的merge commit

# git clone - download repo & git remote add origin & check out initial branch
git clone git@github.com:vejuhust/learn-git.git
git clone git@github.com:vejuhust/learn-git.git demo

# 创建远程repo
git checkout -b cart
git push origin cart

# 删除远程repo
git push origin :cart
git branch -d cart

# 本地清理已删除的远程repo
git remote prune origin

# 列出所有的远程branch
git branch -r

# 对比本地和远程branch状态
git remote show origin


==================
Merge

# Recursive merge, not fast-forward
git merge --no-ff dog

# 将cat branch合并入master
git checkout master
git merge cat

# 可以merge指定commit
git merge 123abc

# 多人协作时先同步代码再发布
git pull
git push

# 出现冲突时需要编辑冲突部分
git pull
git mergetool
git commit -a
git push

# git status -- unmerged


==================
Rebase

# 缓存本地独有修改，重现远程修改，在缓存区重现所有修改
git rebase

# Local branch rebase
git checkout admin
git rebase master
git checkout master
git merge admin

# Conflicts
git rebase --continue
git rebase --skip
git rebase --abort













==================
History

# 查看指定文件的历史
git log --follow README.md

# 优化历史输出格式
git log --pretty=format:"%h %ad - %s [%an]" # 自定义输出格式
git log --pretty=oneline    # 每个commit输出为一行
git log --oneline           # 每个commit输出为一行，SHA1缩减为前6个字符
git log --patch             # patch风格输出
git log --oneline -p        # 简约的patch风格输出
git log --oneline --stat    # 显示文件变化状况
git log --oneline --graph   # 字符图形化显示，突显branch变化

# 只查看一段时间内的记录
git log --until=1.minute.ago
git log --since=1.hour.ago
git log --since=1.month.ago --until=2.weeks.ago
git log --since=2013-07-15 --until=2015-01-15

# 查看指定commit
git log --patch --max-count=1 5f216e3
git show 5f216e3

# 查看具体到行的修改
git diff                    # 与 git diff HEAD 相同
git diff HEAD               # 上次commit后出现的修改
git diff HEAD^              # 对比上上次commit后至今的修改
git diff HEAD~5             # 对比5次之前commit后至今的修改
git diff HEAD^..HEAD        # 比对出上次commit所进行的修改
git diff 123456..abcdefg    # 比对出123456到abcdefg之间所进行的全部修改
git diff master admin       # 对比不同branch

# 显示每一行的最后修改者
git blame index.html --date short

# 查看reflog
git reflog

==================
==================
Rebase

# 缓存本地独有修改，重现远程修改，重现缓存区的所有修改
git rebase

# Local branch rebase
git checkout admin
git rebase master
git checkout master
git merge admin

# Resolve conflicts
git fetch
git rebase --continue
git rebase --skip
git rebase --abort

# Interactive rebase
git rebase -i HEAD~3    # rebase当前HEAD之前三个commit至今
* rebase按行执行，修改行的顺序可以改变rebase顺序
* p, pick - 选择此commit
* r, reword - 选择并修改commit说明
* e, edit - 选择并通过amend方式修改commit
* s, squash - 与上一行commit合并
* f, fixup - 类似squash，但不保留该commit的message
* x, exec = run command (the rest of the line) using shell


==================
Stashing

# 暂存当前修改，并恢复到上一次commit内容
git stash                           # 与 git stash save 相同
git stash save
git stash save "hotfix"             # 添加message信息
git stash save --keep-index         # 仅stash处于unstaged状态的修改
git stash save --include-untracked  # 包含untracked的文件

# 查看被stash的内容
git stash list              # 查看所有stash
git stash list --stat       # 查看所有stash的文件状态，接受类 git log 的参数
git stash show              # 与 git stash show stash@{0} 相同
git stash show stash@{0}    # 查看指定的stash，接受类 git log 的参数

# 恢复被stash的内容
git stash apply                 # 与 git stash apply stash@{0} 相同
git stash apply stash@{0}       # 恢复到指定的stash
git stash branch dev stash@{0}  # 创建新的branch，并pop到此

# 删除stash
git stash drop              # 与 git stash drop stash@{0} 相同
git stash drop stash@{0}    # 删除指定的stash
git stash clear             # 删除全部stash

# 恢复并删除被stash的内容，若apply遇到冲突则不会drop
git stash pop               # 与 git stash apply & git stash drop 相同

# 若出现冲突，先commit或reset，再apply




==================
Purge Histroy

# 清理记录中不想公开的部分

# 主要适用情况——
* 版权纠纷
* 体积过大的二进制文件
* 修改为尚未公开的commit

# 先备份
git clone learn-git learn-git-filter
cd learn-git-filter

# 清洗历史 - 按commit逐条执行command，若中途command执行失败则停止
git filter-branch --tree-filter <command>               # 按commit逐条执行command，然后重新commit
git filter-branch --tree-filter 'rm -fr passwords.txt'  # 删除repo根目录下的passwords.txt
git filter-branch --tree-filter 'find . -name "*.mp4" -exec rm {} \;'   # 删除所有目录中的.mp4
git filter-branch --tree-filter 'rm -fr passwords.txt' -- --all     # 删除所有branch的repo根目录下的passwords.txt
git filter-branch --tree-filter 'rm -fr passwords.txt' -- HEAD      # 删除当前branch的repo根目录下的passwords.txt

git filter-branch --index-filter <command>              # 按commit逐条在staging上执行command，避免重新commit
git filter-branch --index-filter 'git rm --cached --ignore-unmatch passwords.txt'   # 停止追踪repo根目录下的passwords.txt
git filter-branch -f --index-filter <command>           # 强制第二次执行 git filter-branch 命令
git filter-branch -f --prune-empty -- --all             # 删去因为清洗而变空的commit
git filter-branch --index-filter 'git rm --cached --ignore-unmatch passwords.txt' --prune-empty -- --all 




==================
Team work

vim .gitattributes
*       text=auto
*.html  text
*.css   text
*.bat   text eol=crlf
*.sh    text eol=lf
*.jpg   binary
*.png   binary


# 从其他branch复制commit
git cherry-pick 123abc
git cherry-pick -x 123abc                   # 在message中添加来源commit的SHA信息
git cherry-pick --signoff 123abc            # 在message中添加进行操作人员的信息
git cherry-pick --edit 123abc               # 修改message
git cherry-pick --no-commit 123abc 456def   # 一次多个commit，需要自行commit



==================
Submodules

# repo内包含其他repo

# 在repo中添加submodule
git submodule add git@github.com:vejuhust/shared.git
git commit -m "Add shared submodule"
git push

# 更新submodule
cd shared               # Step 1: 更新submodule本身
git checkout master     # 此步不可缺少，因HEAD默认在no branch上
git commit -a -m "Update shared."
git push
cd ..                   # Step 2: 更新所母repo
git commit -a -m "Update shared."
git push    
git push --recurse-submodules=check     # 检查是否忘记push submodule
git push --recurse-submodules=on-demand # 将repo和submodule均push

# 对有submodule的repo进行clone
git clone git@github.com:vejuhust/learn-git.git
git submodule init
git submodule update

# 对有submodule的repo进行pull
git pull
git submodule update




==================
Reflog

# 查看本地reflog
git reflog
git log --walk-reflogs  # 显示细节

# 恢复到指定误删commit
git reset --hard HEAD@{1}

# 恢复指定误删commit到新branch
git branch aviary HEAD@{1}


