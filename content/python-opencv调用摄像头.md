+++
title = "python-opencv调用摄像头"
date = 2018-09-12 16:40:08
[taxonomies]
tags = ["python", "opencv"]
categories = ["Learn"]

+++

# 关于

Python利用opencv的库实现对摄像头的数据的读取，并逐帧显示。

opencv的安装： 终端运行 `pip install python-opencv` 即可。

# 实现

## 用到的库

    import cv2
    import numpy as np #如果你需要对读取到的摄像数据进行进一步的应用。

## 调用摄像头

调用摄像头，`VideoCapture()` 内的数字表示的是摄像头的设备编号，笔记本内建摄像头的编号一般为 `0`，如果你想使用外置摄像头将数字改为 `1`等，随你的摄像头的设备编号决定，可以通过设备管理器查看并尝试。

    cap = cv2.VideoCapture(0)

##  读取并播放

在 `while` 循环中利用 `cap.read()` 读取摄像头的某一帧，通过 `imshow()` 来展示这一帧。等待1个单位时间后，如果检测到 `q` 按键被按下，就跳出循环，即停止读取和展示。

其中 `ret` 变量是`cap`读取之后返回的一个`bool` 变量，可以用它来判断是否读取到数据。

    while(True):
      ret,frame = cap.read(0) #读取数据
      cv2.imshow('Frame',frame) #展示结果
      
      if cv2.waitkey(1) & 0xFF == ord('q'):
        break


## 释放摄像头

在调用摄像头之后不要忘记释放摄像头，否则你的下次调用摄像头的时候可能会有被占用的报错。我们需要释放摄像头并关闭我们用来展示的窗口。

    cap.release()
    cv2.destoryAllWindows()

## 完整代码

附上上述操作的完整代码：

    import numpy as np  
    import cv2

    cap = cv2.VideoCapture(0)

    while(True):
        ret, frame = cap.read()

        cv2.imshow('Frame',frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()

# 其他操作


## 保存视频或者截图

如果我们想要保存其中的某一帧，即截图，使用 `cv2.imwrite()` 可以非常方便的完成该功能，`imwrite`的使用方法，戳 [这里](https://docs.opencv.org/3.0-beta/modules/imgcodecs/doc/reading_and_writing_images.html?highlight=imwrite#cv2.imwrite "imwrite")

对于保存视频需要稍微复杂一些些的操作，我们需要创建一个 `VideoWrite`对象，并指定保存的文件名，然后指定`Fourcc` 值(下节详细)。然后指定每秒的帧数(fps)和帧的大小。该对象的最后一个参数为`isColor`，如果为`True`则编码器需要彩色帧，默认为 `False`,即保存为灰度帧。

**Fourcc**是用于指定__视频编码器__的四字节代码，在 [fourcc.org](http://www.fourcc.org/codecs.php) 中可以找到可用的编码器列表（值得注意的是对于不同的系统适用的编码器似乎还不同） 。在这里我们使用 `cv2.VideoWriter_fourcc()` 对象来指定，比如使用 `cv2.VideoWriter_fourcc(* 'XVID')` 来指定`XVID`编码器。

代码如下：

    import numpy as np
    import cv2

    cap = cv2.VideoCapture(0)

    fourcc = cv2.VideoWriter_fourcc(*'XVID')
    out = cv2.VideoWriter('output.avi',fourcc, 20.0, (640,480))

    while(cap.isOpened()):
        ret, frame = cap.read()
        if ret==True:
            frame = cv2.flip(frame,0)

            out.write(frame)

            cv2.imshow('frame',frame)
            if cv2.waitKey(1) & 0xFF == ord('q'):
                break
        else:
            break

    cap.release()
    out.release()
    cv2.destroyAllWindows()

# 相关来源

[Opencv](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_gui/py_video_display/py_video_display.html "Opencv")
