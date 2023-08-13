# Bev-Joint
将每一帧的bev矢量图拼接起来（实习项目）
> 由于修改的是实习公司的代码，因此不放出具体代码，仅放开我完成的效果（基本都是自己完成）
## 具体效果
如下视频中演示效果：
[![](https://bb-embed.herokuapp.com/embed?v=BV1jS4y1w7SW)](https://player.bilibili.com/player.html?aid=274701012&bvid=BV16F411Z7jX&cid=1233046265&page=1)
## 项目说明
总体思想很简单，就是将每一帧的BEV矢量图，根据每个矢量属于不同实例，将其拼接（目前只是使用**nuScenes数据集**的真实位姿进行推算，之后可以在此真实位姿的基础上添加扰动测试）。
想法很简单，实现起来还是困难重重。具体的实现流程：
* 使用MapTr提供的开源代码进行bev矢量图的建立（使用环视6相机的图片数据，通过学习方法得到bev空间下的矢量信息）
* * 数据集使用 nuScenes，由于nuScenes数据集皆为片段式且少有连续，因此根据nuScenes数据集提供的Token以及每个片段的时间戳，使用python脚本找到其中连续时间最长的8个片段进行拼接。再使用MapTr进行矢量图预测。
  * MapTr进行的矢量图预测，通过修改MapTr的输出部分代码，将输出转为Json文件，存放这每一帧每一个矢量数据点信息
* 将BEV视角下每一帧的矢量信息进行拼接
* * 匹配过程：由于矢量信息只有一个多段折线段，因此最终选择使用折线段扩充之后的图形，计算**IOU进行匹配**（使用Boost中的geometry进行扩充与计算）
  * 滤波过程：使用**Douglas–Peucker算法**，对折线段进行滤波
  * 拟合过程：匹配的两个折线段需要进行拟合才能最终形成一条折线段，拟合过程我使用了三种方法，都无法达到我理想的状态，**暂时选择了简单粗暴的取中点的方式**（后续会适当修改）：
  * * **取中点**：将匹配的两个折线段，选择合适的截取点（因为当前帧的折线段只是匹配上一帧折线段的一部分），然后对截取点之后的两个折线段，互相取点的中点，作为拟合点（对于截取点的判断很复杂，因为匹配的折线段上的点并不是一一对应的）
    * **插值**：进行插值的前提下，这两个折线段必须已经进行了插入合并成了一条折线段，才能进行插值，那么在没有准确位姿的情况下，这种效果并不会好
    * **多项式拟合**：较为理想的方法，但是我实现的效果并不好（可能是我代码写的问题）
    * * 难点1：多项式拟合中，这个多项式次数并不好确定， 次数小了拟合效果不够，次数大了容易出现龙格现象
      * 难点2：矢量折线段的形状千奇百怪，很容易出现一对多，多对一（就是一个x对应好几个y）的情况，那么这种拟合方法是否能够拟合（好想还是多项式次数选择的问题。。。。）
> 整个项目最不好处理的就是拟合部分，其实最近在想这样一个问题：拟合其实代表着这个点在该坐标下有较高的权重，导致拟合曲线会趋向于该点，但是矢量信息的点并不代表这个点的权重比较大，只是代表着这个点是该实例上的一个点，并且该点与其他点合并成为了该实例对象。那么拟合的方法是否不太合理？？这个拟合方法是否可以使用一些**栅格地图**的一些想法？--->这只是我的一点愚见     
