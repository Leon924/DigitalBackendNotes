## 7.1 检查电源地连接

在最后的版图验证阶段，首先需要检查电源地的连接，即检查标准单元、宏单元、IO的电源地连接以及Strap等是否存在Floating。



GUI操作：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB6gpEAArnBKMu1s7f0oDibTADfNLLQy2PjOK2iafMpugpf2EsiaicnItZO8A/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB63HoKuQ6euuLva186wl1uWUCL4yAqMbm1nUTLzr8C6OcncibkDCRx3bA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

相应的脚本：

verify_pg_nets -error_cel std_io_pg \

   -std_cell_pin_connection check \

   -macro_pin_connection all \

   -pad_pin_connection all

## 7.2 DRC检查

版图验证的第二项内容是对版图进行DRC检查。ICC里面不是Signoff级别的DRC检查工具，只能对部分规则进行检查，更加详细准确的结果可以用Calibre或者S家的ICV，这里DRC的结果作为设计者的参考。



1）执行布线DRC检查，保证布线不存在DRC违反

GUI操作：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB6yIl1r7FwZsicSxOI89zVQw6WptsAeQgshaFLp37sXuZtaLFnem1cwtw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB6RGrqMibcWrTfUaicY1B2eVmxDsicNtLibV0Qs2GUP1vtBd9QyNXMibpXz7A/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

相应的脚本：

verify_zrt_route \

-antenna true \

-check_from_user_shapes true \

-check_from_frozen_shapes true \

-report_all_open_nets true

得到下面的报告，此时不能存在天线效应和DRC的违反：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB6XOvALkrf3c8FInO7U601HJtmGln7niaERQt3SwFkP63GcEq8dJ3BGhA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果有违反的话，先执行上面的DRC检查操作，然后输入下面的命令进行修复：

route_zrt_detail -incremental true -initial_drc_from_input true



2）执行整体芯片的DRC检查，ICC不是十分准确，结果仅供参考（有的ICC版本中没有该功能，被新的功能取代了）：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB62ualTxvhDDpBD1nibA1mTVjxqPb3GbDeXXkS46h6PEk3skH4GmB82tA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB6hSMg3ibh0mQicEf28la4XkZicmK4ibwEVMBZZaiazMEdJnyLhFmD1GO8oNA/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

报告如下，不存在DRC违反：

![img](https://mmbiz.qpic.cn/mmbiz_png/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB69hB60lRWX1hxydG83QQ2bNzZxKHhN5gacuZ70moD7QEpWuCOFibrl8g/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 7.3 LVS检查

版图验证的第3项是进行LVS检查，即检查版图和网表的一致性。同样，LVS的结果也仅供参考，最终还是要以Calibre LVS的结果为准。

ICC中的GUI操作如下：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB6LUT0gg8w9sSovwcib4UD4bIhUZaicrOnjvZIicpeyiaCHtJC56xhx5EU4w/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/jfiabYavVPtlW1TB0ILZaSBZSbkib8xUB62F4PWSGaOhiakBIsJS6Rxg1obOK5ia7zYiatYkibjibqKytnlYicv8e85icGw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以忽略的问题：Floating Port、Floating Net

不可忽略的问题：Open、Short



如果出现了LVS问题如何修复呢？

1、Open问题：

用route_zrt_eco进行修复；

2、Short问题：

手动或者用命令删除Short的Nets，然后用route_zrt_eco进行重新绕线。如果是大面积的Short很可能设计没救了，去Floorplan阶段、布局等各个阶段看Congestion找问题。