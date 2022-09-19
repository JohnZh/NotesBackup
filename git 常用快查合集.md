git 常用快查合集

---

# git config

## 命令别名 alias

```shell
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --' (取消暂存文件的命令)
git config --global alias.last 'log -1 HEAD' (查看最后一次提交的信息)
git config --global alias.visual "!gitk" 运行某个外部命令而非git附属工具，加！
```

## 清除 alias

```shell
git config --global --unset alias.ci
```

## 用户配置

```shell
git config --global user.name "John Zhang"
git config --global user.email john@gmail.com
```

## 显示中文路径（文件）

```
git config core.quotepath false
```

默认是 true，遇到路径带中文由于大于 ask2 码的范围（最大值 0x80，128）会使用引号+转义：比如 “\351”（八进制）。



# git tag

## 查看本地所有 tag

```shell
git tag
```

## 查看远端所有的 tag (branch & tag)

```shell
git ls-remote --tags
```

> 没有 --tags 会打印所有的 tags 和 branches

## 轻量级 tag

```shell
git tag 1.0.0-light
```

## 带附注的 tag

```shell
git tag -a 1.2.0 -m "Release 1.2.0"
```

## 切换到指定 tag

```shell
git checkout $tagName
```

## 查看 tag 版本信息

```shell
git show 1.1.1
```

## 删除 tag 和远端 tag

```shell
git tag -d 1.1.1

git push origin :refs/tags/releases-197
```

## 发布 tag

```shell
git push origin 1.2.0 (单个 tag)
git push origin --tags （所有 tags）
```



# git commit

## 当前改变合并到最后一次 commit

```shell
git commit --amend
```



# git log

## 查看某个文件历史版本

```shell
git log /Users/john/Desktop/mcd-android/app/src/main/java/jp/co/mcdonalds/android/view/framesurfaceview/BaseSurfaceView.java
```



# git checkout

## 回退文件到某个版本

```shell
git co 570fcd /Users/john/Desktop/mcd-android/app/src/main/java/jp/co/mcdonalds/android/view/framesurfaceview/BaseSurfaceView.java
```



# git rm

## 删除你不想追踪的文件

```shell
git rm $filename
```

## 删除远端文件/文件夹，但不删除本地的

```shell
git rm --cached .idea/gradle.xml .idea/runConfigurations.xml 
git rm -r --cached $folder 
```

完成后需要执行 commit + push



# git push

## 删除远端分支

```shell
git push origin :$branchName
```



# git remote

## 扫描远端分支删除本地缓存

```shell
get remote prune origin
```



# git revert

## 反做某个 commit，会新建一个 commit（要求 path clean）

```shell
git revert -n 8b89621019c9adc6fc4d242cd41daeb13aeb9861
..
git commit -m 'revert'
git push
```

## 反做某个 merged commit，会新建 commit

```shell
git revert b27267720b2660e6127e8682b505069e1da61c8e -m 1
```

-m 1: 代表两个节点中选择第一个节点作为 revert 后的结果，一般 merge 的时候 base 会是第一个，合入的会是第二个



# git reset

## 删除历史版本（commit）回到某个 commit

```shell
git reset --hard 0d1d7fc32
.. // 如果之前的 commit 已经 push 到远端了，再 push 需要强制 -f
git push -f
```

## 删除最近的 commit，回到上一(N)个 commit

```shell
git reset --hard HEAD^
git reset --hard HEAD^^ //上上个
git reset --hard HEAD-100 //上 100 个
```



# git reflog

## 回到了之前的 commit 之后又想回到放弃的 commit

```shell
git reflog // 查看所有的命令记录
git reset --hard 0d1d7fc32 // 回到那个放弃的 commit
```



# git stash

## 查看某个 stash 里存储的内容

```shell
git stash show -p stash@{0}
```



## 存储当前的修改 git stash

## 查看所有的存储 git stash list

## 应用储藏并将其从 stack 移除

```shell
git stash pop
```

## 从储藏中创建分支，创建成功，储藏移除

```shell
git stash branch $branchName
```



# .gitignore 语法备忘

- 单个文件：/filename



