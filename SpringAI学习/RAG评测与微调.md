## 利用RAGAS评估RAG

使用ragas框架对rag生成内容进行评估打分,相关指标和所需数据结构。

核心概念:
1、提示对象Prompt Object：
	指令：描述LLM应执行的任务，通过
	少量样本示例：使用 examples 提示对象中的变量指定，每个示例都包含一个输入及其对应的输出，LLM会使用它们来学习任务。
	输入模型：每个提示都需要输入来产生输出。在Ragas中，此输入的预期格式使用 input_model变量定义。
	输出模型：执行后，提示会生成输出。此输出的格式由 output_model 提示对象中的变量指定。
示例代码:

图1

1. 清晰简洁的指示
2. 相关的少量示例：(理想情况下为3-5个)的相关少量示例
3. 简单的输入和输出模型：定义简单直观的输入和输出模型或者子任务

​	评估样本是一个单一的结构化数据实例，用于评估和衡量LLM应用程序在特定场景下的性能。它代表了AI应用程序预期处理的单个交互单元或特定用例，分为单轮和多伦用例：SingleTurnSample，MultiTurnSample.

图2

图3	

MultiTurnSample 表示人类、Al以及可选工具之间的多轮交互，以及预期的评估结果。它适用于表示对话代理在更复杂的交互中进行评估。在中 MultiTurnSample息，这些消息共同构成人类用户与AI系统之间的多轮对话。这些消息是类
sage和ToolMessage的实例。

图4

图5

图6

3、评估数据集Evaluation Dataset:
	评估数据集是一组同质的数据样本，在Ragas中，评估数据集使用 EvaluationDataset 类来表示，该类提供了一种结构化的方式来组织和管理用于评估目的的数据样本。
	Samples ：SingleTurnSample或MultiTurnSample实例的集合。每个样本代表一个独特的交互或场景

从 SingleTurnSamples 创建评估数据集：

图7

图8

4、评估指标Evaluation Metrics:
1、答案准确率ACC
衡量的是模型对给定问题的回答与参考标准之间的一致性。该指标通过两个不同的“LLM评委”提示来实现,每个提示会返回一个评分(0、2或4)。该指标会将这些评分转换为[0,1]的范围内，然后取评委给出的两个评分的平均值。分数越高,表示模型的答案与参考标准越接近。
	●0→答复不准确或未解决与参考文献相同的问题。
	· 2→响应部分与参考一致。
	· 4→响应与参考完全一致。

图9

5、语境相关性Context Relevance:
上下文相关性评估检索到的上下文(块或段落)是否与用户输入相关。此操作通过两次独立的“LLM-as-a-judge”提示调用完成,每次调用都会以0、1或2的等级对相关性进行评分。然后将评分转换为[0,1]的 等 级 ，并 取 平 均 值 得 出 最 终 分 数 。 分 数 越 高 , 表 示 上 下 文 与 用 户 查 询 的 匹 配 程 度 越 高

```python
from ragas.dataset_schema import SingleTurnSample
from ragas.metrics import ContextRelevance
sample = SingleTurnSample(
	user_input="When and Where Albert Einstein was born?",
    retrieved_contexts=[
				"Albert Einstein was born March 14, 1879.",
				"Albert Einstein was born at Ulm, in Württemberg, Germany.",
scorer = ContextRelevance(llm=evaluator_llm)
score = await scorer.single_turn_ascore(sample)
print (score)
```

ragas 中所有基于LLM的指标均继承自MetricWithLLM类。这些指标需要在评分前设置一个LLM对象。

```python
from ragas.metrics import FactualCorrectness
scorer = FactualCorrectness(llm=evaluation_llm)
```

excel-EvaluationDataset数据集

“user_input":["查询1号商品”],

"response":[“已生成手卡”],
"retrieved_contexts": [

​	[“历史对话:用户历史:查询商品1\n用户问题:{question}\n回答:“]

"ground_truths":["标准答案A"],

"reference":["参考答案X"]

真实讲解内容:

