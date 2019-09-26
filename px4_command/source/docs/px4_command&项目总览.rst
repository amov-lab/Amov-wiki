.. px4_command&项目总览

=======================
px4_command&项目总览
=======================

第一节 总体概述
===============

px4_command功能包是一个基于PX4开源固件及Mavros功能包的开源项目，旨在为PX4开发者提供更加简洁快速的开发体验。
目前已集成无人机外环控制器修改、目标追踪、激光SLAM定位、双目V-SLAM定位、激光避障等上层开发代码、后续将陆续推
出涵盖任务决策、路径规划、滤波导航、单/多机控制等无人机/无人车/无人船科研及开发领域的功能。
配合板载计算机(树莓派、TX2、Nano)等运算能力比较强的处理器，来实现复杂算法的运行，运行得到的控制指令通过串口或者网口通信发送给底层控制板。


第二节 软件框架
================

.. image:: ../images/framework.png

-   **state_from_mavros.h**:订阅飞控状态,包括无人机当前的状态(/mavros/state),当前位置(/mavros/local_position/pose),当前速度(/mavros/local_position/velocity_local),和当前角度,角速度(/mavros/imu/data)
-   **command_to_mavros.h**:发布px4_command功能包生成的控制量至mavros功能包,可发送期望位置,速度(本地系与机体系),角度,角速度,底层控制(遥控器输入)
-   **px4_pos_estimator.cpp**:订阅激光雷达或者mocap发布的位置信息,并进行坐标转换,在state_from_mavros.h中已订阅飞控发布的位置,速度,欧拉角信息,此处直接使用,根据订阅的数据,发布相应的位置,偏航角给飞控
-   **px4_pos_controller.cpp**:订阅由位置估计发布的DroneState,初始化当前飞机状态的时间.订阅ControlCommand(不知从何发布的数据).发布topic_for_log主题.在选择控制率,检查参数正确后,初始化完成.对move节点中,takeoff,Move_ENU,Move_Body,Hold,Land,Disarm,PPN_land和Trajectory_Tracking等进行逻辑处理.
-   **ground_station.cpp**:订阅自定义日志主题(/px4_command/topic_for_log),订阅视觉系统位置估计PoseStamped主题(/vrpn_client_node/UAV/pose,非mavlink消息,数据包括point位置(x,y,z),四元数方向(w,x,y,z)),订阅飞控姿态四元数AttitudeTarget主题(/mavros/setpoint_raw/target_attitude,#82号mavlink消息).不断的更新视觉传感器状态,并打印当前飞机的状态.
-   **px4_sender.cpp**:订阅自定义消息控制指令主题(/px4_command/control_command),机体系到惯性系坐标转换,move中控制命令的具体实现(0表示位置控制,3表示速度控制)
-   **autonomous_landing.cpp**:降落识别使用xyz均为速度控制.订阅数据包括降落板与无人机的相对位置,降落板与无人机的相对偏航角,视觉flag 来自视觉节点.最后发布位置控制指令
-   **collisiom_avoidance_streo.cpp**:订阅/streo_distance该数据作为计算飞机四个方向的距离判断.
-   **formation_control_sitl.cpp**:多机仿真SITL,只适用于Move_ENU坐标系下,若使用Move_Body,需自行添加修改.
-   **payload_drop.cpp**:订阅/mavros/local_position/pose本地位置.发布遥控器通道值.
-   **square.cpp**:发布/px4_command/control_command命令.子模式xyz均为位置控制.
-   **target_tracking.cpp**:
-   **move.cpp**:发布/px4_command/control_command,并设置子模式xy速度控制(0b10),位置控制.z速度控制(0b01),位置控制
-   **set_mode.cpp**:模拟遥控器,根据mavros服务,进行在SITL下解锁,切换offboard,控制飞行器.
-   **TFmini.cpp**:激光定高雷达的处理,如果需要添加超声波传感器,可参考此代码.

1 下载编译
-----------

a 通过二进制的方法安装Mavros功能包
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

请参考:`mavros的github <https://github.com/mavlink/mavros>`_

如果你已经使用源码的方式安装过Mavros功能包,请先将其删除.

b 在home目录下面创建一个名为 **px4_ws** 的工作空间
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    mkdir -p ~/px4_ws/src
    cd ~/px4_ws/src
    catkin_init_workspace

大部分时候,需要手动 **source** ,打开一个新终端

::

    gedit .bashrc

在打开的 bashrc.txt 文件中添加 source /home/$(your computer name)/px4_ws/devel/setup.bash

c 下载并编译 **px4_command** 功能包
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    cd ~/px4_ws/src
    git clone https://github.com/amov-lab/px4_command
    cd ..
    catkin_make


2 位置估计
------------

GPS在室外可以进行多传感器融合,最后给到飞控内部,得到本地位置的信息.但除了GPS之外的位置数据源构建的一个基于px4的系统,
我们将其称之为外部位置估计系统,例如vicon,optitrack之类的运动捕捉系统,还有像二维激光雷达系统,双目vio之类的基于视觉
的估计系统.

位置估计既可以来源于板载计算机,也可以来源于外部系统.这些数据用于更新机体相对于本地坐标系的位置估计.来自与视觉或者运动
捕捉系统的yaw角信息也可以被融合到姿态估计器当中.

我们所使用的激光雷达或者mocap系统有关数据融合的部分都在**px4_pos_estimator.cpp**代码中.

3 速度控制与位置控制
---------------------

在px4_command中,控制命令的子模式可以选择位置追踪,速度追踪或者是复合追踪.默认为XYZ位置追踪模式(sub_mode=0).速度追踪为sub_mode=3.
在move中控制,提供了XYZ位置追踪模式(0)或者是XYZ速度追踪模式(3).所谓位置控制就是发布期望的位置坐标点,而速度控制即为一直发布期望的速度飞行,
在仿真中测试可以发现,发布期望的速度,如果速度不重新设置为(0,0,0),那么飞行器是不会停止的,在实际过程中,请谨慎使用此模式控制.复合模式的使用常用在
跟踪条件下,对XY进行发布期望速度,对Z发布期望位置.


4 室内定位(激光/视觉)
----------------------

5 视觉追踪
------------

6 视觉引导降落
----------------

7 深度学习追踪
----------------

第三节 硬件链接
===============

1 硬件外设
------------