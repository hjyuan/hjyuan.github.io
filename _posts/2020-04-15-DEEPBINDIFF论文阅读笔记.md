---
layout:     post
title:      DEEPBINDIFF论文阅读笔记
subtitle:   二进制代码相似性检测
date:       2020-04-15
author:     Jie_Y
header-img: img/post-bg-deepbindiff.jpg
catalog: true
tags:
    - 相似性检测
    - 机器学习
    - 二进制
---

# DEEPBINDIFF: Learning Program-Wide Code Representations for Binary Diffing

原文链接：<https://www.ndss-symposium.org/wp-content/uploads/2020/02/24311.pdf>

论文出处：2020 NDSS Symposium会议论文

作者介绍：

1.  Yue
    Duan：加州大学河滨分校博士生，进入康奈尔大学成为博士后研究员，原西安交大本科生

2.  Xuezixiang Li：加州大学河滨分校在读博士生，硕士论文《Learning Program-Wide
    Code Representations for Binary Diffing》为本文原型

3.  Heng
    Yin：加州大学河滨分校计算机与工程系副教授，网络安全与隐私研究和教育中心(CRESP)主任，前三个作者导师，论文《Neural
    Network-based Graph Embedding for Cross-Platform Binary Code Similarity
    Detection》作者

## 一、论文概述

二进制相似性分析通过生成细粒度的基本块级匹配，然后计算两者的相似性得分来分析两个二进制文件之间的相似性，常用于恶意软件分析、安全补丁分析、剽窃检测和基于补丁的漏洞利用生成。现有的方法主要采用基于机器学习的方法进行相似性分析，但是传统的程序分析手段和基于机器学习的方法都存在准确性地、可伸缩性差、粒度粗糙等问题，并且基于机器学习的方法需要大量的训练数据进行神经网络训练。

本文主要通过利用无监督的程序代码表示学习技术解决上述问题，依靠代码语义信息和程序控制流信息生成基本块嵌入，并且采用k-HOP贪婪匹配算法利用基本块嵌入发现最优的相似性结果。作者实现了一个原型DEEPBINDIFF，通过大量二进制文件和真实的OpenSSL漏洞对原型进行评估，结果表明DEEPBINDIFF相比于最先进的工具，跨版本和交叉优化级别都更优。

## 二、背景知识

二进制相似性分析不仅能够提供整个二进制规模相似的精确、细粒度定量的结果，而且揭示代码如何在不同版本和优化级别上发展的。现有的二进制相似性分析包括两种类型。

传统方法包括静态分析和动态分析方法。静态分析方面，BinDiff采用调用图和控制流图执行多对对图同构检测，并利用启发式方法匹配函数和基本块。其他的方法采用对生成的控制和数据流图进行匹配或者将图分解为片段进行相似性检测。而采用的图匹配算法(如Hungarian算法)开销巨大，不能保证最佳匹配。这些方法大多只考虑指令语法，而忽略了语义信息。动态分析方面，通过执行代码在给定的二进制程序上进行动态切片或污染并基于执行期间收集的信息检查语义级别的相似性进行分析。这些方法能够提取语义信息并且对编译器优化和代码混淆具有良好的弹性。但是由于动态分析的性质可能会遇到伸缩性差和代码覆盖范围不完整的问题。

![deepbindiff-1.png](https://i.loli.net/2020/08/10/wOh4SXd3c6H9glv.png)

目前基于机器学习的方法也相继被提出，其利用图表示学习技术并将代码信息合并到嵌入(高维数值向量)中，使用嵌入进行相似性检测。InnerEye和Asm2Vec依靠NLP技术自动提取语义信息并生成用于区分的嵌入。机器学习的方法具有更高的准确性和更好的伸缩性并且可以加速学习过程。

![](https://i.loli.net/2020/08/10/Hd7sTVUQDiIzbmY.png)

但是基于学习表示技术的二进制相似性比较存在三个主要的局限性。

-   现有的基于学习的技术都无法在细粒度的基本块级别上执行有效的全程序二进制比较，当前大多数技术采用函数粒度进行比较，会严重影响性能。

-   没有一种基于学习的技术同时考虑程序范围内的依赖信息和基本块语义信息。依赖项信息可以从过程间控制流图(ICFG)中提取，表示基本块的上下文信息，而基本块语义信息表征每个基本块的唯一性。

-   大多数基于学习的技术都建立在监督学习之上，性能很大程度上取决于训练数据的质量，并且监督学习可能会遭受过度拟合的问题。先进的监督学习技术InnerEye将整体指令视为一个单词，可能会导致out-of-vocabulary问题。

## 三、设计与实现

论文首先将二进制相似性问题定义为：

给定两个二进制程序*p*1=(*B*1*，E*1)和*p*2=(*B*2*，E*2)，二进制差分的目的是找到使*p*1和*p*2之间的相似性最大化的最佳基本块匹配：

![](https://i.loli.net/2020/08/10/WmQJpNVowFURtjy.png)

其中：

-   *B*1={*b*1，*b*2，...，*bn*}和*B*2={*b*1’，*b*2’，...，*bm*’}是*p*1和*p*2中包含所有基本块的两组集合；

-   *E*⊆*B*×*B中的*每个元素*e*相当于两个基本块之间的*控制流依赖*;

-   在*M*(*p*1*，p*2)中的每个元素*mi*表示*bi*和*bi’*之间的匹配对;

-   *sim*(*mi*)定义了两个匹配的基本块之间的定量相似度得分。

因此，该问题可以转化成两个子任务：1)发现*sim*(*mi*)定量地测量两个基本块之间的相似性;
2)找到两组基本块*M*(*p*1*，p*2)之间的最佳匹配。

DEEPBINDIFF架构如下所示，红色块表示中间过程生成的中间数据。系统将两个二进制文件作为输入，并输出基本块级别的相似性结果。首先DEEPBINDIFF利用无监督学习方法学习嵌入，利用嵌入计算基本块之间的相似性得分*sim*(*mi*)。对于这一步骤，首先修改Word2Vec算法提取令牌(操作码和操作数)的语义信息并生成基本块级特征向量。然后将基本块嵌入生成模型建模为网络表示学习问题，将特征向量输入到Text-associated
DeepWalk算法(TADW)中生成包含程序范围控制流上下文信息的基本块嵌入。最后系统使用k-HOP贪婪匹配算法匹配基本块*M*(*p*1*，p*2)。

![](https://i.loli.net/2020/08/10/F9Gskt1BbAOvMaR.png)

整个系统由三个主要部分组成：1)预处理；2)嵌入生成；3)代码比较。预处理包括两个组件：CFG生成和特征矢量生成，生成过程间控制流图(ICFG)和基本块的特征向量。然后将生成的结果发送到嵌入生成组件，该组件利用TADW技术生成基本块嵌入，然后DEEPBINDIFF利用生成的基本块嵌入执行k-HOP贪婪匹配算法进行代码比较。

### 1、预处理

对于CFG生成，通过将调用图与每个函数的控制流图结合，DEEPBINDIFF利用IDA
pro提取基本块信息，并生成提供全程序上下文信息的过程间CFG(ICFG)，其携带控制相关信息。

对于特征向量生成，DEEPBINDIFF首先训练从Word2Vec算法派生的令牌嵌入模型，然后使用该模型生成令牌(操作码和操作数)嵌入，然后根据令牌嵌入生成特征向量。过程如下：

![](https://i.loli.net/2020/08/10/uHqyX3kPxiZsJQd.png)

**随机游走**
对ICFG进行随机游走生成序列化代码以便生成可能的二进制执行路径，利用随机游走提取控制流依赖信息。

**归一化** 对序列化代码随机游走进行归一化。

**模型训练**
将随机游走视为语句，通过令牌嵌入模型训练归一化的随机游走来学习令牌嵌入。利用改进的Word2Vec
CBOW模型训练令牌嵌入生成模型，将令牌(操作码和操作数)视为单词，在ICFG上的归一化随机游走作为围绕令牌的语句和指令，作为目标词周围的上下文。

连续词袋模型(CBOW)生成单词矢量表示，使平均对数概率J(w)最大化。

![](https://i.loli.net/2020/08/10/ik2yngbOlLoKVcm.png)

*p*(*wt* +*j*\|*wt*)是softmax函数：

![](https://i.loli.net/2020/08/10/YRofPa2h5ed9L1y.png)

**特征向量生成**
由于每个基本块可能包含多条指令，并且每条指令依次包含一个操作码和可能的多个操作数，DEEPBINDIFF计算操作数嵌入的平均值，并于操作码嵌入连接以生成指令嵌入，然后对块内的指令进行求和以形成特征向量。

由于指令在相似性方面的重要性不同，文章采用一种根据TF-IDF模型的操作码重要性的加权策略来调整操作码的权重。在生成指令嵌入时，对其建模为两项串联：操作码嵌入*embedpi*与其TF-IDF权重*weightpi*的乘积和操作数嵌入*embedtin*的平均值。对指令嵌入进行求和得到特征向量。

![](https://i.loli.net/2020/08/10/dxXru31QwRTOvUi.png)

### 2、嵌入生成

DEEPBINDIFF首先将两个ICFG合并为一个图，然后将该问题建模为网络表示学习问题，使用Text-associated
DeepWalk算法(TADW)生成基本块嵌入。

对于两个ICFG合并问题，DEEPBINDIFF提取字符串引用并检测外部库调用和系统调用。然后，它为字符串和库函数创建虚拟节点，并绘制从调用点到这些虚拟节点的边。因此，两个ICFG在终端虚拟节点上合并为一个。

Text-associated
DeepWalk是一种无监督的图嵌入学习技术，是对DeepWalk算法的改进。下图表明TADW算法可以将矩阵*M*分解成三个矩阵的乘积：*W*∈*Rk*×\|*v*\|、*H*∈*Rk*×*f*和文本特征*T*∈*Rf*×\|*v*\|。然后，将*W*与*HT相连*以生成2*k*维的顶点表示(嵌入)。

![](https://i.loli.net/2020/08/10/YuK1VyRzBe5b9r8.png)

DeepWalk算法是一种在线图嵌入学习算法，将一组截断的短随机游动作为语言建模问题的语料库，将图顶点作为词汇表，然后使用图顶点上的随机游走来学习嵌入。因此共享的相似邻居的顶点具有相似的嵌入，但是其在分析过程中没有考虑节点特征。而Text-associated
DeepWalk算法能够将顶点特征合并到网络表示学习过程中。DeepWalk相当于因式分解矩阵*M*∈*R*\|*v*\|×\|*v*\|其中每一项*Mij*是顶点*vi*按固定步长随机行走到顶点*vj*的平均概率的对数。

在合并ICFG图后，DEEPBINDIFF将合并后的图和基本块特征向量馈送到TADW中进行多次优化迭代。该算法通过使用交替最小二乘(ALS)算法最小化损失函数将矩阵M分解为三个矩阵，当损失函数收敛或在固定的n次迭代之后停止。

![](https://i.loli.net/2020/08/10/duXtMyvIN83BPeD.png)

### 3、代码相似性比较

采用k-HOP贪婪匹配算法，从ICFG上下文信息中受益，根据从已匹配的k-HOP邻居中的基本块嵌入计算相似度来找到匹配的基本块。如算法所示，在图合并期间通过使用虚拟节点来计算初始匹配集*Setinitial*，然后提取虚拟节点的直接邻居并根据嵌入在邻居之间找到最佳匹配对。然后通过探索合并的ICFG来循环探索已匹配成对的邻居，然后对相邻基本块之间的相似性进行排序选择相似度最高的匹配对。

![](https://i.loli.net/2020/08/10/HxwANE8Y6VtPJXu.png)

## 四、实验评估

论文使用了三种二进制集——Coreutils，Diffutils和Findutils，总共有113个二进制文件。使用GCC
v5.4将它们编译为4种不同的编译器优化级别(O0，O1，O2和O3)，以生成配备有不同优化技术的二进制文件。并从Github上收集了开源C++项目LSHBOX和indicators，包含大量的虚函数。

实验数据集和代码已在Github上给出(<https://github.com/deepbindiff/DeepBinDiff>)。

跨版本比较的精度和召回率：

![](https://i.loli.net/2020/08/10/SWdvOuDKj36i8ty.png)

交叉优化比较的精度和召回率：

![](https://i.loli.net/2020/08/10/guk3YvZQ9UXf8qH.png)

与INNEREYE+k-HOP进行效率比较：

![](https://i.loli.net/2020/08/10/ByYrJbsQLKjHPtT.png)

## 五、总结评价

文章提出了一种基于无监督的代码表示学习技术，利用了令牌嵌入和过程间CFG，生成特征矢量进行二进制相似性比较。DEEPBINDIFF能够处理多种常见的优化技术，能够同时考虑控制流信息和上下文语义信息，实现了多版本和多优化级别的二进制相似性比较。但是它不能对编译器为减少分支错误预测而合并的块进行正确的分类处理；控制流优化的大幅度改变也会影响有效性；不支持跨架构相似性比较。

## 六、心得与启发

本文利用无监督的机器学习方法进行相似性比较，而腾讯科恩实验室采用语义感知神经网络进行相似性比较，并且采用了NLP技术中较新的BERT算法，这两个最新技术的效率和准确度方面孰优孰劣。根据论文中所提的技术，对于深度学习方面知识欠缺较多，无法进行相应的思考。从论文的局限性来说，是否可以将二进制文件表示成中间代码表示然后利用相同的方法进行跨架构研究。现有的技术大都不能解决大幅度改变控制流图的混淆技术，怎样解决这个问题，留待进一步研究。

## 参考

Duan Y, Li X, Wang J, et al. DEEPBINDIFF: Learning Program-Wide Code Representations for Binary Diffing[J].
