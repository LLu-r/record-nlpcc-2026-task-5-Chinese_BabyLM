# record-nlpcc-2026-task-5-Chinese_BabyLM

## 目标

与国际 BabyLM 一致：在受人类认知启发的严格数据预算下，探索如何高效训练中文语言模型，缩小 AI 与人类儿童在语言习得过程中的数据效率差距。

## 赛道和评测指标

### 赛道

1.**NLU 赛道（Natural Language Understanding）**：在有限的训练数据下，评估模型在常识推理、文本分类等标准自然语言理解任务上的表现。

2.**认知建模赛道（Cognitive Modeling）**：采用受认知科学启发的评估指标，考察模型习得中文的过程及行为特征与人类儿童的相似度。

3.**汉字赛道（HANZI Track）**：专门针对字符级（Character-level）和表意文字建模的探索赛道，鼓励抛弃传统的子词（Subword）分词器，直接从汉字特征入手。


### 指标

| 赛道 | 任务 | 数据集 | 来源 |
|---|---|---|---|
| NLU — 零样本 | ZhoBLiMP | 最小对（句法、语义等） | `chinese-babylm-org/zhoblimp` |
| NLU — 微调 | CLUE | AFQMC, OCNLI, TNEWS, CLUEWSC2020 | `clue`（HuggingFace） |
| 汉字 | hanzi-structure | 汉字部件结构 | `chinese-babylm-org/hanzi-structure` |
| 汉字 | hanzi-pinyin | 汉字语音 | `chinese-babylm-org/hanzi-pinyin` |
| Cog | CogBench | fMRI 神经记录 | `zhiheng-qian/cogbench` |

### 按照测试方式来分类

| zero-shot | cogbench | 微调 |
|---|---|---|
|ZhoBLiMP|fmri|afqmc|
|hanzi-structure|word_fmri|cluewsc2020|
|hanzi-pinyin|---|ocnli|
|-|---|tnews|

## 数据集

**方案一.使用官方提供预训练数据集102M**

https://huggingface.co/datasets/chinese-babylm-org/babylm-zho-100M

**方案二.使用自选数据集**

**要求词数不超过 102M（以 Jieba 分词为准）。**


## 测试结果统计

### baseline中平均得分最高的几个模型

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Qwen3-0.6B | 77.62 | 59.85 | 49.80 | 55.00 | 10.40 | 71.57 | 75.08 | 58.10 | 71.71 | 58.79 |
| bert-base-chinese | 83.26 | 57.25 | 47.00 | 56.00 | 10.50 | 72.75 | 74.34 | 57.54 | 72.37 | 59.00 |
| xlm-roberta-base | 84.00 | 57.90 | 37.90 | 55.60 | 10.60 | 73.03 | 73.93 | 56.05 | 63.49 | 56.94 |



### 官方给出的babylm平均分较高的模型

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| babylm-chinese-bert-14m-epoch10 | 74.80 | 53.30 | 39.90 | 55.82 | 9.27 | 69.00 | 60.07 | 54.65 | 64.47 | 53.48 |
| babylm-chinese-bert-14m-epoch20 | 75.82 | 54.45 | 40.10 | 55.89 | 9.46 | 69.00 | 61.66 | 54.96 | 63.82 | 53.91 |

### Erlangshen-DeBERTa测试结果

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
|  Erlangshen-DeBERTa-v2-97M-Chinese | 84.45 | 57.80 | 30.90 | - | - | - | - | - | - | - |
|  Erlangshen-DeBERTa-v2-320M-Chinese | 85.87 | 63.45 | 33.00 | 55.73 | - | - | - | - | - | - |
|  Erlangshen-DeBERTa-v2-710M-Chinese | 85.21 | 64.25 | 24.70 | 55.87 | 11.56 | - | - | - | - | - |
