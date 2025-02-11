

# Generative Pre-trained Transformer -GPT

> 生成式预训练 Transformer.

### 1 High level preview / 数据流总览

> how data flows through a transformer.

**拆词 → 发身份证 → Transformer聊天室讨论上下文 → 用最后一个词的身份证去词库匹配 → 生成下一词 → 循环往复写完整段话！**

1. 将长文本输入切分为 token，并作向量化嵌入。

2. 每个 token 嵌入后的向量对应 [ Embedding martix ] (嵌入矩阵)每一列。

   > 当下嵌入向量仅包含本身含义，尚未包含上下文语境，直接从 [ Embedding martix ] 拉出。
   >
   > 模型具有一个预设的词汇库，以 [ Embedding martix ] 唯一标识。

3. 输入网络 { transformer + MLP }* 96

   > 流入网络的意义在于使得输入在切分后的每个嵌入向量，获得有限上下文的相互含义。
   >
   > 网络一次只能处理特定数量的向量 - 上下文长度，其限制了网络在预测下一个词时能够结合的文本量。

4. 对最后一个嵌入向量乘以 [ Unembedding martix ] (解嵌入矩阵), 得预测下一 token 的 Logits，归一化后得下一个 token 的概率密度分布。

   > Logits 是一列数列，归一化后表示下一个可能 token 的概率分布。
   >
   >  [ Unembedding martix ] 为 [ Embedding martix ] 的转置。

**Points:**

- 感知机阶段 向量不再相互交流，而是并行经历同一处理

  > 这个我很难理解，问问题不是相互交流，问题与其他 token 不相关？

- 机器学习就是采用数据驱动，反馈到模型参数进而指导模型行为的方法

- 权重为模型大脑，随机初始化后训练后所学习得到，其决定了模型的行为模式

- transformer 的本意是什么？

  > 最初用于翻译 中文 <-> 英文

- 归一化就是 softmax？

  > softmax 是一种归一化方法，将输入映射到 0-1 之间的概率分布

- 向量的点积可以衡量他们的对其程度



**Source:**

- GPT3 中[ Embedding martix ] 为 12288 * 50k

  > 表示一个 token 有 12288 个维度表示其语义
  >
  > 词汇库中具有 50k 个 token

![image-20250210211445820](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210211445820.png)

![image-20250210213308413](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210213308413.png)

- 上下文限制

![image-20250210213604255](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210213604255.png)

![image-20250210213618875](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210213618875.png)

![image-20250210213635137](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210213635137.png)

![image-20250210213649467](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210213649467.png)

![image-20250210213657600](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210213657600.png)

-  [ Unembedding martix ] 

![image-20250210214625372](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210214625372.png)



### 2 Attention

- Argumentts

![image-20250210211003117](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20250210211003117.png)

​         