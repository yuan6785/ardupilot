该分支是从tag为Copter-4.5.7拉出来的

解决每次push都会发大量邮件的问题
重命名 .github/workflows为workflows_yx_delte 文件夹后，GitHub 就无法找到工作流配置文件，自然也就不会触发工作流了。这种方法适合临时或快速禁用所有工作流。


搜索笔记[无人机]

仿真编译参考:
https://www.cnblogs.com/qsbye/p/18229006
该网页静态文件我已经存在本项目
0yxgithub/ardupilot/搭建ArduPilot的SITL仿真环境 - qsBye - 博客园.mhtml
另外一篇:
https://blog.csdn.net/weixin_43321489/article/details/132422643
官网仿真文档:
https://ardupilot.org/dev/docs/simulation-2.html  有仿真的介绍(FDM和仿真的概念介绍以及流程图---一定要看)
https://ardupilot.org/dev/docs/setting-up-sitl-on-linux.html  初级仿真---第一步要用的(mac也可以运行---一定要看)
https://ardupilot.org/dev/docs/copter-sitl-mavproxy-tutorial.html  仿真教程--各种命令--重要-----一定要看
https://ardupilot.org/copter/docs/common-parameter-reset.html   重置参数
https://ardupilot.org/dev/docs/plane-sitlmavproxy-tutorial.html  界面使用---重要-----
其他文章
https://www.bilibili.com/opus/882110201551912992   ArduPilot的前世今生(用ardupilot做diy的第一步)

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
mavproxy.py --master=tcp:127.0.0.1:5760
#######
param show *  # 查看所有参数
######
param set BATT_CAPACITY 10000  # 设置电池容量（单位：mAh）
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
#######


简单例子https://ardupilot.org/dev/docs/sitl-on-windows-wsl.html   （---------重要-------）
#########执行飞行命令#########
# 开始垂直起飞高度为40米
mode guided  
arm throttle # 解锁电机响应命令，在另外的解锁下，需要先disarm断开电机
takeoff 40  # 需要时间，不要切换模式
# 绕圈圈
rc 3 1500
mode circle
param set circle_radius 2000
#########重置命令#############
batreset # 重置电池
mode rtl # 飞回起点--需要时间，不要切换模式
mode land # 降落--需要时间，不要切换模式
disarm force # 断开电机
batreset # 重置电池
############################