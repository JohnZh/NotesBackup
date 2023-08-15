Zsh in MacOS



# 什么是 Shell



和系统内核交互的一个桥梁。键盘输入容易记忆的符号（shell 命令），然后 shell 解析，内核执行



# 什么是 zsh



Zsh 就是其中一种 shell

Mac 上要多种 shell，cmd：`cat /etc/shells`

```shell
/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

查看当前的 shell：`echo $SHELL`

切换 shell：`chsh -s /bin/zsh`

> 注意：切换到 zsh 后，.base_profile 里面的环境变量也会失效，需要在 zsh 的 profile 文件里面重新配置
>
> 补充：在对环境变量加载的时候会先加载系统的，然后加载 shell 的环境变量文件，比如 .bash_profile, .baserc, .zprofile



# 什么是 Oh-My-Zsh

可以理解为 Zsh 的加强已经插件管理工具。安装

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```



## Oh-My-Zsh 优点介绍

- 切换路径 cd 可使用方向键选择路径。

  ```shell
  cd [tab][tab]
  ```

- 缩写路径补全，用在进入很深的路径下

  ```sh
  cd ~/D/N [tab] -----> cd ~/Desktop/NotesBackup/
  ```

- 查看最近去过的 10 个目录，输入 d，再选择数字

- 选择最近使用过的 31 个目录，`cd -[tab]`

- 命令选择补全。`git [tab]`

- 命令参数补齐。也是按 tab 

- 大小写字母自动 fix。也是按 tab

- 丰富的主题，自带的主题在：~/.oh-my-zsh/themes 里，设置 `ZSH_THEME="robbyrussell"`

- 别名机制

- 智能纠错

- 插件多

  

  

  

  