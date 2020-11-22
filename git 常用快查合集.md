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

## 删除 tag

```shell
git tag -d 1.1.1
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



# .gitignore 语法备忘

- 单个文件：/filename
