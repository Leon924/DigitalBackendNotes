| Path                                                         | Do what                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| /export/Home/liqiang/lab/HRI/DC-RM_N-2017.09-SP4             | DC main work dir                                             |
| /export/Home/liqiang/lab/HRI/DC-RM_N-2017.09-SP4/work1       | DC wrok dir（在这个文件目录下跑的DC）                        |
| /export/Home/liqiang/lab/HRI/DC-RM_N-2017.09-SP4/rm_setup    | **common_setup.tcl是DC的所需database**，dc_setup.tcl是DC执行期间需要的一些变量设置，这里还有Makefile脚本 |
| /export/Home/liqiang/lab/HRI/DC-RM_N-2017.09-SP4/rm_dc_scripts | DC的脚本（还有FM的也在这里）                                 |

**DC：在work1目录下， 终端，执行make -f ../rm_setup/Makefile1 dc即可执行脚本**

**FM：在work1目录下， 执行Fm脚本，source  ../rm_setup/run_fm.sh即可执行脚本**

**dc1.tcl是最基础的脚本，**

**dc2.tcl是我改动之后，加了门控时钟的**

| Path                                | Do what                          |
| ----------------------------------- | -------------------------------- |
| /export/Home/liqiang/lab/HRI/dc     | 也是一个跑dc的目录，但是不是RM的 |
| /export/Home/liqiang/lab/HRI/fm_syn | fm的目录，常与HRI/dc配合使用     |


