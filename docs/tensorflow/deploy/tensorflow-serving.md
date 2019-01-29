

# TensorFlow Serving 模型部署

TensorFlow Serving 是一个灵活的高性能服务系统，适用于机器学习模型，专为生产环境而设计，可与TensorFlow 模型的完美集成。不仅能够用来部署 Tensorflow 模型服务，也可以部署其它机器学习模型服务。


TensorFlow Serving 支持Docker部署，使用 Docker 部署模型的好处在于，避免了与繁琐的环境配置打交道。使用 Docker，不需要手动安装 Python，更不需要安装 numpy、tensorflow 各种包，直接一个 Docker 容器就包含了全部。

## 入门

```bash
# 1. 下载 TensorFlow Serving Docker 镜像
$ docker pull tensorflow/serving

# 2. 下载示例程序到本地，clone 之后会生成 serving 文件夹
$ git clone https://github.com/tensorflow/serving

# 3. Demo 目录设置到 Shell 变量 TESTDATA
# https://github.com/tensorflow/serving/tree/master/tensorflow_serving/servables/tensorflow/testdata
$ TESTDATA="$(pwd)/serving/tensorflow_serving/servables/tensorflow/testdata"

# 4. 开启 TensorFlow Serving 容器，打开 REST API 端口
# -t Allocate a pseudo-TTY  为容器重新分配一个伪输入终端
# --rm 指定容器停止后自动删除容器
# -p 8501:8501 端口映射
# -v 挂载目录 (宿主机目录:Docker 容器内目录 进行映射)
# -e 指定环境变量，容器中可以使用该环境变量，这里指定 环境变量 MODEL_NAME=half_plus_two
$ docker run -t --rm -p 8501:8501 \
   -v "$TESTDATA/saved_model_half_plus_two_cpu:/models/half_plus_two" \
   -e MODEL_NAME=half_plus_two \
   tensorflow/serving &

# 5. 使用预测 (predict) API 查询 暴露的 RESTful 接口
# -d 设置请求数据，这里同时预测三条数据，会同时返回三条结果
# -X 设置请求方式
$ curl -d '{"instances": [1.0, 2.0, 5.0]}' \
   -X POST http://localhost:8501/v1/models/half_plus_two:predict

# Returns => { "predictions": [2.5, 3.0, 4.5] }
```

> ！！！！！！！！！！！！！！！！！！！！！！！！
> 【注意】： 以上默认配置 Serving 启动成功后，新发布的版本，可以自动识别最新版模型，但是老版本是会自动废弃，相当于只有最新版可用，后续介绍如果定制版本策略，使所有版本同时可用。
> ！！！！！！！！！！！！！！！！！！！！！！！！

## 简介

Serving 镜像提供了两种调用方式：`gRPC` 和 `HTTP` 请求。`gRPC` 默认端口是 `8500`，`HTTP` 请求的默认端口是`8501`。

Serving 镜像中的程序会自动加载镜像内 `/models` 下的模型，通过 `MODEL_NAME` 指定 `/models` 下的哪个模型。

Serving 部署的模型是分版本的，发布新增版本无需重新发布 Docker，Serving 会自动识别出最新的版本，容器内 `/models` 的目录结构如下(通过 `docker exec -it 容器名 bash` 进入容器)：

```bash
models # 模型存放目录
├── half_plus_two # 模型名，模型下面可以有多个版本
    ├── 1547389348 # 模型版本，高阶API导出 saved_model 的时候会默认带上版本，版本是当前时间戳
        └── saved_model.pb # 版本目录下是标准的模型文件
        └── variables
            ├── variables.data-00000-of-00001
            └── variables.index
```

## 一个完整的实例

### 生成模型文件

以下代码会 把 x 乘以 2 赋值给 y，运行之后会在目录 `/tmp/hello_model/<当前时间戳秒>` 下生成 模型文件。

```python
import tensorflow as tf
import datetime as dt

# 定义权重变量 W，初始值是 2.0
W = tf.get_variable('W', initializer=tf.constant(2.0), dtype=tf.float32)

# 定义一个占位符 x,
x = tf.placeholder(tf.float32, name='x')

# OP， y = W * x
y = tf.multiply(W, x, name='y')

with tf.Session() as sess:
    # 初始化所有变量
    sess.run(tf.global_variables_initializer())

    # 已 saved model 形式保存
    tf.saved_model.simple_save(
        sess,
        # 模型保存路径
        "/tmp/tf_models/hello_model/" + str(int(dt.datetime.now().timestamp())),
        # 指名输入变量
        inputs={"param_x": x},
        # 指名输出变量
        outputs={"result_y": y}
    )

```

### Serving 部署模型

```bash
$ docker run -t --rm -p 8503:8501 \
        -v "/tmp/tf_models/hello_model:/models/hello_model" \
        -e MODEL_NAME=hello_model \
        --name tf_model_hello \
        tensorflow/serving
```

### 调用 RESTful 接口

**传一个参数**

```bash
$ curl -d '{"instances":[{"param_x":5}]}' -X POST http://localhost:8503/v1/models/hello_model:predict

# 结果
{
    "predictions": [10.0]
}
```

**传多个参数**

```bash
$ curl -d '{"instances":[{"param_x":3},{"param_x":7}]}' -X POST http://localhost:8503/v1/models/hello_model:predict

# 结果
{
    "predictions": [6.0, 14.0]
}
```

## Serving 的其他 HTTP 接口

> 官方文档 [Client: RESTful API](https://www.tensorflow.org/serving/api_rest)

### 查看版本状态

```javascript
$ curl http://localhost:8503/v1/models/hello_model

{
 "model_version_status": [
  {
   "version": "1548749759",
   "state": "AVAILABLE",
   "status": {
    "error_code": "OK",
    "error_message": ""
   }
  },
  {
   "version": "1548745308",
   "state": "END",
   "status": {
    "error_code": "OK",
    "error_message": ""
   }
  }
 ]
}
```



### 查看模型信息

```javascript
$ curl http://localhost:8503/v1/models/hello_model/metadata
{
    "model_spec":{
        "name":"hello_model",
        "signature_name":"",
        "version":"1548745308"
    },
    "metadata":{
        "signature_def":{
            "signature_def":{
                "serving_default":{
                    "inputs":{
                        "param_x":{
                            "dtype":"DT_FLOAT",
                            "tensor_shape":{
                                "dim":[

                                ],
                                "unknown_rank":true
                            },
                            "name":"x:0"
                        }
                    },
                    "outputs":{
                        "result_y":{
                            "dtype":"DT_FLOAT",
                            "tensor_shape":{
                                "dim":[

                                ],
                                "unknown_rank":true
                            },
                            "name":"y:0"
                        }
                    },
                    "method_name":"tensorflow/serving/predict"
                }
            }
        }
    }
}
```

## 多模型管理

TensorFlow Serving 支持 通过 `models.config` 配置文件，同时发布多个模型 和 自定义一些模型配置。


### models.config

`models.config` 配置文件内容如下：

```javascript
model_config_list {
  config {
    name: 'hello_model'
    base_path: '/models/hello_model/'
    model_version_policy: {all: {}}
  }
  config {
    name: 'hello_model2'
    base_path: '/models/hello_mode2/'
    model_version_policy: {all: {}}
  }
}
```

### Docker 参数

```bash
$ docker run -t --rm -p 8504:8501 \
        -v "/tmp/tf_models:/models" \
        --name tf_model_hello2 \
        tensorflow/serving \
        --model_config_file=/models/models.config
```

### 查看多模型

```bash
curl http://localhost:8504/v1/models/hello_model2
{
    "model_version_status":[
        {
            "version":"1548749759",
            "state":"AVAILABLE",
            "status":{
                "error_code":"OK",
                "error_message":""
            }
        },
        {
            "version":"1548745308",
            "state":"AVAILABLE",
            "status":{
                "error_code":"OK",
                "error_message":""
            }
        }
    ]
}
```

### 调用指定版本的模型

```bash
$ curl  -d '{"instances":[{"param_x":5}]}' -X POST \
>   http://localhost:8504/v1/models/hello_model2/versions/1548745308:predict

{
    "predictions":[ 10 ]
}
```



> 官方文档 
>
> - [How to Configure a Model Server](https://www.tensorflow.org/serving/serving_config)
>
> - [Using TensorFlow Serving with Docker](https://www.tensorflow.org/serving/docker)
>
> [Tensorflow Serving of model with 3 versions present on disk but only latest is available](https://stackoverflow.com/questions/50877939/tensorflow-serving-of-model-with-3-versions-present-on-disk-but-only-latest-is-a)



## Read More

- 「官方文档」[TensorFlow Serving for model deployment in production](https://www.tensorflow.org/serving/)
- [使用tensorflow-serving部署tensorflow模型](https://www.cnblogs.com/weiyinfu/p/9928363.html)

