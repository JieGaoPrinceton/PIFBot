# PIFBot ROS 软件系统分析

## 1. ROS 系统架构

PIFBot 基于 ROS Kinetic 构建，采用标准的 ROS 节点通信架构。

```
┌────────────────────────────────────────────────────────────────┐
│                      ROS 系统架构                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    ROS Master                            │  │
│  │               (节点注册 / 话题管理)                        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                 │
│         ┌────────────────────┼────────────────────┐           │
│         │                    │                    │           │
│         ▼                    ▼                    ▼           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐   │
│  │ 传感器节点   │    │ 控制节点    │    │ 算法节点        │   │
│  │             │    │             │    │                 │   │
│  │ • rplidar   │    │ • rosserial │    │ • slam_gmapping │   │
│  │ • realsense │    │ • cmd_vel   │    │ • rtabmap       │   │
│  │             │    │             │    │ • move_base     │   │
│  └─────────────┘    └─────────────┘    └─────────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

## 2. Catkin 工作空间结构

```
ros/
├── .catkin_workspace      # Catkin 工作空间标记
├── src/                   # 源代码目录
│   ├── ros-slam-nav/      # SLAM 和导航包 (Git Submodule)
│   │   ├── pifbot_bringup/
│   ├── CMakeLists.txt
│   └── ...
├── build/                 # 编译输出 (gitignore)
├── devel/                 # 开发环境 (gitignore)
└── logs/                  # 日志文件 (gitignore)
```

### 2.1 编译配置

```bash
# 初始化工作空间
cd ros/
catkin_init_workspace

# 编译
catkin_make

# 或者使用 catkin build (推荐)
catkin build

# 激活环境
source devel/setup.bash
```

## 3. ROS 功能包结构

### 3.1 典型功能包组织

```
ros-slam-nav/
├── pifbot_bringup/           # 启动文件和配置
│   ├── launch/
│   │   ├── minimal.launch    # 最小启动
│   │   ├── sensor.launch     # 传感器启动
│   │   └── full.launch       # 完整启动
│   └── config/
│       ├── odometry.yaml
│       └── motor_params.yaml
│
├── pifbot_description/       # 机器人模型
│   ├── urdf/
│   │   └── pifbot.urdf
│   ├── meshes/
│   └── launch/
│       └── display.launch
│
├── pifbot_navigation/        # 导航配置
│   ├── launch/
│   │   └── move_base.launch
│   ├── config/
│   │   ├── local_costmap_params.yaml
│   │   ├── global_costmap_params.yaml
│   │   └── dwa_local_planner_params.yaml
│   └── maps/
│
└── pifbot_slam/              # SLAM 配置
    ├── launch/
    │   ├── gmapping.launch
    │   └── rtabmap.launch
    └── config/
```

## 4. 核心话题与服务

### 4.1 话题列表

| 话题名称 | 消息类型 | 发布者 | 订阅者 | 频率 |
|---------|---------|-------|-------|------|
| `/scan` | `sensor_msgs/LaserScan` | rplidar_node | slam, navigation | 5.5 Hz |
| `/odom` | `nav_msgs/Odometry` | rosserial | slam, navigation | 50 Hz |
| `/cmd_vel` | `geometry_msgs/Twist` | navigation | rosserial | 可变 |
| `/imu` | `sensor_msgs/Imu` | rosserial | (可选) | 100 Hz |
| `/map` | `nav_msgs/OccupancyGrid` | slam | navigation | 按需 |
| `/camera/depth/points` | `sensor_msgs/PointCloud2` | realsense | rtabmap | 30 Hz |
| `/tf` | `tf2_msgs/TFMessage` | 各节点 | 所有节点 | 50 Hz |

### 4.2 服务列表

| 服务名称 | 服务类型 | 功能 |
|---------|---------|------|
| `/rosserial/get_loggers` | roscpp/GetLoggers | 获取日志 |
| `/rosserial/set_logger_level` | roscpp/SetLoggerLevel | 设置日志级别 |
| `/map_server/load_map` | nav_msgs/LoadMap | 加载地图 |

## 5. TF 变换树

### 5.1 完整 TF 树

```
map
 └── odom (由 SLAM 发布)
      └── base_link (由里程计发布)
           ├── base_footprint
           ├── laser_link (RPLIDAR)
           ├── camera_link (RealSense)
           │    ├── camera_depth_frame
           │    └── camera_rgb_frame
           ├── imu_link
           ├── left_wheel_link
           └── right_wheel_link
```

### 5.2 URDF 模型片段

```xml
<?xml version="1.0"?>
<robot name="pifbot">
  
  <!-- Base Link -->
  <link name="base_link">
    <visual>
      <geometry>
        <cylinder radius="0.1" length="0.15"/>
      </geometry>
      <material name="blue"/>
    </visual>
    <collision>
      <geometry>
        <cylinder radius="0.1" length="0.15"/>
      </geometry>
    </collision>
    <inertial>
      <mass value="5.0"/>
      <inertia ixx="0.03" ixy="0.0" ixz="0.0"
               iyy="0.03" iyz="0.0" izz="0.05"/>
    </inertial>
  </link>
  
  <!-- Laser Link -->
  <link name="laser_link"/>
  
  <joint name="laser_joint" type="fixed">
    <parent link="base_link"/>
    <child link="laser_link"/>
    <origin xyz="0.0 0.0 0.12" rpy="0.0 0.0 0.0"/>
  </joint>
  
  <!-- Camera Link -->
  <link name="camera_link"/>
  
  <joint name="camera_joint" type="fixed">
    <parent link="base_link"/>
    <child link="camera_link"/>
    <origin xyz="0.08 0.0 0.18" rpy="0.0 0.0 0.0"/>
  </joint>
  
</robot>
```

## 6. SLAM 配置

### 6.1 GMapping 配置

```yaml
# gmapping_params.yaml
map_update_interval: 2.0
maxUrange: 12.0
maxRange: 12.0
minimumScore: 50
sigma: 0.05
kernelSize: 1
lstep: 0.05
astep: 0.05
iterations: 5
lsigma: 0.075
ogain: 3.0
lskip: 0
srr: 0.1
srt: 0.2
str: 0.1
stt: 0.2
linearUpdate: 1.0
angularUpdate: 0.5
temporalUpdate: 3.0
resampleThreshold: 0.5
particles: 30
xmin: -5.0
ymin: -5.0
xmax: 5.0
ymax: 5.0
delta: 0.05
llsamplerange: 0.01
llsamplestep: 0.01
lasamplerange: 0.005
lasamplestep: 0.005
```

### 6.2 RTAB-Map 配置 (3D SLAM)

```yaml
# rtabmap_params.yaml
Mem/IncrementalMemory: true
Mem/InitWMWithAllNodes: false
RGBD/NeighborLinkRefining: true
RGBD/ProximityBySpace: true
RGBD/AngularUpdate: 0.05
RGBD/LinearUpdate: 0.05
RGBD/OptimizeFromGraphEnd: false
Reg/Strategy: 1
Vis/MinInliers: 15
Vis/MaxFeatures: 400
```

## 7. 导航配置

### 7.1 Move Base 配置

```yaml
# move_base_params.yaml
controller_frequency: 10.0
controller_patience: 15.0
planner_frequency: 5.0
planner_patience: 5.0
conservative_reset_dist: 3.0
oscillation_timeout: 10.0
oscillation_distance: 0.2
recovery_behavior_enabled: true
clearing_rotation_allowed: true
```

### 7.2 Costmap 配置

```yaml
# global_costmap_params.yaml
global_costmap:
  global_frame: /map
  robot_base_frame: /base_link
  update_frequency: 2.0
  publish_frequency: 1.0
  static_map: true
  rolling_window: false
  resolution: 0.05
  transform_tolerance: 0.5
  plugins:
    - {name: static_layer, type: "costmap_2d::StaticLayer"}
    - {name: inflation_layer, type: "costmap_2d::InflationLayer"}
  inflation_radius: 0.55
  cost_scaling_factor: 10.0

# local_costmap_params.yaml
local_costmap:
  global_frame: /odom
  robot_base_frame: /base_link
  update_frequency: 5.0
  publish_frequency: 2.0
  static_map: false
  rolling_window: true
  width: 3.0
  height: 3.0
  resolution: 0.05
  transform_tolerance: 0.5
  plugins:
    - {name: obstacle_layer, type: "costmap_2d::ObstacleLayer"}
    - {name: inflation_layer, type: "costmap_2d::InflationLayer"}
```

### 7.3 DWA 局部规划器

```yaml
# dwa_local_planner_params.yaml
DWAPlannerROS:
  max_vel_x: 0.3
  min_vel_x: -0.1
  max_vel_y: 0.0
  min_vel_y: 0.0
  max_vel_trans: 0.3
  min_vel_trans: 0.1
  max_vel_theta: 1.0
  min_vel_theta: 0.4
  
  acc_lim_x: 0.5
  acc_lim_y: 0.0
  acc_lim_theta: 2.0
  
  sim_time: 1.5
  vx_samples: 10
  vy_samples: 0
  vtheta_samples: 20
  
  path_distance_bias: 32.0
  goal_distance_bias: 24.0
  occdist_scale: 0.01
  
  holonomic_robot: false
```

## 8. 启动文件分析

### 8.1 最小系统启动

```xml
<!-- minimal.launch -->
<launch>
  <!-- rosserial 节点 -->
  <node name="rosserial" pkg="rosserial_python" type="serial_node.py">
    <param name="port" value="/dev/ttyUSB0"/>
    <param name="baud" value="115200"/>
  </node>
  
  <!-- 机器人模型 -->
  <param name="robot_description" command="$(find xacro)/xacro '$(find pifbot_description)/urdf/pifbot.urdf.xacro'"/>
  <node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher"/>
</launch>
```

### 8.2 SLAM 启动

```xml
<!-- slam.launch -->
<launch>
  <include file="$(find pifbot_bringup)/launch/minimal.launch"/>
  
  <!-- RPLIDAR -->
  <node name="rplidarNode" pkg="rplidar_ros" type="rplidarNode" output="screen">
    <param name="serial_port" value="/dev/ttyUSB1"/>
    <param name="serial_baudrate" value="115200"/>
    <param name="frame_id" value="laser_link"/>
    <param name="inverted" value="false"/>
    <param name="angle_compensate" value="true"/>
  </node>
  
  <!-- GMapping -->
  <node pkg="gmapping" type="slam_gmapping" name="slam_gmapping" output="screen">
    <rosparam file="$(find pifbot_slam)/config/gmapping_params.yaml" command="load"/>
  </node>
</launch>
```

### 8.3 导航启动

```xml
<!-- navigation.launch -->
<launch>
  <include file="$(find pifbot_bringup)/launch/minimal.launch"/>
  
  <!-- 地图服务器 -->
  <node name="map_server" pkg="map_server" type="map_server" args="$(find pifbot_navigation)/maps/mymap.yaml"/>
  
  <!-- AMCL -->
  <include file="$(find amcl)/examples/amcl_diff.launch"/>
  
  <!-- Move Base -->
  <node pkg="move_base" type="move_base" respawn="false" name="move_base" output="screen">
    <rosparam file="$(find pifbot_navigation)/config/costmap_common.yaml" command="load" ns="global_costmap"/>
    <rosparam file="$(find pifbot_navigation)/config/costmap_common.yaml" command="load" ns="local_costmap"/>
    <rosparam file="$(find pifbot_navigation)/config/global_costmap_params.yaml" command="load"/>
    <rosparam file="$(find pifbot_navigation)/config/local_costmap_params.yaml" command="load"/>
    <rosparam file="$(find pifbot_navigation)/config/dwa_local_planner_params.yaml" command="load"/>
  </node>
</launch>
```

## 9. 依赖包列表

```xml
<!-- package.xml 依赖示例 -->
<package>
  <buildtool_depend>catkin</buildtool_depend>
  
  <depend>roscpp</depend>
  <depend>rospy</depend>
  <depend>std_msgs</depend>
  <depend>sensor_msgs</depend>
  <depend>geometry_msgs</depend>
  <depend>nav_msgs</depend>
  <depend>tf</depend>
  <depend>tf2_ros</depend>
  
  <exec_depend>rosserial_python</exec_depend>
  <exec_depend>rplidar_ros</exec_depend>
  <exec_depend>realsense2_camera</exec_depend>
  <exec_depend>gmapping</exec_depend>
  <exec_depend>rtabmap_ros</exec_depend>
  <exec_depend>move_base</exec_depend>
  <exec_depend>amcl</exec_depend>
  <exec_depend>map_server</exec_depend>
  <exec_depend>robot_state_publisher</exec_depend>
</package>
```

## 10. ROS 仓库

ROS 源代码位于 Git 子模块中：

```
ros/src/ros-slam-nav/
```

**注意**: 需要初始化 Git 子模块以获取完整源代码：
```bash
cd /Users/ll/PIFBot
git submodule update --init --recursive
```
