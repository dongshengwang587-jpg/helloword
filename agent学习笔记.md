## 目录

- [1. 适配器模式](#1-适配器模式)
- [2. 类间关系](#2-类间关系)
- [3. mcp server与mcp client的通信](#3-mcpserver与mcpclient的通信)
- [4. 提示词工程](#4-提示词工程)
- [5. RAG](#5-RAG).
- [6. RAG](#6-缓存机制).

## 1. 适配器模式

适配器模式解决的是接口不兼容的问题.

如下所示，原来的支付接口为pay_money,而新的支付接口为pay，为了解决接口不兼容，引入适配器，将pay_money接口封装成pay接口。

class OldPayment:

    def pay_money(self, money):
        print(f"支付 {money} 元")


class PaymentAdapter:

    def __init__(self, old_payment):
        self.old_payment = old_payment


    def pay(self, amount):

        # 转换接口
        self.old_payment.pay_money(amount)



payment = PaymentAdapter(
    OldPayment()
)


payment.pay(100)



## 2. 类间关系

继承：狗继承动物是最好的例子，继承父类所有方法与属性。

实现：python中没有接口，通过抽象类实现接口。

关联：一个类拥有一个类的引用

聚合：一个类拥有一个类的引用

组合：一个类完全拥有另一个类，比如汽车完全拥有引擎。

依赖：临时调用某个类属于依赖。

# 3. mcp server与mcp client的通信

1.mcp server与mcp client在不同进程时，使用stdio方式通信也就是进程间通信。

2.mcp server与mcp client在不同机器时使用http进行通信，优先选流式http。

# 4. 提示词工程

大模型对promot开头和结尾的内容更敏感.

对应论文地址：Lost in the Middle: How Language Models Use Long Contexts（TACL 2024）

在promot中给出example的效果很好。

大模型应用架构师想什么？

怎样能更准确？答：让更多的环节可控

怎样能更省钱？答：减少 prompt 长度

怎样让系统简单好维护？

提示词模板：

prompt = f"""
角色：{role}
任务说明：{instruction}
例子：{example}
输出格式：{output_format}
用户输入：{input_text}

"""

# 5. RAG

相似度计算常用算法：

余弦相似度与欧氏距离

相似性搜索加速算法：

在向量数据库中，数据按列进行存储，通常会将多个向量组成一个 M×N 的矩阵，其中 M 是向量的维度（特征数），N 是向量的数量（数据库中的条目数），这个矩阵可以看作把多个样本组成。取决于向量的稀疏性和具体的存储优化策略。

这样计算相似性搜索时，本质上就变成了向量与 M×N 矩阵中的每一行进行相似度计算，这里可以用到大量成熟的加速算法：

1. 矩阵分解方法
SVD（奇异值分解）：可以通过奇异值分解将原始矩阵转换为更低秩的矩阵表示，从而减少计算量。
PCA（主成分分析）：类似地，可以通过主成分分析将高维矩阵映射到低维空间，减少计算复杂度。
2. 索引结构和近似算法
LSH（局部敏感哈希）：LSH 可以在近似相似度匹配中加速计算，特别适用于高维稀疏向量的情况。
ANN（近似最近邻）算法：ANN 算法如 KD-Tree、Ball-Tree 等可以用来加速最近邻搜索的计算，虽然主要用于向量空间，但也可以部分应用于相似度计算。
3. GPU 加速

使用图形处理单元（GPU）进行并行计算，可以显著提高相似度计算的速度，尤其是对于大规模数据和高维向量。

4. 分布式计算

由于行与行之间独立，所以可以很便捷地支持分布式计算每行与向量的相似度，从而加速整体计算过程。


# 6. 缓存机制

会话存储在本地，当用户问问题时，首先去存储做相似度匹配，若有高度相似的问题，就直接返回答案，否则调用大模型回答问题，并将问题及答案进行存储。

langchian的CacheBackedEmbeddings组件封装的缓存机制，组件内部实现查询

