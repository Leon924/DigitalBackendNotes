# 1. 设计准备

## 1.1 数据准备

在用ICC开始布局布线之前，应核查输入数据准备是否完备，主要包括：

- Milkyway参考库

信息是以被称为“views”的形式存储的，例如：

CEL:完整的版图信息

FRAM:用于布局布线的抽象化的版图物理信息（只有单元大小、端口名称、端口位置等简单的物理信息）

LM:带有时序和功耗信息的逻辑模型（可选*），该文件对于后端布局布线不是必须的，IC Compiler只通过link_library变量来读取指定的(.db)格式的逻辑库。

对于那些标准单元库、IO库、Memory或者其他Macro，如果设计中没有CEL View以及FRAM View，则可以在Milkway软件中通过简单的read_lef文件的方式生成这些文件。其中lef文件全拼为：Library Exchange Format。

- 工艺文件(.tf文件)

每种工艺对应一个唯一的工艺文件

包含金属层次对应的工艺参数：

​    每个层次/Via对应的编号以及名字

​    工艺的介电常数

​    每个层次/Via的物理和电学特性

​    每个层次/Via的设计规则（最小线宽以及最小线间距等）

​    单位及精度

​    用于显示的每层对应的颜色及模式

- tluplus文件

寄生RC查找表，ICC使用网络几何形状以及该文件来计算互联电阻电容。

若tluplus文件没有时，可由Foundry给的.itf转成tluplus。其中.itf文件全称是Interconnect Technology Format。

用Synopsys公司的Star-RCXT，在shell下用此命令就行：

\> grdgenxo-itf2TLUPlus -i <ITF file> -o <TLU+ file>

- db文件

用于提供STD Cell、IO、Macro的时序、功耗、面积等信息。这里不同于之前的Astro需要根据已有的库文件生成LM View、TIM View、PWR View，ICC可以直接用Design Compiler使用的db文件，因此方便了设计，简化了设计的复杂度。

- 门级网表文件（.v文件）

该文件可以用逻辑综合工具（如Design Compiler, DC）来产生，某些部分可以人为手工修改/编写，在导入ICC中之前，首先需要检查网表的质量，以尽早排除可能造成后端设计困难的问题，比如浮动输入信号、多驱动、未采用寄存器输入输出、输入到寄存器、寄存器到寄存器、寄存器到输出、扇入扇出等。这些问题如果及时发现，并在前端进行改善会比较容易，且非常有利于后端设计的顺利进行。

- 时序约束文件（.sdc文件）

该文件可以由DC工具导出，并人工进行修改，以使其满足设计要求，约束要合理，不能过约束，否则后端软件可能无法达到要求。

- 管脚排列文件（.tdf文件）

该文件主要用来定义设计中所有IO以及IO Corner的位置（上下左右的方位以及排列顺序，也可以定义具体的坐标）。

该文件可以参照ICC中的*set_pad_physical_constraints*命令的文件语法进行编写，也可以在ICC中导出一个该文件进行手工修改。下面简单介绍下该命令：

其部分语法为：

set_pad_physical_constraints

​       objects | -pad_name stringpad_name

​       [-side side_number]

​       [-order order_number]

例如：set_pad_physical_constraints-pad_name clk_block -side 1 -order 1

该命令定义了一个名为clk_block的IO PAD的位置，该名称必须出现在前面所述的门级网表中，否则的话需要在ICC中手动创建该单元，例如网表中没有的电源地、给IO供电的IO需要在后续的11.3.2布局规划阶段进行添加。

Side_number指定了IOPAD摆放在哪条边，其中从左边数，顺时针方向依次为1、2、3、4，即1代表左边，2代表上边，3代表右边，4代表下边。默认该选项设置为0，即代表没有摆在指定哪条边的限制。

Order_number指定IOPAD的排列顺序，数字从1开始，越大越往右/上方摆放。默认该选项也是为0，意思是没有排列顺序的限制。如图1所示：

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlr6ycWicdS31Qct6esrqcQx6fwzxLYunFYBA6vvkkicoK6vjWvlzxX8mXowPeNbIbVianJWA6yptecw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 库设置文件（.synopsys_dc.setup）

该文件可以放在ICC软件启动目录，软件在启动时会自动加载search path、target_library、link_library这些库，或者可以将这些设置单独存为一个脚本，在每次打开ICC时都手动source该脚本，该脚本内容与DC中用的启动文件完全一样。

## 1.2 参考库的创建

在开始后端布局布线之前首先需要准备好标准单元库、IO库、宏单元库（如SRAM）的各种参考库文件，所有文件均为Milkyway格式。参考库的创建可以用Synopsys公司的Milkyway软件来实现。该工具的图形化界面如图1所示：

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlr6ycWicdS31Qct6esrqcQxu1uqVvYxXdgCy18rKMgbgj43pVSXGxg4QJ61GiaS8wuXw1hkXRyaetg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在Milkyway中用read LEF文件的方式创建Milkyway参考库。

首先创建库，在菜单栏中选择“Library”⟶“Create”，在“Library Name”中填写要创建的参考库的名字，“TechnologyFile Name”中填写工艺库文件的路径，并选中“SetCase Sensitive”区分大小写。

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlr6ycWicdS31Qct6esrqcQxtowypIHzsDmBTqoMFGqKAlkic4LLdUU6YSUC0icGrfnMxjZc91R3b0og/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																	图2 在Milkyway中创建参考库

接着在菜单栏中选择“Cell Library”⟶“LEF In”，“Library Name”后边通过点击“Browse”选择刚才创建的参考库，“Tech LEF Files”选择工艺的LEF文件路径，“Cell LEF Files”选择Cell的LEF文件。其他选项默认即可。点击OK之后就会在参考库目录内生成CEL和FRAM两个文件夹，这两个便是创建的CEL View和FRAM View。

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlr6ycWicdS31Qct6esrqcQxHuURCs6uS1ErictRSzJptVypCcSgKrE6ke7ZdzhKxTJOLrmKLjJEhhw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																	图3 在Milkyway软件中读入LEF文件

最后进行电源地端口的声明，即输入命令：

dbSetCellPortTypes"sram_lib" "sadslsck41p64x32m4b1w0c0p0d0t0"'(("VDD""Power")("VSS" "Ground")) #f

其格式为：

*dbSetCellPortTypes"libName" "cellName" '({("portName" {"type"}...} ...)) append?*

该命令的详细内容可以通过help命令来查看，其命令为：

*help "dbSetCellPortTypes"*

下面以一个SRAM为例提供创建参考库的脚本：

\################################################

cmCreateLib

setFormField"Create Library" "Library Name" "sram_lib"

setFormField"Create Library" "Technology File Name""scc40nll_hd_8lm_2tm.tf"

setFormField"Create Library" "Set Case Sensitive" "1"

formOK "CreateLibrary"

 

read_lef

formButton "ReadLEF" "browse..."

setFormField "ReadLEF" "Library Name" "sram_lib"

setFormField "ReadLEF" "Tech LEF Files" "scc40nll_8lm_2tm.lef"

setFormField "ReadLEF" "Cell LEF Files""sadslsck41p1568x32m8b1w0c0p0d0t0.plef"

formOK "ReadLEF"

 

dbSetCellPortTypes"sram_lib" "sadslsck41p1658x32m4b1w0c0p0d0t0"'(("VDD""Power")("VSS" "Ground")) #f

## 1.3 为设计创建library

可以输入下面的命令启动ICC的图形用户界面（GUI）：

\> icc_shell-gui &

打开GUI界面之后，就会看到ICC的MainWindow主窗口。

如果是要用命令行方式启动则输入命令：

\> icc_shell&

在命令行方式下打开GUI的命令为：

\> start_gui（直接输入sta即可）

启动软件后，如果目录下面没有.synopsys_dc.setup，则需要手动source库文件设置的脚本。

或者可以通过MainWindow中设置，即依次选择菜单栏中的“File”⟶“Setup”⟶“ApplicationSetup”。

接着创建设计库，所有的设计都是在该设计库中完成的。可以在MainWindow中的菜单栏中依次选择“File”⟶“Create Library”。在“New Library name”中填写要创建的设计库的名字，在“Technology file”中选择工艺库文件，在“Input referencelibraries: Files”栏中点击“Add”添加参考库文件，包括IO、标准单元、宏单元的参考库。勾选“Open Library”⟶“OK”，完成设计库的创建并打开设计库。如图1所示。

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmDBUmJOY3rWwbic9M1Rb2n5dbA0nbTzFPUprWda69JPVN3aS1dIGtZYCkWFHMQEsg2HRRoyjfVxhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmDBUmJOY3rWwbic9M1Rb2n5icST7w4YiaTFxd2kqu2t8ibfjYPLhWu5dwZZLkDmnufCDJKODc3tHLicKQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图1 在ICC中创建设计库并打开库



打开设计库之后，读入网表文件。在MainWindow中的菜单栏中依次选择“File”⟶“Import Designs”，如图2所示。ICC提供了三种输入文件格式来导入设计，由于db格式和ddc格式会包含一些时序约束，这些约束信息不是特别清晰，所以最好以Verilog格式来创建。点击“Add”添加DC导出的门级网表文件，并在“TopDesign Name”后边填写网表中顶层设计的名称来指定顶层设计，最后点击“OK”即可完成设计中Cell的创建。创建完Cell之后，ICC便会弹出另外一个窗口，即LayoutWindow版图窗口，如图3所示，此时由于没有进行布局规划，所以设计中所有的标准单元、IO、Macro都是叠放在一起的。

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmDBUmJOY3rWwbic9M1Rb2n5zcej8H47DggTiau6Gvp3SrFCs6rWzQmMhWXrFYOlSPpobnO6z0xiauHQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图2 在设计库中以导入Verilog文件的方式导入设计

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmDBUmJOY3rWwbic9M1Rb2n5BTBNviayYc6ASKxxU7UyOwEoPYQDwvJ1JurmLnr9G1c42a52PJGSicwg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图3 ICC中的Layout Window

## 1.4 进行uniquify

为了在布图时进行时钟树综合，必须唯一化DC中的网表。此操作为设计中多次例化的子模块生成唯一的模块/实体定义。原因：1.存在于这些模块内的触发器需要连接到时钟源，而把时钟树连接到这些模块需要单独的时钟连线名。2.物理上要求这些模块有单独的位置。

\> uniquify_fp_mw_cel

确认当前顶层设计

\> current_designmain_pad

之后将网表中例化的单元与参考库中的单元做连接。

进行链接

\> link

## 1.5 设置TLU+文件

在菜单栏中选择“File”⟶“Set TLU+”，分别在“Max TLU+ file”、“Min TLU+ file”中填写最大、最小TLU+文件，并在“Layer name mapping filebetween technology library and ITF file”中填写工艺文件和ITF文件的映射文件，如图1所示。

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmDBUmJOY3rWwbic9M1Rb2n5ZJib1pJwfrS4TZmkdudBh8u5on6Uf0nXM1oyVbmZknSbqueMZf6PibZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](http://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtmDBUmJOY3rWwbic9M1Rb2n5QUG1ichJ2JDSIQoEZNwqZQyMZYsAxvmDngp9ufpfibGFYq9EQtqCpIjg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																			图1设置TLU+文件

## 1.6 读入SDC，设置芯片工作环境

在菜单栏中依次选择“File”⟶“Import”⟶“Read SDC”，Version选择Latest，然后点击OK即可。如果设计中有多个Corner多个Mode，则需要执行相应的MCMM脚本。

## 1.7  检查设计的合理性

在开始后续的布局规划以及布局布线之前，需要检查时序约束是否完善、合理。在PT或ICC中运行check_timing命令，检查是否存在没有时钟的寄存器，没有设置输入输出延迟，未约束路径，交叉时钟域路径等问题，这主要是为了检查约束是否完全，对于未约束路径，输入输出等，一定要补全相应的约束。

之后设置为0互联延时，此时是最理想的情况，在该模式下报告时序，此时不应存在Setup的违反。如果存在违反，需要返回前端修改设计或者可能sdc约束文件存在问题，需要修改约束。

\>set_zero_interconnect_delay_mode true

\> report_timing

另外，还可以用report_constraint命令来报告设计中的时序DRC（Design RuleConstraint）违反，包括Setup、Hold、max capacitance、max transition、max fanout等。此时除了Setup外，其他可以存在违反，这些违反也是后边要修复的目标。>表示将命令执行的结果报告存为某个文件，以方便查看。

\>report_constraint -all_violators > ./rpts/init_design.rpt

上面的命令只针对当前的scenario有效，如果设计中有多个scenario，需要使用下面的命令来指定对所有的scenario均报告违反：

\> report_constraint-all_violators -scenario [all_scenarios] > ./rpts/init_design.rpt

在检查完设计在理想情况下的时序之后要重新将0互联延时模式关闭

\> set_zero_interconnect_delay_mode false