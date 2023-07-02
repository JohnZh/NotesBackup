# node.js，npm，nvm，nrm
- node.js：js 的一种运行环境，Google V8 引擎进行的封装，是一个服务器端的 javascript 的解释器
- npm：包管理工具（ 包含关系，nodejs 中含有 npm）Node Package Manager
- nvm：NVM 是一种用于管理设备上的 Node 版本的工具

## 版本查看
npm：npm -v
node.js：node -v
nvm：nvm -v 

## 安装 nvm
- brew install nvm
- 如果命令行无法执行，~/.bash_profile 输入以下
```bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```


## nvm 查看当前环境 node  的情况
- nvm list

## nvm 安装指定版本 node
- nvm install v12.20.0 

## nvm 安装最新 node，长期支持版本 LTS
- nvm install latest
- nvm install --lts

## nvm 使用特定版本 \ 设置成默认
- nvm use v12.20.0
- nvm alias default v12.20.0
> 切换 node 版本后，npm 版本也会随着改变

## 升级 npm 到最新版本和指定版本
- npm install npm -g
- npm install npm@6.14.8 -g

## 删除 npm 缓存
- npm cache clean -f

## n 模块：专门用来管理 nodejs 的版本，安装
- npm install -g n
```bash
n stable // 把当前系统的 Node 更新成最新的 “稳定版本”
n lts // 长期支持版
n latest // 最新版
n 12.20.0 // 指定安装版本
```

## node & npm 版本对应
https://nodejs.org/zh-cn/download/releases/

## 什么是 nrm
nrm 是一个 npm 源管理器，r: registry

### 切换成淘宝或者官方的源
- npx nrm use taobao
- npx nrm use npm


# yarn, watchman
## yarn
- 是 Facebook 提供的替代 npm 的工具，可以加速 node 模块的下载
- npm install -g yarn
## watchman
- 由 Facebook 提供的监视文件系统变更的工具。安装此工具可以提高开发时的性能（packager 可以快速捕捉文件的变化从而实现实时刷新）。


# RN 组件

## rn/android/ios 对应关系

| RN         | Android    | IOS          |
| ---------- | ---------- | ------------ |
| view       | ViewGroup  | UIView       |
| Text       | TextView   | UITextView   |
| Image      | ImageView  | UIImageView  |
| ScrollView | ScrollView | UIScrollView |
| TextInput  | EditText   | UITextField  | 




# 问题汇总
## gradle 运行环境 jdk 版本不对
gradle 的运行需要 jdk 11，去下载 zulu11 版本，并且在 gradle.properties 里面配置
`org.gradle.java.home=/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home`

### macos 里 jdk 查询
- /usr/libexec/java_home -V

## 例子项目红屏报错问题：Unable to load script.
hint: make sure you're either running a Metro server ...

### fix 方法：替换 watchman 版本，用老的 watchman

5 月 30 号之后的 watchman 都有问题，所以使用5月 16 号的
```shell
watchman shutdown-server 
# To ensure an old version isn't still running brew uninstall watchman # Get rid of the buggy (e.g., 2022.05.30.00) version 

curl https://raw.githubusercontent.com/Homebrew/homebrew-core/8651d8e23d308e564414188509f864e40548f514/Formula/watchman.rb > "$(brew --repository)/Library/Taps/homebrew/homebrew-core/Formula/watchman.rb" 

brew install watchman 
# This installs the functioning 2022.05.16.00 version dowloaded above

brew pin watchman 
# Tell brew to leave this older version alone (don't forget to unpin once this problem is solved) 

cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/ 

git checkout -- watchman.rb 
# To avoid brew showing warnings about having uncommitted git changes
```

**curl 命令解释**：实际上就是从 https://github.com/Homebrew/homebrew-core/commits/master/Formula/watchman.rb 下载下 5 月 16 号提交的 hashcode 为 8651d8e 的 **Raw** 文件: https://raw.githubusercontent.com/Homebrew/homebrew-core/8651d8e23d308e564414188509f864e40548f514/Formula/watchman.rb ，并替换 /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/ 路径下的 watchman.rb

最后再次执行：
- brew install watchman