---
title: 理解LSTM
date: 2020-04-13 12:42:21
author: Riser
top: false
cover: false
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: false
summary: 本文帮助更好理解LSTM网络
categories: deep learning
tags:
  - LSTM
  - RNN
---

# 理解LSTM

转载翻译自 http://colah.github.io/posts/2015-08-Understanding-LSTMs/ 

### 循环神经网络（Recurrent Neural Networks）

人对一个问题的思考不会完全从头开始。比如你在阅读本片文章的时，你会根据之前理解过的信息来理解下面看到的文字。在理解当前文字的时候，你并不会忘记之前看过的文字，从头思考当前文字的含义。

传统的神经网络并不能做到这一点，这是在对这种序列信息（如语音）进行预测时的一个缺点。比如你想对电影中的每个片段去做事件分类，传统的神经网络是很难通过利用前面的事件信息来对后面事件进行分类。

而循环神经网络（下面简称RNNs）可以通过不停的将信息循环操作，保证信息持续存在，从而解决上述问题。RNNs如下图所示



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-ad37bd8b463c0f00.png)

可以看出A是一组神经网络（可以理解为一个网络的自循环），它的工作是不停的接收![x_{t}](https://math.jianshu.com/math?formula=x_{t})并且输出![h_{t}](https://math.jianshu.com/math?formula=h_{t})。从图中可以看出A允许将信息不停的再内部循环，这样使得它可以保证每一步的计算都保存以前的信息。

这样讲可能还是有点晕，更好的理解方式，也是很多文章的做法，将RNNs的自循环结构展开，像是将同一个网络复制并连成一条线的结构，将自身提取的信息传递给下一个继承者，如下图所示。



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-42172a6dae3d3388.png)

这种链式的结构揭示了RNNs与序列和列表类型的数据密切相关。好像他们生来就是为了处理序列类型数据的。

谁说不是呢！在过去的几年里，RNNs在语音识别、文字建模、翻译、字幕等领域有很成功的应用。在Andrej Karpathy写的博客 [The Unreasonable Effectiveness of Recurrent Neural Networks](https://links.jianshu.com/go?to=http%3A%2F%2Fkarpathy.github.io%2F2015%2F05%2F21%2Frnn-effectiveness%2F)中讨论了RNNs取得的惊人成果，这里就不详细讨论了。

很对成功的案例都有一个共性，就是都用了一种叫LSTMs的特殊的RNNs网络结构。下面就来看看什么是LSTMs。

### 长依赖存在的问题

从之前的描述可以看出来，RNNs理论上是可以将以前的信息与当前的任务进行连接，例如使用以前的视频帧来帮助网络理解当前帧。如果RNNs能做到这一点，那将会是非常的有用。但是他们能做到这点吗？答案是不一定。

有时候我们需要利用近期的信息来执行来处理当前的任务。例如，考虑用一个语言模型通过利用以前的文字信息来预测下一个文字。如果我们需要预测“the clouds are in the sky”这句话的最后一个字，我们不需要其他的信息，通过前面的语境就能知道最后一个字应该是sky。在这种情况下，相关信息与需要该信息的位置距离较近，RNNs能够学习利用以前的信息来对当前任务进行相应的操作。如下图所示通过输入的![x_{1}、x_{2}](https://math.jianshu.com/math?formula=x_{1}、x_{2})信息来预测出![h_{3}](https://math.jianshu.com/math?formula=h_{3})


![](https:////upload-images.jianshu.io/upload_images/6983308-bda6fb8e152bf765.png?imageMogr2/auto-orient/strip|imageView2/2/w/972/format/webp)



假设现在有个更为复杂的任务，考虑到下面这句话“I grew up in France… I speak fluent French.”，现在需要语言模型通过现有以前的文字信息预测该句话的最后一个字。通过以前文字语境可以预测出最后一个字是某种语言，但是要猜测出French，要根据之前的France语境。这样的任务，不同之前，因为这次的有用信息与需要进行处理信息的地方之间的距离较远，这样容易导致RNNs不能学习到有用的信息，最终推导的任务可能失败。如下图所示。



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-bc057d1b292d35f6.png)

理论上RNNs是能够处理这种“长依赖”问题的。通过调参来解决这种问题。但是在实践过程中RNNs无法学习到这种特征。[Hochreiter (1991) [German\]](https://links.jianshu.com/go?to=http%3A%2F%2Fpeople.idsia.ch%2F~juergen%2FSeppHochreiter1991ThesisAdvisorSchmidhuber.pdf) 和[Bengio, et al. (1994)](https://links.jianshu.com/go?to=http%3A%2F%2Fwww-dsi.ing.unifi.it%2F~paolo%2Fps%2Ftnn-94-gradient.pdf)深入研究过为什么RNNs没法学习到这种特征。

幸好LSTMs这种特殊的RNNs是没有这个问题的。

### LSTM 网络

Long Short Term Memory networks（以下简称LSTMs），一种特殊的RNN网络，该网络设计出来是为了解决长依赖问题。该网络由 [Hochreiter & Schmidhuber (1997)](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.bioinf.jku.at%2Fpublications%2Folder%2F2604.pdf)引入，并有许多人对其进行了改进和普及。他们的工作被用来解决了各种各样的问题，直到目前还被广泛应用。

所有循环神经网络都具有神经网络的重复模块链的形式。 在标准的RNN中，该重复模块将具有非常简单的结构，例如单个tanh层。标准的RNN网络如下图所示



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-2f0d4a87883d2c8c.png)

LSTMs也具有这种链式结构，但是它的重复单元不同于标准RNN网络里的单元只有一个网络层，它的内部有四个网络层。LSTMs的结构如下图所示。



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-169c41fa64ff202f.png)

在解释LSTMs的详细结构时先定义一下图中各个符号的含义，符号包括下面几种



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-f8c19513a9a46570.png)

图中黄色类似于CNN里的激活函数操作，粉色圆圈表示点操作，单箭头表示数据流向，箭头合并表示向量的合并（concat）操作，箭头分叉表示向量的拷贝操作

### LSTMs的核心思想

LSTMs的核心是细胞状态，用贯穿细胞的水平线表示。

细胞状态像传送带一样。它贯穿整个细胞却只有很少的分支，这样能保证信息不变的流过整个RNNs。细胞状态如下图所示



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-ebff76b101c09b28.png)

LSTM网络能通过一种被称为门的结构对细胞状态进行删除或者添加信息。

门能够有选择性的决定让哪些信息通过。其实门的结构很简单，就是一个sigmoid层和一个点乘操作的组合。如下图所示



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-d0c116d9de3d9660.png)

因为sigmoid层的输出是0-1的值，这代表有多少信息能够流过sigmoid层。0表示都不能通过，1表示都能通过。

一个LSTM里面包含三个门来控制细胞状态。

### 一步一步理解LSTM

前面提到LSTM由三个门来控制细胞状态，这三个门分别称为忘记门、输入门和输出门。下面一个一个的来讲述。

LSTM的第一步就是决定细胞状态需要丢弃哪些信息。这部分操作是通过一个称为忘记门的sigmoid单元来处理的。它通过查看![h_{t-1}](https://math.jianshu.com/math?formula=h_{t-1})和![x_{t}](https://math.jianshu.com/math?formula=x_%7Bt%7D)信息来输出一个0-1之间的向量，该向量里面的0-1值表示细胞状态![C_{t-1}](https://math.jianshu.com/math?formula=C_{t-1})中的哪些信息保留或丢弃多少。0表示不保留，1表示都保留。忘记门如下图所示。


![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-5fb98869b61eced2.png)



下一步是决定给细胞状态添加哪些新的信息。这一步又分为两个步骤，首先，利用![h_{t-1}](https://math.jianshu.com/math?formula=h_%7Bt-1%7D)和![x_{t}](https://math.jianshu.com/math?formula=x_%7Bt%7D)通过一个称为输入门的操作来决定更新哪些信息。然后利用![h_{t-1}](https://math.jianshu.com/math?formula=h_%7Bt-1%7D)和![x_{t}](https://math.jianshu.com/math?formula=x_%7Bt%7D)通过一个tanh层得到新的候选细胞信息![\tilde C_{t}](https://math.jianshu.com/math?formula=\tilde C_{t})，这些信息可能会被更新到细胞信息中。这两步描述如下图所示。


![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-43b42ce338d0566d.png)

下面将更新旧的细胞信息![C_{t-1}](https://math.jianshu.com/math?formula=C_%7Bt-1%7D)，变为新的细胞信息![C_{t}](https://math.jianshu.com/math?formula=C_{t})。更新的规则就是通过忘记门选择忘记旧细胞信息的一部分，通过输入门选择添加候选细胞信息![\tilde C_{t}](https://math.jianshu.com/math?formula=%5Ctilde%20C_%7Bt%7D)的一部分得到新的细胞信息![C_{t}](https://math.jianshu.com/math?formula=C_%7Bt%7D)。更新操作如下图所示


![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-cb48d627cc8df11e.png)



更新完细胞状态后需要根据输入的![h_{t-1}](https://math.jianshu.com/math?formula=h_%7Bt-1%7D)和![x_{t}](https://math.jianshu.com/math?formula=x_%7Bt%7D)来判断输出细胞的哪些状态特征，这里需要将输入经过一个称为输出门的sigmoid层得到判断条件，然后将细胞状态经过tanh层得到一个-1~1之间值的向量，该向量与输出门得到的判断条件相乘就得到了最终该RNN单元的输出。该步骤如下图所示


![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-977224dfe3f34477.png)

还是拿语言模型来举例说明，在预测动词形式的时候，我们需要通过输入的主语是单数还是复数来推断输出门输出的预测动词是单数形式还是复数形式。

### LSTM的变种

之前描述的LSTM结构是最为普通的。在实际的文章中LSTM的结构存在各种变式，虽然变化都不会太大，但是也值得一提。
 其中一个很受欢迎的变式由[Gers & Schmidhuber (2000)](https://links.jianshu.com/go?to=ftp%3A%2F%2Fftp.idsia.ch%2Fpub%2Fjuergen%2FTimeCount-IJCNN2000.pdf)提出，它在LSTM的结构中加入了“peephole connections.”结构，peephole connections结构的作用是允许各个门结构能够看到细胞信息，具体如下图所示。


![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-38089213ba42e38e.png)



上图描绘的是所有门都能看到细胞信息，还有一些变式是在其中的某些门引入细胞信息。

还有一种变式是在忘记门与输入门之间引入一个耦合。不同于之前的LSTM结构，忘记门和输入门是独立的，这个变式是在忘记门删除历史信息的位置加入新的信息，在加入新信息的位置删除旧信息。该结构如下图所示。



![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-31cce97beb8612f8.png)

一种比其他形式变化更为显著的LSTM变式是由 [Cho, et al. (2014)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1406.1078v3.pdf)提出的门循环单元（GRU）。它将忘记门和输入门合并成一个新的门，称为更新门。GRU还有一个门称为重置门。如下图所示


![](https://gitee.com//Riser_Wu/images/raw/master/img/6983308-f3d8a02ed1b8b24d.png)

*其中重置门为上图中前面那个门，决定了如何将新的输入信息与前面的记忆相结合。更新门为上图中后面那个门，定义了前面记忆保存到当前时间步的量。*由于该变式的简单有效，后来被广泛应用。

这里介绍的只是一些较为有名的LSTM变式，关于LSTM的变式其实还有很多种，像 [Yao, et al. (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1508.03790v2.pdf)提出的Depth Gated RNNs。还有其他用于解决长依赖问题的方法，如由 [Koutnik, et al. (2014)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1402.3511v1.pdf)提出的 Clockwork RNNs。

至于哪种变式效果最好？各种差异对LSTM的影响有多少？这些问题 [Greff, et al. (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1503.04069.pdf)做了一些对比，结论是他们基本是一样的。 [Jozefowicz, et al. (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Fjmlr.org%2Fproceedings%2Fpapers%2Fv37%2Fjozefowicz15.pdf)测试了一万多种RNN结构，发现在某些指定任务上有些变式还是由于标准LSTMs的。

### 总结

之前也提到过RNNs取得了不错的成绩，这些成绩很多是基于LSTMs来做的，说明LSTMs适用于大部分的序列场景应用。
 一般文章写法会堆一堆公式吓唬人，希望本文一步一步的拆分能有助于大家的理解。
 LSTMs对于RNNs的使用是一大进步。那么现在还有个问题，是否还有更大的进步？对于很多研究者来说，但是是肯定的，那就是attention的问世。attention的思想是让RNN在每一步挑选信息的时候都能从更大的信息集里面挑选出有用信息。例如，利用RNN模型为一帧图片生成字母，它将会选择图片有用的部分来得到有用的输入，从而生成有效的输出。事实上， [Xu, *et al.*(2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1502.03044v2.pdf) 已经这么做了，如果你想更深入的了解attention，这会是一个不错的开始。attention方向还有一些振奋人心的研究，但还有很多东西等待探索......

在RNN领域attention并不是唯一一个可以研究的点。比如[Kalchbrenner, *et al.* (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1507.01526v1.pdf)提出的Grid LSTMs，[Gregor, *et al.* (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1502.04623.pdf), [Chung, *et al.* (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1506.02216v3.pdf), 和 [Bayer & Osendorfer (2015)](https://links.jianshu.com/go?to=http%3A%2F%2Farxiv.org%2Fpdf%2F1411.7610v3.pdf)将RNNs用于生成模型的研究都非常有意思。
 在过去几年RNNs方面的研究非常的多，相信以后的研究成果也会更为丰富。