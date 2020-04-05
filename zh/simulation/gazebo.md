# Gazebo 仿真

[ Gazebo ](http://gazebosim.org)是用于自主机器人的强大3D模拟环境，其特别适用于测试物体避障和计算机视觉。 本文描述了如何使用它来进行单机的软件在环仿真。 Gazebo 也可以适用于 [硬件在环仿真](../simulation/hitl.md) 和 [多机仿真](../simulation/multi-vehicle-simulation.md) 。

**支持机型：**四旋翼 （[Iris](../airframes/airframe_reference.md#copter_quadrotor_wide_3dr_iris_quadrotor) 和 [Solo](../airframes/airframe_reference.md#copter_quadrotor_x_3dr_solo)），六旋翼（Typhoon H480），[通用四旋翼构型 VTOL ](../airframes/airframe_reference.md#vtol_standard_vtol_generic_quad_delta_vtol)，尾座式，固定翼，无人车，潜艇/无人水下航行器。

> **Warning** Gazebo 通常与 [ROS](../ros/README.md) 一起使用，ROS 可以理解成 Offboard 自动飞行控制的 API 工具包。 如果您计划同时使用ROS和PX4，则**应遵循**[ ROS说明安装 ](../simulation/ros_interface.md)去安装ROS和Gazebo（从而避免安装冲突）。

{% youtube %}https://www.youtube.com/watch?v=qfFF9-0k4KA&vq=hd720{% endyoutube %}

{% mermaid %} graph LR; Gazebo-->Plugin; Plugin-->MAVLink; MAVLink-->SITL; {% endmermaid %}

> **Note** 有关仿真器、仿真环境和仿真配置（以支持的机型为例）的基本信息，请参见 [仿真 ](/simulation/README.md)。

## 安装 {#installation}

Gazebo 9 的安装在标准的环境编译已有说明。

* ** macOS：** [ Mac 上的开发环境](../setup/dev_env_mac.md)。
* **Linux：**Ubuntu LTS/Debian Linux 上安装 Gazebo、 JMAVSim 和 Nuttx（Pixhawk）的开发环境。</li> 
    
    * ** Windows：**暂不支持。</ul> 
    
    其他安装说明可在 [gazebosim.org](http://gazebosim.org/tutorials?cat=guided_b&tut=guided_b1) 上找到。
    
    ## 运行仿真
    
    启动 PX4 SITL 和 Gazebo 进行仿真，并加载机架配置（支持多旋翼，固定翼，VTOL，光流和多机仿真）。
    
    运行仿真最简单的方法是在 PX4 *Firmware* 存储库的根目录下打开一个终端，并调用 `make` 编译所需目标。 例如，要开始一个四旋翼飞行器的仿真（默认）：
    
    ```sh
    cd /path/to/Firmware
    make px4_sitl gazebo
    ```
    
    下文列出了支持的载具类型及对应的 `make` 指令（点击链接查看载具图像）。
    
    > **Note** 使用指令 `make px4_sitl list_vmd_make_targets` 获取构建目标的完整列表（并过滤掉以 `gazebo_` 开头的目标）。
    
    | 载具类型                                                                                                | 指令                                     |
    | --------------------------------------------------------------------------------------------------- | -------------------------------------- |
    | [四旋翼](../simulation/gazebo_vehicles.md#quadrotor)                                                   | `make px4_sitl gazebo`                 |
    | [具有光流的四旋翼](../simulation/gazebo_vehicles.md#quadrotor_optical_flow)                                 | `make px4_sitl gazebo_iris_opt_flow`   |
    | [3DR Solo（四旋翼）](../simulation/gazebo_vehicles.md#3dr_solo)                                          | `make px4_sitl gazebo_solo`            |
    | <span id="typhoon_h480"></span>[Typhoon H480（六旋翼）](../simulation/gazebo_vehicles.md#typhoon_h480) （支持视频流） | `make px4_sitl gazebo_typhoon_h480`    |
    | [标准构型的固定翼](../simulation/gazebo_vehicles.md#standard_plane)                                         | `make px4_sitl gazebo_plane`           |
    | [标准垂起](../simulation/gazebo_vehicles.md#standard_vtol)                                              | `make px4_sitl gazebo_standard_vtol`   |
    | [尾座式垂起](../simulation/gazebo_vehicles.md#tailsitter_vtol)                                           | `make px4_sitl gazebo_tailsitter`      |
    | [Ackerman 地面车辆（UGV/Rover）](../simulation/gazebo_vehicles.md#ugv)                                    | `make px4_sitl gazebo_rover`           |
    | [HippoCampus TUHH（UUV：无人水下航行器）](../simulation/gazebo_vehicles.md#uuv)                               | `make px4_sitl gazebo_uuv_hippocampus` |
    | [船（USV：无人驾驶水面艇）](../simulation/gazebo_vehicles.md#usv)                                              | `make px4_sitl gazebo_boat`            |
    
    > **Note** 如果发生构建错误， [文件和代码安装指南](../setup/dev_env.md) 是一个有用的参考。
    
    以上指令启动了一个具有完整 UI 的载具。 其他选项包括：
    
    * 为了使 Gazebo 保持运行，请[分别启动 PX4 和 Gazebo](#start_px4_sim_separately)，并在需要时单独重新启动 PX4（比重新启动两者更快）。
    * 在 [无头模式](#headless) 运行仿真将不会启动 Gazebo UI（使用的资源更少，速度更快）。
    
    ## 让飞行器起飞
    
    上文的 `make` 指令首先构建PX4，然后同时运行 PX4 与 Gazebo 仿真器。
    
    PX4运行后将启动如下所示的 PX4 shell 脚本。
    
        ______  __   __    ___ 
        | ___ \ \ \ / /   /   |
        | |_/ /  \ V /   / /| |
        |  __/   /   \  / /_| |
        | |     / /^\ \ \___  |
        \_|     \/   \/     |_/
        
        px4 starting.
        
        INFO  [px4] Calling startup script: /bin/sh etc/init.d-posix/rcS 0
        INFO  [param] selected parameter default file eeprom/parameters_10016
        [param] Loaded: eeprom/parameters_10016
        INFO  [dataman] Unknown restart, data manager file './dataman' size is 11798680 bytes
        INFO  [simulator] Waiting for simulator to connect on TCP port 4560
        Gazebo multi-robot simulator, version 9.0.0
        Copyright (C) 2012 Open Source Robotics Foundation.
        Released under the Apache 2 License.
        http://gazebosim.org
        ...
        INFO  [ecl/EKF] 5188000: commencing GPS fusion
        
    
    控制台将打印出“PX4”的形状，加载指定机型的初始化和参数文件，等待（并连接到）仿真器。 一旦 INFO 打印出的 [ecl/EKF] 状态为 `commencing GPS fusion` ，则表明该载具已准备就绪可以解锁。
    
    > **Note** 在 Gazebo 中右击四旋翼模型允许从上下文菜单启用跟随模式，这样可以方便地将其保持在视图中。
    
    ![Gazebo UI](../../assets/simulation/gazebo/gazebo_follow.jpg)
    
    你可以通过输入以下指令让飞机起飞：
    
    ```sh
    pxh> commander takeoff
    ```
    
    ## 使用/配置选项
    
    ### 无头模式 {#headless}
    
    Gazebo 可以在 *无头(headless)* 模式下运行，在这种模式下 Gazebo UI 不会启动。 这会使 Gazebo 的启动速度更快并且使用更少的系统资源（即为一种更“轻量级”的运行仿真的方式）。
    
    只需在 `make` 指令前添加 `HEADLESS=1`，如下所示：
    
    ```bash
    HEADLESS=1 make px4_sitl gazebo_plane
    ```
    
    ### 设置自定义起飞位置 {#custom_takeoff_location}
    
    可以使用环境变量重写 SITL Gazebo 中的默认起飞位置。
    
    要设置的变量有：`PX4_HOME_LAT`、`PX4_HOME_LON` 和 `PX4_HOME_ALT`。
    
    例如：
    
        export PX4_HOME_LAT=28.452386
        export PX4_HOME_LON=-13.867138
        export PX4_HOME_ALT=28.5
        make px4_sitl gazebo
        
    
    ### 更改仿真速率
    
    可以通过设置环境变量 `PX4_SIM_SPEED_FACTOR` 来加快或减慢仿真环境相对于实际时间的流速。
    
        export PX4_SIM_SPEED_FACTOR=2
        make px4_sitl_default gazebo
        
    
    更多相关信息请参考：[Simulation > Run Simulation Faster than Realtime](../simulation/README.md#simulation_speed).
    
    ### 使用操纵杆
    
    通过 *QGroundControl* 可支持操纵杆或者拇指操纵杆（[设置说明在此](../simulation/README.md#joystickgamepad-integration)）。
    
    ### 提高距离传感器的性能
    
    当前的默认世界是 [PX4/sitl_gazebo/worlds/**iris.world**](https://github.com/PX4/sitl_gazebo/tree/master/worlds), 它使用高度图作为地面。
    
    这可能会在使用距离传感器时造成困难。 如果出现意外的结果，我们建议您将 **iris.model** 中的模型默认设定从 `uneven_ground` 改为 `asphalt_plane`。
    
    ### 模拟 GPS 噪声 {#gps_noise}
    
    Gazebo 可以模拟类似于实际系统中常见的 GPS 噪声（否则报告的GPS值将是无噪声/完美的）。 这在处理可能受 GPS 噪声影响的应用（例如精度定位）时非常有用。
    
    如果目标载具的 SDF 文件包含` gpsNoise `元素的值（值为 `<gpsNoise>true</gpsNoise>`），则GPS噪声将被模拟。 默认情况下, 它在许多载具的 SDF 文件中启用：**solo.sdf**、**iris.sdf**、**standard_vtol.sdf**、**delta_wing.sdf**、**plane.sdf**、**typhoon_h480** **tailsitter.sdf**。
    
    启用/禁用GPS噪音：
    
    1. 构建任何 gazebo 目标以生成 SDF 文件（适用于所有机型）。 例如： ```make px4_sitl gazebo_iris``` >**Tip** 在后续版本中不会覆盖 SDF 文件。
    2. 打开目标载具的 SDF 文件（例如**./Tools/sitl_gazebo/models/iris/iris.sdf **）。
    3. 搜索 `gpsNoise` 元素： 
            xml
            <plugin name='gps_plugin' filename='libgazebo_gps_plugin.so'>
             <robotNamespace/>
             <gpsNoise>true</gpsNoise>
            </plugin>
        
        * 如果已预设，则启用 GPS。 您可以通过删除以下行来禁用它：`<gpsNoise> true </gpsNoise>`
        * 如果未预设，则禁用 GPS 。 您可以通过将` gpsNoise `元素添加到` gps_plugin `部分来启用它（如上所示）。
    
    下次构建/重新启动 Gazebo 时，将使用新的 GPS 噪声设置。
    
    ## 加载指定的世界 {#set_world}
    
    PX4支持多个 [Gazebo Worlds](../simulation/gazebo_worlds.md) ，均存储在 [PX4/sitl_gazebo/worlds](https://github.com/PX4/sitl_gazebo/tree/master/worlds) 中。在默认情况下，Gazebo 显示的是一个平坦无特征的平面，如 [empty.world](https://github.com/PX4/sitl_gazebo/blob/master/worlds/empty.world) 中所定义。
    
    您可以在PX4配置目标中将它们指定为最终选项，进而加载任意世界。 例如，要加载 *warehouse* 世界，您可以在编译时添加如下信息：
    
        make px4_sitl_default gazebo_plane_cam__warehouse
        
    
    > **Note** 模型（`plane_cam`）后面有两个下划线，表示使用默认调试器（无）。 详见 [Building the Code > PX4 Make Build Targets](../setup/building_px4.md#make_targets)。
    
    您也可以使用环境变量 `PX4_SITL_WORLD` 来指定要加载的世界的完整路径。 如果您想测试一个暂未包含在PX4中的新世界，这种方式非常有用。
    
    ## 分别启动 Gazebo 和 PX4  {#start_px4_sim_separately}
    
    对于扩展开发会话而言，将 Gazebo 和 PX4 分别启动或在IDE中启动可能更加方便。
    
    除了现有 cmake 目标（使用 px4 的参数运行` sitl_run.sh `的以加载正确的模型）之外，它还会创建一个名为 `px4_<mode>` 的启动器目标，这是原始 sitl px4 应用程序的一个瘦包装器。这个瘦包装器只嵌入应用程序参数，如当前工作目录和模型文件的路径。
    
    分别启动 Gazebo 和 PX4：
    
    * 通过在终端中指定环境变量 `_ide` 来运行 gazebo（或任何其他 sim 卡）服务器和客户端查看器： 
            sh
            make px4_sitl gazebo___ide 或者 
        
            sh
            make px4_sitl gazebo_iris_ide
    
    * 在 IDE 中选择要调试的 `px4_<mode>` 目标（例如 `px4_iris` ）。
    * 直接从 IDE 启动调试会话。
    
    这种方法可显著缩短调试周期，因为仿真器（例如 Gazebo ）总是在后台运行，而您只需重新运行 px4 这个非常轻量的进程。
    
    ## 模拟的航测摄影机
    
    *Gazebo* 的航测摄像机模拟了一台 [MAVLink 摄像机](https://mavlink.io/en/services/camera.html)，它捕捉地理标记的JPEG图像，并将摄像机捕捉到的信息发送到所连接的地面站。 该摄像机同时支持视频流。 它可用于测试摄像机的拍摄性能，特别是在航测任务中。
    
    每当捕获到图像时，相机都会发送出 [CAMERA_IMAGE_CAPTURED](https://mavlink.io/en/messages/common.html#CAMERA_IMAGE_CAPTURED) 消息。 捕获的图像将被保存到：**Firmware/build/px4_sitle_default/tmp/frames/DSC_n_.jpg** (*n* 从 00000 开始，每当捕获到新的图像该数值便会增加一次)。
    
    模拟一个带有该相机的飞机：
    
        make px4_sitl_default gazebo_plane_cam
        
    
    > **Note** 摄像机还支持/响应以下 MAVLink 指令： [MAV_CMD_REQUEST_CAMERA_CAPTURE_STATUS](https://mavlink.io/en/messages/common.html#MAV_CMD_REQUEST_CAMERA_CAPTURE_STATUS), [MAV_CMD_REQUEST_STORAGE_INFORMATION](https://mavlink.io/en/messages/common.html#MAV_CMD_REQUEST_STORAGE_INFORMATION), [MAV_CMD_REQUEST_CAMERA_SETTINGS](https://mavlink.io/en/messages/common.html#MAV_CMD_REQUEST_CAMERA_SETTINGS), [MAV_CMD_REQUEST_CAMERA_INFORMATION](https://mavlink.io/en/messages/common.html#MAV_CMD_REQUEST_CAMERA_INFORMATION), [MAV_CMD_RESET_CAMERA_SETTINGS](https://mavlink.io/en/messages/common.html#MAV_CMD_RESET_CAMERA_SETTINGS), [MAV_CMD_STORAGE_FORMAT](https://mavlink.io/en/messages/common.html#MAV_CMD_STORAGE_FORMAT), [MAV_CMD_SET_CAMERA_ZOOM](https://mavlink.io/en/messages/common.html#MAV_CMD_SET_CAMERA_ZOOM), [MAV_CMD_IMAGE_START_CAPTURE](https://mavlink.io/en/messages/common.html#MAV_CMD_IMAGE_START_CAPTURE), [MAV_CMD_IMAGE_STOP_CAPTURE](https://mavlink.io/en/messages/common.html#MAV_CMD_IMAGE_STOP_CAPTURE), [MAV_CMD_REQUEST_VIDEO_STREAM_INFORMATION](https://mavlink.io/en/messages/common.html#MAV_CMD_REQUEST_VIDEO_STREAM_INFORMATION), [MAV_CMD_REQUEST_VIDEO_STREAM_STATUS](https://mavlink.io/en/messages/common.html#MAV_CMD_REQUEST_VIDEO_STREAM_STATUS), [MAV_CMD_SET_CAMERA_MODE](https://mavlink.io/en/messages/common.html#MAV_CMD_SET_CAMERA_MODE).
    
    

<span></span>

    
    > **Note** 模拟相机的执行代码见 [PX4/sitl_gazebo/src/gazebo_geotagged_images_plugin.cpp](https://github.com/PX4/sitl_gazebo/blob/master/src/gazebo_geotagged_images_plugin.cpp)。
    
    ## 仿真降落/飞行终止 {#flight_termination}
    
    *Gazebo* 可用于仿真在[飞行终止](https://docs.px4.io/master/en/advanced_config/flight_termination.html)期间展开[降落伞](https://docs.px4.io/master/en/peripherals/parachute.html)（飞行终止由 *Gazebo* 中模拟的 PWM 指令触发）。
    
    `IF750A` 机身上装有降落伞。 仿真该型飞行器，请运行以下指令：
    
        make px4_sitl gazebo_if750a
        
    
    要使车辆进入飞行终止状态，您可以强制使其在[安全检查](https://docs.px4.io/master/en/config/safety.html)中失败，并确保飞行终止设置为故障保护动作。 比如，您可以通过强制设置 [侵犯地理围栏](https://docs.px4.io/master/en/config/safety.html#geofence-failsafe) 来做到这一点。
    
    获取更多信息，请见：
    
    * [飞行终止](https://docs.px4.io/master/en/advanced_config/flight_termination.html) 
    * [降落伞](https://docs.px4.io/master/en/peripherals/parachute.html)
    * [ 安全配置（故障保护） ](https://docs.px4.io/master/en/config/safety.html)
    
    ## 视频流 {#video}
    
    Gazebo 的 PX4 SITL 支持来自连接到设备模型的 Gazebo 相机传感器的 DP 视频流。 启用视频流后，您可以从 *QGroundControl*（在UDP端口5600上）连接到它，并观看在 Gazebo 环境下该仿真载具的视频，就像从真实的摄像机观看一样。 视频使用 *GStreamer* 管道进行流式传输，并且可以使用 Gazebo UI 中的按钮进行启用/禁用。
    
    Gazebo 摄像机传感器可在以下机型上被支持/启用：
    
    * [Typhoon H480](#typhoon_h480)
    
    ### 必备组件
    
    视频流需要 *GStreamer 1.0*。 [当您设置 Gazebo 时](#installation)，所需的依赖项应该已经安装完毕（它们包含在 macOS 和 UbuntuLinux 系统下的标准 PX4 安装脚本/说明中）。
    
    > **Note** 仅供参考，依赖项包括：`gstreamer1.0-plugins-base`，`gstreamer1.0-plugins-good`，`gstreamer1.0-plugins-bad`，`gstreamer1.0-plugins-ugly`，`libgstreamer-plugins-base1.0-dev`。
    
    ### 启动/停止视频流
    
    对支持的目标载具，视频流将自动启动。 例如，要在台风 Typhoon H480 上启动视频流：
    
        make px4_sitl gazebo_typhoon_h480
        
    
    可以使用 Gazebo 界面中的 打开/关闭 按钮来 启用/禁用 视频流。
    
    ![视频 打开/关闭 按钮](../../assets/simulation/gazebo/sitl_video_stream.png)
    
    ### 如何查看 Gazebo 视频
    
    最简单的方式是在 *QGroundControl* 中查看 SITL/Gazebo 摄像机视频流。 只需打开 QGroundControl 中 **软件配置> 通用设置** 找到 **视频源** 选择 *UDP 视频流* 然后将 **UDP 端口号** 默认设置为 *5600*：
    
    ![Gazebo 的 QGC 视频流设置](../../assets/simulation/gazebo/qgc_gazebo_video_stream_udp.png)
    
    来自 Gazebo 的视频会像真实相机一样显示在* QGroundControl* 中。
    
    ![QGC 视频流 Gazebo 示例](../../assets/simulation/gazebo/qgc_gazebo_video_stream_typhoon.jpg)
    
    > **Note** 台风世界模型不是一个好的选择。
    
    也可以使用 *Gstreamer 管道* 观看视频。 只需在终端中输入如下指令：
    
    ```sh
    gst-launch-1.0  -v udpsrc port=5600 caps='application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264' \
    ! rtph264depay ! avdec_h264 ! videoconvert ! autovideosink fps-update-interval=1000 sync=false
    ```
    
    ## 扩展与定制
    
    若要扩展或者是自定义仿真接口，请编辑 `Tools/sitl_gazebo` 文件夹。 源码也可以在 [sitl_gazebo 库](https://github.com/px4/sitl_gazebo) 上获取。
    
    > **Note** 构建系统会强制执行正确的 GIT 子模块，包括仿真器。 这不会覆盖目录文件中的改动。
    
    ## 更多信息
    
    * [ROS 与 Gazebo 仿真](../simulation/ros_interface.md)
    * [Gazebo Octomap](../simulation/gazebo_octomap.md)