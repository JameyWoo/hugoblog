

### 贝叶斯方法完整代码

```py
import seaborn as sns  
iris = sns.load_dataset('iris')

X_iris = iris.drop('species', axis=1)
y_iris = iris['species']

print(X_iris)
from sklearn.cross_validation import train_test_split
Xtrain, Xtest, ytrain, ytest = train_test_split(X_iris, y_iris, random_state=1)

from sklearn.naive_bayes import GaussianNB
model = GaussianNB()
model.fit(Xtrain, ytrain)
y_model = model.predict(Xtest)

from sklearn.metrics import accuracy_score
accuracy_score(ytest, y_model)
```
---

## 步骤分析

### 一-首先获取数据.

这里我们在线导入seaborn库的iris(鸢尾花)数据

```python
import seaborn as sns
iris = sns.load_dataset('iris')
```
[这是github上的说明](https://github.com/mwaskom/seaborn-data), 可直接下载csv文件

### 二-将数据格式化

```py
X_iris = iris.drop('species', axis=1)
y_iris = iris['species']
```
将类别那一列删除生成新的对象赋值给X_iris, y_iris为分类.
[pandas的drop方法参考](https://blog.csdn.net/sinat_32547403/article/details/73822660)

### 三-将数据切分为训练数据和测试数据

```py
from sklearn.cross_validation import train_test_split
Xtrain, Xtest, ytrain, ytest = train_test_split(X_iris, y_iris, random_state=1)
```

`cross validation`是**交叉验证**的意思, [参考文章](https://blog.csdn.net/CherDW/article/details/54881167)
参数`random_state`是固定的随机种子, [参考文章](http://sofasofa.io/forum_main_post.php?postid=1001204)

### 四-调用高斯朴素贝叶斯实现训练

```cpp
from sklearn.naive_bayes import GaussianNB
model = GaussianNB()
model.fit(Xtrain, ytrain)
y_model = model.predict(Xtest) # 进行预测
```
调用`Gaussian naive Bayes`模型, 并进行拟合, 预测

### 五-对测试结果进行评估

[评估方法介绍](https://blog.csdn.net/CherDW/article/details/55813071)

```py
from sklearn.metrics import accuracy_score
accuracy_score(ytest, y_model)
```
得到准确率为`0.9736842105263158`
`metrics`是指标的意思.

说明对特征明显的数据, 即使是非常简单的分类算法也可以高效地进行分析.