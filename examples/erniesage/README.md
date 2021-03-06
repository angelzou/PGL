# 使用PGL实现ERNIESage

[ENG Readme](./README.en.md)

## 背景介绍 

在很多工业应用中，往往出现如下图所示的一种特殊的图：Text Graph。顾名思义，图的节点属性由文本构成，而边的构建提供了结构信息。如搜索场景下的Text Graph，节点可由搜索词、网页标题、网页正文来表达，用户反馈和超链信息则可构成边关系。

<img src="./docs/source/_static/text_graph.png" alt="Text Graph" width="800">

**ERNIESage** 由PGL团队提出，是ERNIE SAmple aggreGatE的简称，该模型可以同时建模文本语义与图结构信息，有效提升 Text Graph 的应用效果。其中 [**ERNIE**](https://github.com/PaddlePaddle/ERNIE) 是百度推出的基于知识增强的持续学习语义理解框架。

**ERNIESage** 是 ERNIE 与 GraphSAGE 碰撞的结果，是 ERNIE SAmple aggreGatE 的简称，它的结构如下图所示，主要思想是通过 ERNIE 作为聚合函数（Aggregators），建模自身节点和邻居节点的语义与结构关系。ERNIESage 对于文本的建模是构建在邻居聚合的阶段，中心节点文本会与所有邻居节点文本进行拼接；然后通过预训练的 ERNIE 模型进行消息汇聚，捕捉中心节点以及邻居节点之间的相互关系；最后使用 ERNIESage 搭配独特的邻居互相看不见的 Attention Mask 和独立的 Position Embedding 体系，就可以轻松构建 TextGraph 中句子之间以及词之间的关系。

<img src="./docs/source/_static/ernie_aggregator.png" alt="ERNIESage" width="800">

使用ID特征的GraphSAGE只能够建模图的结构信息，而单独的ERNIE只能处理文本信息。通过PGL搭建的图与文本的桥梁，**ERNIESage**能够很简单的把GraphSAGE以及ERNIE的优点结合一起。以下面TextGraph的场景，**ERNIESage**的效果能够比单独的ERNIE以及GraphSAGE模型都要好。

<img src="./docs/source/_static/ERNIESage_result.png" alt="ERNIESage_result" width="800">

**ERNIESage**可以很轻松地在PGL中的消息传递范式中进行实现，目前PGL提供了4个版本的ERNIESage模型：

- **ERNIESage v1**: ERNIE 作用于text graph节点上;
- **ERNIESage v2**: ERNIE 作用在text graph的边上;
- **ERNIESage v3**: ERNIE 作用于一阶邻居及起边上;
- **ERNIESage v4**: ERNIE 作用于N阶邻居及边上;

<img src="./docs/source/_static/ERNIESage_v1_4.png" alt="ERNIESage_v1_4" width="800">

## 环境依赖
- paddlepaddle>=1.7
- pgl>=1.1

## Dataformat
示例数据```data.txt```中使用了NLPCC2016-DBQA的部分数据，格式为每行"query \t answer"。
```text
NLPCC2016-DBQA 是由国际自然语言处理和中文计算会议 NLPCC 于 2016 年举办的评测任务，其目标是从候选中找到合适的文档作为问题的答案。[链接: http://tcci.ccf.org.cn/conference/2016/dldoc/evagline2.pdf]
```

## How to run

我们采用了[PaddlePaddle Fleet](https://github.com/PaddlePaddle/Fleet)作为我们的分布式训练框架，在```config/*.yaml```中，有部分用于训练ERNIESage的配置, 其中ERNIE模型```ckpt_path```以及词表```ernie_vocab_file```在[ERNIE](https://github.com/PaddlePaddle/ERNIE)下载。


```sh
# 分布式GPU模式或单机模式ERNIESage
sh local_run.sh config/erniesage_v2_gpu.yaml

# 分布式CPU模式训练ERNIESage
sh local_run.sh config/erniesage_v2_cpu.yaml
```

**NOTE**：为了方便用户们学习使用ERNIESage，我们在百度AIStudio中提供了可以直接运行的ERNIESage实例，详情可见：https://aistudio.baidu.com/aistudio/projectdetail/667443.

## Hyperparamters

- learner_type: `gpu` or `cpu`; gpu 使用fleet Collective 模式, cpu 使用fleet Transpiler 模式.

## Citation
```
@misc{ERNIESage,
  author = {PGL Team},
  title = {ERNIESage: ERNIE SAmple aggreGatE},
  year = {2020},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/PaddlePaddle/PGL/tree/master/examples/erniesage},
}
```
