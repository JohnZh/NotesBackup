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



# 安装 Whisper

安装最新发布

```
pip3 install -U openai-whisper
```

这个安装好后还需要配置环境变量，因此可以直接用 brew 来安装：

```
brew install openai-whisper
```



Whisper 需要 ffmpeg 命令行工具

```
# on MacOS using Homebrew (https://brew.sh/)
brew install ffmpeg
```



## Whisper 使用

```
whisper jvideo.mp4  // 直接识别语言，然后翻译
whisper jvideo.mp4 --language Japanese // 制定语言
whisper jvideo.mp4 --language Japanese --task translate //翻译成 En
```



# 补充：翻译完的非熟悉外语如何处理

推荐 https://www.nikse.dk/subtitleedit/online

点击「Auto-translate」，选择翻译引擎，然后在弹出窗口中选择字幕要翻译的语言，并**将页面拖动到最下方**（非常重要），确定所有文字都被翻译后点击 OK 按钮

除了网页翻译字幕，本地端的神经机器翻译也是种好选择。macOS 用户推荐使用 [Argos Translate](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2Fargosopentech%2Fargos-translate)，这是基于 OpenNMT 的开源神经机器翻译。如果你的动手能力较强，可以尝试 [Opus-MT](https://sspai.com/link?target=https%3A%2F%2Fgithub.com%2FHelsinki-NLP%2FOpus-MT)。