# 5. 布线


时钟树综合结束之后，接下来的工作是布线(routing)。

Floorplan阶段，生成电源地网络时已经完成了电源地网络的布线，floorplan阶段给标准单元供电的rail已经生成，place结束后标准单元上下两边都放在了rail上面，routing阶段主要是完成标准单元的信号线的连接。

在开始布线之前先介绍基于格点的布线理论。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtjyDBbtkhWU5A3sw5WRj87tAF3j8v4LN0njg5cd8b4EibjicenJnZK9R1A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 基于格点的布线理论

在上图中，黄色和蓝色的虚线称之为track，track是没有实际宽度的，但两条track之间有间距称之为pitch，基于格点的布线要求所有的金属走线要走在track之上，而实际走出的金属线称之为trace，trace是有宽度的。

不同的金属线走线方向是不同的，奇数层金属默认走水平方向，偶数层默认走竖直方向。

两条track的交点称之为gridpoint。

标准单元的高宽都被设计成了pitch的整数倍，而在布局时标准单元的pin都被放在了gridpoint上面这样都为布线作好了准备。

Routing主要由以下四个步骤完成：

Global routing（全局布线）

Track assignment（布线通道分配）

Detail Routing（详细布线）

Search andrepair（布线修补）

ICC有两种布线模式，一种是传统的类似于Astro中的基于格点的布线方法，另一种是Zroute模式，这是对传统基于格点的布线模式的延伸，线可以不用非得沿着格点进行布线。由于这种布线方法能非常有效解决很多DRC的违反，所以这里主要基于该模式进行讲解。这种模式可以在菜单栏中可以进行下面的操作进行选择：“Route”→ “Zroute Mode”。一般软件默认的也是该模式。



#### 5.1 加载天线效应文件

在开始布线之前要首先加载天线效应文件，以防止在布线过程中发生天线效应规则的违反。如果发现违反也可以通过引导EDA工具来修复违反。之前的Astro中使用的是.clf格式的天线效应问题，ICC则使用.tcl格式的文件。

\> source /xxx/SCC40NLL_HS_RVT_V0p2b/astro/clf/antenna_8lm_2tm.tcl

####  

#### 5.2 时钟线布线

在菜单栏中依次选择“Route” → “Net Group Route”，勾选“All clock nets”，点击OK即可完成所有时钟线的布线，如图2所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtjcq7xwlvLv04pc4aAmd02jaOuwdUibcA8fpDjQNVOReCMmu0myOBmt5A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 基于Zroute模式的时钟线布线



#### 5.3 普通信号线的布线

如果设计规模很小，那么Global routing、Track assignment、Detail Routing、Search and repair可以一步完成，在GUI里面可以通过“Route” → “Auto Route”来完成。如果设计规模很大，则需要分步执行，依次选择执行“Route”下面的“Global Route” → “Track Assignment”⟶“Detail Route”。



#### 5.4 检查布线之后的时序

用report_timing -max以及-min查看setup与hold是否满足设计要求，最后用report_constraint -all_violators报告设计中是否存在时序DRC的违反。

如果存在时序DRC的违反，可以用focal_opt-drc_nets all -effort high来修复。如果想让软件专门修复hold的违反则可以用：

\>focal_opt -hold_endpoints all -effort high

如果时序DRC还存在违反，那么可能是前面的操作没有执行好，或者就是约束的问题，需要修改约束。



#### 5.5 做布线的DRC检查

在做完布局布线之后，需要对版图的质量进行检查，这里只用做关于布线的DRC即可，ICC对于整体的DRC检查不是很完全、准确，整体的DRC可以用Calibre来做。

在GUI中可以通过“Route” → “Verify Route”来做布线DRC的检查，如图3所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtjlvicYbFE2bMzw3aMGickGfVWGlA4l7Olbcnfsn4t2ibg1TAxVsZ65juQQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 布线DRC的检查

得到如图4所示的报告：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtjdNf4FzUPJxxpbF6uicpBbp07QD7e34ZVmDl8hv0JoxMibKwPFbfOMdTA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图4布线DRC检查结果



报告中显示了设计中是否存在物理DRC的违反以及天线效应的违反。另外该操作也可以用命令来执行，相应的命令为：

\>verify_zrt_route \

-antenna true \

-check_from_user_shapestrue \

-check_from_frozen_shapestrue \

-report_all_open_netstrue

在报告中不能存在物理DRC的违反。如果有违反的话，先执行上面的DRC检查操作，然后输入下面的命令进行修复：

\> route_zrt_detail -incremental true-initial_drc_from_input true