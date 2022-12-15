18年的东西，
特征还是通过ROIs提取，但相对于上篇论文,只提取到RGB没有进行映射到色度。
而是将一段视频转化成一张三通道图。
**特征提取过程是这样的：**
- 划分n个ROIs，每个ROI提取出RGB三个通道。
- 对于其中一个ROI 的 R 通道，
  - avgpooling后会变成一个值，将其映射到[0,255]上可以变成一个灰度值。
  - T个时间观测，可以变成T个灰度值，也就是一个灰度值反映的sequence，一个灰度长条
  - n个ROI的R通道，就可以变成一个灰度图
  - n个ROI的R,G,B通道，得到三个灰度图，也就是一张三通道的输入特征图（论文中的spatial-temporal map就是三通道分别映射至RGB的彩图，即所谓的时空表示）
  - Eventually, we get a spatial-temporal representation from the
raw RGB video sequence with the size of n ×T ×3 as the
input of our deep HR estimation network.**（一段视频转化成一张三通道图）**

只有600段视频用于训练和测试（就600张图）**数据集太小了。** 于是...<br>
*we propose an algorithm to generate synthetic heart rhythm to replicate the color changes caused by real heart rhythm.*<br>
我们提出了一种生成合成心律的算法，以复制真实心律引起的颜色变化
大批量地合成心率->合成时空图，引导网络区学习如何将RGB视频序列映射成心率值的先验知识，再进一步在自己的训练集上fine-tune。

