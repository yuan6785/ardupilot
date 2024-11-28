该分支是从tag为Copter-4.5.7拉出来的

a. ardupilot是安装在UAV的飞控上面的操作系统,可以随时根据传感器的数据来调整UAV的平衡;
b. sitl是无需任何硬件在pc上面模拟传感器数据的模拟飞行模式,参考本文的【sitl的原理】; SITL要把硬件参数录入，比如飞机的电池电量， 重量， 动力范围等输入进去; 这样可以防止摔机
c. hitl硬件仿真(读取正式无人机传感器数据) https://ardupilot.org/dev/docs/hitl-simulators.html  ----自己制作，一般不需要，硬件成本太高了。

解决每次push都会发大量邮件的问题
重命名 .github/workflows为workflows_yx_delte 文件夹后，GitHub 就无法找到工作流配置文件，自然也就不会触发工作流了。这种方法适合临时或快速禁用所有工作流。


搜索笔记[无人机]

仿真编译参考:
https://www.cnblogs.com/qsbye/p/18229006
该网页静态文件我已经存在本项目
0yxgithub/ardupilot/搭建ArduPilot的SITL仿真环境 - qsBye - 博客园.mhtml
仿真的另外一篇:
https://dronechina.net/t/topic/1046/2
另外一篇:
https://blog.csdn.net/weixin_43321489/article/details/132422643
官网仿真文档:
https://ardupilot.org/dev/docs/simulation-2.html  有仿真的介绍(FDM和仿真的概念介绍以及流程图---一定要看)
https://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html  初级仿真---第一步要用的(mac也可以运行---一定要看)
https://ardupilot.org/dev/docs/copter-sitl-mavproxy-tutorial.html  仿真教程--各种命令--重要-----一定要看
https://ardupilot.org/dev/docs/plane-sitlmavproxy-tutorial.html  界面使用---重要-----
https://ardupilot.org/copter/docs/common-parameter-reset.html   重置参数
https://ardupilot.org/copter/docs/parameters-Copter-stable-V4.5.7.html  完整参数列表(硬件参数，可以自己手动配置你的无人机大概的硬件参数)
其他文章
https://www.bilibili.com/opus/882110201551912992   ArduPilot的前世今生(用ardupilot做diy的第一步)
比较好的中文diy教程
https://doc.cuav.net/tutorial/copter/

#官方mac 编译
https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md

CXX=clang++ CC=clang ./waf configure


最后我用的命令(不要参考文中的编译版本，用我下面的版本和命令)
############
pyenv activate py3.8_virtualenv_test   #指定python3.8版本
git clone git@github.com:yuan6785/ardupilot.git
git checkout fork_tag_copter_457_main  # 指定ardupilot的版本为Copter-4.5.7
----
cd ardupilot
CXX=clang++ CC=clang ./waf configure
# 使用 pip 安装剩余的包
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple imgviz==1.5.0
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple empy==3.3.4
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pexpect==4.8.0
# pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pymavlink
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple mavproxy==1.8.46
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple dronecan==1.0.26
----
# 编译到sitl模式
./waf clean
./waf configure --board sitl --debug
./waf -j8 copter -v  # 编译----如果飞行命令执行不了，请重新执行此命令初始化整个环境----重要-----
# https://ardupilot.org/dev/docs/sitl-on-windows-wsl.html ----- 入门飞行控制命令
python Tools/autotest/sim_vehicle.py --console --map -w -v ArduCopter  # 将会启动飞控界面--这里会出现一个地址，你可以可以在命令行界面回车直接控制sitl, 下面mavproxy.py会用这个地址可以实现远程控制
# sudo apt-get install -y flightgear
# python Tools/autotest/sim_vehicle.py -L KSFO -v ArduCopter
----
# 地面站飞行控制，地址输入sitl的运行地址，即sim_vehicle.py运行的地址即可
# https://ardupilot.org/mavproxy/docs/getting_started/quickstart.html  ------ 学习如何控制
mavproxy.py --master=tcp:127.0.0.1:14450  # 这里用的是sitl的启动后输出的"--out" "127.0.0.1:14550"的地址，不是--master的地址
#######
param show *  # 查看所有参数
######
param set BATT_CAPACITY 10000  # 设置电池容量（单位：mAh）--电池充满时容量
param show BATT_CAPACITY
param show SIM_BATT_CAP_AH    # 也是电池容量，但是为0。 不知道什么含义
param show SIM_BATT_VOLTAGE   # 模拟电池电压 
######
# param set SIM_BATTERY {VALUE}
param set FORMAT_VERSION 0
param show SIM_BATTERY
param show FORMAT_VERSION
reboot
batreset # 重置电池
#################
disarm force  # 用了arm命令后需要用disarm命令来断开电机
#######重置#######
输入 mode rtl，飞行器会返回起飞点。
输入 mode land，飞行器会直接降落。
rc 3 1000   # 还原油门
#######加载参数#####
param load <path to parameter file>
######保存参数######
param save <path to parameter file>  # 启动默认在mav.parm下面



简单例子: https://ardupilot.org/dev/docs/sitl-on-windows-wsl.html   （---------重要-------）
#########执行飞行命令#########
# 开始垂直起飞高度为40米
mode guided  
arm throttle # 解锁电机响应命令，在另外的解锁下，需要先disarm断开电机
takeoff 40  # 需要时间，不要切换模式
# 绕圈圈
rc 3 1500   # 1000: 表示最小油门（无人机怠速或停转）。 1500: 表示中立油门（无人机悬停或匀速飞行）。 2000: 表示最大油门（全速飞行）。 0: 表示无控制状态
mode circle
param set circle_radius 2000
#########重置命令#############
batreset # 重置电池
mode rtl # 飞回起点--需要时间，不要切换模式
mode land # 降落--需要时间，不要切换模式
rc 3 1000 # 还原油门--重要--需要在arm throttle后执行, 或者重置所有rc(rc all 1000)
disarm force # 断开电机
batreset # 重置电池
##########只有在上面的停止状态才能执行reboot##################
reboot # reboot 命令会重启 APM（自动驾驶仪），用于恢复飞控系统的正常状态，常用于调试、配置更改后或系统异常时。
############################

















sitl的原理
https://doc.cuav.net/tutorial/copter/simulation.html
###############
SITL模拟器（循环中的软件）
SITL（循环中的软件）模拟器允许您在没有任何硬件的情况下运行Plane，Copter或Rover。 它是使用普通C ++编译器构建的自动驾驶仪代码，为您提供了一个本机可执行文件，允许您在没有硬件的情况下测试代码。

本文概述了SITL的优势和体系结构。

SITL允许您直接在PC上运行ArduPilot(本来应该安装在无人机上调整无人机的平衡，接受其他命令执行飞行计划等)，无需任何特殊硬件。 它利用了ArduPilot是一个可以在各种平台上运行的便携式自动驾驶仪。 您的PC只是ArduPilot可以构建和运行的另一个平台。

在SITL中运行时，传感器数据来自飞行模拟器中的飞行动力学模型。 ArduPilot拥有多种内置的飞行器模拟器，可以连接到几个外部模拟器。 这使得ArduPilot可以在各种类型的模型上进行测试。 例如，SITL可以模拟：

多旋翼飞机

固定翼飞机

地面车辆

相机的万向节

天线跟踪器

各种可选的传感器，如激光雷达和光学流量传感器

添加新的模拟飞行器类型或传感器类型很简单。

ArduPilot在SITL上的一大优势是它可以让您访问C ++开发所需的各种开发工具，如交互式调试器，静态分析器和动态分析工具。 这使得开发和测试ArduPilot的新功能变得更加简单。

运行SITL
APM SITL环境上运行了 Linux和Windows。 指令看设置SITL Linux上和在Windows上设置SITL为更多的信息。

SITL架构
注意下面的图像端口号仅供参考，可能会有所不同。 例如，ArduPilot和图像上的模拟器之间的端口是5501/5502，但根据您的环境，它们可能会变成5504/5505或其他端口号。
###############