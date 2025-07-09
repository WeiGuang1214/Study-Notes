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

##### 评估指标Evaluation Metrics:

1、Context Precision:上下文准确率，衡量上下文retrieved_contexts中相关词块比例的指标。识别检索到的上下文是否相关。
基于LLLM:
	Context Precision without reference：用于判断retrieved_contexts和response Context Precision with reference：retrieved_contexts和reference

```python
from ragas import SingleTurnSample
from ragas.metrics import LLMContextPrecisionWithReference
context_precision = LLMContextPrecisionWithReference(llm=evaluator_llm)
sample = SingleTurnSample(
	user_input="Where is the Eiffel Tower located?",
	reference="The Eiffel Tower is located in Paris.",
	retrieved_contexts=["The Eiffel Tower is located in Paris."]
)
await context_precision.single_turn_ascore(sample)
```

##### 传统向量距离：参考上下文的上下文精度,

```python
sample = SingleTurnSample(
	retrieved_contexts=["The Eiffel Tower is located in Paris."],
	reference_contexts=["Paris is the capital of France.", "The Eiffel,
	Tower is one of the most famous landmarks in Paris."])
await context_precision.single_turn_ascore(sample)
```

2、Context Recall：上下文召回率衡量成功检索到的相关文档(或信息片段)数量，所以一定需要上下文retrieved_contexts和参考reference。

```python
from ragas.dataset_schema import SingleTurnSample
from ragas.metrics import LLMContextRecall
sample = SingleTurnSample(
	user_input="Where is the Eiffel Tower located?",
	response="The Eiffel Tower is located in Paris.",
	reference="The Eiffel Tower is located in Paris.",
	retrieved_contexts=["Paris is the capital of France."],
	context_recall = LLMContextRecall(llm=evaluator_llm)
await context_recall.single_turn_ascore(sample)
```

3、ContextEntityRecall，上下文实体召回率,，基于reference和retrieved_contexts中同时存在的实体数量，对于reference中单独存在的实体数量的比率。

```python
reference="The Eiffel Tower is located in Paris.",
retrieved_contexts=["The Eiffel Tower is located in Paris."],
```

4、ResponseRelevancy，响应相关性，回答和用户提问的余弦相似度距离。

```python
user_input="When was the first super bowl?",
response="The first superbowl was held on Jan 15, 1967",
retrieved_contexts=[
	"The First AFL-NFL World Championship Game was an American football game played on January 15, 1967, at the Los Angeles Memorial Coliseum in Los Angeles."]
```

5、Faithfulness，忠诚度衡量的是response与的retrieved context事实一致性。其范围从0到1，分数越高,，一致性越好。

```python
sample = SingleTurnSample(
	user_input="When was the first super bowl?",
	response="The first superbowl was held on Jan 15, 1967",
	retrieved_contexts=[
		"The First AFL-NFL World Championship Game was an American football game played on January 15, 1967, at the Los Angeles Memoria Coliseum in Los Angeles."
	]
)
```

6、答案准确率ACC
	衡量的是模型对给定问题的回答与参考标准之间的一致性。该指标通过两个不同的“LLM评委”提示来实，每个提示会返回一个评分(0、2或4)。该指标会将这些评分转换为[0,1]的范围内，然后取评委给出的两个评分的平均值。分数越高，表示模型的答案与参考标准越接近。
	0→答复不准确或未解决与参考文献相同的问题。
	2→响应部分与参考一致。
	4→响应与参考完全一致。

```python
user_input="When was Einstein born?",
user_input="When was Einstein born?",
response="Albert Einstein was born in 1879.",
reference="Albert Einstein was born in 1879."
```

7、语境相关性
上下文(块或段落)是否与用户输入相关，以0、1或2的等级对相关性进行评分。然后将评分转换为[0,1]的等级，并取平均值得出最终分数。分数越高，表示上下文与用户查询的匹配程度越高。

```python
sample = SingleTurnSample(
user_input="When and Where Albert Einstein was born?",
retrieved_contexts=[
	"Albert Einstein was born March 14, 1879.",
	"Albert Einstein was born at Ulm, in Württemberg, Germany.",
	]
)
scorer = ResponseGroundedness(llm=evaluator_llm) # 以上demo缺少scorer
```

8、回应立足点
回应立足点衡量的是检索到的语境对回应的支持程度或“立足点”。它评估回应中的每个主张是否可以在提供的语境中找到全部或部分依据。

```python
sample = SingleTurnSample(
user_input="When and Where Albert Einstein was born?",
retrieved_contexts=[
	"Albert Einstein was born March 14, 1879.",
	"Albert Einstein was born at Ulm, in Württemberg, Germany.",
	]
)
scorer = ResponseGroundedness(llm=evaluator_llm)
```

9、事实正确性
事实正确性FactualCorrectnessresponse用于比较和评估生成的与reference的事实准确性。此指标用于 确 定 生 成 的 响 应 与 参 考 的 一 致 程 度 。 事 实 正 确 性 得 分 范 围 从 0 到 1， 值 越 高 表 示 性 能 越 好

```python
sample = SingleTurnSample(
response="The Eiffel Tower is located in Paris.",
reference="The Eiffel Tower is located in Paris. I has a height of
1000ft.“
)
```

10、BLEU 分数
衡量响应与参考之间的相似性,BLEU分数的范围是0到1，其中1表示响应与参考完全匹配。这是一个非基于LLM的指标。

```python
sample = SingleTurnSample(
response="The Eiffel Tower is located in India.",
reference="The Eiffel Tower is located in Paris."
)
scorer = BleuScore()
await scorer.single_turn_ascore(sample)
```

11、ROUGE 分数
基于n-gram召回率、准确率和F1分数来衡量生成结果response与reference文本之间的重叠度。
ROUGE分数的范围是0到1,其中1表示响应与参考文献完全匹配

12、ExactMatch精确匹配
响应与参考文本完全匹配,则该指标返回1,否则返回0。

```python
sample = SingleTurnSample(
response="India",
reference="Paris"
)
scorer = ExactMatch()
await scorer.single_turn_ascore(sample)
```

13 、 AspectCritic 二 进 制 评 分 0,1, 表 明 提 交 的 内 容 是 否 与 定 义 的 方 面 相 符 , 提 前 预 定 义 一 个 问 题 ,
来判断是否符合这个定义的问题：

```python
from ragas.dataset_schema import SingleTurnSample
from ragas.metrics import AspectCritic
sample = SingleTurnSample(
	user_input="Where is the Eiffel Tower located?",
	response="The Eiffel Tower is located in Paris.",
)
scorer = AspectCritic(	
	name="maliciousness",
	definition="Is the submission intended to harm, deceive, or exploit users?",
	ltm=evatuator_llm
)
await scorer.single_turn_ascore(sample)
```

14、自定义评分标准:

```python
from ragas.dataset_schema import SingleTurnSample
from radas.metrics import RubricsScore
sample = SingleTurnSample(
	response="The Earth is flat and does not orbit the Sun.",
	reference="Scientific consensus, supported by centuries of evidence, confirms that the Earth is a spherical planet that orbits the Sun. This has been demonstrated through astronomical observations, satellite imagery, and gravity measurements.",)
rubrics = {
"score1_description": "The response is entirely incorrect and
fails to address any aspect of the reference.",
"score2_description": "The response contains partial accuracy but
includes major errors or significant omissions that affect its
relevance to the reference.",
"score3_description": "The response is mostly accurate but lacks
clarity, thoroughness, or minor details needed to fully address the
reference.",
"score4_description": "The response is accurate and clear, with
only minor omissions or slight inaccuracies in addressing the
reference.",
"score5_description": "The response is completely accurate,
cloar and tharauahlw addraccoc tha rofaronco withont any arroce
```

