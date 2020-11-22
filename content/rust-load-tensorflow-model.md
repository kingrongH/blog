+++
title = "Rust 加载tensorflow模型用于预测"
date = 2020-05-01 15:36:25
[taxonomies]
tags = ["rust", "tensorflow"]
categories = ["Rust"]

+++

最近的时间做了一个项目：基于Rust的车牌图像处理，因为爱好Rust，而项目又不得不做，于是乎用Rust重新造了一个轮子，但是也不是没有什么用处，起码这个轮子变得很快并且推广了Rust在机器学习方面的应用，私心里还颇有些得意的。但是这类项目用到了深度学习，而Rust并没有纯深度学习框架的实现，所以使用tensorflow binding是较为理想的方案。

本文将介绍如何使用Rust加载Tensorflow 的模型

<!--more-->

多数时候我们并不直接使用Rust + Tensorflow进行训练，如果要这么做的话，岂不是白白浪费了前人给我们筑的基石，所以我们的目的限于使用Rust + Tensorflow 加载模型然后进行预测。之所以这么做是因为python用作预测的话可能并不能直接用于生产使用，因为速度太慢了，所以使用Rust + Tensorflow 加载训练好的模型就会是一个很好的选择。


## Tensorflow模型

参照[tensorflow-rust](https://github.com/tensorflow/rust)的例子[addition](https://github.com/tensorflow/rust/blob/master/examples/addition.rs)加载已经导出的tensorflow模型是相当简单的一件事情：

```Rust
// Load the computation graph defined by addition.py.
let mut graph = Graph::new();
let mut proto = Vec::new();
File::open(filename)?.read_to_end(&mut proto)?;
graph.import_graph_def(&proto, &ImportGraphDefOptions::new())?;
```
可以看到从上面的代码中，我们已经导入Tensorflow的模型为一个图（Graph）。操作方法就是将模型读到缓存里面，然后利用Tensorflow的方法`import_graph_def`直接导入该模型。这个图里面有很多的节点，也被称做是操作（Operation）,包含我们对输入输出的种种操作，Tensorflow 支持的种种操作我们可以从[这里](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/core/ops/ops.pbtxt)找到，这些操作都是通过名字（name）和图（graph）关联起来的。目前我们是从别的语言训练好导出的模型中导入的这张图，也就是说这张图里本身就已经含有了种种Operation。如果我们需要输入内容进行预测，首先就需要知道输入和输出操作的名称，然后通过名称获取对应的操作，再进行输入，获取输出。

我们可以通过如下的方式获取未知模型的输入操作和输出操作的名称：

```Rust
fn main() -> Result<(), Box<dyn Error>> {
    let mut file = File::open("./detect.pb")?;
    let mut buffer = Vec::new();
    file.read_to_end(&mut buffer)?;
    let mut graph = Graph::new();
    graph.import_graph_def(&buffer, &ImportGraphDefOptions::new())?;
    let operation_iter = graph.operation_iter();
    let names: Vec<String> = operation_iter.map(|op| {
        op.name().unwrap()
    }).collect();
    println!("names: {:?}", names);
    Ok(())
}
```
这是打印了所有的`Opearation`的名称。

知道了名称之后，我们就可以导入数据进行预测了。

```Rust
use tensorflow::{ SessionRunArgs, SessionOptions, Session, Tensor, Graph, ImportGraphDefOptions };
use image::GenericImageView;

use std::fs::File;
use std::io::prelude::*;
use std::error::Error;

const INPUT_NAME: &'static str = "image_tensor";
const OUT_DETECT_BOX: &'static str = "detection_boxes";
const OUT_DETECT_SCORES: &'static str = "detection_scores";

fn main() -> Result<(), Box<dyn Error>> {
    let mut file = File::open("./detect.pb")?;
    let mut buffer = Vec::new();
    file.read_to_end(&mut buffer)?;
    let mut graph = Graph::new();
    graph.import_graph_def(&buffer, &ImportGraphDefOptions::new())?;

    let img = image::open("test.jpg")?;
    let (width, height) = img.dimensions();
    let img_data = &img.to_rgb().to_vec();
    let img_tensor = Tensor::new(&[1, height as u64, width as u64, 3]);
    let img_tensor = img_tensor.with_values(&img_data)?;

    let session = Session::new(&SessionOptions::new(), &graph)?;
    let mut args = SessionRunArgs::new();
    args.add_feed(&graph.operation_by_name_required(INPUT_NAME)?, 0, &img_tensor);
    let box_token = args.request_fetch(&graph.operation_by_name_required(OUT_DETECT_BOX)?, 0);
    let scores_token = args.request_fetch(&graph.operation_by_name_required(OUT_DETECT_SCORES)?, 0);
    session.run(&mut args)?;
    let boxes: Tensor<f32> = args.fetch(box_token)?;
    let scores: Tensor<f32> = args.fetch(scores_token)?;

    println!("boxes: {}\nscores: {}", boxes, scores);

    Ok(())
}
```

上面的例子中，可以看到我们配置了三个操作名称："image_tensor"、 "detection_boxes"、 "detection_scores"，这三个是典型的[Object Detection](https://github.com/tensorflow/models/tree/master/research/object_detection)的`Opearation`名称，第一个是输入操作的名称，后两个是输出操作的名称。而且我们输入的是一个图片数据。得到输出之后，可以进一步对输出数据做处理


## Keras 模型

使用Tensorflow 加载tensorflow的模型自然而然是没有什么问题的，但是现阶段用Keras和Tensorflow作为后端进行训练会是多数人的选择，这样训练得到的模型，我们又怎么导入然后进行预测呢？Rust + Kearas？不不不，Keras只是python实现的，我们不能用Rust调用python。但是我们可以将Keras模型加载出来然后再导出为Tensorflow模型，这样就是皆大欢喜了。

使用Tensorflow作为后端的Keras用的是全局`Session`，我们只需要拿到对应的图保存对应的model并不难。

```python
def freeze_session(session, keep_var_names=None, output_name=None, clear_devices=True):
    graph = session.graph
    with graph.as_default():
        freeze_var_names = list(set(v.op.name for v in tf.global_variables()).difference(keep_var_names or []))
        output_names = output_name or []
        output_names += [v.op.name for v in  tf.global_variables()]
        input_graph_def = graph.as_graph_def()
        if clear_devices:
            for node in input_graph_def.node:
                node.device = ""
            frozen_graph = convert_variables_to_constants(session, input_graph_def, output_names, freeze_var_names)
            return frozen_graph
```

你可以在 [convert_keras_to_tensorflow](https://github.com/kingrongH/convert_keras_to_tensorflow) 找到一个详细的例子

我们加载转换好的tensorflow模型，就可以搞一些事情了。
