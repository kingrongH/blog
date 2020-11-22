+++
title = "Python播放MP3 总结"
date = 2018-11-15 14:15:08
[taxonomies]
tags = ["python", "MP3"]
categories = ["Python"]

+++

# 关于

最近在做一个项目，需要在树莓派中使用到python来播放MP3文件，树莓派这个设备比较特殊，不像是我们库和驱动都比较齐全的电脑，里面很多东西都是精简化的。所以使用各种各样的库的过程中我总结了一下自己的经验。

如何使用python播放MP3文件。

# 方法

花了大量的时间来进行测试，最终整理出以下的方法：

* playsound
* mp3play
* pygame
* pyglet
* pyaudio
* python-vlc
* subprocess & mpg123

## playsound 
	
playsound 在我的Linux的电脑上工作正常而且工作良好。使用方法也是非常简单，

1. 使用 `````pip3 install playsound` 来进行安装
2. 代码简单
	
		import playsound 
		playsound.playsound('/path/to/your/mp3file')

3. `playsound` 的文档：[playsound](https://pypi.org/project/playsound/)

但是`playsound` 在我的树莓派上却无论如何也跑不起来，程序执行完毕也没有错误提示，弃之。

## mp3play

`mp3play` 在我的Linux电脑上甚至都没有跑起来，原因是它支持python2或者别的什么原因

1. 使用 `pip install mp3play` 进行安装
2. 代码示例
	
		import mp3play

		filename = r'C:\Documents and Settings\Michael\Desktop\music.mp3'
		clip = mp3play.load(filename)

		clip.play()

3. `mp3play` 参考文档：[mp3play](https://pypi.org/project/mp3play/) 
	
在树莓派上同样跑不起来，弃之

## pygame

`pygame` 相较于前两者来说是一个大型的东西，它有更多的用处，但是我只想用它来播放一个mp3，没有在我的电脑上进行测试，直接在树莓派上测试成功。

1. 使用`pip3 install pygame` 来进行安装
2. 代码示例

		from pygame import mixer 

		mixer.init()
		mixer.music.load('/path/to/your/mp3file')
		mixer.music.play()

3. `pygame` 参考文档：[pygame](https://www.pygame.org/docs/) 

`pygame` 虽然在树莓派上一次性测试成功，但是播放出来的音频却是语速失调的音频，人说话的声音听起来就像是萝莉音，弃之。

## pyglet 

`pyglet` 和`pygame`是一个级别的，但是就播放音频来说它的使用明显又复杂许多。 在树莓派上的使用失败。

1. 使用 `pip3 install pyglet` 来进行安装
2. 代码示例

		import pyglet
		from pyglet.gl import *
		pyglet.options['audio'] = ('openal', 'directsound', 'silent')

		music = pyglet.resource.media('music.mp3')
		music.play()

		pyglet.app.run()

3. `pyglet` 参考文档：[pyglet](https://pyglet.readthedocs.io/en/pyglet-1.3-maintenance/programming_guide/media.html)
`pylet` 虽然看起来专业很多但是对于我的项目的使用来说就很复杂，在树莓派上测试失败，弃之。在使用`pyglet`的过程中遇到一个问题在树莓派上似乎没有办法解决，那就是提示`AVbin`的相关问题，要么缺失，要么就是其他的。在树莓派上无论是下在稳定的还是最新的版本的`AVbin`都无法解决这个问题。一些相关的文档贴在这里希望能帮到各位：

[AVbin Downloads](http://avbin.github.io/AVbin/Download.html)
[Install AVbin](https://stackoverflow.com/questions/10302873/python-pyglet-avbin-how-to-install-avbin) 

## pyaudio

`pyaudio` 使用似乎还要进行一些巧妙的设置才能进行正常的播放，我没有对它进行测试

1. 使用 `pip3 install pyaudio ` 来进行安装
2. `pyaudio` 官方的文档：[pyaudo](http://people.csail.mit.edu/hubert/pyaudio/)

## python-vlc

`python-vlc` 是VLC 的python接口库其github地址为[python-vlc](https://github.com/oaubert/python-vlc) ，该库的使用需要调用VLC的库，理论上是可以播放vlc支持的媒体文件。但是在我单薄的树莓派上就测试失败了。进过多次测试并解决了一些些问题，但是失败了。

1. 使用`pip3 install python-vlc` 来进行安装，没有VLC还需要安装VLC：树莓派上`sudo apt-get install vlc`
2. 代码示例
	
		import vlc 
		p = vlc.MediaPlayer('/path/to/your/mp3file')
		p.play()

3. `python-vlc` 的文档：[python-vlc](https://wiki.videolan.org/PythonBinding)

使用`python-vlc`来播放mp3从代码上看是十分简单的，但是还需要在树莓派中安装vlc的，依赖许多，很容易撞见各种问题，弃之。


## subprocess & mpg123

`subprocess` 并不是一个直接播放音频的库，使用它来调用系统软件来播放音频，代码十分简单，音调语速上也是正常的。

1. 使用 `sudo apt-get install mpg123` 
2. 代码示例

		import subprocess
		subprocess.Popen(['mpg123', '-q', '/path/to/your/mp3file']).wait()

3. 参考文档：[subprocess](https://docs.python.org/3/library/subprocess.html) 

这个方法或许是最为简单的方法了，依赖少，代码简洁，易于控制。在树莓派上一次试验就成功了。

**********************************************

# 参考文档

[play mp3 on python](https://stackoverflow.com/questions/20021457/playing-mp3-song-on-python)

[play mp3 with pyaudio](https://stackoverflow.com/questions/26673746/playing-mp3-files-with-pyaudio)


