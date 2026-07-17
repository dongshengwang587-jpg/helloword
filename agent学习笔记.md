## 目录

- [1. 适配器模式](#1-适配器模式)
- [2. 类间关系](#1-类间关系)
- [3. mcp server与mcp client的通信](#1-mcpserver与mcpclient的通信)



























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


