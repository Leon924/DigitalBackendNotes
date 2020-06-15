# 4. 时钟树综合

####  4.1 时钟树布线规则的定义

1）删除之前定义过的NDR（Non-default Rule）

\>remove_routing_rules -all

2）为时钟线定义双倍宽度、双倍间距这些类似的NDR

在ICC LayoutWindow中的菜单栏上依次选择“Route” → “Routing Setup” → “Define Routing Rule” → “New”来新建一个NDR。在Rule name中填入clk_dsdw，“Width multiplier”中填2，“Spacing multiplier”中填2，点击Apply即可创建一个名为clk_dsdw的双倍线宽，双倍线间距的NDR。

该操作相应的命令为：

\>define_routing_ruleclk_dsdw \

-default_reference_rule-multiplier_spacing 2 \

-multiplier_width2 ;#for double width

3）为时钟布线选择自己定义的NDR，以及金属层次，在Sink端用默认规则进行布线（不用自己定义的NDR）。可以执行下面的命令：

\>set_clock_tree_options-routing_rule clk_dsdw \

-layer_list {M3 M4 M5 M6} \

-use_default_routing_for_sinks1 ;#apply rule to all but leaf nets

该命令对应的gui操作在“clock” → “Set Clock Tree Options”中。



#### 4.2 选择用于生成时钟树的references

在进行时钟树综合之前首先需要给软件指定，用哪些单元来创建时钟树。一般可以用时钟Buffer或者时钟反相器来创建。时钟Buffer与一般Buffer的不同之处在于，它的上升下降时间基本相同。使用Buffer构建时钟树的优点是：逻辑简单，便于post-CTS对时钟树的修改；缺点是：面积大，功耗大，insertion delay大。使用反相器构建时钟树的优点是：面积小，功耗小，insertiondelay小，对时钟duty cycle(占空比)有利；缺点是：不易做时钟树的修改。

在ICC中进行该操作的GUI步骤依次为：“Clock”⟶“Set Clock Tree References”。在“Clock trees”中选择需要创建时钟树的时钟的名字，下方选择创建时钟树所需的Cell。一般库里面的时钟Buffer或者Inverter都是以CLK或者CK开头的，可以用通配符进行查找这些单元。

这里给出相应的命令：

\>set_clock_tree_references-references \

{CLKBUFHSV12CLKBUFHSV16 CLKBUFHSV2 CLKBUFHSV20 \

CLKBUFHSV3CLKBUFHSV4 CLKBUFHSV6 CLKBUFHSV8 \

CLKNHSV12CLKNHSV16 CLKNHSV2 CLKNHSV20 CLKNHSV3 \

CLKNHSV4 LKNHSV6CLKNHSV8}

####  

#### 4.3 开始时钟树综合

在菜单栏中依次选择“Clock”⟶“Core CTS and Optimization”。在“Perform”选项下面选择第三项“CTS only (CTS, CTO, clockrouting)”之后取消勾选“RouteClock nets”，意思是只建立时钟树，不进行时钟线布线，留待之后专门进行时钟线布线。如图1所示：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtjZ9xcSurWwiaNtccuIMicPZtGgYxdvWbicsLtcQSysvsAIiby2t6h2Pq74w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 时钟树综合

相应的命令为：

\>clock_opt -only_cts-no_clock_route



#### 4.4 报告CTS之后的结果

做完CTS之后需要查看CTS的结果，评价CTS质量的指标是时钟偏斜Skew，输入下面两个命令中的任何一个均可：

\>report_clock_tree -summary (报告的是Global Skew)

\>report_clock_timing -type skew (报告的是Local Skew)

这两条命令只会会报出当前scenario下面的Skew，要想报出所有scenarios下面的Skew可以在后边加上-scenarios[all_scenarios]选项。

如图2所示是ICC报告的某一个scenario下的时钟树的大概情况，从上面可以看到每个时钟树的名字、每个时钟树上面在CTS阶段插入的Buffer的数目和面积、时钟树的最长路径延时还有就是最重要的时钟skew的大小。

report_clock_tree -summary可以报告出时钟树的总体情况，主要包括以下几个方面：

1）Clock 即时钟树的名字；

2）Sinks 即时钟树Sink pin的数目；

3）CTBuffers 即CTS阶段插入的时钟树单元的数目；

4）ClkCells 时钟树中所有单元的总数，包括已存在的门；

5）Skew  时钟偏斜；

6）LongestPath 最长时钟路径；

7）TotalDRC 时钟树中所有DRC违反的数目；

8）BufferArea 时钟树中所有CTBuffers的总面积。

这里需要注意的是skew不能太大，因为CTS之前的时钟都是理想的，用了一个uncertainty来模拟这个skew，如果实际的skew与之前预设的uncertainty偏差太大，那么setup也许会有违反，同时CTS之后也会很难修hold。

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtj2lxAMBOkqXTGQZD2sjiaPvQH8Cyib1SuSLXsQJiboIsCiaUz7KicticE5YlQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 ICC报告的时钟skew

####  

#### 4.5 POST-CTS

CTS之后由于时钟树已经建立，所以需要将时钟网络以及高扇出网络的理想属性移除，并开始修复hold违反。下面逐一进行讲解如何操作。

1）重新定义关于clockuncertainty的定义，去掉其中估计的clock skew的部分，移除高扇出网络的理想属性，将时钟网络进行传播，命令为：

\>foreach_in_collectionclk [get_clocks] {

remove_clock_latency$clk

remove_ideal_network[all_fanout -flat -clock_tree]

set_propagated_clock[get_attr $clk sources]

set_clock_uncertainty-setup 1 $clk

set_clock_uncertainty-hold 0.1 $clk

}

2）CTS之后开始关心hold time，命令为：

\> set_fix_hold[all_clocks]

3）CTS之后开始修holdviolation

在菜单栏中依次选择“Clock” → “Core CTS and Optimization”。在“Perform”选项下面选择第一项“CTS, clock routing, …”之后勾选“Fix hold time violation forall clocks”修复hold的违反，并取消勾选“Route Clock nets”，意思是不进行时钟线布线，留待之后专门进行时钟线布线,如图3所示。



![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtnS5XlAPaGU55LtVFC1WWtjLrtU4rEGeJGfc27P7TgLIqckPEplON42micwjEwktgo04YibRdI7ruTg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 CTS之后开始修复hold违反

相应的命令为：

\> clock_opt-no_clock_route -fix_hold_all_clocks

之后需要对所有的标准单元的电源地进行逻辑连接

\>derive_pg_connection -power_net {VDD} -power_pin {VDD} \

-ground_net {VSS}-ground_pin {VSS}

\>derive_pg_connection -power_net {VDD} -ground_net {VSS} -tie