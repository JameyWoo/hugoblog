---
title: "Handson Ml Chp2"
date: 2019-03-30
draft: false

categories: ["机器学习"]
---


批量学习（batch learning），一次性批量输入给学习算法，可以被形象的称为填鸭式学习。
在线学习（online learning），按照顺序，循序的学习，不断的去修正模型，进行优化。

batch learning 如果数据很大的话，可以使用MapReduce技术，或者使用online learning。

performance measure 使用RMSE（root mean square error），也就是均方根误差。看https://www.jianshu.com/p/9ee85fdad150 。学到了root 是根号的意思。

hypothesis ：假设
outlier ：异常值

有很多异常值（很大）的话，可能倾向于用MAE。

> RMSE 和 MAE 都是测量预测值和目标值两个向量距离的方法。有多种测量距离的方法，或范数：
计算对应欧几里得范数的平方和的根（RMSE）：这个距离介绍过。它也称作ℓ2范数，标记为 （或只是 ）。
计算对应于ℓ1（标记为 ）范数的绝对值和（MAE）。有时，也称其为曼哈顿范数，因为它测量了城市中的两点，沿着矩形的边行走的距离。
更一般的，包含n个元素的向量v的ℓk范数（K 阶闵氏范数），定义成

> ℓ0（汉明范数）只显示了这个向量的基数（即，非零元素的个数），ℓ∞（切比雪夫范数）是向量中最大的绝对值。

> 范数的指数越高，就越关注大的值而忽略小的值。这就是为什么 RMSE 比 MAE 对异常值更敏感。但是当异常值是指数分布的（类似正态曲线），RMSE 就会表现很好。

entry ：条目，entries
histogram ：统计直方图
histogram ：后端

数据可能经过预处理，这不一定会出错，但你得知道数据是怎么来的。

feature scaling ：特征缩放

如果右边的属性很长，会处理这些属性，使其变为正态分布。 

P49：生成随机排列以获得比较随机的train和test集划分。random seed控制结果。
P50：使用hash方法来生成稳定的train、test集，当得到新的数据时，原来的划分保持不变，新的实例同样进行划分。
调用的库是hashlib，可以稍微研究一下python中的hash方法。
增加一列索引index。
可以使用scikit learn提供的函数

P51：讲到要使用分层采样，因为不同层次的样本数量是不一样的，如果全都随机采样的话，误差会比较大。
strata：层
stratify：分层
使用sklearn提供的分层类，配合pandas的where等，分层正确可以获得较大的性能提升。
还可以查看分层比例。

pandas的where方法
> Series.where(cond, other=nan, inplace=False, axis=None, level=None, errors=‘raise’, try_cast=False, raise_on_error=None)
如果 cond 为真，保持原来的值，否则替换为other， inplace为真标识在原数据上操作，为False标识在原数据的copy上操作。

insight：见解，洞察力
density：密度

分析数据，数据的可视化：可以得到数据之间的联系，哪些数据更有用，从而更加有技巧地进行训练；而且，从数据的可视化中，就可以得出一些只是看数据得不到的结论，图像能使信息暴露出来，所以matplotlib在机器学习中会有那么重要的作用，而看到的哪些讲机器学习的书、比赛的notebook，他们都有大量的数据可视化，这是呈现数据，探索数据的过程，也是：为什么此模型能够比其他人的模型更加强大的原因。但是数据分析有套路，要多看熟悉这些套路。

alpha参数的作用：它是一个设置透明度的参数。看到书上用它时，觉得没啥用啊，不是每个点都会设置一样的透明度吗？但是转念一想，存在透明度的话，点多的地方，颜色就会深，而点少的地方颜色就会浅。这就是透明度的作用啊！

correlation：相关；关联
median：中位数
coefficient：率；系数
correlation coefficient：相关系数

可以使用corr方法获取属性两两之间的（linear）相关性。（可以说是很强了！没想到居然还提供了这种方法。不过相关系数是怎么计算的？）
得到了相关性，然后呢？有什么作用吗？
书上说：想要移除掉相关性大的属性，避免重复。所谓数据清洗。

属性的结合，可能会获取更加有效的数据。

数据清洗：missing features
三种策略
sklearn提供了一种Imputer类来实现确实数据的填充

要设计自己的读取、划分数据函数，一种通用的方法，能够对不同的数据进行划分。

想到一个问题：如何将自己写的类，在python中import？就像numpy那样？但是numpy不是一个类而是一个包，那么如何制作自己的包？

interfeace：接口

P61讲述了sklearn的设计风格
评估器、转换器等的设计模式，统一性。

将文本属性转化成数字。标签编码。

sklearn.preprocessing 

独热编码OneHotEncoder

sparse matrix：稀疏矩阵。书上一般说SciPy sparse matrix

text attribute -> num -> one hot; text -> one hot; 可选scipy

设计自定义的transformers，三种method，fit，transform，fit_transform。
使用两个基类，提供一些功能。

feature scaling：特征缩放
normalization、standardization(less attected by outliers)
只能向数据集拟合：这样更准确。

pipelines class
数值属性和文本属性的特征可以分开设置pipeline，然后可以使用FeatureUnion来合并，真是方便！不过这个pipeline的设计有点麻烦。

选择模型、拟合、计算error。
underfitting and overfitting
cross validation：交叉验证。比较可靠的评估方式。
utility function：效用函数。越大越好 
cost function：成本函数。越小越好

overfitting的可怕，还以为放出决策树回归出来会吊打线性回归呢，结果被吊打了，哈哈。

ensemble learning：集成学习。如random forests

把模型和训练结果保存下来，以便以后的对比。
可以使用pickle，或者sklearn.externals.joblib。
可以说是很方便了。

fine tune：微调

超参数：hyperparameter
> 在机器学习的上下文中，超参数是在开始学习过程之前设置值的参数，而不是通过训练得到的参数数据。通常情况下，需要对超参数进行优化，给学习机选择一组最优超参数，以提高学习的性能和效果。
在机器学习的上下文中，超参数是在开始学习过程之前设置值的参数。 相反，其他参数的值通过训练得出。
超参数：
定义关于模型的更高层次的概念，如复杂性或学习能力。
不能直接从标准模型培训过程中的数据中学习，需要预先定义。
可以通过设置不同的值，训练不同的模型和选择更好的测试值来决定

超参数的一些示例：
* 树的数量或树的深度
* 矩阵分解中潜在因素的数量
* 学习率（多种模式）
* 深层神经网络隐藏层数
* k均值聚类中的簇数

amazing Grid Search！直译是网格搜索，但是显然不能这么理解。
这是sklearn提供的一种超级方便的选择hyperparameter的工具，简直是开挂啊。fine tune果然不赖。
还有一些分析的技巧。不过话说想要选好一组适合的超参数要训练好多组啊。
甚至直接获取the best estimator

Randomized Search 和 Gird相比，有几个有点。但目的和作用是一样的。

Ensemble Method 集成方法

分析最佳模型和他们的误差，可以获得更深的对问题的理解。比如可以给出每个属性对于做出准确预测的相对重要性，然后去掉某些属性，是否会使得分类更加准确。

maintain：维护