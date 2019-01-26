# 核心概念

## 简介

TensorFlow （TF）是一种 **机器学习框架**，而 **机器学习** 经常和 **人工智能**、**深度学习** 联系在一起，那么三者到底是什么关系呢？简单来讲三者可以理解为包含于被包含的关系。

其中最大的是**人工智能（以下简称为 AI）**，AI最早起源于1956年的达特茅斯会议，当时 AI 的几位先驱在会上展示了最早的 AI 程序：Logic Theorist，能够自动推导数学原理第二章前52个定理中的38个，甚至其中一个定理的证明过程比书中给出的还要优雅，他们甚至曾尝试要发表这一新的证明方式（不过后来被拒了）。总之，简单来说，AI就是赋予机器以人的智能，让机器具有学习和认知的能力。

**机器学习（以下简称为ML）**则是实现 AI 的一种方法。举个简单的垃圾邮件过滤的例子，我们人类判断一个邮件是否是垃圾邮件很简单，通过标题或内容很快就可以辨别。但是让机器完成这样的任务就没这么简单。对于经典的ML 算法，首先需要从原始数据（邮件）中提取 **特征**，比如发信人地址、邮件标题、邮件内容关键词等，从而将文字的邮件转换成包含多个特征的向量，然后利用逻辑回归算法在已经标定的数据集上进行训练，得到每个特征的权重。这些权重构成我们的预测模型，对于一封新的邮件，就可以用这个模型判断其是否是垃圾邮件。

传统的 ML 最大的问题就是特征提取。比如让机器识别照片中的动物是猫还是狗，如何设计特征？**深度学习（以下简称为DL）**正是为了解决特征提取的问题，我们不再需要人工设计特征，而是让算法从数据中自动学习特征，将简单的特征组合形成复杂的特征来解决这些问题，所以 DL 可以说是实现 ML 的一种技术。



## TensorFlow是什么

Tensorflow是一个Google开发的第二代机器学习系统，克服了第一代系统 DistBelief 仅能开发神经网络算法、难以配置、依赖Google内部硬件等局限性，应用更加广泛，并且提高了灵活性和可移植性，速度和扩展性也有了大幅提高。字面上理解，TensorFlow 就是以 **张量（Tensor）**在**计算图（Graph）**上**流动（Flow）**的方式的实现和执行机器学习算法的框架。具有以下特点：

- **灵活性。**TensorFlow 不是一个严格的「神经网络」库。只要可以将计算表示成数据流图，就可以使用TensorFlow，比如科学计算中的偏微分求解等。（实际上其官网的介绍中对 TF 的定位就是基于数据流图的科学计算库，而非仅仅是机器学习库）
- **可移植性。**同一份代码几乎不经过修改既可以部署到有任意数量CPU、GPU 或 TPU（Tensor Processing Unit，Google专门为机器学习开发的处理器）的PC、服务器或移动设备上。
- **自动求微分。**同 Theano 一样，TensorFlow 也支持自动求微分，用户不需要再通过反向传播求解梯度。
- **多语言支持。**TensorFlow 官方支持 Python、C++、Go 和 Java 接口，用户可以在硬件配置较好的机器中用Python 进行实验，在资源较紧张或需要低延迟的环境中用C++进行部署。
- **性能。**虽然TensorFlow最开始发布时仅支持单机，在性能评测上并不出色，但是凭借 Google 强大的开发实力，TensorFlow 性能已经追上了其他框架。

## Google为什么开源Tensorflow

Google第一代分布式机器学习框架DistBelief在内部大规模使用后没有选择开源，而第二代 TensorFlow 于2015年11月在 GitHub 上开源，并在持续快速开发迭代中。TensorFlow 最早由 Google Brain 的工程师开发，设计初衷是加速机器学习的研究，并快速地将研究原型转化为产品。Google 选择开源 TensorFlow 的原因很简单：第一是希望借助社区的力量，大家一起完善 TensorFlow。第二是回馈社区，Google 希望让这个优秀的工具得到更多的应用，提高学术界和工业界使用机器学习的效率。

## TensorFlow大事记

- 2015年11月9日，Google Research 发布了文章：TensorFlow - Google’s latest machine learning system, open sourced for everyone，正式宣布其新一代机器学习系统开源。
- 2016年4月13日，TensorFlow v0.8发布，提供分布式计算支持。
- 2016年4月29日，开发 AlphaGo 的 DeepMind 宣布从 Torch7 平台转向TensorFlow。
- 2016年4月12日，基于TensorFlow的世界最准确的语法解析器 SyntaxNet 宣布开源。
- 2016年6月27日，TensorFlow v0.9 发布，提高对移动设备的支持。
- 2016年8月30日，TF-Slim——TensorFlow的高层库发布，用户可以更简单快速地定义模型。
- 2017年2月15日，TensorFlow v1.0发布，提高了速度和灵活性，并且承诺提供稳定的Python API。

## 符号式编程 vs 命令式编程

和我们一般常用的 **命令式（Imperative）编程模式** 不同，TF采用的是 **符号式（Symbolic）编程**。我们先认识一下两种编程模式。

命令式编程是很常见的编程模式，大多数 Python 或 C++ 程序都采用命令式编程。命令式编程明确输入变量，根据程序逻辑逐步运算。下面是一段命令式编程的Python代码:

```python
import numpy as np

a = np.ones(10)
b = np.ones(10) * 2

c = b * a
d = c + 1
```

执行完第一步 `a = np.ones(10)` 后，程序得到了输入变量`a`，第二句后得到了 `b`，当执行`c = b * a`时，程序通过乘法计算而得到了`c`。

**符号式编程则将计算过程抽象为计算图**，所有输入节点、运算节点和输出节点均符号化处理。将上述命令式编程代码转换为符号式编程：

```python
A = Variable('A')
B = Variable('B')

C = B * A
D = C + Constant(1)

# compiles the function
f = compile(D)
d = f(A=np.ones(10), B=np.ones(10) * 2)
```

当前四步执行后，程序并没有A、B、C、D的值，仅仅是**构建了由这四个符号构成的计算图**。

> 注：只是搭建了一个计算的架子：A * B + 1，最后一步通过给 A 和 B 喂数据，来获取计算结果



大多数符号式编程的程序中都或显式或隐式地包含编译的步骤，将前面定义的计算图打包成可以调用的函数，而实际的计算则发生在编译后。

Theano 和 TensorFlow 属于典型的符号式编程模式，而 Torch 则是命令式编程模式。**在灵活性上，命令式编程模式更优，而在内存和计算上，符号式编程效率更高**。

## TensorFlow基本概念

要使用TensorFlow，我们必须理解TensorFlow

- 使用图（`Graph`）表示计算流程
- 在会话（`Session`）中执行图
- 使用张量（`Tensor`）表示数据
- 使用变量（`Variable`）维护状态
- 使用 `feed` 和 `fetch` 为任意的操作赋值或从中获取数据

TF 使用 graph表示计算流程。图中的节点称为操作（Operation，以下简称OP）。每个 OP 接受 0 到多个Tensor，执行计算，输出 0 到多个Tensor。图是对计算流程的描述，需要在Session中运行。Session将计算图的OP分配到CPU或GPU等计算单元，并提供相关的计算方法，并且会返回OP的结果。

### 张量（Tensor）

TF 使用 Tensor 表示所有数据，相当于 Numpy 中的 ndarray，0维的数值、一维的矢量、二维的矩阵到n维数组都是Tensor。相对于Numpy，TensorFlow提供了创建张量函数的方法，以及导数的自动计算。

### 变量（Variable）

在训练模型时，Variable 被用来存储和更新参数。Variable 包含张量储存在内存的缓冲区中，必须显式地进行初始化，在训练后可以写入磁盘。下面代码中的 Variable 充当了一个简单的计数器角色：

```python
import tensorflow as tf

# Create a Variable, that will be initialized to the scalar value 0.
# 创建一个变量，名字是 counter，初始值是 0
state = tf.Variable(0, name="counter")

# Create an Op to add one to `state`.
# 一下三步操作实现了一个 counter = counter + 1 的逻辑

# 创建一个 常量 1，
one = tf.constant(1)
# 对变量 counter + 1
new_value = tf.add(state, one)
# counter 加 1后 分配给自己
update = tf.assign(state, new_value)

# 以上操作是在画一个计算图，并没有真的开始计算，sess.run 的时候才真正开始计算

# Variables must be initialized by running an `init` Op after having
# launched the graph.  We first have to add the `init` Op to the graph.
init_op = tf.initialize_all_variables()

# Launch the graph and run the ops.
with tf.Session() as sess:
    # Run the 'init' op
    sess.run(init_op)
    # Print the initial value of 'state'
    # 获取初始值 打印
    print(sess.run(state))
    # Run the op that updates 'state' and print 'state'.
    for _ in range(3):
        # 计算
        sess.run(update)
        # 打印计算后的结果
        print(sess.run(state))

# output:
# 0
# 1
# 2
# 3

```

上述代码中 `assign()` 操作同 `add()` 一样，都是在构建计算图而没有执行实际的计算。直到 `run()` 函数才会真正执行赋值等计算操作。

一般来说，用户使用一系列 `Variable` 来表示一个统计模型，在训练过程中运行图计算来不断更新，训练完成后可以使用这些 `Variable` 构成的模型进行预测。

### Feed

TensorFlow 除了可以使用 Variable 和 Constant 引入数据外，还提供了 Feed 机制实现从外部导入数据。一般Feed 总是与占位符 placeholder 一起使用。

```python
import tensorflow as tf

input1 = tf.placeholder(tf.float32)
input2 = tf.placeholder(tf.float32)

# input1 * input2
output = tf.multiply(input1, input2)

with tf.Session() as sess:
    print(sess.run([output], feed_dict={input1: [7.], input2: [2.]}))

# output:
# [array([ 14.], dtype=float32)]

```

### Fetch

要获取操作的输出，需要执行会话的 run() 函数，并且提供需要提取的 OP。下面是获取输出的典型例子：

```python
import tensorflow as tf

input1 = tf.constant([3.0])
input2 = tf.constant([2.0])
input3 = tf.constant([5.0])

# 2 + 5 = 7
intermed = tf.add(input2, input3)
# 3 * 7 = 21
mul = tf.multiply(input1, intermed)

with tf.Session() as sess:
    result = sess.run([mul, intermed])
    print(result)

# output:
# [array([21.], dtype=float32), array([7.], dtype=float32)]

```

### 图和会话

由于 TF 采用符号式编程模式，所以 TF 程序通常可以分为两部分：**图的构建** 和 **图的执行**。



#### 图的构建

构建图的第一步，是创建源 OP(source op)，源操作不需要任何输入，例如常量(constant)，源操作的输出被传递给其它操作做运算。

Python库中，OP构造器的返回值代表被构造出的OP的输出，这些返回值可以传递给其它OP构造器作为输入。

TensorFlow Python库有一个默认图 (default graph)，OP构造器可以为其增加节点。这个默认图对许多程序来说已经足够用了。

```python
import tensorflow as tf

# Create a Constant op that produces a 1x2 matrix.  The op is added as a node to the default graph.
#
# The value returned by the constructor represents the output of the Constant op.
matrix1 = tf.constant([[3., 3.]])
# Create another Constant that produces a 2x1 matrix.
matrix2 = tf.constant([[2.], [2.]])

# Create a Matmul op that takes 'matrix1' and 'matrix2' as inputs.
# The returned value, 'product', represents the result of the matrix multiplication.
# 矩阵乘法
product = tf.matmul(matrix1, matrix2)

```

上面使用 TensorFlow 提供的默认图构建了包含三个节点的计算图：两个 `constant()` 操作和一个 `matmul()` 操作。要实际执行**矩阵乘法**，必须在 Session 中运行该计算图。



#### 图的执行

要执行计算图，首先需要创建Session对象，如果不提供参数，Session构造器将运行默认图。

```python
# Launch the default graph.
sess = tf.Session()

# To run the matmul op we call the session 'run()' method, passing 'product'
# which represents the output of the matmul op.  This indicates to the call
# that we want to get the output of the matmul op back.
#
# All inputs needed by the op are run automatically by the session.  They
# typically are run in parallel.
#
# The call 'run(product)' thus causes the execution of three ops in the
# graph: the two constants and matmul.
#
# The output of the op is returned in 'result' as a numpy ndarray object.
result = sess.run(product)
print(result)
# ==> [[ 12.]]

# Close the Session when we're done.
sess.close()

```



Session结束后，需要关闭以释放资源。用户也可以使用with控制语句自动关闭会话。

```python
with tf.Session() as sess:
    result = sess.run([product])
    print(result)
```



## Read More

- [一文读懂TensorFlow（附代码、学习资料）](https://mp.weixin.qq.com/s/SlitM8JToD7dN5E5Ue9wjA)
- [TensorFlow学习笔记1：入门](http://www.jeyzhang.com/tensorflow-learning-notes.html)