# TensorFlow 模型格式

> 本文来自：[TensorFlow 到底有几种模型格式？](https://cloud.tencent.com/developer/article/1009979)



## CheckPoint(\*.ckpt)

在训练 TensorFlow 模型时，每迭代若干轮需要保存一次**权值**到磁盘，称为 `checkpoint`。

这种格式文件是由 `tf.train.Saver()` 对象调用 `saver.save()`生成的，**只包含若干 Variables 对象序列化后的数据，不包含图结构，所以只给 checkpoint 模型不提供代码是无法重新构建计算图**。

- `checkpoint`：检查点文件，文件保存了一个目录下所有的模型文件列表
- `*.ckpt.data-xxx` ： 保存模型中每个变量的取值
- `*.ckpt.meta`： 文件保存了TensorFlow计算图的结构，可以理解为神经网络的网络结构



## GraphDef(\*.pb)

这种格式文件包含 protobuf 对象序列化后的数据，包含了计算图，可以从中得到所有运算符（operators）的细节，也包含张量（tensors）和 Variables 定义，**但不包含 Variable 的值，因此只能从中恢复计算图**，但一些训练的权值仍需要从 checkpoint 中恢复。

TensorFlow 一些例程中用到 `*.pb` 文件作为预训练模型，这和上面 GraphDef 格式稍有不同，属于冻结（Frozen）后的 GraphDef 文件，简称 **FrozenGraphDef 格式**。这种文件格式不包含 Variables 节点。将 GraphDef 中所有 Variable 节点转换为常量（其值从 checkpoint 获取），就变为 FrozenGraphDef 格式。

`.pb` 为二进制文件，实际上 protobuf 也支持文本格式（`*.pbtxt`），但包含权值时文本格式会占用大量磁盘空间，一般不用。

## SavedModel

在使用 TensorFlow Serving 时，会用到这种格式的模型。**该格式为 GraphDef 和 CheckPoint 的结合体，另外还有标记模型输入和输出参数的 SignatureDef**。从 SavedModel 中可以提取 GraphDef 和 CheckPoint 对象。

```bash
assets/
assets.extra/
variables/
    variables.data-?????-of-?????
    variables.index
saved_model.pb
```

其中 `saved_model.pb`（或 `saved_model.pbtxt`）包含使用 MetaGraphDef protobuf 对象定义的计算图；assets 包含附加文件；variables 目录包含 tf.train.Saver() 对象调用 save() API 生成的文件。

更多细节可以参考 Github TensorFlow 项目： [tensorflow/python/saved_model/README.md](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/python/saved_model)

## 小结

部署在线服务（Serving）时官方推荐使用 `SavedModel` 格式，而部署到手机等移动端的模型一般使用 `FrozenGraphDef` 格式，TensorFlow Lite 有专门的轻量级模型格式 `*.lite`，和 FrozenGraphDef 十分类似。这些格式之间关系密切，可以使用 TensorFlow 提供的 API 来互相转换。