# 操作步骤
参考教程：  
1. [net-snmp-5.7.3配置编译安装](https://www.cnblogs.com/oloroso/p/4595123.html)
2. [snmp-agent 初步了解](https://blog.csdn.net/backkom_jiu/article/details/79579064) 
3. [net-snmp的MIBs扩展（linux下）](https://www.cnblogs.com/oloroso/p/4599501.html)<-这个教程有点小问题，读写时OID最后少了`.0`
4. [linux net-snmp（之mib2c工具生成表格代码）](https://blog.csdn.net/chengcheng1024/article/details/105529250)
5. 待补充
## 安装SNMP
&emsp;&emsp;根据*参考教程1和2*安装SNMP，建议安装`5.7.3`版本。安装过程可能会报错，根据错误安装相应的库即可。
## 写MIB文件
* 使用ANS.1写mib文件，例如*参考教程3*中myTest.txt或者myTest.mib。
* DEFINITIONS ::= BEGIN 前的名称是库名。
* IMPORTS标识当前库文件需要引入其它的类文件,编译才不会出错.引入格式为 [对象]FROM[库文件]。
* NODENAME为当前节点的名称,如sysDescs,这个名称应该是唯一的,NODETYPE为当前节点的类型,如MODULE-IDENTITY,OBJECT-TYPE等等,
DATA为节点的内容说明,包含数据类型,访问模式,状态,描述,PARENT_NODENAME为当前节点的父节点,这个父节点可能在本类中,也可以要引用的外部类中,id为当前节点在父类中的索引顺序号。
* MIB存放于/usr/share/snmp/mibs目录下，因为这个目录是snmpd的默认目录，只要把MIB库放入该目录就可以自动加载MIB库，否则需要修改/etc/snmp/snmp.conf文件，添加mibs +/path/to/XXX 并重启snmpd。
## MIB引入和snmp转换
* export MIBS=+库名称或MIB文件名，如`export MIBS=+TEST-MIB`，也可以在根目录下.profile文件最后一行加这句，这样可以开机自动export。
* 开启snmpd：
  |功能             |指令                       |
  |:---             |:---                      |
  |开启snmpd        |sudo service snmpd start   |
  |关闭snmpd        |sudo service snmpd stop    |
  |查看snmpd状态    |sudo service snmpd status  |
  |另一种开启snmpd  |sudo snmpd -d -f -Lo       |  

  [/etc/snmp]sudo snmpd -d -f -Lo这种方式开启snmpd后需要使用kill命令关闭进程，先`ps -aux|grep snmpd`查看snmpd进程的进程号，然后`sudo kill -9 进程号`杀死进程。
* snmp转换查看，使用指令`snmptranslate -Tp -IR 节点名`，如snmptranslate -Tp -IR test，转换成功会显示树信息。
## MIB编译转.c文件
* 使用mib2c将MIB转换成C文件  
  |模板类型      |指令                                                               |备注                   | 
  |:---         |:---                                                              |---:                   |
  |int_watch    |sudo env MIBS="库名或文件名" mib2c -c mib2c.int_watch.conf 节点名  |数据只读用这个         |
  |scalar       |sudo env MIBS="库名或文件名" mib2c -c mib2c.scalar.conf 节点名     |数据读写用这个         |
  |iterate      |sudo env MIBS="库名或文件名" mib2c -c mib2c.iterate.conf 节点名    |表格用这个，转换选1    |
* 根据*参考教程3和4*修改生成的`节点名.C`文件
* 使用指令`sudo net-snmp-config --compile-subagent 可执行文件名称 节点名.c`将.c文件转换为可执行文件
* 开启snmpd
## 使用snmpget、snmpset等指令进行读写
* 读指令`sudo snmpget -c public -v 2c localhost TEST-MIB::readObject.0` 或者 `sudo snmpget -c public -v 2c localhost 1.3.6.1.4.1.77587.1.0`
* 写指令`sudo snmpset -c private -v 2c localhost TEST-MIB::writeObject.0 s "123"` 或者 `sudo snmpset -c private -v 2c localhost 1.3.6.1.4.1.77587.2.0 s "123"`
## 注意事项
* 将1203v0305.mib文件中FROM后的NTCIP8004v02改为NTCIP8004-A-2004
* MIB中有表格时，根据*教程4*需要在表格中增加一列ROW_STATUS类型的列才能进行表格列的增删操作，否则表格为空表格无法get和set。
