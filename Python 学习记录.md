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
