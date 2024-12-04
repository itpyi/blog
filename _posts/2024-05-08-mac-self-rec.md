---
title: Mac 录制系统声音
category: [技术笔记]
---

希望截取一段新闻联播录音做语音分析，因此想录制电脑输出的声音。搜索了一圈，很多方案都过于笨重。看到少数派上的[这篇文章](https://sspai.com/post/61420)，使用 [BackgroundMusic](https://github.com/kyleneideck/BackgroundMusic)即可实现。

少数派文章里的下载命令有些过时，可参考 BackgroundMusic 的说明，使用如下命令：
```bash
brew install --cask background-music
```

录制时需将电脑输出通道和录制输入通道都设置为 Background Music。