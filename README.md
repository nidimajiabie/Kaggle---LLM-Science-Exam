# Kaggle---LLM-Science-Exam
Use LLMs to answer difficult science questions

1. 赛题背景

竞赛受OpenBookQA数据集的启发， 要求参与者使用大型语言模型（LLM）回答一些科学上的困难问题，从ABCDE五个选项中选出正确答案。通过这项工作，研究者们希望更好地理解LLM测试自身的能力，以及在资源受限环境中运行LLM的潜力。随着大型语言模型（LLM）能力范围的扩大，越来越多的研究领域正在使用LLM来表征。由于许多现有的 NLP 基准已被证明对于最先进的模型来说是微不足道的，因此也有一些有趣的工作表明 LLM 可用于创建更具挑战性的任务来测试更强大的模型。与此同时，量化和知识蒸馏等方法被用来有效地缩小语言模型并在更普通的硬件上运行它们。 Kaggle 环境提供了一个独特的视角来研究这一问题，提交内容受到 GPU 和时间限制。此挑战的数据集是通过提供从维基百科提取的一系列科学主题的 gpt3.5 文本片段，并要求其编写多项选择题（带有已知答案），然后过滤掉简单的问题来生成的。目前， 我们估计 Kaggle 上运行的最大模型约有 100 亿个参数，而 gpt3.5的参数为 1750 亿个。如果一个问答模型能够在由比其规模大 10 倍的问题编写模型编写的测试中表现出色，这将是一个真正有趣的结果；另一方面，如果一个较大的模型能够有效地击败较小的模型，这对LLM自我基准表现和测试的能力具有引人注目的影响。

2. 数据简介

竞赛官方只提供了200个问题样本，具体见https://www.kaggle.com/competitions/kaggle-llm-science-exam/data

3. 外部数据

STEM 文本语料库1： https://www.kaggle.com/datasets/mbanaei/all-paraphs-parsed-expanded

STEM 文本语料库2： https://www.kaggle.com/datasets/mbanaei/stem-wiki-cohere-no-emb

Wikipedia完整语料库： https://www.kaggle.com/datasets/jjinho/wikipedia-20230701

https://www.kaggle.com/datasets/cdeotte/60k-data-with-context-v2

https://www.kaggle.com/datasets/cdeotte/40k-data-with-context-v2

https://www.kaggle.com/datasets/cdeotte/99k-data-with-context-v2


4. 算法流程——基于RAG检索增强生成架构的自然语言问答模型

4.1 数据提取：分别对样本提示语prompt和外部大型知识语料库进行清洗和整理， prompt重复三次与问题选项拼接构成检索文本（query），将语料库进行整理和区分后形成三种：完整的所有文本（full text）、 STEM相关文本片段1、 STEM相关文本片段2。

4.2 向量化Embedding：分别对检索文本与三种文本语料库使用embedding模型gte-base生成embedding（特征向量）。

4.3 创建索引（Index）：使用Faiss对三种文本语料库的embedding创建Index，以便后续查询。

4.4 检索（Retrieval）：使用向量内积（Inner Product, IP）相似度进行语义相似搜索，使用query文本embedding逐一在三种文本语料库Index中查询，获取top10文本作为最终检索结果。

4.5 生成（Generation）：将基于完整文本（full text）语料检索结果作为训练集文本外部知识库（context），由context（知识库） + prompt（问题） + option（选项）构成完整样本。

4.6 模型微调（Finetune）：将所得完整样本输入deberta-v3-large模型进行五分类训练，基于下述形式进行RAG和风格行为词汇适配：五分类形式：外部知识文本{context}，请回答题目{question prompt }，选项{ option [1-5]}中最正确的是ABCDE中的哪一个？

4.7 推理答案（Inference）：将在三种语料知识库得到检索文本分别输入微调后的deberta-v3-large模型预测每个选项概率，将预测结果按照加权平均得到最终的前三答案选项。


5. 总结

5.1 RAG（检索增强） 通过为LLM提供回答查询时使用的事实背景，使LLM变得更加实用。

5.2 在实际项目中，提示词是非常脆弱和敏感的，当前的大模型对提示具有非常高的依赖性，这种依赖性与模型的能力成反比，也就是模型的能力越弱，对提示的依赖越强。选择不同的模型、不同的数据，甚至不同的索引，都需要调整提示来得到一个比较优秀的结果。

5.3 基于向量化的相似性是 RAG 的标准检索机制，提高Embedding model将使得LLM性能更高。
