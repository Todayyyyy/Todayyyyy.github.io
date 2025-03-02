---
title: MIDAS M32 DN32-LIVE调音台直播链路
---



作者：段恺乐



# 一、路由详解

先对不同接口、通道的具体设置进行讲解。

## Inputs

Inputs通道的routing思路是，选择inputs通道的sources，它们来自于local in以及其他通道接口的out。

1-24：Card 1-24；

也就是说，将机架的输出通道以及电脑音乐输入到了调音台的Inputs通道上，机架或CARD上的信号从哪儿来，稍后会进行解释。

Inputs通道的source主要分类为：麦克风（LEAD、HS）以及电脑音频（音乐、音效等）；

## MIXBUS

mixbus的功能分为：PA、OB、Back、FX、SUB；

然后又会通过HS、LEAD两种不同的麦克风分别汇总到不同的总线上；

## Matrix

从母线的职能层级来说，matrix＞main L/R＞mixbus、group、aux等；

matrix输出，从功能上将，分为OB、Main L/R、S L/R；

因此，matrix作为主要的输出通道。

## OUTPUTS

M32调音台上，[BLOCK]一栏中的路由，均是选择的in（输入）信号，不是out（输出）信号；

而outputs，是M32调音台上虚拟的（virtual、digital）输出通道，每个可以选择自己通道的信号来源，也可以自由的assign给其他out通道使用。

在大型直播商务中，outputs按功能分为主输出（main、side、sub）、BackUP、EF、OB等类型。

信号来源根据功能需求，大多来自于mixbus或者matrix。

为了方便处理，将OUTPUTS中1-8作为主输出，其他功能的放置在9-16通道中。具体原因会在稍后解释。

## CARD

在直播处理中，我将CARD out的routing理解为进入电脑机架的声音来源（source）。

1-8：outputs 9-16；用于处理9-16中的effect bus，进行特殊化处理。（因此将EF等其他mixbus集中发送给OUTPUTS 9-16中才方便传输到电脑机架中。）

9-16：local 9-16；麦克风接入。

17-24：card 1-8；用于传输电脑的音乐。

25-32：local 25-32；麦克风接入。

（PS：local指的是本地XLR in）

## XLR

均选择outputs通道。

## AUX in

1-6：user in 1-6；

其中，user in 1-6的信号来自于card in 25-32中的信号。

最后，aux in被发送至了mixbus中。

应该是将处理后的特殊效果或者音乐发送给了aux。



# 实现功能

## 1.监听

有的岗位需要实时监听直播间的音频情况，因此需要将信号汇总到到一个监听母线并进行输出。

## 2.输出

直播现场、直播间的音频需要分开输出，并且直播现场要按照现场的声音制作规格进行分频段发送输出。因此需要2-3个输出母线用于信号的输出。

## 3.通讯

可以通过麦克风进行内部的交流，不过在后勤工作人员之间，用三方内通还是更为普遍的选择。



> ![mix_console](image/Works/mix_console.jpg)
>
> ![a61e3fcf952de401320980c0e72288c](image/Works/mix_console2.jpg)