# SavedModel CLI

您可以使用 SavedModel 命令行界面 (CLI) 检查并执行 SavedModel。例如，您可以使用 CLI 检查模型的 `SignatureDef`。CLI 让您能够快速确认输入的 [Tensor dtype 和形状](https://www.tensorflow.org/guide/tensors) 是否与模型匹配。

如果您已安装了 TensorFlow，则您的系统上已经安装 SavedModel CLI。

## 查看帮助 -h

```bash
(venv-tf) xxx $ saved_model_cli -h
usage: saved_model_cli [-h] [-v] {show,run,scan} ...

saved_model_cli: Command-line interface for SavedModel

optional arguments:
  -h, --help       show this help message and exit
  -v, --version    show program's version number and exit

commands:
  valid commands

  {show,run,scan}  additional help

```

可以看到 saved_model_cli 支持三个子命令 `show`,`run`,`scan`

## show 子命令

```bash
(venv-tf) xxx $ saved_model_cli show -h
usage: saved_model_cli show [-h] --dir DIR [--all] [--tag_set TAG_SET]
                            [--signature_def SIGNATURE_DEF_KEY]

Usage examples:
To show all tag-sets in a SavedModel:
$saved_model_cli show --dir /tmp/saved_model

To show all available SignatureDef keys in a MetaGraphDef specified by its tag-set:
$saved_model_cli show --dir /tmp/saved_model --tag_set serve

For a MetaGraphDef with multiple tags in the tag-set, all tags must be passed in, separated by ';':
$saved_model_cli show --dir /tmp/saved_model --tag_set serve,gpu

To show all inputs and outputs TensorInfo for a specific SignatureDef specified by the SignatureDef key in a MetaGraph.
$saved_model_cli show --dir /tmp/saved_model --tag_set serve --signature_def serving_default

To show all available information in the SavedModel:
$saved_model_cli show --dir /tmp/saved_model --all

optional arguments:
  -h, --help            show this help message and exit
  --dir DIR             directory containing the SavedModel to inspect
  --all                 if set, will output all information in given SavedModel
  --tag_set TAG_SET     tag-set of graph in SavedModel to show, separated by ','
  --signature_def SIGNATURE_DEF_KEY
                        key of SignatureDef to display input(s) and output(s) for
```

## run 子命令

```bash
(venv-tf) xxx $ saved_model_cli run -h
usage: saved_model_cli run [-h] --dir DIR --tag_set TAG_SET --signature_def
                           SIGNATURE_DEF_KEY [--inputs INPUTS]
                           [--input_exprs INPUT_EXPRS]
                           [--input_examples INPUT_EXAMPLES] [--outdir OUTDIR]
                           [--overwrite] [--tf_debug] [--worker WORKER]
                           [--init_tpu]

Usage example:
To run input tensors from files through a MetaGraphDef and save the output tensors to files:
$saved_model_cli show --dir /tmp/saved_model --tag_set serve \
   --signature_def serving_default \
   --inputs input1_key=/tmp/124.npz[x],input2_key=/tmp/123.npy \
   --input_exprs 'input3_key=np.ones(2)' \
   --input_examples 'input4_key=[{"id":[26],"weights":[0.5, 0.5]}]' \
   --outdir=/out

For more information about input file format, please see:
https://www.tensorflow.org/guide/saved_model_cli

optional arguments:
  -h, --help            show this help message and exit
  --dir DIR             directory containing the SavedModel to execute
  --tag_set TAG_SET     tag-set of graph in SavedModel to load, separated by ','
  --signature_def SIGNATURE_DEF_KEY
                        key of SignatureDef to run
  --inputs INPUTS       Loading inputs from files, in the format of '<input_key>=<filename>, or '<input_key>=<filename>[<variable_name>]', separated by ';'. The file format can only be from .npy, .npz or pickle.
  --input_exprs INPUT_EXPRS
                        Specifying inputs by python expressions, in the format of "<input_key>='<python expression>'", separated by ';'. numpy module is available as 'np'. Will override duplicate input keys from --inputs option.
  --input_examples INPUT_EXAMPLES
                        Specifying tf.Example inputs as list of dictionaries. For example: <input_key>=[{feature0:value_list,feature1:value_list}]. Use ";" to separate input keys. Will override duplicate input keys from --inputs and --input_exprs option.
  --outdir OUTDIR       if specified, output tensor(s) will be saved to given directory
  --overwrite           if set, output file will be overwritten if it already exists.
  --tf_debug            if set, will use TensorFlow Debugger (tfdbg) to watch the intermediate Tensors and runtime GraphDefs while running the SavedModel.
  --worker WORKER       if specified, a Session will be run on the worker. Valid worker specification is a bns or gRPC path.
  --init_tpu            if specified, tpu.initialize_system will be called on the Session. This option should be only used if the worker is a TPU job.

```





## scan 子命令

```python
$ saved_model_cli scan -h
usage: saved_model_cli scan [-h] --dir DIR [--tag_set TAG_SET]

Usage example:
To scan for blacklisted ops in SavedModel:
$saved_model_cli scan --dir /tmp/saved_model
To scan a specific MetaGraph, pass in --tag_set

optional arguments:
  -h, --help         show this help message and exit
  --dir DIR          directory containing the SavedModel to execute
  --tag_set TAG_SET  tag-set of graph in SavedModel to scan, separated by ','

```



## Read More

- [How to serve a tensorflow-module, specifically Universal Sentence Encoder?](https://stackoverflow.com/questions/52715499/how-to-serve-a-tensorflow-module-specifically-universal-sentence-encoder)
- [使用 CLI 检查并执行 SavedModel](https://www.tensorflow.org/guide/saved_model#cli_to_inspect_and_execute_savedmodel)

