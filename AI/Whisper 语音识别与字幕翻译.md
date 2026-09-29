# 安装 Whisper

pip 的用法见 [[Python 学习记录]]。

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