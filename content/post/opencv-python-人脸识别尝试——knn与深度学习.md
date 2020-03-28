---
title: "Opencv Python 人脸识别尝试——knn与深度学习"
date: 2019-06-08
draft: false

categories: ["深度学习"]
---


## 引言
人脸识别和人脸检测不同，人脸检测时检测到人脸位置，而人脸识别是基于人脸数据库，进行一些识别操作如识别某一个人像是数据库中的哪个标签。

需要说明的是，使用knn和Dense层的神经网络作为人脸识别算法只是我的尝试，在实际的使用中基本不使用这两种算法的。同时，经过实际测试，这样得到的结果极不准确，甚至可以说毫无效果（苦笑）。

---
## 人脸数据获取
进行人脸识别首先要有人脸数据库，我们可以用opencv调用摄像头，进行人脸检测，并将人脸灰度图片写入到(200, 200)的pgm文件作为我们的人脸数据库。

code
```py
import cv2

def generate():
    face_cascade = cv2.CascadeClassifier( # haar级联文件-人脸
        './cascades/haarcascade_frontalface_default.xml'
    )
    camera = cv2.VideoCapture(0)
    count = 0

    while True:
        ret, frame = camera.read()
        frame = cv2.flip(frame, 1)  # 翻转为正常角度
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY) #转灰度处理
        faces = face_cascade.detectMultiScale(gray, 1.3, 5) # 识别

        for (x, y, w, h) in faces:
            img = cv2.rectangle( # 画框图
                frame, (x, y), (x + w, y + h), (255, 0, 0), 2
            )
            # cv2.putText(
            #     img, 'name', (x, y - 20), cv2.FONT_HERSHEY_SIMPLEX, 1, 255, 2
            # )
            f = cv2.resize(gray[y: y+h, x: x+w], (200, 200)) # 统一大小
            cv2.imwrite('./data/at/name/%s.pgm' % str(count), f) # 将人脸数据写入到pgm文件中
            count += 1
        if count == 200:
            break

        cv2.imshow('camera', frame)
        if cv2.waitKey(int(1000 / 12)) & 0xff == ord("q"): # 等待ｑ键
            break

    camera.release()
    cv2.destroyAllWindows()
    return

if __name__ == '__main__':
    generate()
```
## 处理图片读取到并数组中
**这个代码是读取数据的模块，在后面的代码中多次调用以获取数据**。
将上一步存储的图片数据转化为可处理的numpy数组，提供了两种相似的接口函数。

read_images 为普通的读取到灰度的pgm图片返回的数组
read_images_binary 是将pgm图片进行了二值化处理得到的数组数据

返回的数据类型为 list, list, dict
分别为 图片数据、图片标签、标签和人名的映射字典

```py
import os, sys, re
import cv2
import numpy as np

def read_images(path, sz=None): # 读取自己的图片数据库
    c = 0
    X, y = [], []
    dic = {}
    for dirname, dirnames, filenames in os.walk(path): # 遍历文件夹下文件
        for subdirname in dirnames:
            subject_path = os.path.join(dirname, subdirname)
            # print("subdirname = ", dirname)
            # print("subject_path = ", subject_path)
            for filename in os.listdir(subject_path):
                if filename[-4:] != '.pgm': # 如果文件不是pgm文件,那么跳过
                    continue
                filepath = os.path.join(subject_path, filename) # 生成完整的文件名
                # print(filepath)
                im = cv2.imread(os.path.join(subject_path, filename), \
                                cv2.IMREAD_GRAYSCALE) # 读取pgm图片
                # print(np.shape(im))
                # 改变大小
                if sz is not None:
                    im = cv2.resize(im, (200, 200)) # 调整图片大小

                X.append(np.asarray(im, dtype=np.uint8))
                y.append(c)
            # print(re.findall(r'./data/at/(.*)', subject_path))
            dic[c] = re.findall(r'./data/at/(.*)', subject_path)[0]
            c = c + 1
    # print("c = ", c)
    # print(X)
    return [X, y], dic

def read_images_binary(path, sz=None): # 读取自己的图片数据库
    c = 0
    X, y = [], []
    dic = {}
    for dirname, dirnames, filenames in os.walk(path): # 遍历文件夹下文件
        for subdirname in dirnames:
            subject_path = os.path.join(dirname, subdirname)
            # print("subdirname = ", dirname)
            # print("subject_path = ", subject_path)
            for filename in os.listdir(subject_path):
                if filename[-4:] != '.pgm': # 如果文件不是pgm文件,那么跳过
                    continue
                filepath = os.path.join(subject_path, filename) # 生成完整的文件名
                # print(filepath)
                im = cv2.imread(os.path.join(subject_path, filename), \
                                cv2.IMREAD_GRAYSCALE) # 读取pgm图片
                ret, im = cv2.threshold(im, 120, 255, cv2.THRESH_BINARY)
                if sz is not None:
                    im = cv2.resize(im, (200, 200)) # 调整图片大小

                X.append(np.asarray(im, dtype=np.uint8))
                y.append(c)
            # print(re.findall(r'./data/at/(.*)', subject_path))
            dic[c] = re.findall(r'./data/at/(.*)', subject_path)[0]
            c = c + 1
    # print("c = ", c)
    # print(X)
    return [X, y], dic
```

## 调用opencv内置函数进行人脸识别
内置的三种人脸识别函数
```py
model = cv2.face.EigenFaceRecognizer_create() # 这个版本的opencv名字改了,和书上的有点不一样
model = cv2.face.LBPHFaceRecognizer_create(1, 8, 8, 90)
model = cv2.face.FisherFaceRecognizer_create()
```
调用opencv-python提供的这三种人脸识别函数，发现效果都不好。人脸总是检测错误，准确率极低，要它何用？（不知道是不是我使用姿势不太正确）于是我催生出自己实现人脸识别算法那的念头。

观察框出的人脸，觉得一个人的人脸图片，单单框出了人脸，那应该相似度很高啊。类似于手写数字识别，简单的knn算法可以达到很高的正确率，那么是不是可以用knn较好的解决这个问题。

实践出来的结果是：不是的。

code
```py
import cv2
import numpy as np
from PIL import Image, ImageDraw, ImageFont
from getData import read_images

def cv2ImgAddText(img, text, left, top, textColor=(0, 255, 0), textSize=20):
    img = Image.fromarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    draw = ImageDraw.Draw(img)
    fontText = ImageFont.truetype(
        "SimHei.ttf", textSize, encoding="utf-8")
    draw.text((left, top), text, textColor, font=fontText)
    return cv2.cvtColor(np.asarray(img), cv2.COLOR_RGB2BGR)

def face_rec(): # 人脸识别

    [X, y], label2name = read_images('./data/at')
    print("label2name:", label2name)
    print(np.shape(X))
    print("label2name = ", label2name)
    print(np.shape(X))
    # print("y = ", y)
    y = np.asarray(y, dtype=np.int32)

    # 调用人脸识别函数生成模型
    # model = cv2.face.EigenFaceRecognizer_create() # 这个版本的opencv名字改了,和书上的有点不一样
    model = cv2.face.LBPHFaceRecognizer_create(1, 3, 8, 8)
    model.train(np.asarray(X), np.asarray(y)) # 训练
    camera = cv2.VideoCapture(0)
    face_cascade = cv2.CascadeClassifier( # 分类器检测人脸
        './cascades/haarcascade_frontalface_default.xml'
    )
    while True:
        read, img = camera.read()
        img = cv2.flip(img, 1)  # 翻转为正常角度
        faces = face_cascade.detectMultiScale(img, 1.3, 5)
        for (x, y, w, h) in faces:
            img = cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
            roi = gray[x: x+2, y: y+h]
            roi = cv2.resize(roi, (200, 200), interpolation=cv2.INTER_LINEAR)
            params = model.predict(roi) # 预测
            print(params) # 返回 (图片标签, 置信度)
            if params[0] >= 0:
                name = label2name[params[0]]
            else:
                name = "未知"
            print("检测到%s" % name)
            print("label: %s, Confidence: %.2f" % (params[0], params[1]))
            img = cv2ImgAddText(img, name, x, y - 20, (255, 255, 0), 20)
            # cv2.putText(img, name, (x, y - 20),
            #             cv2.FONT_HERSHEY_SIMPLEX, 1, 255, 2) # putText只能写上ascii中的部分字符, 呵呵
        # print(img)

        cv2.imshow('camera', img)
        if cv2.waitKey(1000 // 12) & 0xff == ord("q"):
            break
    cv2.destroyAllWindows()

if __name__ == "__main__":
    face_rec() # 人脸识别
```


## knn算法进行人脸识别
简单的knn模板

knn 实现
```py
import numpy as np
import matplotlib.pyplot as plt
from getData import read_images

def knn(xtest, data, label, k): # xtest为测试的特征向量，data、label为“训练”数据集，k为设定的阈值
#     print(xtest.shape)
#     print(label.shape)
    exp_xtest = np.tile(xtest, (len(label), 1)) - data
    sq_diff = exp_xtest**2
    sum_diff = sq_diff.sum(axis=1)
    distance = sum_diff**0.5
    # print(distance)
    sort_index = distance.argsort()
    classCount = {}
    for i in range(k):
        one_label = label[sort_index[i]]
        classCount[one_label] = classCount.get(one_label, 0) + 1
    sortedClassCount = sorted(classCount.items(), key = lambda x:x[1], reverse=True)
    print(distance.sum())
    print(classCount)
    print(sortedClassCount)
    return sortedClassCount[0][0]

def main():
    [X, y], label2name = read_images('./data/at/')
    print(np.shape(X))
    X = np.array(X)
    Xx = X.reshape(X.shape[0], 40000)
    xdata, label = Xx, y
    result = knn(np.array(xdata[67]), xdata, label, 3)
    print("result = ", label2name[result])
    return result

if __name__ == '__main__':
    main()
```
调用knn的主体部分
```py
import cv2
import numpy as np
from PIL import Image, ImageDraw, ImageFont
from getData import read_images_binary
from knn import knn


def cv2ImgAddText(img, text, left, top, textColor=(0, 255, 0), textSize=20):
    img = Image.fromarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    draw = ImageDraw.Draw(img)
    fontText = ImageFont.truetype(
        "SimHei.ttf", textSize, encoding="utf-8")
    draw.text((left, top), text, textColor, font=fontText)
    return cv2.cvtColor(np.asarray(img), cv2.COLOR_RGB2BGR)

def face_rec(): # 人脸识别

    [X, y], label2name = read_images_binary('./data/at')
    X = np.array(X)
    Xx = X.reshape(X.shape[0], 40000)
    xdata, label = Xx, y

    y = np.asarray(y, dtype=np.int32)
    face_cascade = cv2.CascadeClassifier( # 分类器检测人脸
        './cascades/haarcascade_frontalface_default.xml'
    )
    videoCapture = cv2.VideoCapture(0)
    while True:
        read, img = videoCapture.read()
        img = cv2.flip(img, 1)
        faces = face_cascade.detectMultiScale(img, 1.3, 5)
        for (x, y, w, h) in faces:
            img = cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
            roi = gray[x: x+2, y: y+h]
            roi = cv2.resize(roi, (200, 200), interpolation=cv2.INTER_LINEAR)
            roi = np.array(roi).reshape(40000, )
            result = knn(roi, xdata, label, 3)
            print(result) # 返回结果的标签
            name = label2name[result]
            print("检测到%s" % name)
            img = cv2ImgAddText(img, name, x, y - 20, (255, 255, 0), 20)
        cv2.imshow('camera', img)
        if cv2.waitKey(1000 // 12) & 0xff == ord("q"):
            break
    cv2.destroyAllWindows()

if __name__ == "__main__":
    face_rec() # 人脸识别
```

## 使用Dense层神经网络进行人脸识别
简单的神经网络多分类器实现（效果不好，大概是数据太少，每个人只有200张(200, 200) 的pgm图片数据。

调用keras的高级API，搭积木一样的建立神经网络。

即使是这么少的数据，训练一次的时间也要好几分钟。调参调了好久，关键是调不出效果啊～

```py
from getData import read_images, read_images_binary
import numpy as np
from tensorflow.keras import models
from tensorflow.keras import layers

def to_onehot(y): # 将标签转化为独热码oen-hot
    print("max of y = ", np.max(y))
    onehots = np.zeros((len(y), np.max(y) + 1), dtype=np.float)
    for i in range(len(y)):
        onehots[i][y[i]] = 1.0
    print("shape of onehots : ", np.shape(onehots))
    return onehots

def get_model_dense(size=(200, 200)): # 设置一个全连接层网络，返回模型，未训练
    model = models.Sequential([
        layers.Dense(16, activation='relu', input_shape=((40000, ))),
        layers.Dense(16, activation='relu'),
        layers.Dense(16, activation='relu'),
        layers.Dense(4, activation='softmax')
    ])
    model.compile(
        optimizer='rmsprop',
        loss='categorical_crossentropy',
        metrics=['accuracy']
    )
    return model

def k_fold_validation(train_data, train_targets):
    k = 4
    num_val_samples = len(train_data) // k
    num_epochs = 5
    all_scores = []

    for i in range(k):
        print('processing fole # ', i)
        val_data = train_data[i * num_val_samples: (i + 1) * num_val_samples]
        val_targets = train_targets[i * num_val_samples: (i + 1) * num_val_samples]

        partial_train_data = np.concatenate(
            [train_data[:i * num_val_samples],
            train_data[(i + 1) * num_val_samples:]],
            axis=0
        )
        partial_train_targets = np.concatenate(
            [train_targets[:i * num_val_samples],
             train_targets[(i + 1) * num_val_samples:]],
            axis = 0
        )
        model = get_model_dense()
        model.fit(
            partial_train_data,
            partial_train_targets,
            epochs=num_epochs,
            batch_size=3
        )
        '''
        model5开始，对图片进行了阈值为１２０的图片二值化处理
        model6调整了参数（防止过拟合），从６４调成１６
        model7加了一层
        '''
        model.save('dense_model_7.h5')
        val_mse, val_mae = model.evaluate(val_data, val_targets)
        all_scores.append(val_mae)

def main():
    [X, y], label2name = read_images_binary('./data/at/') # 调用人脸数据
    X = np.array(X).reshape(len(X), 40000) / 255
    y = to_onehot(np.array(y))
    index = np.arange(len(X))
    np.random.shuffle(index) # 生成打乱的索引
    print("index = ", index)
    X = X[index] # 得到打乱的数据
    y = y[index]
    print(X)
    print(y)
    print(X.shape)
    print(y.shape)
    k_fold_validation(X, y)


if __name__ == "__main__":
    main()
```
主体部分
```py
import cv2
import numpy as np
from PIL import Image, ImageDraw, ImageFont
from getData import read_images
from knn import knn
from tensorflow.keras import models


def cv2ImgAddText(img, text, left, top, textColor=(0, 255, 0), textSize=20):
    img = Image.fromarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    draw = ImageDraw.Draw(img)
    fontText = ImageFont.truetype(
        "SimHei.ttf", textSize, encoding="utf-8")
    draw.text((left, top), text, textColor, font=fontText)
    return cv2.cvtColor(np.asarray(img), cv2.COLOR_RGB2BGR)

def face_rec(): # 人脸识别

    [X, y], label2name = read_images('./data/at')
    print("label2name: ", label2name)
    X = np.array(X)
    Xx = X.reshape(X.shape[0], 40000)
    xdata, label = Xx, y

    y = np.asarray(y, dtype=np.int32)
    face_cascade = cv2.CascadeClassifier( # 分类器检测人脸
        './cascades/haarcascade_frontalface_default.xml'
    )
    videoCapture = cv2.VideoCapture(0)

    model = models.load_model('dense_model_6.h5')

    while True:
        read, img = videoCapture.read()
        img = cv2.flip(img, 1)
        faces = face_cascade.detectMultiScale(img, 1.3, 5)
        for (x, y, w, h) in faces:
            img = cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
            gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
            roi = gray[x: x+2, y: y+h]
            roi = cv2.resize(roi, (200, 200), interpolation=cv2.INTER_LINEAR)
            roi = np.array(roi).reshape(40000, )
            prediction = model.predict(np.array([roi]))
            # print(np.shape(roi))
            print("dnn predict result :", prediction)
            index = np.argmax(prediction) # 最大值所在的索引
            name = label2name[index]
            print("检测到%s" % name)
            img = cv2ImgAddText(img, name, x, y - 20, (255, 255, 0), 20)
        cv2.imshow('camera', img)
        if cv2.waitKey(1000 // 12) & 0xff == ord("q"):
            break
    cv2.destroyAllWindows()

if __name__ == "__main__":
    face_rec() # 人脸识别
```

## 一些小知识点
### opencv putText无法写中文
putText()不能直接写上中文，那就用PIL库曲线救国了。下面是一个demo。
```py
#coding=utf-8
#中文乱码处理

import cv2
import numpy
from PIL import Image, ImageDraw, ImageFont

def cv2ImgAddText(img, text, left, top, textColor=(0, 255, 0), textSize=20):
    img = Image.fromarray(cv2.cvtColor(img, cv2.COLOR_BGR2RGB))
    draw = ImageDraw.Draw(img)
    fontText = ImageFont.truetype(
        "SimHei.ttf", textSize, encoding="utf-8")
    draw.text((left, top), text, textColor, font=fontText)
    return cv2.cvtColor(numpy.asarray(img), cv2.COLOR_RGB2BGR)

img = cv2.imread('./test.png')
img = cv2ImgAddText(img, "你好世界", 140, 60, (255, 255, 0), 20)
cv2.imshow('image', img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```
### 二值化图片
使用`cv2.threshold()`函数，设置`cv2.THRESH_BINARY`参数进行二值化。可以设置阈值。
```py
import cv2
import numpy as np

img = cv2.imread('test.jpg')
# img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
cv2.imwrite('test.jpg', cv2.Canny(img, 50, 120))
cv2.imshow('canny', cv2.imread('test.jpg'))
ret, img = cv2.threshold(img, 110, 255, cv2.THRESH_BINARY)
cv2.imshow('binary', img)
cv2.waitKey()
cv2.destroyAllWindows()
```

### 路径目录测试
由于代码文件和图片文件中间隔了两个文件夹，且图片所在文件夹名称代表图片标签名称，所以如何遍历文件夹，读取到有用的信息是个技术活。
```py
import os

path = './data'
for dirname, dirnames, filenames in os.walk(path):
    for subdirname in dirnames:
        subject_path = os.path.join(dirname, subdirname)
        # print(subject_path)
        for filename in os.listdir(subject_path):
            if filename[-4:] == '.pgm':
                filepath = os.path.join(subject_path, filename)
                print(filepath)
```

### np.random.shuffle() 测试
`np.random.shuffle()` 能够将某一迭代对象进行打乱。
会直接改变传递给他的对象，而不会返回值，需要注意。

用这个函数，生成索引的随机排列，可以很方便的得到打乱的数据和标签，从而更好的进行训练。
```py
import numpy as np

a = range(0, 10)
print(a)
print(type(a[9]))
print(np.array(a))
index = np.array(a)
np.random.shuffle(index)
print(index)

index = np.arange(7)
print(index)
```

###  keras保存和恢复模型
`from tensorflow.keras import models`
`model.save('dense_model_7.h5')`
`model = models.load_model('dense_model_6.h5')`
就不用每次都重新训练了。