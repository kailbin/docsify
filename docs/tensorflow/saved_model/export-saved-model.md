# 导出 Saved Model 格式



## 低阶 API 导出

```python
import tensorflow as tf

# 定义权重变量 W，初始值是 2.0
W = tf.get_variable('W', initializer=tf.constant(2.0), dtype=tf.float32)

# 定义一个占位符 x,
x = tf.placeholder(tf.float32, name='x')

# OP， y = W * x
y = tf.multiply(W, x, name='y')

with tf.Session() as sess:
    # 初始化所有变量
    sess.run(tf.global_variables_initializer())
    # 给 x 赋值 并运行
    print(sess.run(y, {x: 10.0}))
    # 输出 20.0

    # 已 saved model 形式保存
    tf.saved_model.simple_save(
        sess,
        # 模型保存路径
        "./hello_model",
        # 指名输入变量
        inputs={"param_x": x},
        # 指名输出变量
        outputs={"result_y": y}
    )
```

## 模型文件如下

```bash
$ tree hello_model/
hello_model/
├── saved_model.pb
└── variables
    ├── variables.data-00000-of-00001
    └── variables.index

1 directory, 3 files
```

## saved_model_cli 查看模型文件

```bash
$ saved_model_cli show --dir ./hello_model/ --all

MetaGraphDef with tag-set: 'serve' contains the following SignatureDefs:

signature_def['serving_default']:
  The given SavedModel SignatureDef contains the following input(s):
    inputs['param_x'] tensor_info:
        dtype: DT_FLOAT
        shape: unknown_rank
        name: x:0
  The given SavedModel SignatureDef contains the following output(s):
    outputs['result_y'] tensor_info:
        dtype: DT_FLOAT
        shape: unknown_rank
        name: y:0
  Method name is: tensorflow/serving/predict
```

## saved_model_cli 调用模型

```bash
(venv-tf) xxx $ saved_model_cli run \
	--dir hello_model/ \
	--tag_set serve \
	--signature_def serving_default \
	--input_exprs "param_x=[7.]"

# 输出
# Result for output key result_y:
# [14.]
```

> [How to serve a tensorflow-module, specifically Universal Sentence Encoder?](https://stackoverflow.com/questions/52715499/how-to-serve-a-tensorflow-module-specifically-universal-sentence-encoder)



## 使用 Python API 调用 模型文件

```python
import tensorflow as tf

export_dir = "./hello_model"

graph = tf.Graph()

with tf.Session(graph=graph) as sess:
    # 加载模型
    tf.saved_model.loader.load(sess, [tf.saved_model.tag_constants.SERVING], export_dir)

    # 从图中获取 参数和结果信息
    x = graph.get_tensor_by_name("x:0")
    y = graph.get_tensor_by_name("y:0")

    # 调用模型进行运算，参数 x 设置为 7.0， 结果写入变量 y
    result = sess.run(y, feed_dict={x: 7.})
    # 打印模型计算的结果结果 14.0
    print(result)
```



## 使用 Java API 调用 模型文件

### 添加 Maven 依赖

```xml
<dependency>
    <groupId>org.tensorflow</groupId>
    <artifactId>tensorflow</artifactId>
    <version>1.12.0</version>
</dependency>
```

### Java API 示例

```java
public static void main(String[] args) {
    // 打印 TensorFlow 版本 (1.12.0)
    System.out.println(TensorFlow.version());

    // saved model 文件路径
    String path = "./hello_model";

    // 加载 模型文件
    try (SavedModelBundle b = SavedModelBundle.load(path, "serve")) {

        // 创建 Session
        try (Session sess = b.session()) {
            // 定义参数，值是 7.0F
            Tensor x = Tensor.create(7.0F);

            // 运行模型，获取结果 14.0f.
            Tensor<?> tensor = sess.runner()
                    // 投喂 模型参数 x
                    .feed("x", x)
                    // 获取模型计算的结果
                    .fetch("y")
                    .run()
                    .get(0);

            // 打印结果
            System.out.println(tensor.floatValue());
        }
    }
}
```



## 高阶 API 导出

**TODO** 参考 鸢尾花示例，模型导出

**TODO**

**TODO**

**TODO**

## Read More

- [Tensorflow: how to save/restore a model?](https://stackoverflow.com/questions/33759623/tensorflow-how-to-save-restore-a-model)
- 官网 > 保存和恢复 > [构建和加载 SavedModel](https://www.tensorflow.org/guide/saved_model#build_and_load_a_savedmodel)