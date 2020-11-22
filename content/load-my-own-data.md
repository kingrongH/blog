+++
title = "tensorflow 加载自己的数据"
date = 2018-09-03 19:29:30
[taxonomies]
tags = ["tensorflow"]
categories = ["Learn"]

+++

# 关于

很多时候我们学习tensorflow或者其他deeplearning的课程的时候，我们都被牵着鼻子走，使用一些捏造出来的数据，苦于不能处理我们自己。所以我们今天来尝试一下怎么加载自己的图片数据，将图片数据转换成我们通常学习的数组数据，便于应用我们学习到的深度学习的知识。

我们需要使用的库有`os`、`np`、`cv2`、`pickle` 等。

![lib](https://i.loli.net/2018/09/04/5b8def3e54dd5.png "lib")


# 代码

    import numpy as np 
    import matplotlib.pyplot as plt
    import os
    import cv2
    import random
    import pickle

    DATADIR = "./Datasets/PetImages"
    CATEGORIES = ['Cat','Dog']
    IMG_SIZE = 50  #将size设置为50可以识别出物体 又降低了数组大小

    training_data = []

    def create_training_data():
        for category in CATEGORIES:
            path = os.path.join(DATADIR, category) # path to cats or dogs dir  
            class_num = CATEGORIES.index(category)   #0 is cat , 1 is dog 
            for  img in os.listdir(path):
                try:
                    img_array = cv2.imread(os.path.join(path,img), cv2.IMREAD_GRAYSCALE)   #将图片数据转换为数组 因为颜色并不影响识别所以采用IMREAD_GRAYSCALE
                    new_array = cv2.resize(img_array, (IMG_SIZE,IMG_SIZE))
                    training_data.append([new_array,class_num])
                except Exception as e:
                    pass

    create_training_data()
    random.shuffle(training_data) #将所有数据进行随机排序有助于提高训练识别水平


    X_temp = []
    y = []

    for feature,label in training_data:     #分离图片数据和标签信息
        X_temp.append(feature)
        y.append(label)

    X = np.array(X_temp).reshape(-1,IMG_SIZE,IMG_SIZE,1)


    #保存数据 以供下次调用或者其他程序调用
    pickle_out = open('X.pickle',"wb")
    pickle.dump(X,pickle_out)
    pickle_out.close()

    pickle_out = open('y.pickle',"wb")
    pickle.dump(y,pickle_out)
    pickle_out.close()

    #加载数据举例
    pickle_in = open("X.pickle","rb")
    X = pickle.load(pickle_in)


`来源`： `[sentdex](https://pythonprogramming.net/loading-custom-data-deep-learning-python-tensorflow-keras/)`

