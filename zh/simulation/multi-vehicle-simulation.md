# Gazebo 多机仿真

本文介绍如何使用 Gazebo 仿真器配合软件在环仿真进行多机仿真（仅适用于 Linux ）。

> **Tip** 如果您不需要 Gazebo 或 ROS 提供的功能， [ JMAVSim的多机仿真](../simulation/multi_vehicle_jmavsim.md) 更容易设置。

本文给出了演示实例, 打开了 Gazebo 仿真器，在空的的世界中显示了两个 Iris 无人机。 然后, 你可以使用 *QGroundControl* 配合 MAVROS 控制多机, 其使用方式与单机控制类似。

## 仿真前准备

* [PX4 ROS/Gazebo 开发环境](../setup/dev_env_linux_ubuntu.md#rosgazebo) **Note** 编写此教程的环境为 Ubuntu 18.04 和 ROS 版本为 Melodic 以及 Gazebo版本为 Gazebo 9 。 也可以看 [Gazebo 仿真](../simulation/gazebo.md)
* [MAVROS](http://wiki.ros.org/mavros)
* 克隆最新的 PX4 源码固件。

## 开始仿真

构建一个示例设置, 请按照以下步骤操作:

1. 克隆 PX4 固件源码, 然后编译 SITL 代码 
      cd Firmware_clone
       git submodule update --init --recursive
       DONT_RUN=1 make px4_sitl_default gazebo

2. 配置系统环境变量： 
  
      source Tools/setup_gazebo.bash $(pwd) $(pwd)/build/px4_sitl_default
       export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:$(pwd):$(pwd)/Tools/sitl_gazebo

3. 启动 launch 文件： ```roslaunch px4 multi_uav_mavros_sitl.launch```
  
  > **Tip** 你可以在上述 *roslaunch* 中指定 `gui:=false` ，以便不启动界面的情况下启动 gazebo。

本指南设置打开了 Gazebo 仿真器界面, 在空世界中显示了两个 Iris 无人机。

然后, 你可以使用 *QGroundControl* 和 MAVROS 控制多机，控制方式与单机类似。

* *QGroundControl* 中有一个下拉选项，你可以选择指定的飞行器。
* MAVROS 要求你在 topic/service 路径之前包含合适的命名空间，（例如，你会用到 */uav1/mavros/mission/push* ）。

## 发生了什么？

对每一个仿真的飞行器，有如下要求：

* **Gazebo model**：在路径`Firmware/Tools/sitl_gazebo/models/rotors_description/urdf/<model>_base.xacro`下被定义成`xacro` 文件。看[这里](https://github.com/PX4/sitl_gazebo/tree/02060a86652b736ca7dd945a524a8bf84eaf5a05/models/rotors_description/urdf)。 目前，模型 `xacro`文件设定以 base. xacro 结尾。 此模型应该有一个名为 `mavlink_udp_port` 的参数, 该参数定义了与 px4 节点通信的 udp 端口。 模型的 `xacro` 文件将用于生成包含您选择的 udp 端口的 `urdf` 模型。 若要定义 udp 端口，请在每个飞行器的启动文件中设置 `mavlink_udp_port`，请参阅例子[here](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/launch/multi_uav_mavros_sitl.launch#L37)。
  
  > **注意** 如果你在使用同一个飞行器模型，你不需要为每一个飞行器设置一个单独的**`xacro`**文件。 相同的 **`xacro`** 文件就可以了。

* **PX4 node**: 这是 sitl px4 应用程序。它通过在Gazebo模型中定义的同一 udp 端口 （即 `mavlink_udp_port`）与模拟器 、gazebo 进行通信。 要在 px4 sitl 应用程序端设置 udp 端口, 您需要在启动文件中设置 `SITL_UDP_PRT` 参数, 以匹配前面讨论的 `mavlink_udp_port`, 请参阅 [here](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/posix-configs/SITL/init/ekf2/iris_2#L46)。 启动文件中的开始文件路径由参数 `vehicle`和`ID`产生，参考[这里](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/launch/multi_uav_mavros_sitl.launch#L36)。 开始文件中的每一个飞行器的`MAV_SYS_ID`参数都要与启动文件中的`ID`相匹配。参考[这里](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/launch/multi_uav_mavros_sitl.launch#L25)。 这样能够帮助你确保启动文件和开始文件中的设置相同。

* **MAVROS node**（可选）: 如果要通过 ros 控制车辆, 可以在启动文件中运行一个单独的 mavros 节点， 请参阅 [here](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/launch/multi_uav_mavros_sitl.launch#L41), 以便连接到 px4 sitl 应用程序。 您需要在启动文件中一些特殊的端口上启动 mavlink 流, 请参阅 [这里](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/posix-configs/SITL/init/ekf2/iris_1#L68)。 这些特殊端口需要与launch文件中为MAVROS节点设置的相符合。参考[这里](https://github.com/PX4/Firmware/blob/4d0964385b84dc91189f377aafb039d10850e5d6/launch/multi_uav_mavros_sitl.launch#L26)。

启动文件 `multi_uav_mavros_sitl.launch`做了以下内容,

* loads a world in gazebo, 
      <!-- Gazebo sim -->
        <include file="$(find gazebo_ros)/launch/empty_world.launch">
            <arg name="gui" value="$(arg gui)"/>
            <arg name="world_name" value="$(arg world)"/>
            <arg name="debug" value="$(arg debug)"/>
            <arg name="verbose" value="$(arg verbose)"/>
            <arg name="paused" value="$(arg paused)"/>
        </include>

* for each vehicle,
  
  * creates urdf model from xacro, loads gazebo model and runs PX4 SITL app instance
    
          <!-- PX4 SITL and vehicle spawn -->
          <include file="$(find px4)/launch/single_vehicle_spawn.launch">
              <arg name="x" value="0"/>
              <arg name="y" value="0"/>
              <arg name="z" value="0"/>
              <arg name="R" value="0"/>
              <arg name="P" value="0"/>
              <arg name="Y" value="0"/>
              <arg name="vehicle" value="$(arg vehicle)"/>
              <arg name="rcS" value="$(find px4)/posix-configs/SITL/init/$(arg est)/$(arg vehicle)_$(arg ID)"/>
              <arg name="mavlink_tcp_port" value="4560"/>
              <arg name="ID" value="$(arg ID)"/>
          </include>
        
  
  * runs a mavros node
    
          <!-- MAVROS -->
          <include file="$(find mavros)/launch/px4.launch">
              <arg name="fcu_url" value="$(arg fcu_url)"/>
              <arg name="gcs_url" value=""/>
              <arg name="tgt_system" value="$(arg ID)"/>
              <arg name="tgt_component" value="1"/>
          </include>
        
  
  > **Note** The complete block for each vehicle is enclosed in a set of `<group>` tags to separate the ROS namespaces of the vehicles.

要在此模拟中添加第三个iris四旋翼, 需要考虑两个主要部分：

* add `UAV3` to **multi_uav_mavros_sitl.launch** 
  * duplicate the group of either existing vehicle (`UAV1` or `UAV2`)
  * increment the `ID` arg to `3`
  * select a different port for `mavlink_udp_port` arg for communication with Gazebo
  * selects ports for MAVROS communication by modifying both port numbers in the `fcu_url` arg

* create a startup file, and change the file as follows:
  
  * make a copy of an existing iris rcS startup file (`iris_1` or `iris_2`) and rename it `iris_3`
  * `MAV_SYS_ID` value to `3`
  * `SITL_UDP_PRT` value to match that of the `mavlink_udp_port` launch file arg
  * the first `mavlink start` port and the `mavlink stream` port values to the same values, which is to be used for QGC communication
  * the second `mavlink start` ports need to match those used in the launch file `fcu_url` arg
    
    > **Note** Be aware of which port is `src` and `dst` for the different endpoints.

## Multiple Vehicles using SDF Models

This section shows how developers can simulate multiple vehicles using vehicle models defined in Gazebo SDF files (instead of using models defined in the ROS Xacro file, as discussed in the rest of this topic).

The steps are:

1. Install *xmlstarlet* from your Linux terminal: ```sudo apt install xmlstarlet```
2. Use *roslaunch* with the **multi_uav_mavros_sitl_sdf.launch** launch file: ```` roslaunch multi_uav_mavros_sitl_sdf.launch vehicle:=<model_file_name> ```
  
  > **Note** that the vehicle model file name argument is optional (`vehicle:=<model_file_name>`); if omitted the [plane model](https://github.com/PX4/sitl_gazebo/tree/master/models/plane) will be used by default.

This method is similar to using the xacro except that the SITL/Gazebo port number is automatically inserted by *xmstarlet* for each spawned vehicle, and does not need to be specified in the SDF file.

To add a new vehicle, you need to make sure the model can be found (in order to spawn it in Gazebo), and PX4 needs to have an appropriate corresponding startup script.

1. You can choose to do either of: 
  * modify the **single_vehicle_spawn_sdf.launch** file to point to the location of your model by changing the line below to point to your model: ```$(find px4)/Tools/sitl_gazebo/models/$(arg vehicle)/$(arg vehicle).sdf``` > **Note** Ensure you set the `vehicle` argument even if you hardcode the path to your model.
  * copy your model into the folder indicated above (following the same path convention). 
2. The `vehicle` argument is used to set the `PX4_SIM_MODEL` environment variable, which is used by the default rCS (startup script) to find the corresponding startup settings file for the model. Within PX4 these startup files can be found in the **Firmware/ROMFS/px4fmu_common/init.d-posix/** directory. For example, here is the plane model's [startup script](https://github.com/PX4/Firmware/blob/master/ROMFS/px4fmu_common/init.d-posix/1030_plane). For this to work, the PX4 node in the launch file is passed arguments that specify the *rCS* file (**etc/init.d/rcS**) and the location of the rootfs directory (`$(find px4)/ROMFS/px4fmu_common`). For simplicity, it is suggested that the startup file for the model be placed alongside PX4's in **Firmware/ROMFS/px4fmu_common/init.d-posix/**.

## Additional Resources

* See [Simulation](../simulation/README.md) for a description of the UDP port configuration.
* See [URDF in Gazebo](http://gazebosim.org/tutorials/?tut=ros_urdf) for more information about spawning the model with xacro.
* See [RotorS](https://github.com/ethz-asl/rotors_simulator/tree/master/rotors_description/urdf) for more xacro models.