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
./waf -j8 copter -v  # 编译
python Tools/autotest/sim_vehicle.py --console --map -w -v ArduCopter  # 将会启动飞控界面--这里会出现一个地址，你可以可以在命令行界面回车直接控制sitl, 下面mavproxy.py会用这个地址可以实现远程控制
# sudo apt-get install -y flightgear
# python Tools/autotest/sim_vehicle.py -L KSFO -v ArduCopter
----
# 地面站飞行控制，地址输入sitl的运行地址，即sim_vehicle.py运行的地址即可
# https://ardupilot.org/mavproxy/docs/getting_started/quickstart.html  ------ 学习如何控制
mavproxy.py --master=tcp:127.0.0.1:5760