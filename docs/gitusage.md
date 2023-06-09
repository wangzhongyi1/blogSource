# git 日常使用

## git 基本使用
- 在 github 上新建一个仓库
- 在本地使用 `git clone xxx` 命令拉取远程仓库到本地
- 修改文件后使用 `git add xxx` 添加更改
  + `git add .` 添加本次修改的所有文件到暂存区
  + `git add <filename>` 指定本次修改的文件到暂存区 
- 使用 `git commit -m "xxx"` 添加本次提交注释
- 使用 `git push` 推送到远程仓库
  + `git push origin main` 指定推送到远程的 main 仓库
  + `git push` 实际上是简写
    * `gut push -u origin master` 推送到远程的 master 仓库，并且记住当前操作
    * 下次直接使用 `git push` 就会自动推送到远程 master 仓库，就不用每次都指定推送的远程仓库了

## 查看本次修改内容
- `git diff`
## 撤销本次修改内容
### 未使用 git add 缓存代码时
- `git checkout -- readme.md`
> 本地修改会消失

### 已经使用了 git add 缓存代码时
- `git reset HEAD readme.md`
- 放弃所有的缓存 `git reset HEAD .`
> 此命令用来清除 git 对于文件修改的缓存。仅仅撤出 add 区域，本地修改不会消失

## git 查看分支图
- `git log --graph --decorate --oneline --simplify-by-decoration --all`

## 版本回退
- `git reset --hard HEAD^` 回退到上一个版本
- 回退到指定版本
    + `git log` 查看日志，`git reflog` 查看所有操作的日志
    + `git reset --hard 版本号`
    + `git push -f` 强制推送到远程仓库

## commit 以后，想撤销
### 撤销到 add 状态，即不改变修改的代码，仅仅撤出 commit 区域
- `git reset --soft HEAD^`
### 直接撤销本次修改，会回退到上个本版的代码
- `git reset --hard HEAD^`

> 可查看：https://www.jianshu.com/p/a9f327da3562

## git 反做某次提交
- 如果你有 commit1，commit2(最新)，但此时要回退commit1的内容，同时要保留commit2内容
- `git revert -n 版本号` 需要回退的commit1版本号
- 可能会有冲突，需要手动解决，要重新 git add
- 然后正常 `git commit` 来生成一条记录，同时commit1改动的内容已经被取消了
- 也可以你自己去手动把 commit1 改动的内容删除掉，`git revert` 可以理解为代替你做了这件事

## git修改 commit注释
- `git rebase -i HEAD~2` 会展示最近的两次提交注释,最后面的数字可更改
    + 按键盘 `i` 键，进入编辑模式
    + 把你想要修改的注释的前面的 `pick` 更改为 `edit`
    + 然后按 `esc` 退出编辑模式，再输入 `:wq` 保存并退出
- `git commit --amend` 开始编辑注释
- `git rebase --continue`
- `git push --force origin master` 强制推送远程仓库，当然你也可以不强制

## git push 某个特定的commit
- `git push <remotename> <commit SHA>:<remotebranchname>`
    + `remotename` 远程仓库名，默认为 origin
    + `commit SHA` commit的id
    + `remotebranchname` 远程分支名
    + 例如：`git push origin 503edc5aaf:master`
> 上面的命令会将暂存区内 `commit SHA` 代表的提交 ***以及其之前尚未推送到远程的提交一起提交到远程***
- 如果想要通过上面命令推送一个指定的commit，就需要保证这个提交之前没有其他的提交了。如果不是，可以通过`git rebase i`改变提交的位置，使其之前没有其他提交
    + `git rebase i` 会列出暂存区中所有未push的commit
    + i键 进入编辑模式，光标定位到当前行，输入命令 `:m 0` 就会移入行首，然后输入 `:x` 退出
    + 最后通过上面的命令 push 这个特定的 commit
    + 参考：https://blog.csdn.net/u010730126/article/details/100743019

## git 合并某个分支的某次 commit 
- 例如：在 master分支想要合并 dev分支上的某次提交
  + 切换到 dev分支，拷贝下来你想要被合并的 commitId
  + 切换到 master分支，执行 `git cherry-pick 62ecb3` 来进行操作 

## git 设置和取消代理
> 使用情况：当你 `git clone xxx` 时，由于天朝网络原因可能克隆不下来，就可以利用 ssr v2ray vpn 等工具
> 开启科学上网工具后，然后设置 git 代理，就会走代理，到外网去拉取资源
### 查看代理
- `git config --global --get http.proxy`
- `git config --global --get https.proxy`
### 开启代理
- 开启 socks 代理
  + `git config --global http.proxy 'socks5://127.0.0.1:10809'` 后面的 127.0.0.1:10809 就是指向科学上网工具的服务地址
  + `git config --global https.proxy 'socks5://127.0.0.1:10809'`
- 开启 http/https 代理
  + `git config --global http.proxy http://127.0.0.1:10809`
  + `git config --global https.proxy http://127.0.0.1:10809`
### 取消代理
- `git config --global --unset http.proxy`
- `git config --global --unset https.proxy`