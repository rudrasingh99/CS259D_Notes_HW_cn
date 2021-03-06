# Adaptive Intrusion Detection System via Online Machine Learning

<!-- TOC -->

- [背景知识和启发](#背景知识和启发)
- [本文目标](#本文目标)
- [数据来源](#数据来源)
- [特征](#特征)
- [自适应入侵检测系统（A-IDS）](#自适应入侵检测系统a-ids)
    - [混合算法](#混合算法)
        - [记号](#记号)
        - [算法](#算法)
- [实验](#实验)
- [局限](#局限)
- [参考资料](#参考资料)

<!-- /TOC -->

## 背景知识和启发

* 网络环境多样
* 攻击类型持续变化
* 充满敌意的环境
* IDS的性能很大程度上取决于所选的分类器
    * 误用 IDS
        * 检测已知的攻击
        * SNORT：基于签名的 IDS
    * 异常 IDS
        * 检测新的攻击
        * FP 更高
    * **检测表现因环境而异**
    * **没有免费的午餐**
* 混合 IDS
    * 高成本
    * 多数投票规则
    * 错误
        * 真理未必掌握在多数人手中
* 对冲和提升（Hedge/Boosting）
    * 在线学习框架
    * 选择时段 T 中表现最好的 IDS
    * 持续变换攻击，一次来破坏最好的 IDS 的优势
    * 结果，所有的 IDS 表现都糟糕
* IDS 的适应性
    * 适应不断变化的敌对环境
    * 如何选择最好的 IDS

## 本文目标

* 新的高效的在线学习框架
* 自适应入侵检测系统（Adaptive Intrusion Detection System，A-IDS）

## 数据来源

* ECML-PKDD HTTP 2007
    * 50,000 样本，20% 是攻击数据
        * 攻击和正常数据
    * 攻击类型
        * 跨站脚本攻击
        * SQL 注入
        * LDAP 注入
        * XPATH 注入
        * 路径遍历
        * 执行命令
        * SSI 攻击
* CSIC HTTP 2010
    * 61,000 样本，41% 是攻击数据
        * 攻击和正常数据
    * 流量数据
        * 真实数据
        * 为此论文特意开发
        * 电子商务 Web 应用
        * Apache 服务器
    * 攻击类型
        * SQL 注入
        * 缓冲区溢出
        * 信息收集
        * CRLF 注入
        * XSS
        * SSI 攻击
        * 参数篡改

## 特征

* 30个特征
    * 长度
    * 数量
    * 最大值
    * 最小值
    * 头部类型
    * 四类字符
        * 子母
        * 数字
        * 非字母数字字符
            * 程序语言中有特殊含义的字符
            * “特殊含义”的字符
        * 其他
    * 信息熵
    * 程序语言关键字


## 自适应入侵检测系统（A-IDS）

* 基本 IDS
    * 基本分类器
        * 朴素贝叶斯
        * 贝叶斯网络
        * 决策桩
        * RBF 网络
    * **对于基本分类器的选择不做假设**
    * 10-folds 交叉验证
* *损失更新（Loss Update）*
    * 对冲和提升算法
* *混合更新（Mixing Update）*
    * Bousquet & Warmuth 算法（Bousquet & Warmuth，2002）
    * 快速恢复 IDS 的性能
        * 记住过去的平均权重矢量
* 监督学习框架
    * 综合基本 IDS 的结果
    * 获取当前样本的真实标签
    * 衡量预测值和真实值之间的损失
    * 维护基本 IDS 的权重

### 混合算法

#### 记号

* $$T$$，样本数量
* $$t$$，$$(t=1,...,T)$$，一个时段或一个实验
* $$n$$，基本IDS数量
* $$i$$，$$i \in \{ 1, 2, ..., n \}$$，基本 IDS 的序数
* $$x_t$$,由 $$n$$ 个 IDS 输出的结果构成的向量
    * A-IDS 接受 $$x_t$$
    * $$x_t = (x_{t,1}, ..., x_{t,n})$$
        * 其中 $$x_{t, i} \in \{0\text{(normal)}, 1\text{(attack)}\}$$
* $$pred(t)$$，A-IDS 的预测
* $$y_t$$, $$y_t \in \{0, 1\}$$，在 $$t$$ 时刻第 $$t$$ 个样本的真是标签
* $$L$$，损失函数
    * 对于实验 $$t$$ 以及第 $$i$$ 个 IDS
    * $$L_{t, i} = (y_t - x_{t, i})^2$$
    * 计算真实结果和基本 IDS 预测值的损失
* $$v_t$$，A-IDS 维护的权重向量
    * $$v_t = (v_{t,1}, v_{t,2}, ..., v_{t,n})$$
    * $$v_{t,i} \geq 0$$
    * $$\sum_{i = 1}^n v_{t,i} = 1$$

#### 算法

* 参数
    * $$\eta > 0$$，学习率
    * $$0 \leq \alpha \leq 1$$
    * $$n$$
* 初始化
    * $$v_1 = v_0^m = (1/n, ...,  1/n, ..., 1/n)$$
* For $$t=1 \to T$$
    * 预测
        * $$\hat y_t = v_t \cdot x_t$$
        * $$pred(t)$$:
            * $$0$$, $$0 \leq \hat y_t \leq 0.5$$
            * $$1$$, $$\hat y_t \geq 0.5$$
    * *损失更新（Loss Update）*
        * 从 $$n$$ 个候选的基本 IDS 中找出最好的
        * $$L_{t, i} = (y_t - x_{t, i})^2$$
        * $$e^{-\eta L_{t,i}}$$, a factor
        * $$v_{t,i}^m = {v_{t,i} e^{-\eta L_{t,i}}} / {\sum_{j=1}^n v_{t,j} e^{-\eta L_{t,j}}}$$
    * *混合更新（Mixing Update）*
        * $$av_t = \frac{1}{t} \sum_{q=0}^{t-1} v_q^m$$，平均权重向量
        * $$v_{t+1} = \alpha av_t + (1-\alpha)v_t^m$$

## 实验

* 专家设置（没有混合更新）
    * 单一 IDS 无法检测到所有类型
        * 在有特定方面表现良好
        * 需要组合使用
    * 损失更新
        * $$\eta = 0.1$$
    * 没有混合更新
        * $$v_{t_1,i} = v_{t,i}^m$$
        * 组合不起作用
        * 表现结果等同于最好的 IDS
* 专家组合（A-IDS）
    * 模拟攻击环境
        * 数据随机洗牌
    * 10-folds 交叉验证
    * 混合算法
        * $$\eta = 0.1$$
        * $$\alpha = 0.001$$
        * 平均过去的更新
* 专家扩展（A-ExIDS）

## 局限

* 损失更新是双刃剑
    * 最好的 $$v_{t,i}$$ （$$L_{t,i} = 0$$）控制了$$v_{t,i}^m$$
    * 如果 IDS 暂时表现不佳，后来表现良好，却难以恢复
    * 结果
        * 对 IDS 性能变化适应缓慢
        * 攻击者可以持续变换攻击模式

## 参考资料

* Adaptive Intrusion Detection System via Online Learning, 2012
* CS 259D Session 10
