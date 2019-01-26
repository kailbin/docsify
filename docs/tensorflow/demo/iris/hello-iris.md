# 鸢尾花分类

> 本文来自：[Tensorflow-iris-案例教程-零基础-机器学习](https://www.jianshu.com/p/b86c020747f9)



## 预备知识

> 来自：[iris-经典案例解析-机器学习](https://www.jianshu.com/p/da18f0cd7f60)

已知鸢尾花 iris 分为三个不同的类型：**山鸢尾花 Setosa**、**变色鸢尾花 Versicolor**、**韦尔吉尼娅鸢尾花 Virginica**，这个分类主要是依据鸢尾花的 **花萼(Sepal)长度、宽度** 和 **花瓣(Petal)的长度、宽度** 四个指标。我们并不知道具体的分类标准，但是植物学家已经为 150朵 不同的鸢尾花进行了分类鉴定，我们也可以对每一朵鸢尾花进行准确测量得到花萼花瓣的数据。

那么问题来了，你女朋友家的一株鸢尾花开花了，她测量了一下，花萼长宽花瓣长宽分别是 3.1、2.3、1.2、0.5，然后她就问你：“我家这朵鸢尾花到底属于哪个品种？”

作为一个传统程序员，一定会很崩溃，因为我们不清楚具体的分类标准，也不能用 `if...else...`条件判断解决问题：

```java
...
if(SepalLength > 4.1 && SepalLength < 2.2 && ....) {
  return “Setosa”
}else(...){
  return “Versicolor”
}
...
```



幸运的是，研究**机器学习的科学家已经为我们提供了一些成熟的算法**，这些算法可以自动从我们 150条 数据中寻找规律，并自动为我们生成所需要的分类方法函数。

机器学习是如何分析数据并找到规律的？

某智商不高的机器学习科学家发明了最糟糕的分类方法，叫做 **random 乱猜分类器**：

```javascript
function guess(rowData){
	// 特征
    string species;
    // 随机数除以3的余数，0或1或2
    int n = random() % 3; 
    if(n == 0){
        species = "Virginica";
    }else if(n == 1){
        species = "Versicolor";
    }else if(n == 2){
        species = "Setosa";
    }
    return species
}
```

这是一个毫无技术含量的分类方法，但它形式上是正确的。**对于任何一朵新的鸢尾花，这个函数有 33.3% 的可能猜对。**

**TensorFlow 使用更加复杂的神经网络分类器**，可以处理比上面这种情况复杂几百几千倍的难题，比如图像识别、语音识别等。接下来，我们以鸢尾花分类这个示例作为入门。



## 了解数据集

> 训练数据集 http://download.tensorflow.org/data/iris_training.csv
>
> 测试数据集 http://download.tensorflow.org/data/iris_test.csv

以上两个数据集，打开后的效果大致如下：

```
120,4,setosa,versicolor,virginica
6.4,2.8,5.6,2.2,2
5.0,2.3,3.3,1.0,1
4.9,2.5,4.5,1.7,2
4.9,3.1,1.5,0.1,0
5.7,3.8,1.7,0.3,0
...
```

用 Excel 打开是一个表格：

| 120  | 4    | setosa | versicolor | virginica |
| ---- | ---- | ------ | ---------- | --------- |
| 6.4  | 2.8  | 5.6    | 2.2        | 2         |
| 5    | 2.3  | 3.3    | 1          | 1         |
| 4.9  | 2.5  | 4.5    | 1.7        | 2         |
| 4.9  | 3.1  | 1.5    | 0.1        | 0         |
| 5.7  | 3.8  | 1.7    | 0.3        | 0         |
| ...  | ...  | ...    | ...        | ...       |

每个文件的第一行包含两个数字

- 第一个数字是说这个文件中有多少行数据，每一行代表一朵花。training 有 120朵，test 有 30朵
- 第二个数字都是4，表示每朵花有几个 **特征futures** 数据。从前面的鸢尾花案例分析文章中我们知道，测量了花萼的长度、花萼的宽度、花瓣的长度、花瓣的宽度这4个数据，所以这里都是4

第一行还包含了三个英文单词，就是鸢尾花的三个类型

- 山鸢尾花 Setosa
- 变色鸢尾花 Versicolor
- 韦尔吉尼娅鸢尾花 Virginica。

下面是很多很多的数字行。每一行有5个数字，逗号分割。前面4个对应花萼花瓣4个特征 futrues。

每行数字最后一个都是0、1或2，这里的意思是0代表Setosa，1代表Versicolor，2代表Virginica。012这个顺序和第一行最后三个单词是对应的。这就是植物学家鉴定之后给每朵花写上的 **标签 label** 

接下来，我们会先把**训练**数据**“喂”**给计算机，让它吃掉并消化好，这样它就**能自己从数据中找到植物学家区分三种鸢尾花类型的规则**。

然后我们再用训练出来的规则尝试去**评估**我们的测试数据，如果鉴定出来的结果和植物学家的看法一致，那么我们就可以说这个规则很有效。

最后我们会用你女朋友测量出来的鸢尾花数据，使用训练出来的模型，进行评估，来确定她养的是哪个品种的鸢尾花。

## 加载数据

```python
import os
# pip3 install pandas
import pandas as pd
# pip3 install tensorflow
import tensorflow as tf

# 定义 csv 文件每一列的标题，用于表示数据的特征
FUTURES = ['SepalLength', 'SepalWidth', 'PetalLength', 'PetalWidth', 'Species']
# 鸢尾花种类
SPECIES = ['Setosa', 'Versicolor', 'Virginica']

# 当前文件 py 脚本文件的绝对路径 /iris/hello_iris.py
print(os.path.realpath(__file__))
# 当前文件 py 脚本文件所在的目录 /iris
print(os.path.dirname(os.path.realpath(__file__)))

# 获取当前 py 脚本的文件夹
dir_path = os.path.dirname(os.path.realpath(__file__))
# 拼接训练数据集路径 /iris/data/iris_training.csv
train_path = os.path.join(dir_path, 'data/iris_training.csv')
# 拼接评估数据集路径 /iris/data/iris_test.csv
test_path = os.path.join(dir_path, 'data/iris_test.csv')

# 读取训练集数据，列名是 FUTURES，去除首行
train_data = pd.read_csv(train_path, names=FUTURES, header=0)
# 数据拆分成：train_x=前4列 ，train_y=第5列
train_x, train_y = train_data, train_data.pop('Species')

# 读取测试集数据，列名是 FUTURES，去除首行
test_data = pd.read_csv(test_path, names=FUTURES, header=0)
# 数据拆分成：train_x=前4列 ，train_y=第5列
test_x, test_y = test_data, test_data.pop('Species')

"""
print(test_x.head())
   SepalLength  SepalWidth  PetalLength  PetalWidth
0          5.9         3.0          4.2         1.5
1          6.9         3.1          5.4         2.1
2          5.1         3.3          1.7         0.5
3          6.0         3.4          4.5         1.6
4          5.5         2.5          4.0         1.3
...

print(test_y.head())
0    1
1    2
2    0
3    1
4    1
...
"""


# 特征列
feature_columns = []
# 循环每一个训练集，处理成 tf 的特征列格式
for key in train_x.keys():
    feature_columns.append(tf.feature_column.numeric_column(key=key))

# 等效写法
# feature_columns = [tf.feature_column.numeric_column(key=key) for key in train_x.keys()]

"""
print(feature_columns)
[
	_NumericColumn(key='SepalLength', shape=(1,), default_value=None, dtype=tf.float32, normalizer_fn=None), 
	_NumericColumn(key='SepalWidth', shape=(1,), default_value=None, dtype=tf.float32, normalizer_fn=None), 
	_NumericColumn(key='PetalLength', shape=(1,), default_value=None, dtype=tf.float32, normalizer_fn=None), 
	_NumericColumn(key='PetalWidth', shape=(1,), default_value=None, dtype=tf.float32, normalizer_fn=None)
]
"""
```



## 训练

直接使用 TensorFlow 提供的估算器（estimator）来从花朵数据推测分类规则。`tf.estimator`里面包含了很多估算器，这里我们使用**深度神经网络分类器** `DNNClassifier`(**Deep Neural Network Classifier**)。


```python

# 设置日志级别
tf.logging.set_verbosity(tf.logging.INFO)

# 深度神经网络分类器
classifier = tf.estimator.DNNClassifier(
    # 设置特征列
    feature_columns=feature_columns,
    # 设定深层神经网络分类器的复杂程度
    hidden_units=[10, 10],
    # 对应三种花朵类型
    n_classes=3
)

def train_input_fn(features, labels, batch_size):
    features = dict(features)
    inputs = (features, labels)
    dataset = tf.data.Dataset.from_tensor_slices(inputs)
    dataset = dataset.shuffle(1000).repeat().batch(batch_size)
    return dataset.make_one_shot_iterator().get_next()

# 训练
classifier.train(
    input_fn=lambda: train_input_fn(features=train_x, labels=train_y, batch_size=100),
    steps=1000
)
```

训练过程的日志输出：

```bash
INFO:tensorflow:Running local_init_op.
INFO:tensorflow:Done running local_init_op.
INFO:tensorflow:Saving checkpoints for 0 into /var/folders/xj/gbfrnmws2vxbkrgklqk5pv0w0000gn/T/tmpu7zmvecf/model.ckpt.
INFO:tensorflow:loss = 163.1145, step = 1
INFO:tensorflow:global_step/sec: 480.968
INFO:tensorflow:loss = 17.749083, step = 101 (0.208 sec)
INFO:tensorflow:global_step/sec: 757.048
INFO:tensorflow:loss = 11.6443815, step = 201 (0.132 sec)
INFO:tensorflow:global_step/sec: 687.749
INFO:tensorflow:loss = 9.859166, step = 301 (0.146 sec)
INFO:tensorflow:global_step/sec: 729.293
INFO:tensorflow:loss = 6.759341, step = 401 (0.137 sec)
INFO:tensorflow:global_step/sec: 702.647
INFO:tensorflow:loss = 5.0290885, step = 501 (0.142 sec)
INFO:tensorflow:global_step/sec: 755.469
INFO:tensorflow:loss = 7.097002, step = 601 (0.132 sec)
INFO:tensorflow:global_step/sec: 678.998
INFO:tensorflow:loss = 6.965787, step = 701 (0.149 sec)
INFO:tensorflow:global_step/sec: 737.039
INFO:tensorflow:loss = 9.172378, step = 801 (0.134 sec)
INFO:tensorflow:global_step/sec: 687.493
INFO:tensorflow:loss = 4.080602, step = 901 (0.146 sec)
INFO:tensorflow:Saving checkpoints for 1000 into /var/folders/xj/gbfrnmws2vxbkrgklqk5pv0w0000gn/T/tmpu7zmvecf/model.ckpt.
INFO:tensorflow:Loss for final step: 2.8866835.
```

## 评估

```python
def eval_input_fn(features, labels, batch_size):
    features = dict(features)
    inputs = (features, labels)
    dataset = tf.data.Dataset.from_tensor_slices(inputs)
    dataset = dataset.batch(batch_size)
    return dataset.make_one_shot_iterator().get_next()


# 评估
eval_result = classifier.evaluate(
    input_fn=lambda: eval_input_fn(test_x, test_y, 100)
)

# 打印评估结果
for key, value in eval_result.items():
    print(key, ":", value)

"""
accuracy : 0.93333334
average_loss : 0.05918344
loss : 1.7755033
global_step : 1000
"""
```

评估结果的理解

- [准确率 (accuracy)](https://developers.google.com/machine-learning/glossary/#accuracy)：[**分类模型**](https://developers.google.com/machine-learning/glossary/#classification_model)的正确预测所占的比例。在[**多类别分类**](https://developers.google.com/machine-learning/glossary/#multi-class)中，准确率的定义是：`正确预测数/样本总数`
- 平均损失(average loss) = 损失 / 样本个数；这里测试样本个数是30个，：average_loss = loss / 30
- [损失 (Loss)](https://developers.google.com/machine-learning/glossary/#loss)：一种衡量指标，用于衡量模型的[**预测**](https://developers.google.com/machine-learning/glossary/#prediction)偏离其[**标签**](https://developers.google.com/machine-learning/glossary/#label)的程度。或者更悲观地说是**衡量模型有多差**。要确定此值，模型必须定义损失函数。例如，线性回归模型通常将[**均方误差**](https://developers.google.com/machine-learning/glossary/#MSE)用作损失函数，而逻辑回归模型则使用[**对数损失函数**](https://developers.google.com/machine-learning/glossary/#Log_Loss)。



## 预测

```python

# 预测
for i in range(0, 100):
    print('\n请输入特征，用逗号分割: SepalLength,SepalWidth,PetalLength,PetalWidth')
    a, b, c, d = map(float, input().split(','))
    predict_x = {
        'SepalLength': [a],
        'SepalWidth': [b],
        'PetalLength': [c],
        'PetalWidth': [d],
    }
    predictions = classifier.predict(
        input_fn=lambda: eval_input_fn(
            predict_x,
            labels=[0],
            batch_size=100
        )
    )

    for pred_dict in predictions:
        print(pred_dict)
        class_id = pred_dict['class_ids'][0]
        probability = pred_dict['probabilities'][class_id]
        print(SPECIES[class_id], "的概率是：", 100 * probability)

"""
请输入特征，用逗号分割: SepalLength,SepalWidth,PetalLength,PetalWidth
6.3,2.9,5.6,1.8
...
{'logits': array([-7.881872 ,  0.2766737,  6.4064646], dtype=float32), 'probabilities': array([6.2188411e-07, 2.1723057e-03, 9.9782699e-01], dtype=float32), 'class_ids': array([2]), 'classes': array([b'2'], dtype=object)}

Virginica 的概率是： 99.1868257522583
"""
```

## 小结

以上代码片段全都在 hello_iris.py 文件里面，全部拷贝到一个文件即可执行。

后续会介绍如何把一个模型 导出、部署。

## Read More

- 官方文档
  - [预创建的 Estimator](https://www.tensorflow.org/guide/premade_estimators)
  - [创建自定义 Estimator](https://www.tensorflow.org/guide/custom_estimators)