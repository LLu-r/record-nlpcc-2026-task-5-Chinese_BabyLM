# record-nlpcc-2026-task-5-Chinese_BabyLM

## babylm-chinese-14M实验结果在最后

## 目标

与国际 BabyLM 一致：在受人类认知启发的严格数据预算下，探索如何高效训练中文语言模型，缩小 AI 与人类儿童在语言习得过程中的数据效率差距。

## 赛道和评测指标

### 赛道

1.**NLU 赛道（Natural Language Understanding）**：在有限的训练数据下，评估模型在常识推理、文本分类等标准自然语言理解任务上的表现。

2.**认知建模赛道（Cognitive Modeling）**：采用受认知科学启发的评估指标，考察模型习得中文的过程及行为特征与人类儿童的相似度。

3.**汉字赛道（HANZI Track）**：专门针对字符级（Character-level）和表意文字建模的探索赛道，鼓励抛弃传统的子词（Subword）分词器，直接从汉字特征入手。


### 评测指标（9项）

| 赛道 | 任务 | 数据集 | 来源 |
|---|---|---|---|
| NLU — 零样本 | ZhoBLiMP | 中文最小对（句法、语义等） | `chinese-babylm-org/zhoblimp` |
| NLU — 微调 | CLUE | AFQMC, OCNLI, TNEWS, CLUEWSC2020 | `clue`（HuggingFace） |
| 汉字 | hanzi-structure | 汉字部件结构 | `chinese-babylm-org/hanzi-structure` |
| 汉字 | hanzi-pinyin | 汉字语音 | `chinese-babylm-org/hanzi-pinyin` |
| Cog | CogBench | fMRI 神经记录 | `zhiheng-qian/cogbench` |

#### 1.中文最小对
官方论文《A Systematic Assessment of Language Models with Linguistic Minimal Pairs in Chinese》介绍了ZhoBLiMP基准测试集，ZhoBLiMP 的全称可以理解为“中文语言学最小对比对基准。

规则：给模型看两个句子（Pair）。这两个句子在绝大部分词汇和结构上完全一样，只有极其微小的一处差异。其中一句是符合中文语法的（可接受的），另一句是不符合中文语法的（不可接受的）。

评测标准：让语言模型分别计算这两个句子的生成概率。如果模型给“正确的句子”算出的概率 大于 “错误的句子”，就说明模型真正掌握了这个语法，记为正确（1分）；否则记为错误（0分）。

#### 2.汉字

**A. hanzi_pinyin（汉字拼音赛道）：这个赛道测试模型是否知道哪些字同音。**

例子：

sentence_good（正确的句子）：“帮”和“邦”的声母、韵母和声调完全相同。

sentence_bad（错误的句子）：“帮”和“碰”的声母、韵母和声调完全相同。

**B. hanzi_structure（汉字结构赛道）：这个赛道测试模型是否知道汉字的偏旁部首拆解。**

sent_good（正确的句子）：乒字上边的部分是丘。

sent_bad（错误的句子）：乒字上边的部分是甫。

#### 3.CLUE中文评测

| 数据集 | 任务类型 |  输出维度 (num_labels) | 评价指标 |
| :--- | :--- | :--- | :--- |
| **TNEWS** | 单句分类 |  15 | Accuracy |
| **AFQMC** | 句对分类 |  2 | Accuracy |
| **OCNLI** | 推理三分类 |  3 | Accuracy |
| **CLUEWSC2020** | 共指二分类 |  2 | Accuracy |

&nbsp;

| 数据集 | 考察能力 | 任务类型 | 模型输入拼装逻辑 (输入 -> 分类头) |
| :--- | :--- | :--- | :--- |
| **TNEWS** | 短文本分类 (主题感知) | 15 分类 | `[CLS] 标题文本 [SEP]`  $\rightarrow$ Linear(15) |
| **AFQMC** | 句子对匹配 (语义相似度) | 二分类 (0/1) | `[CLS] 句子A [SEP] 句子B [SEP]`  $\rightarrow$ Linear(2) |
| **OCNLI** | 逻辑推理 (蕴含/矛盾/中立) | 三分类 | `[CLS] 前提句子 [SEP] 假设句子 [SEP]`  $\rightarrow$ Linear(3) |
| **CLUEWSC2020** | 共指消解 (代词消歧) | 二分类 (T/F) | `[CLS] ...[名词]实体[/名词]...[代词]代指[/代词]... [SEP]` $\rightarrow$ Linear(2) |


#### 4.认知建模 Cog赛道

Cog 赛道它不看模型的输出，它看模型的隐层向量与真实人类的“脑电波”（fMRI 数据）是否同频。

**实验背景：**

科学家让一批中国母语者躺在功能性磁共振成像（fMRI）扫描仪里，让他们阅读或听一系列的中文故事/句子。扫描仪会记录下他们在处理每个词、每句话时，大脑皮层各个区域（特别是语言中枢）的血液氧合水平变化（BOLD 信号），这就是真实的人脑认知信号。

**计算方法**

**1.提取模型表征：** 把人类被试读过的中文句子输入到你的预训练模型中。提取模型在处理这些句子时，Transformer 层产生的隐层状态向量（Hidden States），这可以看作是模型对这句话的“内部心理表征”。（记作自变量矩阵 $X$）。

**2.对齐与映射：** 岭回归拟合 (Ridge Regression)由于模型的向量维度（比如你的 256 维）和人脑 fMRI 扫描仪记录的体素（Voxel）维度根本不在一个空间里，不能直接比较。必须用机器学习在它们之间架起一座桥梁。官方使用岭回归（Ridge Regression，即带有 L2 正则化的线性回归）来训练一个映射模型。目标：找到一个权重矩阵，使得模型的隐层向量乘以这个权重后，能尽可能逼近人脑真实的 fMRI 激活值。

$$Loss = \sum_{i=1}^{n} (y_i - \hat{y}_i)^2 + \lambda \sum_{j=1}^{p} w_j^2$$

**3.预测人脑反应** (Prediction on Test Set)在保留的测试集句子上，把模型的隐层向量输入刚才拟合好的岭回归模型，算出一个预测的大脑反应。

**4.最终打分：** 计算相关系数 (Pearson / Spearman Correlation)拿模型预测的脑部活动。官方使用 皮尔逊相关系数（衡量线性相关）或 斯皮尔曼相关系数（衡量单调相关）来打分。分值意义：相关系数越接近 1，说明你的模型和人脑的语言处理机制越“同频”；越接近 0，说明模型是用完全违背人类认知规律的方式在进行黑盒计算。

### 按照测试方式来分类

| zero-shot | cogbench | 微调（CLUE） |
|---|---|---|
|ZhoBLiMP|fmri|afqmc|
|hanzi-structure|word_fmri|cluewsc2020|
|hanzi-pinyin|---|ocnli|
|-|---|tnews|

## 预训练数据集

**方案一.使用官方提供预训练数据集102M**

https://huggingface.co/datasets/chinese-babylm-org/babylm-zho-100M

**方案二.使用自选数据集**

**要求词数不超过 102M（以 Jieba 分词为准）。**


## 测试结果统计

### baseline中平均得分最高的几个模型

| Model | zhoblimp | hanzi_struc | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc20 | mean |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Qwen3-0.6B | 77.62 | 59.85 | 49.80 | 55.00 | 10.40 | 71.57 | 75.08 | 58.10 | 71.71 | 58.79 |
| bert-base-chinese | 83.26 | 57.25 | 47.00 | 56.00 | 10.50 | 72.75 | 74.34 | 57.54 | 72.37 | **59.00** |
| chinese-bert-wwm-ext | 84.86 | 58.05 | 29.60 | 55.80 | 10.40 | 73.15 | 75.36 | 58.32 | 74.34 | 57.76 |
| xlm-roberta-base | 84.00 | 57.90 | 37.90 | 55.60 | 10.60 | 73.03 | 73.93 | 56.05 | 63.49 | 56.94 |



### 官方给出的babylm平均分较高的模型

| Model | zhoblimp | hanzi_struc | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc20 | mean |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| babylm-chinese-bert-14m-epoch10 | 74.80 | 53.30 | 39.90 | 55.82 | 9.27 | 69.00 | 60.07 | 54.65 | 64.47 | 53.48 |
| babylm-chinese-bert-14m-epoch20 | 75.82 | 54.45 | 40.10 | 55.89 | 9.46 | 69.00 | 61.66 | 54.96 | 63.82 | 53.91 |

### Erlangshen-DeBERTa测试结果

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |mean|
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |:--- |
|  Erlangshen-DeBERTa-v2-97M-Chinese | 84.45 | 57.80 | 30.90 | 55.67 | 11.28 | 72.01 | 75.97 | 56.56 | 64.47 |56.57|
|  Erlangshen-DeBERTa-v2-320M-Chinese | 85.87 | 63.45 | 33.00 | 55.73 | 10.67 | 76.51 | 81.46 | 58.09 | 88.49 |61.47|
|  Erlangshen-DeBERTa-v2-710M-Chinese | 85.21 | 64.25 | 24.70 | 55.87 | 11.56 | 74.63 | 80.78 | 57.13 | 63.49 |57.51|

注：微调部分出错，f1=0，模型输出的分类标签全为0，在排查问题。

已确定的问题：
Erlangshen-deberta为float16（半精度）加载。官方评测代码在编写时没有考虑混合精度的向下兼容，分类头还是写死的 float32，导致了 Half and Float 的报错。

暂时解决办法：在官方测评pipline的/finetune/classifier_model.py中加载分类模型转换为float32

```python
self.transformer: nn.Module = AutoModel.from_pretrained(
    config.model_name_or_path,
    trust_remote_code=True,
    revision=config.revision_name,
    torch_dtype=torch.float32      ##转换为float32,全精度加载模型
)

self.enc_dec: bool = config.enc_dec
self.causal: bool = config.causal
```

## 后续工作（暂时）

### 模型架构选择

既然DeBERTa架构在中文最小对评测上表现比较好，后续我打算尝试不同大小的DeBERTa-v2架构在Chinese BabyLM上的表现。

### 模型大小选择

目前的方案：尝试14M及14M以上参数量的模型

官方的Baseline BabyLM包括bert、T5、pythia等架构，参数量都为14M（一千四百万）

其中babylm-chinese-bert-14M的结构：

```json
{
  "architectures": [
    "BertForMaskedLM"
  ],
  "attention_probs_dropout_prob": 0.1,
  "classifier_dropout": null,
  "directionality": "bidi",
  "hidden_act": "gelu",
  "hidden_dropout_prob": 0.1,
  "hidden_size": 224,
  "initializer_range": 0.02,
  "intermediate_size": 672,
  "layer_norm_eps": 1e-12,
  "max_position_embeddings": 512,
  "model_type": "bert",
  "num_attention_heads": 8,
  "num_hidden_layers": 5,
  "pad_token_id": 0,
  "pooler_fc_size": 224,
  "pooler_num_attention_heads": 8,
  "pooler_num_fc_layers": 3,
  "pooler_size_per_head": 28,
  "pooler_type": "first_token_transform",
  "position_embedding_type": "absolute",
  "torch_dtype": "float32",
  "transformers_version": "4.38.0",
  "type_vocab_size": 2,
  "use_cache": true,
  "vocab_size": 50004
}
```
#### 为什么选择14M？

##### 缩放定律计算

按照DeepMind的Chinchilla 缩放定律(Hoffmann, J., Borgeaud, S., Mensch, A., et al. (2022). Training Compute-Optimal Large Language Models. arXiv preprint arXiv:2203.15556.)

DeepMind 通过实验拟合得出，在给定训练总算力（FLOPs）的约束下，为了达到最优的训练效果（即模型损失最小），模型的参数量（N）与训练数据的 Token 数量（D）应当成等比例缩放。其推导出的“算力最优前沿”（Compute-Optimal Frontier）的经验法则公式为：

$$D \approx 20 \times N$$

官方提供的预训练数据大小为102M：

$$D = 102,000,000$$

代入公式：

$$N = \frac{102,000,000}{20} = 5,100,000$$

**约5.1M参数为最优**，14M快赶上这个参数量的三倍了。

&nbsp;

##### pythia家族

官方给了基准测试集结果包括他们自己训练的 Zh-Pythia 家族**14M 到 1.4B** 而pythia家族中最小的模型就是**14M**

### 模型

目前还没有完全搞清楚如何针对当前任务设计最合适的模型，大语言模型给的一份方案参考：

```json
{
  "model_type": "deberta-v2",
  "architectures": [
    "DebertaV2ForMaskedLM"
  ],
  "torch_dtype": "float32",
  
  "vocab_size": 21128,
  "hidden_size": 256,
  "num_hidden_layers": 12,
  "num_attention_heads": 8,
  "intermediate_size": 1024,
  
  "hidden_act": "gelu",
  "hidden_dropout_prob": 0.1,
  "attention_probs_dropout_prob": 0.1,
  "max_position_embeddings": 512,
  "type_vocab_size": 0,
  "initializer_range": 0.02,
  "layer_norm_eps": 1e-7,
  "pad_token_id": 0,
  
  "relative_attention": true,
  "pos_att_type": ["p2c", "c2p"],
  "position_biased_input": false,
  "max_relative_positions": -1
}
```

#### 理由：

```
1. 算力预算分配参数（决定了你的 14M 怎么花）在语言模型中，最大的“吸金兽”是词表（Embedding 层），其次才是用来做逻辑推理的 Transformer 层。
vocab_size: 21128理由：这是标准的中文字粒度词表（兼容原生 bert-base-chinese 的分词器）。对于小模型来说，强行缩减词表会导致大量汉字变成 [UNK]，扩大词表又会浪费参数。21128 是中文场景下兼顾覆盖率和体积的唯一“甜点值”。
hidden_size: 256理由：特征维度。如果你设为 BERT 标准的 768，光词表层就会吃掉 $21128 \times 768 \approx 16M$ 的参数，直接超标。将隐藏层压缩到 256，词表税（Embedding Tax）就降到了约 5.4M，我们才能省下近 9M 的预算给逻辑推理层。
num_hidden_layers: 12理由：网络深度，也就是“大脑的层数”。很多 14M 的开源模型为了做宽，只能做 5 层或 6 层。但深度往往比宽度更能决定模型的语法抽象能力。我们用 256 的宽度省下了空间，硬生生盖起了 12 层的深层网络。在语法树解析（比如 ZhoBLiMP 评测）中，12 层的结构拥有绝对优势。
intermediate_size: 1024理由：前馈神经网络（FFN）的扩张维度。行业黄金准则是 hidden_size 的 4 倍（$256 \times 4 = 1024$）。这保证了模型在每层处理完注意力交互后，有足够的非线性空间来记忆局部特征。
num_attention_heads: 8理由：模型同时从多少个角度理解句子。必须能被 hidden_size 整除（$256 \div 8 = 32$）。每个头负责 32 维的子空间，足以捕捉中文的“主谓宾”和“修饰关系”。(参数核算：5.4M 词表 + 12 层 $\times$ 约 0.78M/层 $\approx$ 14.8M 总参数，完美卡点。)

2. DeBERTa-v2 的纯正基因（不可删改的标识）既然叫 DeBERTa-v2，就必须舍弃 BERT 笨拙的绝对位置编码，开启其引以为傲的解耦注意力（Disentangled Attention）。
relative_attention: true理由：开启相对位置编码的主开关。它让模型不再死记硬背“这个词在第 3 个位置”，而是关注“词 A 在词 B 前面 2 个位置”。
pos_att_type: ["p2c", "c2p"]理由：解耦注意力的灵魂核心。p2c (position-to-content) 捕捉某个位置和目标内容的关系，c2p (content-to-position) 捕捉某内容和目标位置的关系。这是 DeBERTa 吊打原始 BERT 的核心数学创新。
position_biased_input: false理由：关闭底层输入时的绝对位置偏置。因为相对位置编码已经接管了位置计算，底层再加就是画蛇添足。
max_relative_positions: -1理由：让模型动态适应相对距离，不再设置硬性的最大相对位置截断。

3. 训练稳定性与防雷设计
type_vocab_size: 0理由：这是区分句子 A 和句子 B 的标志符。DeBERTa 靠相对位置就能分清句子边界，不再需要 token_type_ids。设为 0 可以进一步省下一部分参数（虽然不多），并让你在构建 Dataloader 时少传一个无用的 Tensor，极大简化代码。
torch_dtype: "float32"理由：绝对的防雷补丁。我们在微调排错时吃过这个大亏。从零预训练 14M 这种极小参数模型时，强制锁定为 float32 单精度，能够彻底避免前期因为梯度下溢出导致的 NaN 爆炸问题。
hidden_dropout_prob & attention_probs_dropout_prob: 0.1理由：预训练防过拟合的标配。因为你的数据量可能只有 1 亿 Token，模型很容易“死记硬背”。保留 10% 的随机神经元失活，能强迫模型学习真正的语法规律。
```

### 官方给出的提升成绩的思路

以下仅为建议，并非硬性要求。参赛者可在数据预算范围内自由探索其他方法。

数据策略： 提升语料质量（过滤、去重），调整训练轮数，或尝试训练顺序 / 课程学习。

模型架构： 尝试 Transformer 编码器、解码器、编码器-解码器、Mamba / 状态空间模型，或扩散语言模型。

训练方法： 探索强化学习、偏好优化、多阶段预训练等高级技术。

认知赛道调优： 隐藏层的选择与特征提取方式（例如平均池化、特定层的表征）对 fMRI 对齐成绩有显著影响。

```text
______________________________________________________________________________________________
```
## 实验部分

按照官方的要求，只允许使用不超过102M的限制数据集来完成该任务。所以continue training没有继续。

### babylm-chinese-deberta-v2-14M-epoch10(统计UNK之前的实验结果)

我沿用Erlangshen-deberta-v2架构，构建了一个参数量14.6M的模型。直接在官方提供的102M数据集上训练了10个epoch

我没有额外训练tokenizer，而是构建了大小与Erlangshen-deberta-v2-97M相同的词表，使用其tokenizer。

具体架构如下:

```json
{
  "architecture_2_config_class": {
    "DebertaV2ForMaskedLM": "DebertaV2Config"
  },
  "architectures": [
    "DebertaV2ForMaskedLM"
  ],
  "attention_probs_dropout_prob": 0.1,
  "hidden_act": "gelu",
  "hidden_dropout_prob": 0.1,
  "hidden_size": 256,
  "initializer_range": 0.02,
  "intermediate_size": 1024,
  "layer_norm_eps": 1e-7,
  "max_position_embeddings": 512,
  "max_relative_positions": 512,
  "model_type": "deberta-v2",
  "num_attention_heads": 8,
  "num_hidden_layers": 12,
  "pad_token_id": 0,
  "pooler_hidden_size": 256,
  "pooler_size_per_head": 128,
  "position_biased_input": false,
  "pos_att_type": [
    "c2p",
    "p2c"
  ],
  "relative_attention": true,
  "talking_head": false,
  "type_vocab_size": 0,
  "vocab_size": 12800,
  "tie_word_embeddings": true
}
```

### 实验测试结果

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |mean|
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |:--- |
| babylm-chinese-deberta-v2-14M-epoch3 | 68.53 | 52.95 | 68.20 | 55.74 | 6.81 | 69.00 | 60.51 | 53.06 | 63.49  |54.35|
| babylm-chinese-deberta-v2-14M-epoch10| 71.10 | 53.25 | 36.60 | 55.91 | 7.21 | 69.05 | 64.54 | 53.23 | 63.49 |52.71|

&nbsp;

### 课程学习（Cirriculum Learning）babylm-chinese-deberta-v2-14M-CL

#### 测试阶段实验设置：

把数据切分出来，分为4个部分，分阶段训练模型。

| 训练阶段 | 覆盖 Epoch | 数据筛选条件 | Max Sequence Length |
| :--- | :--- | :--- | :--- |
| **幼教期** | Epoch 1 - 2 | 仅保留长度 `< 32` 的短句 | 64 |
| **小学期** | Epoch 3 - 5 | 引入长度 `< 64` 的中等句子 | 128 |
| **中学期** | Epoch 6 - 8 | 引入长度 `< 128` 的长句 | 256 | 
| **冲刺期** | Epoch 9 - 10 | **全量数据** | 512 |

#### 实验结果统计：

//统计UNK之前

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |mean|
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |:--- |
| babylm-chinese-deberta-v2-14M-CL-final(epoch10) | 66.06 | 53.15 | 63.90 | 55.73 | 6.88 | 69.25 | 59.53 | 52.40 | 63.82 |54.52|

#### 统计UNK之后的结果：

**重做tokenizer再训练**

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |mean|
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |:--- |
| babylm-chinese-deberta-v2-final-lr1e-4-epoch10|59.02|49.35|50.25|55.79|6.86|69.05|59.86|51.99|63.49| 51.74 |
| **babylm-chinese-deberta-v2-final-lr5e-4-epoch10**|69.06|57.30|47.20|55.69|7.35|69.44|65.86|53.45|63.49|54.32|

&nbsp;

**学习率 LR= 5e-4 增加训练轮数**

#### 实验设置：

| 训练阶段 | 覆盖 Epoch | 数据筛选条件 | Max Sequence Length |
| :--- | :--- | :--- | :--- |
| **幼教期** | Epoch 1 - 2 | 仅保留长度 `< 32` 的短句 | 64 |
| **小学期** | Epoch 3 - 5 | 引入长度 `< 64` 的中等句子 | 128 |
| **中学期** | Epoch 6 - 10 | 引入长度 `< 128` 的长句 | 256 | 
| **冲刺期** | Epoch 11 - 30 | **全量数据** | 512 |

#### 实验结果统计：

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |mean|
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |:--- |
|babylm-chinese-deberta-v2-final-lr5e-4-epoch30(fine-tuning LR=3e-5)|75.23|61.00|51.40|55.68|8.50|70.57|67.66|54.25|63.49|56.42|
|babylm-chinese-deberta-v2-final-lr5e-4-epoch30(fine-tuning LR=2e-5)|↑|↑|↑|↑|↑|70.44|68.00|54.16|63.49|56.43|


**fine-tuning测评阶段多轮调试后的结果**

修改pipline配置

```yaml
finetune_hparams:
  lr: 2.0e-5
  batch_size: 32
  max_epochs: 10
  sequence_length: 128
  seed: 42

  task_overrides:
    afqmc:
      lr: 2.0e-5
      batch_size: 16
      max_epochs: 15
    ocnli:
      lr: 2.0e-5
      batch_size: 32
      max_epochs: 10
    tnews:
      lr: 3.0e-5
      batch_size: 32
      max_epochs: 15
    cluewsc2020:
      lr: 5.0e-5
      sequence_length: 256
      batch_size: 8
      max_epochs: 30
```

| Model | zhoblimp | hanzi_structure | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc2020 |mean|
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |:--- |
|babylm-chinese-deberta-v2-final-lr5e-4-epoch30|75.23|61.00|51.40|55.68|8.50|71.11|68.00|54.69|64.14|56.64|




&nbsp;

### 官方给出的babylm平均分较高的模型

| Model | zhoblimp | hanzi_struc | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc20 | mean |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| babylm-chinese-bert-14m-epoch10 | 74.80 | 53.30 | 39.90 | 55.82 | 9.27 | 69.00 | 60.07 | 54.65 | 64.47 | 53.48 |
| babylm-chinese-bert-14m-epoch20 | 75.82 | 54.45 | 40.10 | 55.89 | 9.46 | 69.00 | 61.66 | 54.96 | 63.82 | 53.91 |

### baseline中在大语料上训练的平均得分最高的几个模型

| Model | zhoblimp | hanzi_struc | hanzi_pinyin | word_fmri | fmri | afqmc | ocnli | tnews | cluewsc20 | mean |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Qwen3-0.6B | 77.62 | 59.85 | 49.80 | 55.00 | 10.40 | 71.57 | 75.08 | 58.10 | 71.71 | 58.79 |
| bert-base-chinese | 83.26 | 57.25 | 47.00 | 56.00 | 10.50 | 72.75 | 74.34 | 57.54 | 72.37 | **59.00** |
| chinese-bert-wwm-ext | 84.86 | 58.05 | 29.60 | 55.80 | 10.40 | 73.15 | 75.36 | 58.32 | 74.34 | 57.76 |
| xlm-roberta-base | 84.00 | 57.90 | 37.90 | 55.60 | 10.60 | 73.03 | 73.93 | 56.05 | 63.49 | 56.94 |



### 后续任务

官方明确指出，模型必须在AoE时间6月11日23：59前（北京时间6月12日19:59）上传至huggingface，此后会公布最终评比测试数据和pipline。参赛者将最终的分数统计并上传到Leaderboard。

主办方会公布4个排名：3个赛道各一个排名+总分排名。6月20日后联系每个排名榜上前三名的队伍进行复现。
