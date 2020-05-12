# 3. 布局

下面讲解用ICC进行布局的流程。

#### 3.1 布局前的时序设置

在布局之前，需要将所有的scenario都激活：set_active_scenarios -all，如果没有MCMM，则不用执行该操作。

之后对于每个scenario都需要将高扇出网络以及时钟网络设置为理想的网络：

\>set_ideal_nerwork[all_fanout -flat -clock_tree]

#### 3.2 设置placement的约束

一般在电源地Strap下面放置标准单元时，可能会在Strap的地方存在Congestion。这时可以限制工具在电源地Srtap下面少放或者不放置标准单元，相关设置命令如下：

例如，将TM1和TM2的PG线下方设置为PartialBlockage：

\>set_pnet_options-partial {TM1 TM2}

不允许TM1和TM2的PG线下方放置任何标准单元：

\>set_pnet_options-complete { TM1 TM2}

允许软件在TM1和TM2的PG线下方放置任何标准单元，在不指定的情况下，软件默认的也是该设置，即允许软件在所有的PG线下方自由放置任何标准单元的：

\>set_pnet_options-none {TM1 TM2}

#### 3.3 开始布局

\>place_opt -effort high-congestion

#### 3.4 查看布局之后的拥塞

\>report_congestion

#### 3.5 报告布局之后的时序

报告最大路径延时，查看是否存在Setup Slack：

\>report_timing

报告是否存在时序DRC的违反：

\>report_constraint-all_violators

此时设计中最好不要存在Setup的违反，可以存在Max Cap/tran、hold、Min Cap的违反。

如果设计中有MCMM，那么可以用下面的命令产生MCMM的报告同时产生相应的网页文件，方便查看：

\>create_qor_snapshot-name cel_name

图1是产生的报告：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtk0zmkLy0061NFwaxAWtytwNJmkEcJcuRakf0pAia8lHyrvMUiaFoKpdW4UpgicRziaYMm7vDLNVfic3AA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 ICC中生成的报告

从图中的报告可以看出，设计中一共有12个scenario，s1-s4用来检查setup，s5-s12检查hold。设计中不存在setup的违反。在布局之后最好不要存在setup的违反，如果有违反，最好执行相应的优化，将这些违反解决掉。而其他DRC的违反存在一部分，如果这些违反是库文件中限定的，则应尽量解决，如果是sdc文件引起的，那可能是约束过于苛刻造成的，可以忽略或者更改sdc约束