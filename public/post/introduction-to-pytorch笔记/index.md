

# Introduction to PyTorch 笔记

## Part 1 - Tensors in PyTorch (Solution).ipynb

1. 最基本的神经网络, 使用矩阵计算. 

2. 激活函数, sigmoid, softmax, relu等
3. 使用pytorch生成随机数(用来初始化weights). 似乎用不同的norm函数影响较大
4. 介绍了前向传播的实现方式, 矩阵相乘 + 偏置
5. 矩阵改变形状, 从numpy转化来/去



## Part 2 - Neural Networks in PyTorch (Exercises).ipynb

1. 加载MNIST数据集, 设置是训练或者测试, 用DataLoader进行分批加载, 可设置随机化

2. 使用了transforms转换器转换数据, 比如归一化, 转化为tensor, 改变图片大小等

    ```python
    train_transforms = transforms.Compose([transforms.RandomRotation(30),
                                           transforms.RandomResizedCrop(224),
                                           transforms.RandomHorizontalFlip(),
                                           transforms.ToTensor(),
                                           transforms.Normalize([0.5, 0.5, 0.5], 
                                                                [0.5, 0.5, 0.5])])
    ```

3. 练习自定义网络权重, 并实现网络的前向传播

4. 自己用pytorch实现了softmax. 使用sum, argmax等函数需要注意设置dim

    ```python
    def softmax(x):
        return torch.exp(out) / torch.exp(out).sum(dim=1).view(64, 1)
    ```

5. 自定义网络结构(class方式), 继承自nn.Module. 自定义层次, 激活函数等, 实现init, forward函数

    ```py
    class Network(nn.Module):
        def __init__(self):
            super().__init__()
            
            # Inputs to hidden layer linear transformation
            self.hidden = nn.Linear(784, 256)
            # Output layer, 10 units - one for each digit
            self.output = nn.Linear(256, 10)
            
            # Define sigmoid activation and softmax output 
            self.sigmoid = nn.Sigmoid()
            self.softmax = nn.Softmax(dim=1)
            
        def forward(self, x):
            # Pass the input tensor through each of our operations
            x = self.hidden(x)
            x = self.sigmoid(x)
            x = self.output(x)
            x = self.softmax(x)
            
            return x
    ```

6. 有model对象, 可方便地查看模型的权重, 偏置值, 还可以进行更改, 如使用随机值

7. 使用nn.Sequential()搭建网络, 和之前其实类似

    ```py
    model = nn.Sequential(nn.Linear(input_size, hidden_sizes[0]),
                          nn.ReLU(),
                          nn.Linear(hidden_sizes[0], hidden_sizes[1]),
                          nn.ReLU(),
                          nn.Linear(hidden_sizes[1], output_size),
                          nn.Softmax(dim=1))
    ```

    

8. 多次出现helper辅助代码, 实现一些功能, 如下所示

```py
import matplotlib.pyplot as plt
import numpy as np
from torch import nn, optim
from torch.autograd import Variable


def test_network(net, trainloader):

    criterion = nn.MSELoss()
    optimizer = optim.Adam(net.parameters(), lr=0.001)

    dataiter = iter(trainloader)
    images, labels = dataiter.next()

    # Create Variables for the inputs and targets
    inputs = Variable(images)
    targets = Variable(images)

    # Clear the gradients from all Variables
    optimizer.zero_grad()

    # Forward pass, then backward pass, then update weights
    output = net.forward(inputs)
    loss = criterion(output, targets)
    loss.backward()
    optimizer.step()

    return True


def imshow(image, ax=None, title=None, normalize=True):
    """Imshow for Tensor."""
    if ax is None:
        fig, ax = plt.subplots()
    image = image.numpy().transpose((1, 2, 0))

    if normalize:
        mean = np.array([0.485, 0.456, 0.406])
        std = np.array([0.229, 0.224, 0.225])
        image = std * image + mean
        image = np.clip(image, 0, 1)

    ax.imshow(image)
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    ax.spines['left'].set_visible(False)
    ax.spines['bottom'].set_visible(False)
    ax.tick_params(axis='both', length=0)
    ax.set_xticklabels('')
    ax.set_yticklabels('')

    return ax


def view_recon(img, recon):
    ''' Function for displaying an image (as a PyTorch Tensor) and its
        reconstruction also a PyTorch Tensor
    '''

    fig, axes = plt.subplots(ncols=2, sharex=True, sharey=True)
    axes[0].imshow(img.numpy().squeeze())
    axes[1].imshow(recon.data.numpy().squeeze())
    for ax in axes:
        ax.axis('off')
        ax.set_adjustable('box-forced')

def view_classify(img, ps, version="MNIST"):
    ''' Function for viewing an image and it's predicted classes.
    '''
    ps = ps.data.numpy().squeeze()

    fig, (ax1, ax2) = plt.subplots(figsize=(6,9), ncols=2)
    ax1.imshow(img.resize_(1, 28, 28).numpy().squeeze())
    ax1.axis('off')
    ax2.barh(np.arange(10), ps)
    ax2.set_aspect(0.1)
    ax2.set_yticks(np.arange(10))
    if version == "MNIST":
        ax2.set_yticklabels(np.arange(10))
    elif version == "Fashion":
        ax2.set_yticklabels(['T-shirt/top',
                            'Trouser',
                            'Pullover',
                            'Dress',
                            'Coat',
                            'Sandal',
                            'Shirt',
                            'Sneaker',
                            'Bag',
                            'Ankle Boot'], size='small');
    ax2.set_title('Class Probability')
    ax2.set_xlim(0, 1.1)

    plt.tight_layout()
```



## Part 3 - Training Neural Networks (Exercises).ipynb

1. 梯度下降与反向传播. 反向传播算法是求导过程中链式法则的应用
    $$
    \large \frac{\partial \ell}{\partial W_1} = \frac{\partial L_1}{\partial W_1} \frac{\partial S}{\partial L_1} \frac{\partial L_2}{\partial S} \frac{\partial \ell}{\partial L_2}
    $$

$$
\large W^\prime_1 = W_1 - \alpha \frac{\partial \ell}{\partial W_1}
$$

2. 通常会导入的包

    ```py
    import torch
    from torch import nn
    import torch.nn.functional as F
    from torchvision import datasets, transforms
    ```

    

3. 介绍了损失函数. 

    比如交叉熵nn.CrossEntropyLoss(), 一般赋值给criterion

    还有比如nn.NLLLoss()

    

4. 介绍了自动求导autograd, 调用.backward()可查看导数

5. 介绍了optimizer, 在torch.optim中. 有SGD(随机梯度下降等). Adam更好. 

    ```py
    from torch import optim
    
    # Optimizers require the parameters to optimize and a learning rate
    optimizer = optim.SGD(model.parameters(), lr=0.01)
    ```

    

6. 训练中记得清零梯度`optimizer.zero_grad()`
7. optimizer调用step()方法更新模型的权重

8. 一个较完整的训练过程

    ```py
    ## Your solution here
    
    model = nn.Sequential(nn.Linear(784, 128),
                          nn.ReLU(),
                          nn.Linear(128, 64),
                          nn.ReLU(),
                          nn.Linear(64, 10),
                          nn.LogSoftmax(dim=1))
    
    criterion = nn.NLLLoss()
    optimizer = optim.SGD(model.parameters(), lr=0.003)
    
    epochs = 5
    for e in range(epochs):
        running_loss = 0
        for images, labels in trainloader:
            # Flatten MNIST images into a 784 long vector
            images = images.view(images.shape[0], -1)
        
            # TODO: Training pass
            optimizer.zero_grad()
            output = model.forward(images)
    #         print(output.shape)
    #         print(labels.shape)
            
            loss = criterion(output, labels)
            
            running_loss += loss.item()
            
            loss.backward()
            optimizer.step()
        else:
            print(f"Training loss: {running_loss/len(trainloader)}")
    ```

    

## Part 4 - Fashion-MNIST (Exercises).ipynb

使用pytorch对数据集Fashion-MNIST进行分类的练习, 有如下过程

1. 导入必要的库
2. 导入数据集并格式化
3. 定义网络结构
4. 定义optimizer, loss函数等
5. 开始训练, 分epoch和batch
6. 前向传播后计算损失函数, 求梯度, 反向传播更新权重
7. 训练结束, 输出正确率, 损失值等等

完整代码如下

```python
import torch
import torch.nn.functional as F

from torch import nn, optim
from torchvision import datasets, transforms

# Define a transform to normalize the data
transform = transforms.Compose([transforms.ToTensor(),
                                transforms.Normalize((0.5,), (0.5,))])

# Download and load the training data
trainset = datasets.FashionMNIST('~/.pytorch/F_MNIST_data/', download=True, train=True, transform=transform)
trainloader = torch.utils.data.DataLoader(trainset, batch_size=64, shuffle=True)

# Download and load the test data
testset = datasets.FashionMNIST('~/.pytorch/F_MNIST_data/', download=True, train=False, transform=transform)
testloader = torch.utils.data.DataLoader(testset, batch_size=64, shuffle=True)

# 定义网络结构
class MyFashionMnist(nn.Module):
  def __init__(self):
    super().__init__()
    self.fc1 = nn.Linear(784, 256)
    self.fc2 = nn.Linear(256, 64)
    self.fc3 = nn.Linear(64, 10)
    
  def forward(self, x):
    x = x.view(-1, 784)
    x = F.relu(self.fc1(x))
    x = F.relu(self.fc2(x))
    x = F.log_softmax(self.fc3(x))
    
    return x
  
model = MyFashionMnist()

optimizer = optim.Adam(model.parameters(), lr=0.003)

criterion = nn.NLLLoss()

epochs = 20

for e in range(epochs):
  running_loss = 0 # 损失
  for images, labels in trainloader:
    output = model(images)
    loss = criterion(output, labels)
    
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
    
    running_loss += loss.item()
   
  else:
    print("loss: ", running_loss / len(trainloader))
    
sum = correct = 0

# 用测试集测试正确率
for images, labels in testloader:
  output = torch.exp(model(images))
  result = torch.argmax(output, dim=1)
  correct += (result == labels).sum()
  sum += len(images)
  
print("correct = ", correct.item())
print("sum = ", sum)
print("rate = {}".format(correct.item() / sum))
```



写了一份基于keras的代码做对比, 过程是类似的. 相对而言keras更黑箱所以代码短一些

```py
import tensorflow as tf
from tensorflow.keras import datasets, models, layers
import os

os.environ['TF_CPP_MIN_LOG_LEVEL'] = '2'

(train_images, train_labels), (test_images, test_labels) = datasets.fashion_mnist.load_data()
train_images = tf.reshape(train_images, [-1, 784])
test_images = tf.reshape(test_images, [-1, 784])

print(train_images.shape, train_labels.shape, test_images.shape, test_labels.shape)

model = models.Sequential()
model.add(layers.Dense(256, activation='relu', input_shape=(784, )))
model.add(layers.Dense(64, activation='relu'))
model.add(layers.Dense(10, activation='softmax'))
print("model build!")

model.compile(optimizer='rmsprop',
              loss='categorical_crossentropy',
              metrics=['accuracy'])

# 将训练集随机化
idx = tf.range(len(train_images))
idx = tf.random.shuffle(idx)
print(idx)
train_images = tf.gather(train_images, indices=idx)
# train_images = train_images[idx]
train_labels = tf.gather(train_labels, indices=idx)
# train_labels = train_labels[idx]

# 将label转化成one hot
train_labels = tf.one_hot(train_labels, depth=10)
test_labels = tf.one_hot(test_labels, depth=10)

# 划分出训练集和验证集
partial_x_train = train_images[3000:]
x_val = train_images[:3000]
partial_y_train = train_labels[3000:]
y_val = train_labels[:3000]

print(partial_x_train.shape, partial_y_train.shape, x_val.shape, y_val.shape)

history = model.fit(partial_x_train,
                    partial_y_train,
                    epochs=20,
                    batch_size=64,
                    validation_data=(x_val, y_val))

result = model.evaluate(test_images, test_labels)
```

差别是:

1.  keras的训练起来要比pytorch快得多. 不知道是不是因为keras在gpu条件满足情况下自动调用了gpu, 而pytorch用的是cpu
2. 准确率用pytorch写的反而要高, 这么一个简单的网络正确率达到了86%-87%, 而反观keras的, 在和pytorch的超参数差不多的情况下, 正确率只有70%左右. 设置其他的超参数才能稍高一些



## Part 5 - Inference and Validation (Exercises).ipynb

1. 这部分主要讲验证, 比如用topk()来衡量正确

    但用topk()时总是出错, 所以后面改用argmax()了

2. 介绍了dropout的使用, 可以明显地减少过拟合. 也就是训练时的损失和验证损失差不多, 但同时训练时正确率更低一些

    dropout也真是玄学

3. 定义了自己的带dropout层的网络

    ```py
    ## TODO: Define your model with dropout added
    
    from torch import nn, optim
    import torch.nn.functional as F
    
    class Classifier(nn.Module):
        def __init__(self):
            super().__init__()
            self.fc1 = nn.Linear(784, 256)
            self.fc2 = nn.Linear(256, 128)
            self.fc3 = nn.Linear(128, 64)
            self.fc4 = nn.Linear(64, 10)
            
            self.dropout = nn.Dropout(p=0.2)
            
        def forward(self, x):
            # make sure input tensor is flattened
            x = x.view(x.shape[0], -1)
            
            x = self.dropout(F.relu(self.fc1(x)))
            x = self.dropout(F.relu(self.fc2(x)))
            x = self.dropout(F.relu(self.fc3(x)))
            x = F.log_softmax(self.fc4(x), dim=1)
            
            return x
    ```

4. 在训练时记录了各个时间点的损失, 所以用下面的代码可以轻松画出损失的变化值, 来判断是否发生过拟合

    ```py
    import matplotlib.pyplot as plt
    
    train_losses = torch.Tensor(train_losses) / len(trainloader)
    test_losses = torch.Tensor(test_losses) / len(testloader)
    
    plt.plot(train_losses.numpy(), label='train_losses')
    plt.plot(test_losses.numpy(), label='test_losses')
    plt.legend()
    ```

    

5. 在验证时需要调用model.eval(), 避免验证进入dropout. 训练时要验证, 验证之后要用model.train()进入训练模式



## Part 6 - Saving and Loading Models.ipynb

这部分将如何保存和恢复模型, 因为这一节notebook运行较麻烦原因没有很认真去看...



## Part 7 - Loading Image Data (Exercises).ipynb

这部分讲如何从文件夹中加载数据集

1. 文件夹格式, 主文件夹下有多个子文件夹, 分别代表图片的类别. 可以在这两层文件夹之间加一层来区分训练集和测试集.

2. 常规步骤:

    1. 定义转化器transform(改变图片大小, 中心裁剪部分图片, 转化成tensor, 翻转图片等)
    2. 使用datasets.ImageFolder()方法加载数据集
    3. 使用torch.utils.data.DataLoader()方法得到生成器dataloader

    代码实现:

    ```py
    data_dir = 'Cat_Dog_data/train'
    
    transform = transforms.Compose([transforms.Resize(255),
                                   transforms.CenterCrop(224),
                                   transforms.ToTensor()])
    # TODO: compose transforms here
    dataset = datasets.ImageFolder(data_dir, transform=transform)
    # TODO: create the ImageFolder
    dataloader = torch.utils.data.DataLoader(dataset, batch_size=32, shuffle=True)
    # TODO: use the ImageFolder dataset to create the DataLoader
    ```

    

3. 最后一部分是用之前所学网络结构实现猫狗分类(老师说很可能不成功, 因为之前只学了full connection net, 且只训练过MNIST这种简单的数据集, 像这种彩色的, 大图片的分类, 那些简单的网络可能效果非常不好, 所以我没尝试)



## Part 8 - Transfer Learning (Exercises).ipynb

1. 这部分讲的是迁移学习, 用别人预训练好的模型就可以做好多很厉害的东西啦. 样例使用的是densenet121, 分为feature和classifier部分, 我们需要改动的是classifier部分, 从开始的ImageNet输出1000类改成2类的分类器(猫狗分类). 直观上, 感觉像是那很复杂很复杂的feature部分是将图片的特征提取出来了, 作为一个1024长度的向量是输入, 然后我们再用之前学过的知识对这个输入进行分类???

2. 自定义classifier, 看着有点怪, 不像之前自己定义的网络

    ```py
    # Freeze parameters so we don't backprop through them
    for param in model.parameters():
        param.requires_grad = False
    
    from collections import OrderedDict
    classifier = nn.Sequential(OrderedDict([
                              ('fc1', nn.Linear(1024, 500)),
                              ('relu', nn.ReLU()),
                              ('fc2', nn.Linear(500, 2)),
                              ('output', nn.LogSoftmax(dim=1))
                              ]))
        
    model.classifier = classifier
    ```

    3. 最后有一个自己使用预训练模型进行猫狗分类的练习, 我自己做出来正确率居然是51%(嗯, 不错, 对了一半...). 视频里讲的跟我差不多的方法是95+%的正确率. 我果然太天真. 
    4. 不过过程就是这样了, 从torchvision.models加载某一预训练模型, 然后查看他的网络结构, 修改最后一部分(分类器), 注意freeze网络参数. 然后开始使用gpu训练(据比较gpu和cpu的训练速度是几百倍的差别, 然而在colab上进行训练即使是一个epoch也要训练几分钟的...), 这部分和普通的网络训练一样, 最后是测试(或者将验证部分嵌入到训练部分, 在训练过程中不断获悉正确率)