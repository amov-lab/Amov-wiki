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
-   **autonomous_landing.cpp**:
-   **collisiom_avoidance_streo.cpp**:
-   **formation_control_sitl.cpp**:
-   **payload_drop.cpp**:
-   **square.cpp**:
-   **target_tracking.cpp**:
-   **move.cpp**:
-   **set_mode.cpp**:
-   **setpoint_track.cpp**:
-   **TFmini.cpp**:

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

3 速度控制与位置控制
---------------------




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