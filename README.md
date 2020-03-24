# chinese-sequence-ner

## chinese-sequence-ner 中文命名实体识别

命名实体识别作为序列标注类的典型任务，其使用场景特别广泛。本文基于PyTorch搭建**HMM、CRF、BiLSTM、BiLSTM+CRF及BERT**模型，实现中文命名识别任务。

其中HMM、CRF、BiLSTM和BiLSTM+CRF模型文件在models文件夹下，训练文件融合在main.py中，并做了比较及集成。BERT模型使用
huggingface提供的transformers实现，预训练模型使用哈工大讯飞开源的<a href="https://github.com/microsoft/unilm" target="_blank">中文BERT-wwwm</a>，模型训练
代码在bert_base_ner.py文件中

### HMM

HMM是关于时序的概率模型，描述由一个隐藏的马尔科夫链随机生成不可观测的状态随机序列，再由各个状态生成一个观测而产生观测随机序列的过程（李航 统计学习方法）。HMM基于2个假设—齐次马尔科夫假设与观测独立性假设，这是HMM相较于其它几个模型性能差的主要原因。HMM由三要素—初始状态向量、观测概率矩阵及状态转移概率矩阵所确定。模型涉及HMM的2大问题，参数即三要素的学习算法和解码预测算法。

HMM模型训练结果：

precision|recall|f1-score
:----:|:----:|:----:
0.9129|0.9097|0.9107


### 条件随机场(CRF)

HMM是典型的生成模式，对联合概率建模且受限于2个假设，其性能天花板相对较低。CRF（我们用的是特殊CRF，即线性链条件随机场）是典型的判别模式。CRF通过引入自定义的特征函数，不仅可以表达观测之间的依赖，还可表示当前观测与前后多个状态之间的复杂依赖，可以有效克服HMM模型面临的问题

CRF模型训练结果：

precision|recall|f1-score
:----:|:----:|:----:
0.9543|0.9543|0.9542


### BiLSTM
循环神经网络RNN的改进型LSTM，特别适用于时序文本数据的处理。LSTM依靠神经网络超强的非线性拟合能力以及隐状态对信息的传递，在训练时将样本通过高维空间中的复杂非线性变换，学习到从样本到标注的函数。双向LSTM对前后向的序列依赖的捕获能力更强。
相比于传统统计学习方法，BiLSTM不需要做复杂的特征工程，简单直接。

BiLSTM模型训练结果:

precision|recall|f1-score
:----:|:----:|:----:
0.9553|0.9552|0.9551


### BiLSTM + CRF

LSTM的优点是能够通过hidden state的传递学习到观测序列（输入的字）之间的依赖，在训练过程中，LSTM能够根据目标（比如识别实体）自动提取观测序列的特征，但缺无法学习到状态序列（输出的标注）之间的关系。在命名实体识别等序列标注任务中，标注之间是有一定的关系的，比如B类标注（表示某实体的开头）后面不会再接一个B类标注，E类标注（表示某实体的结尾）后面不会再接一个E类标注。
所以LSTM在解决NER这类序列标注任务时，虽然可以省去很繁杂的特征工程，但是无法有效学习到标注之间的上下文关系。相反，CRF的优点就是能对隐含状态建模，学习状态序列的特征，但它的缺点是需要手动提取序列特征。
所以一般的做法是，将两者结合起来，在LSTM后面再加一层CRF，以获得两者的优点。

模型训练结果：

precision|recall|f1-score
:----:|:----:|:----:
0.9578|0.9578|0.9576

### 模型ensembel结果

precision|recall|f1-score
:----:|:----:|:----:
0.9558|0.9558|0.9557


### BERT

BERT基于Transformers Encoder，使用MLM获取双向融合信息，在海量连续语料上进行预训练，得到的预训练模型，基于下游任务只需简单的fine-tuning就能获得特别好的结果。

模型训练结果：

precision|recall|f1-score
:----:|:----:|:----:
0.9494|0.9646|0.9569


### 所有模型结果汇总

模型|precision|recall|f1-score
:----:|:----:|:----:|:----:
HMM|0.9129|0.9097|0.9107
CRF|0.9543|0.9543|0.9542
BiLSTM|0.9553|0.9552|0.9551
BiLSTM+CRF|0.9578|0.9578|0.9576
ensembel|0.9558|0.9558|0.9557
BERT|0.9494|0.9646|0.9569


### 结果分析：


* 隐马尔科夫模型相比其它几种模型确实性能差距很大，其基于的齐次马尔科夫假设和观测独立假设限制了性能天花板
* BiLSTM+CRF模型相比BiLSTM模型性能提升很少，可能的解释推测为数据集太小，不足于让CRF层学到多少状态转移之间的信息
* ensembel性能全面不如BiLSTM+CRF，可以解释为其他3个模型拖累了BiLSTM+CRF
* BERT模型只在recall上比BiLSTM+CRF表现好，表明在中文NER上BERT并不一定是最好的模型，可能为模型太强导致的过拟合。