《GLOBAL-TO-LOCAL MEMORY POINTER NETWORKS FOR TASK-ORIENTED DIALOGUE 》阅读笔记

##### 1.论文的贡献

作者提出了global-to-local memory pointer (GLMP) networks将KB(knowledge base)嵌入学习框架，提高了copy-mechanism的准确率，缓解了词汇溢出的问题，在simulated bAbI Dialogue dataset and human-human Stanford Multi-domain Dialogue dataset 两个数据集中超越了之前的效果。

##### 2.前人的主要贡献

传统流式处理方法将任务型对话分为了几个独立的阶段：自然语言理解(NLU)、对话管理(DM)、自然语言生成(NLG)。传统框架中，如果某个步骤错误，会导致后序步骤错上加错，并且每个阶段都相对昂贵。（个人认为这种分阶段的框架清晰明了，可能是没有充分利用各个阶段之间的相关信息以及涉及过多人工干预，导致效果或者代价昂贵）

后面有人证明RNN生成答案和MN(Memory Network)对KB进行建模能够改善效果。**Pointer networks** (Vinyals et al., 2015) or **copy mechanism** (Gu et al., 2016) is crucial to successfully generate system responses because directly copying essential words from the input source to the output not only reduces the generation difficulty, but it is also more like a human behavior.

将问题转化的工作：Some works view the task as a **next utterance retrieval problem**.

Some approaches treat the task as a **sequence generation problem**.

##### 3.组件与机制

###### 组件：

Memory Network(MN)

功能：The MN is well-known for its multiple hop reasoning ability (Sukhbaatar et al., 2015), which is appealing to strengthen copy mechanism.

最基本的MN介绍：https://zhuanlan.zhihu.com/p/29590286

RNN

###### 机制：	

**copy mechanism**：

从知识库和输入中直接获取可以输出的内容，减少重新生成。

**attention**：

知识或输入对输出的贡献是动态变化的，而之前的工作是计算出权重c，然后知识或输入对输出的权重都是c.具体解释，需要些一篇专门文章。

**multi-hop design**：

多步推理形式的阅读理解从多个文本段获取答案，而传统抽取式阅读理解大部分只要预测答案在原文的位置即可。在任务型对话系统中，应该指的是从KB和对话历史的不同地方推理得到答案。

##### 4.模型与框架

###### 整体框架：

First, **the global memory encoder uses a context RNN to encode dialogue history and writes its hidden states into the external knowledge**.
Then the last hidden state is used **to read the external knowledge and generate the global memory pointer** at the same time. 

On the other hand, during the decoding stage, **the local memory decoder first generates sketch responses by a sketch RNN.** 

Then **the global memory pointer and the sketch RNN hidden state are passed to the external knowledge as a filter and a query.** 

The local memory pointer returns from the external knowledge can copy text from the external knowledge to replace the sketch tags and obtain the final system response.

![image-20201205172720618](C:\Users\xmh\AppData\Roaming\Typora\typora-user-images\image-20201205172720618.png)

框架的组件：

###### 4.1 EXTERNAL KNOWLEDGE

Global contextual representation

KB memory module将知识表示为三元组(Subject, Relation, Object),dialogue memory module将知识表示为三元组{($user, turn1, word),...}。将其中的单词先进行词向量编码，然后转化为词袋表示。

Knowledge read and write

初始化一个query向量，计算query与memory module中的词袋表示得到第k步的注意力概率$p_k$。然后,将注意力概率与memory module的词袋表示相乘得到memory module的输出，然后query通过输出进行增量更新。	

###### 4.2 GLOBAL MEMORY ENCODER

使用Context RNN将对话历史转化为隐含状态（RNN的输出）存入memory module。将memory module的表示加上隐含状态得到新的表示。Context RNN的最终隐含态用来query EK得到EK步骤的soft memory attention.

Global memory pointer用来表示memory中的词汇在回答中的概率。实现会有标准的Global memory pointer(0,1组成)，如果用户提问的单词没在回答中出现，那么为0，否则为1。这样就可以计算两者的交叉熵，将训练global memory pointer转化为一个多分类问题。EK 中计算注意力概率将query向量和memory表示相乘，然后经过softmax函数输出注意力概率。Global memory pointer与注意力概率不同的是，使用的是sigmoid函数。

###### 4.3 LOCAL MEMORY DECODER

Given **the encoded dialogue history** h, **the encoded KB information** q, and **the global memory pointer** G, our local memory decoder first initializes its sketch RNN using the concatenation of h and q, and generates a sketch response that excludes slot values but includes the sketch tags.

使用**global memory pointer**修改memory module中的表示，然后由sketch RNN根据修改后的表示生成的隐含状态h进行query EK，得到注意力概率. 作者使用multi-hop机制最后一个hop计算得到的注意力概率作为local memory pointer来表示memory中的分布。local memory pointer的标准值是统计回答中的单词在总的实体列表（通过KB得到的关系字典统计得到的列表）中的最后位置。最后计算masked-cross-entroy，然后学习训练（这段不是很理解）。

最后整个模型的损失值由global memory pointer, local memory pointer以及sketch的误差经过加权求和得到。

##### 5.结论

贡献所说的内容。模型在bAbI Dialogue任务和Stanford Multi-domain Dialogue任务的效果超越了之前的工作。

