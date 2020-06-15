# 2 布局规划

## 流程简介

​	芯片设计进入深亚微米时代后，Floorplan变得极为重要，它将影响到芯片的面积、速度、信号完整性和设计周期。一个好的Floorplan，不仅能获得更好的QoR ( Quality of Result )，也将大大减少后续布局布线以及时序收敛所耗费的时间和精力。所以后端设计上流程中的反复主要是发生在这一步。如果这一步做得比较好，则后面一次通过的几率就比较高，反之如果回溯到这一步，则花费的时间开销就会很大。因此可以说，做好Floorplan设计，芯片设计就成功了一半。

Floorplan的主要目的是为模块、I/O接口、电源焊盘分配相对的位置，并定义时钟和电源分配。在Floorplan之前我们需要知道每个模块的门级电路尺寸和运行频率，以及模块之间的连接关系，这样才能设计出更好的Floorplan。同时也不能等到网表完全确定后才开始这一步，Floorplan是确定芯片面积、线长、线拥塞情况的决定要素，现代EDA工具在Floorplan阶段就可以估计线延迟。因此，实际项目中，往往是前端逻辑初步定义好之后就要开始尝试Floorplan，这样可以发现一些逻辑设计阶段不能发现的问题。对于面积很大的ASIC芯片物理设计，往往采用层次化设计方案，通过顶层设计规划、子模块划分和实现，以及芯片顶层组装实现。对于顶层划分，往往综合考虑设计层次，连接关系以及面积，将整个设计划分成多个子模块，然后再对每个子模块分别进行物理设计，而顶层就可以将这些模块当成仅有IO口的黑盒子宏模块进行物理设计。

Floorplan的主要内容：

- 确定芯片的尺寸
- 标准单元的排列形式
- IO单元及宏单元的位置
- 电源地网络的分布

推荐Floorplan按照以下步骤进行：



#### 1. 在设计中添加physical only cells

#### 2. 读入IO约束文件

#### 3. 创建Floorplan

#### 4. 加入Pad filler

#### 5. 宏单元放置

#### 6. 布局障碍的放置

#### 7.添加EndCAP

#### 8. 添加N well和衬底接触单元

#### 9. 电源地规划

#### 10. 自动做floorplan的placement，作为floorplan的参考



之后逐一进行详细介绍。



## 2.1 在设计中添加physical only cell

#### 1.在设计中添加physicalonly cells

Physical only cells是那些在网表中没有，而在实际芯片中需要存在的一些单元，如电源地IO、给IO供电的IO以及一些衬底、阱接触单元等。

在Layout Window的菜单栏中依次选择“ECO”⟶“CreateCell”。

1）添加IO Corner

IO Corner的作用是连接芯片拐角处两边的IO Pad，连接衬底以及衬底以上的各个层，使得IO Pad内部的电路形成一个电源地的供电环路。同时也使它们的衬底、阱等各个层连续，不至于出现DRC的违反，如图1所示：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtmb7NObWMpLqNeYje8yML6nbLe4NIUhFaDe0tEt464neuN3vPNyHvuMeYmP2jLZfwUYpWOd8hDDSQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 IO Corner连接相邻两边IO的衬底以及衬底以上的各个层

只要在“Reference cell”后边选择相应的参考单元并在“Newcell names”中填写创建的Corner的名字，点击OK即可，如图2所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmb7NObWMpLqNeYje8yML6nkiabwjIv2a7jvYpKia5qEf0OyzGPrvy3uwNRTRr041ou7tcEaBUnNSfQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 添加IO Corner

2）添加为Core供电的IO（VDD VSS）

3）添加为IO供电的IO（VDD_IO VSS_IO）

**注意，IO Corner是不用围成环的，甚至IO Corner都不是必须的！但是给Core和IO供电的IO是必须的。**当芯片只有一边有IO时，或者只有芯片的对边有IO时，此时为了节省芯片面积，不需要IO Corner。如图3所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmb7NObWMpLqNeYje8yML6nk5rhCF9QEEcQeMo4BajVkac6HVI9mn1DfwRk27ypqpcszoeCX3WhEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmb7NObWMpLqNeYje8yML6nOaEgF1O39aK5ib3LuMrLjyxgl3IpPWV7gVomTIuVNjcSAKDNd4iaVV2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 芯片1边和2边（对边）有IO的情况，此时不需要IO Corner

但是对于含IO的边都需要添加一组VDD_IO和VSS_IO来给IO供电。对于右边那种情况，如果用IOCorner将其连接起来的话（面积牺牲比较大，一般也不会这样做），在不考虑IO上的功耗的情况下，只需要一组VDD_IO和VSS_IO就可以了，而非两组。

对于相邻两边含IO或者三、四边含IO的情况，一般在拐角处都会加上IO Corner，如图4所示：![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtmb7NObWMpLqNeYje8yML6nEwPLYBvTMb4xjiazzoC4NOqaeOBcNic9eIY7G1P2xO78mibdtHvuW3gxA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 芯片2边（相邻），3边和4遍有IO的情况需要IO Corner

这里以4边都含IO的设计为例进行讲解，因此四个拐角处都需要添加IOCorner，如图5所示：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtmb7NObWMpLqNeYje8yML6njDlaVPsmicVxPVjNgmiaoyqSBE7olXgwK8gUiaUo91LWxsA2ZsGFnYnvw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 最常见的芯片4边都有IO的情况

步骤1）2）3）的tcl命令为：

\> create_cell {CORNER1 CORNER2CORNER3 CORNER4} {PCORNERRN}

\> create_cell {VDD} PVDD1RN

\> create_cell {VSS} PVSS1RN

\> create_cell {VDD_IO} PVDD2RN

\> create_cell {VSS_IO} PVSS2RN

该例子中分别给Core和IO只添加了一组供电的IO，但是在实际项目中，需要考虑Core的功耗以及IR-drop来决定需要几组VDD和VSS以及它们的分布情况。同样需要考虑IO的功耗以及IO上的电压降来决定需要VDD_IO和VSS_IO以及它们的分布情况。



## 2.2 读入IO约束文件（.tdf文件）

在设计中创建完Physical Only Cells之后，便可以读入管脚约束文件了，它指定了每个IO在整个芯片中的位置和排列顺序，对于Block Level的设计而言，因为设计中没有IO，所以可以读入pin顺序和位置的tdf文件。

对于Chip level的设计而言，该文件主要用来定义设计中所有IO以及IOCorner的位置（上下左右的方位以及排列顺序，也可以定义具体的坐标），对于Block level的设计而言，它用来指定所有pin的位置和所用的metal的层次。

该文件可以参照ICC中的set_pad_physical_constraints命令的文件语法进行编写，也可以在ICC中导出一个该文件进行手工修改。下面简单介绍下该命令：

其部分语法为：

set_pad_physical_constraints

​        objects |-pad_name stringpad_name

​        [-sideside_number]

​        [-order order_number]

例如：set_pad_physical_constraints -pad_name clk_block -side 1 -order 1

该命令定义了一个名为clk_block的IO PAD的位置，该名称必须出现在前面所述的门级网表中，否则的话需要在ICC中手动创建该单元，例如网表中没有的电源地、给IO供电的IO需要在后续的11.3.2布局规划阶段进行添加。

Side_number指定了IO PAD摆放在哪条边，其中从左边数，顺时针方向依次为1、2、3、4，即1代表左边，2代表上边，3代表右边，4代表下边。默认该选项设置为0，即代表没有摆在指定哪条边的限制。

Order_number指定IO PAD的排列顺序，数字从1开始，越大越往右/上方摆放。默认该选项也是为0，意思是没有排列顺序的限制。如图1所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtllE8xGAeu2BnOp0OlceT8CR9p56JWmA67SemSTHcvORq1W0thnbdZkeQBGU5T1ArDC6pfEezKXsg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 tdf管脚约束

 

该文件是后端设计中为数不多需要手工编写的，在chip的IO数目非常多的情况下，编写该文件便需要面临以下几个问题：

​	2.1 容易出错，当IO数目超过上百个时，手工编写便极其容易出错，容易出现漏或者重复的情况；

​	2.2 在某些情况下，需要指定IO的具体坐标，而坐标是需要计算的，如果手工来计算的话太麻烦，且容易出错，如果用tcl脚本完成的话，只用通过简单的迭代计算就能将所有IO的坐标计算出来，最终体现在tdf文件里面；

​	2.3 在纳米工艺中，IO和PAD往往是分离的，IO的摆放还稍微容易一点，可是PAD的摆放却不容易。为了减小IO和PAD的面积，多采用Stagger形式的IO和PAD，IO一般放在PAD的下方，即DUP（Design Under Pad）形式，这样PAD的摆放会比较困难，还需要和IOalign到一起（在有的工艺下，IO的宽度还是不等宽的，这样就增大了Align的难度），如图2所示：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtkkjfmfNu9F2ib9eebxZweAq43aoEYfF6NZJNic51XtGoh6icKnJfVY8x3ASqtkAhTJr5DnXaia1oOE9g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 Stagger结构DPU(Design Under Pad)的IO和PAD



而tdf文件是决定IO的排布的，如果分开来摆放IO和PAD的话步骤会太过繁琐，其实可以通过写一个spec，并写一个通用的脚本来完成tdf文件的生成和PAD的摆放。具体Spec的编写和tcl实现脚本可以看推文：

[28nm工艺下，自动生成管脚排列文件，给设计加PAD，并在PAD上面打Label的流程（含脚本）](http://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247484089&idx=2&sn=e68505780cf5d59be5d4979901693426&chksm=e8292677df5eaf61e757801b9f28088bbda5b094a2db7e8f08d33d406a8c4b2a4be2b9297d9f&scene=21#wechat_redirect)



**操作：**在LayoutWindow的菜单栏中依次选择“Floorplan”⟶“ReadPin/Pad Physical Constraints”，在栏中填写前边准备好的tdf文件，然后点击OK。

相应的命令为：read_pin_pad_physical_constraints /xxx/data/main_pad.tdf



## 2.3 创建Floorplan

在读入管脚约束文件（.tdf文件）后，可以创建Floorplan得到芯片大概的物理形状和尺寸，在这一步执行完毕之后，IO或者pin就会按照前面的约束进行摆放。

**操作：**在LayoutWindow的菜单栏中依次选择“Floorplan”→“Create Floorplan”,弹出如图1所示的框。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtllE8xGAeu2BnOp0OlceT8Cc3ap2clN34MPdfonsdhMPcjAyvbq60UEjnkB07WWQ3RMCgicbXLm6mg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

图1 创建Floorplan

在ICC中有三种布局规划控制方案，其中：

1）Aspect ratio：

这种是指定芯片高度和宽度比值的方案，可以设置Core的利用率。

当设计中不含Macro时多种这种方案，在最开始Trail的时候一般设置较小的利用率，看Timing、DRC、LVS等结果如何，此时用时较短，能在短时间内给设计者一个参考，让设计者对Floorplan方案进行评估。如果设计的问题不大，那么可以逐渐提高利用率，减小芯片的面积。

2）Width/Height

指定高度和宽度，一般对于含有Macro的设计，多用这种方案。

相应的命令为：

\> create_floorplan [-options]

创建完Floorplan之后需要移除terminal，命令为

\> remove_terminal *

该操作前后的效果对比如图2所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtllE8xGAeu2BnOp0OlceT8CSgnKISdruLP4yibM878eHntdZXYjV3Y3xBeakTGU7GK0UaRarqibnRCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

(a) 创建Floorplan前

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtllE8xGAeu2BnOp0OlceT8CyKG00CCE0LFxuhuVw7bUyxiaIvNCeUgCvnN5nLF7zxicWo5goDMJY7Xg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

(b)创建Floorplan后
图2 创建Floorplan前后版图的变化

从图中可以看出，创建Floorplan之前，所有的IO、宏单元以及标准单元都是叠放在一起的；Floorplan之后，所有的IO都已经摆放正确，且所有的宏单元都随机排列在芯片上方，所有的标准单元都排列在芯片右方。



## 2.4 加入Pad filler

​	前面讲解了IO Corner的作用，和它的作用相似，加Padfiller是为了连接两个两邻的IO pad，实现从衬底到各层金属的相连，为IO内部电路形成电源地供电环路。**注意：**和IO Corner一样，IO Filler也不是必须的，在打线和封装允许的情况下，可以将IO与IO挨着放置，中间不留空隙，不插入IOFiller（**即，在打线和封装能够实现的情况下（需要咨询封装厂），允许****IO****与IO****挨着放置，这样也不会出现DRC****和LVS****的违反。不会出现DRC****违反的意思是，即使IO****挨着放，相邻两个IO****内部所有层次的间距都不会有DRC****违反！不会出现LVS****违反的意思是，即使IO****挨着放，相邻两个IO****内部所有层次都不会出现短路！**）。除此之外（也就是IO和IO中间有空隙），如果IO中间没有插入IO Filler的话，可能会出现供电问题以及DRC的违反。

**操作：**在LayoutWindow的菜单栏中依次选择“Finishing”⟶“Insert Pad Filler”，在“Pin/Blockage cells”框中填写Pad Filler的Cell名。“Pin/Blockage overlap cells”在框中填写那些尺寸小的允许交叠的Pad Filler的名字，因为某些情况下不允许交叠的话可能这些Pad没法连成一个环，一般这些Pad Filler的宽度小于1 μm。例如某个PadFiller的名字是PFILL10RN，则后边的数字一般代表了该Filler的宽度，也就是说这个Filler宽10 μm，PFILL01RN代表它的宽度为0.1μm。在填写时要注意：一定要按照从大到小的顺序进行填写，因为软件会按照从前到后的顺序插入PadFiller。

按照上面的表述，在Pin/Blockagecells中依次填写PFILL20RN PFILL10RNPFILL5RN PFILL2RN PFILL1RN PFILL01RN PFILL001RN，在Pin/Blockage overlap cells中依次填写PFILL01RN PFILL001RN，Boundary placement中勾选TopBottom Left Right，点击OK即可，如图1所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmict0OC5Whg4PCyEsADhd3nNCZYiccfwHGkWhCwH5Q2MStrB0ggYBDsKojr68icI3PufpbBKxlEA0JQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 ICC中插入Pad Filler

相应的命令为：

\> insert_pad_filler -cell{PFILL20RN PFILL10RN PFILL5RN PFILL2RN PFILL1RN PFILL01RN PFILL001RN}-overlap_cell {PFILL01RN PFILL001RN}



## 2.5 宏单元的放置


对于Floorplan而言，人们更倾向于靠自己的经验去摆放Macro，摆放它们时不仅要考虑面积、互联线长等传统问题，还需要考虑Place阶段，Macro的摆放对于Place的影响。因为Macro从本质上讲就是一个巨型的标准单元，很多Macro也存在于各个模块内，在Floorplan阶段并不能和模块对等的考虑。对于这个问题，人们根据实际生活中的经验，提出一种边缘摆放**（****edgeplace****）**的方法。

边缘摆放的好处主要来源于下面两点：

1）从目前芯片设计的趋势来看，芯片中除了计算单元外就是随机存储单元RAM、只读存储单元ROM等。这些存储单元占据的芯片面积在有些设计中甚至超过百分之五十。对于存储单元来说，存在数据端口和存储端口，并且周围需要有一些可测性电路。这使得这些单元引线众多且功耗巨大。将它们贴边放置，不仅有利于这些单元的供电，而且防止这些单元过多的引脚对其他单元的布线造成影响。

2）标准单元在布局时，按照Row所划定的高度一排一排的摆放，这样既有利于算法的设计，又有利于工业制造。并且在给各个器件供电时，可以使用横向的电源线将处于同一高度的器件连接在一起统一供电。若是将标准单元都摆放在芯片区域的中心，而大的Macro摆放在四周，就可以使标准单元方便的只用一条电源线连接在一起，而不会被高度不统一的Macro打断。对电源网格的设计提供了巨大便利。

而Macro的摆放原则基本如下，可以参照下面这张图：



![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmict0OC5Whg4PCyEsADhd3nk74oucjU1cicIRDvKJTuxD0SqteygNt6vNszYTQwc7OULEibyFHxlF6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图1 Macro摆放原则

1）模块尽量摆放在靠近相应输入输出口（IOpad）的位置。一般来说对于大型的Macro，他们不仅仅需要与芯片内部的其他Macro或者标准单元进行数据交换，还需要与芯片外部的器件进行通信。比如，锁相环单元需要接收外部晶振信号，存储单元需要接收外部地址等。这种数据交换就是靠IOpad进行的，因此摆放在离相应的数据端口附近，有利于减少互联线长度，减少线上延迟，并节约布线资源。

2）大的Macro摆放尽量贴近版图的边缘和角落，这样有利于空间的利用，**要尽量留出一个连续且尽量接近圆形****/****方形的Core****区域来摆放标准单元**。

3）Macro与Macro之间要留有一定空隙，给予布线资源。特别是在Macro的间隙有端口的时候更是如此，**设计者可以通过相邻****Marco****边界上端口的多少来决定留有多大的间隙比较合适，这样才不至于出现Pin Access****的问题**。在使用EDA软件的Floorplan设计时，同样可以给Macro加上晕环（halo）来控制Macro与Macro之间的距离。在ICC中这被成为Keepout Margin，有hard和soft之分，hard区域不允许任何Cell放置在该区域，soft则在coarse place的时候不允许任何Cell放入其内，但是在optimization以及legalization的时候是允许Cell放入其内的，也就是只允许Buffer加入其中。Keepout Margin与Placement Blockage不同，它并不是独立存在的，而是依附于Macro周围，可随Macro移动的。所以它是专门用来控制Macro和其他单元之间距离的一种功能。

4）合理设置Macro摆放的角度。在考量Macro摆放的角度时，不仅仅考虑空间摆放的因素，还要根据端口的连接关系与互连模块的位置来决定。如前面原理图中存储模块的端口方向朝向中央，因为中间的标准单元需要与存储模块进行数据交换，存在互连关系。在实际设计时，不仅要根据端口与标准单元之间的连接关系，还要考虑Macro与Macro之间的互连关系进行综合判断。

手工对宏单元进行摆放后，需要将他们设置为dont_touch属性，以防止在布局阶段软件对它进行移动。命令为：set_dont_touch_placement[all_macro_cells]。效果如下：



![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmict0OC5Whg4PCyEsADhd3nfDh67XSjib5q1myDpBmr1Xy1ibUuUiaIia7slGkTbQawOibelO05gY4LSDw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



图2 宏单元放置

从图中可以看出选中的宏单元上面出现了一个叉号，同时左下角出现其对应的属性，*is_fixed*和*is_placed*都是true，表示他们已经放置好，且位置固定。



**！前方高能！**

此外，通过**不同工艺节点**下后端实践，发现一些**非常重要的经验**，分享如下（不同的Foundary要求可能会有所差异）：

1、在180 nm工艺节点下，Macro的摆放比较随意，对其Poly的方向没有硬性规定，不同Macro之间的Poly方向不用相同，也不用和STD Cells的Poly方向相同；

2、在40 nm工艺节点下，要求变严格了，整个Chip中所有Macro的Poly的方向必须一致，因此相同的Macro不能进行旋转操作，要想旋转，必须统一旋转。但是Macro中Poly的方向不用和STD Cells的Poly方向相同；         

3、在28 nm工艺下，要求变得更加严格，整个Chip中Macro的Poly的方向必须一致，且必须和STD Cells中Poly的方向一致。因此在摆放Macro之前，最好确定一下Macro应该如何摆放，否则继续往下做都是白费功夫。

## 2.6 布局障碍的放置（placement blockage）

前面讲解了Floorplan阶段摆放Macro的操作，这里接着讲后续的操作。

摆放完Macro之后需要在Macro周围放置Hard Placement Blockage，它会阻止任何Cell放置在该区域，甚至包括后边的Tap Cell以及Core Filler Cell。同时它也会阻止离它很近的地方放置太多的Cell，如果放置的Cell太多，就会在Macro出pin的地方产生一些Congestion。如果不放置Hard Placement Blockage，那么软件就会在布局阶段自动在Macro周围放置Cell，因此就会产生一些N Well或者P注入、N注入等层次间距的违反，同时也可能会产生一些Congestion。

Hard Placement Blockage的作用和Keep out margin一样，其在Macro外部扩展的距离非常有讲究，如果太小，可能会引起DRC甚至是LVS的问题，其间距过大则会浪费芯片面积，使得物理空间利用率降低。因此该值的选取会比较靠经验，且不同工艺下，该值也往往不同。

如果是Hard Placement Blockage摆放距离不当引起的DRC问题，那么还比较容易解决，可是如果是LVS的问题，那么就麻烦了。下面根据做项目的经验来谈一下我遇到过的问题。

在40nm工艺下，chipfinish完成之后在Calibre里面做DRC LVS，DRC问题不大，可是LVS死活就是过不了，本来以为是Flow的问题，各种改spice文件，改版图，可还是不行，后来觉得可能真的是LVS存在问题。最终找到了问题所在，由于标准单元Nwell伸出标准单元的Boundary，而Keep-out margin留的不太够，使得标准单元的Nwell与SRAM里面的一个SP形成了Overlap，从而形成了一个寄生二极管。

## 2.7 添加EndCAP

在28 nm以及更小尺寸的工艺中，为了保证栅以及氧化层的一致性，需要在标准单元Row两端放置EndCAP。它相当于一种Dummy管子，用来保证两边的标准单元左右环境的一致性。避免在光刻时，由于最两端标准单元左右环境的不一致导致其性能有所差异。

一般对于28 nm以及更高端的工艺中才会有这类单元。如果工艺库中含有这类单元，最好加上。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnXggg76GfBPA59iaabJgNOBicgoCgMiagAIrKAN5tLNM7iaM0jUxFAPCJWYibOkm77CK3YoTl1sicojMpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在ICC中添加这类单元的命令为：

\>add_end_cap -respect_blockage-lib_cell $ICC_H_CAP_CEL

注意：在后端面试中，曾有面试官问过，“加Endcap Cell的深层原因是什么？”。上面我提到了，是为了保证芯片边缘和芯片中间栅和氧化层的一致性，避免因为光刻、刻蚀的原因使得芯片边缘和芯片中间栅、氧化层等结构不一致。其实可以继续深入，边缘处结构和中间结构不一致，将使得芯片边缘处标准单元的时序和中间标准单元不一致。而库中定义的时序的值一般都是常规情况下（也就是芯片中间）的标准单元的时序。因而，如果不加Endcap的话，那么边缘处的标准单元的时序便和库中定义的值有一定的出入，而工具是无法考虑该现象的。因此PR工具或者STA工具分析的结果便于流片后测试的结果有出入。严重的话可能会导致芯片Fail掉。

## 2.8 添加Nwell和衬底接触单元



在超深亚微米工艺中，由于标准单元的N阱和衬底接触占的面积比较大，所以不在每个标准单元中都做相应的接触，而是单独做一个Cell，每隔一定的间距放一些，由此能够节省很多面积。在该单元中包含了N阱接触和衬底接触。不同的工艺库可能名称不一样，一般名字为FILLTIE*。

摆放方式：一般标准单元库中会给出两种摆放方式，一种是Normal（如图1所示），另一种是Flipped（如图2所示）。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgI8ftcn9D1euqlJSDViabo8ESlAt3Hq3iaLRn9klaaib8eOreIHKauey4w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 FILLTIE Cell的Normal放置方式

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgSeMonSkxfkkNTzHAEUYxJrvdHPBNO1OopC2X1lSUaJq58psxGbhtibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 FILLTIE Cell的Flipped放置方式

库中一般会定义Normal放置方式中两个TapCell之间的最大间距，以20 μm为例，则每个Tap Cell距离它最近的TapCell的间距最大应该是20 μm。图2所示的方案满足同样的规则，不过由于每一行Row上面都有Tap Cell（且临近两行Row上面摆放的TapCell是反转的），因此同一行的两个F之间的间距最大可以是40 μm。

**注意：**该间距的设置一定不要超过库文件中定义的值，否则在衬底和阱接触的地方电阻过大，可能会引起闩锁效应。

操作：在ICC的菜单栏中依次选择“Finishing”→“AddTap Cell Array”，得到图3：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgIuVwdPVcow1p48KO65w1eMnrmOichUibgpswPNuxS5et9R9McaJzlVJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 添加Tap Cell阵列

ICC有三种方式来放置Tap Cell，如上图所示，一种是Normal，也是工具默认的选项，工具会按照设定的间距在每一行Row上面都放置Tap Cell。第二种是every_other_row，这种模式是隔行放置方式，对应于前面的图1。第三种是stagger_every_other_row，这种是每行均放置Tap Cell，不过相邻两行是以stagger的方式放置的，对应于前面的图2。

这里采用第三种方式，由前所述，这种情况下同行最大间距为40 μm，所以可以设定为38 μm，相应的命令为：

\>add_tap_cell_array \

-master_cell_name {FILLTIEHS} \

-distance 38 \

-pattern stagger_every_other_row \

-respect_keepout

摆放完毕之后效果如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgj8kjFpX0YWqj51kVryvEibGvib8NewiaakDoleQfRVjTFP03Q46wniaEug/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 Tap Cell的stagger_every_other_row放置结果

 

从图中可以看出，同一行中的相邻两个Tap Cell的间距为38。

将所有单元的电源地端口进行逻辑连接，命令为：

\> derive_pg_connection \

-power_net {VDD} -power_pin {VDD} \

-ground_net {VSS} -ground_pin {VSS}

\> derive_pg_connection -power_net {VDD} -ground_net {VSS} -tie



## 2.9 电源地规划

电源地规划是Floorplan中一个非常难非常复杂，出问题非常多的部分，这里先做简要的介绍，以便于初学者能快速入门。

电源地规划包括以下内容：

- 创建Core PG Rings；
- 创建Macro PG Rings，并连接Macro的PG pins；
- 将IO的电源地连接到CorePG Rings上；
- 创建PG Strap；
- 电源地Rail布线；
- 检查IR-drop。

其中，创建CorePG Rings和PG Strap这两个步骤可以用手工指定的方式来设计，也可以用ICC中的SynthesizePower Network来实现，后者只需要设计者对功耗、电压降、金属层次、宽度等给出相应的约束，软件就会给出一个尽量满足要求的电源地规划方案。前提是设计库中已经指定了TLU+文件，且进行IR-drop分析也需要用到该文件。

有些Designer会觉得电源地规划，当然是布的越多越好了，这种说法不太正确，过多的电源地会占用太多的布线资源，另外，在电源地下方会尽量限制摆放的标准单元的密度，以防止出现拥塞，因此，电源地是不能太多的，否则整个设计的利用率会大大降低。因此其实这是在IR-drop和Congestion之间的折衷。在不影响Congestion的情况下可以尽量多布电源地线。



下面对上面所述6个内容分别做以介绍。



1）创建Core PG Rings；

这里采用手工创建的方式，在菜单栏中依次选择“Preroute” → “Create Rings”，之后在最上方选择“Rectangular”表示以矩形方式创建，另一选项“Rectilinear”表示以线条方式创建，该选项可以创建非矩形形状的环。对于Core PGRings的创建一般用矩形来创建，所以选择“Rectangular”。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgz2tTiceTngjUib0VgodwI6vuCM3rxc339bg5MQIibyfwAUKpAibb85fCBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 创建Core PG Rings



在“Nets”栏中填写为哪些nets创建电源地环，在“Around”一栏中填写为哪些区域/模块创建电源地环，由于这里是为Core创建，所以选择“Core”。此外还可以为Macro以及Region来创建。

在“Side”一栏中选择为哪条边创建环，可以为其指定偏移“Offset”、宽度“Width”、以及层次“Layer”。如果设置了偏移，还需要指定该偏移是相对于Core的绝对偏移还是满足DRC间距的偏移。

该操作的命令为：create_rectangular_rings [-options]。



2）创建Macro PG Rings、MacroPG Straps，并连接Macro的PGpins；

如果是为Macro创建PG Rings，则有三种选择，即“Specified Macro”可以为选中的单/多个Macro分别创建、“Specified as a Group”可以为某些选定的组的外围来创建，组内部则不创建、“ExceptMacros”可以为那些选定的Macro之外的Macro来进行创建。

该操作的命令和上面一样，也是：create_rectangular_rings [-options]。

同时还可以用template来创建Macro的PG Ring，这里不做讲解。

所以为了降低整个Macro的IR-drop，可以在Macro上边多打一些PG Strap，另外某些Memory的datasheet上面也可能会对它上面的Strap的数目以及宽度做出很多限制。



3）将IO的电源地连接到CorePG Rings上；

相应的GUI操作依次为：“Preroute” → “Preroute Instances”，相应的命令为：

\>preroute_instances [-options]

由于最大线宽的限制，给电源地IO布线不能布的太宽，这时候可以通过多次连接来实现非常好的电源地IO布线方案。如下图所示，分别在lowend、high end以及中心向两边偏移一定的距离来布四条线。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgYAu8vK5iaRfTviaqxsdzaRdQxwnvBib1HzxAMAhBLhic5xzvWxtUibicN9Gg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 用多条线将IO的电源地连接到CorePG Rings上



如果说电源地环有多条，那么在不指定extend_for_multiple_connections的时候软件只会连接最外边的电源地环，要想实现与每圈环都接触，那么需要设置在后边的-options中添加extend_for_multiple_connections，并指定一个-extension_gap距离，在该距离内同一个net上的power mesh都会连接到一起。



注意：如果不做其他设置，那么在某些位置，软件经常会形成下面的连接，为了在IO上出pin连接到PG Ring上，会换很多层，形成很多Via，在这些地方经常会发生DRC违反，且连接非常难看。如果手工来连接的话是非常简单的，你看中间部分软件连接的就非常好。除此之外，其实在IO上，可以出pin的有很多层次的，可以做多层金属的连接，这样IR-drop会更低，接触也更好，可是工具默认是只连接一层的。那么如何连接的更漂亮，不违反sign off DRC，且做多层金属连接而非1层呢？这里留个悬念![img](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_ios/u1F61D.png?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhg1zibRte3P3PMFZ7MnqpRO0libyiaBKYEicBv7b3glXrqsZD5pWMOSxqoHA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 工具默认的连接可能出现sign off DRC违反，且连接不是很理想，需要某些特殊方法来解决



4）创建PG Strap；

由于创建PG Strap除了手工创建外还可以采用另一种更简单的方式来产生，即Synthesize Power Network，所以这里用这种方式进行讲解。

首先移除之前定义的一些层次约束，由于之前已经手动产生了Core PG Rings所以需要跳过Ring的约束。

\>set_fp_rail_constraints -remove_all_layers

\>set_fp_rail_constraints -skip_ring   -nets "VDD VSS"

之后需要为Power Network进行一些相应的设置，依次选择“Preroute”⟶“Power Network Constraints”⟶“Strap Layers Constraints”，如图4所示，可以设置相应的层次、最大最小Strap的数目、宽度，设置完毕一层之后点击Set，然后设置另一层。设置完毕后点击Close。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgxNNDhO65K7l9Iaphlic0ALGOAbJhk3umytHOdibiaE9ZdiaPj1zdfNSd1w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4 ICC中的Power Network Constraints



设置完约束之后便可以进行电源地网络的综合了。在菜单栏中依次选择“Preroute”⟶“Synthesize Power Network”。如图5所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgKrkXPXGcaT5Ctaia0xibhwIibyXCtw3CCfZCnh1npqmK6FFGC3DX3UFaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图5 在ICC中以SynthesizePower Network的方式创建PG Strap



选择“By Power network by nets”，在“Nets”中填写PG Nets名称，之后填写供电电源值、预计的功耗，在“Synthesizeoptions”一栏中选中“Synthesizepower plan”，勾选“Specifiedpad masters”以指定IO Pad的方式来设置从什么地方进行供电，并在后边的框中选择相应的供电IO。

上述选项设置完毕之后，点击“Apply”，便可以观看设计中的IRdrop，如图6所示，如果不满足则需要修改约束并重新生成。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlkn56icAFSgCWt0TgZBsHhgPnYzMDyZ90ISBm5pSlajD55o6VpQHtFthOnqqcnSg7IOZrws4KybkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图6 Synthesize Power Network产生的IR drop图



如果IR drop已经满足了自己的要求，可以点击“Commit”，软件便会尽量产生出满足自己定义的规则的PG Strap。注意该操作一旦执行，不可撤销。所以最好在执行该操作前对设计进行保存。或者如果执行了该操作，发现有问题，想要撤销，可以用remove_route_by_type -pg_strap来删除PG Strap。



该步骤的脚本如下：

set_fp_rail_constraints-remove_all_layers

 

set_fp_rail_constraints -skip_ring \

-nets "$MW_GROUND_NET$MW_POWER_NET"

 

set_fp_rail_constraints-add_layer -layer TM1 \

​    -direction vertical \

​    -max_strap 12 -min_strap 10 \

​    -max_width 2 -min_width 1.5 \

​    -spacing minimum

 

set_fp_rail_constraints-add_layer -layer TM2 \

​    -direction horizontal \

​    -max_strap 20 -min_strap 18 \

​    -max_width 2 -min_width 1.5 \

​    -spacing minimum

 

synthesize_fp_rail-nets "$MW_POWER_NET $MW_GROUND_NET" \

​    -voltage_supply 0.8 \

​    -synthesize_power_plan \

​    -power_budget 40 \

​    -pad_masters {VDD:PVDD1RN_X.FRAM VDD:PVDD1RN_Y.FRAM VSS:PVSS1RN_X.FRAM }



注意：在供电IO多于一个的情况下要用分别指定的放置，即：

{VDD:PVDD1RN_X.FRAM VDD:PVDD1RN_Y.FRAM VSS:PVSS1RN_X.FRAM }

不要写成：

{VDD:{PVDD1RN_X.FRAM PVDD1RN_Y.FRAM} VSS:PVSS1RN_X.FRAM }

如果IR-drop达到了要求，要执行则用下面的命令：

commit_fp_rail



5）电源地Rail布线（给标准单元供电的电源和地线）

该步的GUI操作为：“Preroute”→“Preroute Standard Cells”。相应的命令为preroute_standard_cells[-options]，此处需要注意选择fill_empty_rows，因为某些Row上面可能没有放置标准单元，所以需要填充这些空的Row。

例如可以用下面的命令：

\> preroute_standard_cells-nets {VDD VSS} \

   -connect horizontal \

   -fill_empty_rows \

   -port_filter_mode off \

   -cell_master_filter_mode off \

   -cell_instance_filter_mode off \

   -voltage_area_filter_mode off \

   -route_type {P/G Std. Cell Pin Conn}



6）检查IR-drop

做电源网络分析（Power Network Analysis, PNA），查看电源规划的IR Drop。这里只介绍该操作的命令：

\>analyze_fp_rail-nets "VDD VSS" \

​    -power_budget 10 \

​    -voltage_supply 1.1 \

​    -pad_masters { VDD:PVDD1RN.FRAMVSS:PVSS1RN.FRAM 

接下来会继续讲解后续的步骤

## 2.10 自动做Floorplan的Placement，对Floorplan的结果进行评估

这里讲解一下Floorplan最后一个部分，自动做floorplan的placement，作为floorplan的参考，对floorplan的结果进行评估。这一步是对前面做的Floorplan做以评判，看其质量如何，如果Floorplan做的不好，那么后边也许就没有继续做下去的必要了，因为后边的流程可能根本就走不通，因此这一步至关重要，它是对Floorplan的质量做以简单的评估，如果质量很差，那么需要参考之前的Floorplan方案，并做以调整，直到结果在可以接受的范围内，然后才能继续往下做。



为了对手工Floorplan的结果做以评判，需要在拥塞和时序上对它进行检查，所以可以在布局之前，首先让软件快速的做一个粗略的布局，可以输入以下命令来在Floorplan阶段做快速布局：

\>create_fp_placement -congestion_driven-timing_driven

之后需要报告设计中的拥塞情况，输入命令：

\>report_congestion

这里提到了拥塞（Congestion），它对于后端设计至关重要，至于什么是拥塞，拥塞该如何去解决，可以查看推文：

[数字后端中的拥塞（Congestion）及其解决方案](http://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247484746&idx=1&sn=3909497bb786f68360d11245f35c8da8&chksm=e8292184df5ea8923349dba61c8e1e1e09fa9640a2d85d043ba603d481006513a91d58d75251&scene=21#wechat_redirect)

[美国下注15亿美元重点搞芯片？AI在IC设计及EDA软件中的应用 AI在DRC中的应用](http://mp.weixin.qq.com/s?__biz=MzIyMjc3MDU5Mw==&mid=2247484694&idx=1&sn=b9f9bc44ac41a6d640508444a40c1d9b&chksm=e82921d8df5ea8ce6fbcd6c93dcbf768c41362ed9748bf46d10e3d66cd1030524447346b3b99&scene=21#wechat_redirect)



Floorplan阶段在SoC设计中至关重要，将来的设计中可能有非常多的模块，芯片规模非常大，有哪些指标来评价Floorplan方案的好坏？



1、时序，用零延时模式检查看是否存在Setup违反，最好不要有，可以存在一些留待后边CTS修复，即用CCD（Concurrent Clock and Data）来修；

2、Congestion，设计中的拥塞决定后边Routing是否有可能布的通。经验值，如果max overflow超过10基本不可能布通，最好不要超过3-5；total overflow最好不要超过2%;

3、IR-drop，总的不要超过电压的5%，它会对时序以及芯片的性能稳定性造成一定的影响；

4、DRC，这非常重要，通常也容易被忽略。



**下面对Floorplan阶段的DRC问题进行专门的介绍**

该阶段的DRC主要是对电源地网络以及Macro与Macor之间的间距进行DRC检查。最好是导出GDS到Calibre里面去做DRC，而非在ICC里面做DRC，因为ICC用的是FRAM View，且DRC规则不全，无法对Macro之间的间距违反做出检查，且有些DRC规则是不全的，所以结果不完整。

该阶段最好不要放置标准单元，因为如果放置了标准单元，那么标准单元之间没有填充Filler，因此会存在很多N well间距的违反。

Power Mesh DRC检查：

到了深亚微米，在Floorplan阶段还有一个非常重要的问题是Redundant Via的问题，就是，在上下两块金属接触面积到一定程度时，DRC规则会对接触位置最少Via的数量做出限制。在布线阶段，可以设置Redundant Via的规则，让软件在布线中考虑这些问题。由于Floorplan阶段是Power Mesh是手工完成的，因此很有可能会在这个网络上存在很多Redundant Via的问题，可是power mesh的这个规则的违反在ICC里面却检查不出来。所以在ICC里面做完Floorplan之后要导入Calibre里面检查，如果Via数量不够，那么需要调整线宽，使得软件可以在换层的地方插入更多的Via，做完之后看一下Via的数量，如果满足要求那么就OK了。

标准单元和Macro之间间距（Placement Blockage）：

同样，根据这些规则去设置Placement Blockage，该距离不能太小，否则可能会引起DRC和LVS的违反。举个例子，之前我在设计中，给SRAM周边外部加了Placement Blockage，因此外部的Standard cell不会离它太近。可是到了Sign off阶段，发现一直存在着LVS的错误，找了很久发现，在standard cell和SRAM挨着的地方，由于standard cell的N well是要向外伸出一部分的，该部分刚好与Macro里面的SP重叠在了一起，产生了一个二极管，因此导致出现了LVS错误，这些问题都是在Floorplan阶段产生的。

Macro和Macro之间的间距：

最好就是看Calibre里面DRC规则，比如Nwell最小间距，然后在ICC里面打开Cel View视图，根据N well间距去调节Macro之间的距离，这样既能充分利用空间，又不至于出现DRC违反。

