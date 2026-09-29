# macOS 的 Python 

macOS 系统下可以安装多个 Python，系统自带，brew 安装的，用户手动安装。
- 系统的一般在 /usr/bin/python、/usr/bin/python3
- brew 安装的一般在 /opt/homebrew/bin/python3.13

检查方法
- 使用 which cmd: `which -a python` or `which -a python3`
- `brew list | grep python`  看所有安装的 Python


# Python 虚拟空间

Python 允许你创建多个 **虚拟环境（venv/virtualenv/conda）**

- 创建：python3 -m venv my_env
- 激活：source my_env/bin/activate
- 退出：deactivate
- 删除：rm -rf my_env


# 什么是 PIP

pip 是**Python 包管理工具**，该工具提供了对Python 包的查找、下载、安装、卸载的功能。 目前如果你在python.org 下载最新版本的安装包，则是已经自带了该工具。 注意：Python 2.7.9 + 或Python 3.4+ 以上版本都自带pip 工具



# MacOS 下的 PIP

查看 Python 版本：

```
# python --version
Python 3.9.6
```

```shell
// pip --version     # Python2.x 版本命令
pip3 --version    # Python3.x 版本命令

pip install SomePackage              # 最新版本
pip install SomePackage==1.0.4       # 指定版本
pip install 'SomePackage>=1.0.4'     # 最小版本

eg.
pip install Django==1.7
pip install --upgrade SomePackage // 升级
pip uninstall SomePackage
pip search SomePackage
pip show 
pip show -f SomePackage
pip list // 列出已安装的包
pip list -o // 查看可升级的包
```



## pip 升级

```
pip install --upgrade pip    # python2.x
pip3 install --upgrade pip   # python3.x
```
