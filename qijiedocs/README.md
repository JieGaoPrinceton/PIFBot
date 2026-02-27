# PIFBot 技术文档索引

## 文档概述

本目录包含 PIFBot 项目的完整技术分析文档，涵盖项目概述、系统架构、硬件、固件、ROS 软件、机械设计以及部署指南等方面。

## 文档列表

| 序号 | 文档名称 | 主要内容 |
|:----:|---------|---------|
| 01 | [项目概述](./01_项目概述.md) | 项目简介、核心功能、技术栈、规格参数、项目特点 |
| 02 | [系统架构](./02_系统架构.md) | 整体架构设计、数据流分析、ROS 节点架构、通信协议、电源架构 |
| 03 | [硬件系统分析](./03_硬件系统分析.md) | OpenSTM32 控制板、传感器系统、驱动系统、计算系统、电源系统 |
| 04 | [固件系统分析](./04_固件系统分析.md) | STM-IDF 框架、里程计模块、运动控制、rosserial 通信、PID 控制器 |
| 05 | [ROS软件系统分析](./05_ROS软件系统分析.md) | Catkin 工作空间、功能包结构、话题与服务、TF 变换、SLAM/导航配置 |
| 06 | [机械设计分析](./06_机械设计分析.md) | 机械架构、层板设计、底盘设计、驱动系统、装配关系、制造文件 |
| 07 | [项目部署指南](./07_项目部署指南.md) | 环境准备、项目构建、硬件组装、固件烧录、软件配置、调试排错 |
| 08 | [STM32核心系统技术文档](./08_STM32核心系统技术文档.md) | **以STM32为核心**：硬件接口、固件架构、外设驱动、ROS通信、运动学模型 |

## 项目快速导航

### 核心技术栈

```
┌─────────────────────────────────────────────────────────────┐
│                        PIFBot 技术栈                         │
├─────────────────────────────────────────────────────────────┤
│  操作系统    │ Ubuntu 16.04 LTS                             │
│  机器人框架  │ ROS Kinetic Kame                             │
│  嵌入式框架  │ STM-IDF (STM32)                              │
│  主控制器    │ STM32F4 (ARM Cortex-M4 with FPU)             │
│  单板计算机  │ Raspberry Pi 3 B+                            │
│  传感器      │ RPLIDAR A1 + Intel RealSense D415 + MPU9250  │
│  驱动方式    │ 差速驱动                                     │
│  SLAM        │ GMapping / RTAB-Map                          │
│  导航        │ move_base + DWA                              │
│  许可证      │ MIT License                                  │
└─────────────────────────────────────────────────────────────┘
```

### 机器人规格速查

| 参数 | 数值 |
|-----|------|
| 底盘直径 | 200 mm |
| 最大高度 | 212 mm |
| 轮子直径 | 65 mm |
| 轴距 | 165 mm |

### 关键文件位置

```
PIFBot/
├── README.md                    # 项目说明
├── LICENSE                      # MIT 许可证
├── assets/demo/                 # 演示资源
│   └── openSTM32F4_LQFP64.pdf   # 控制板原理图
├── firmware/                    # 固件代码
│   └── ros-stm32-base-control/
│       ├── Core/                # STM32 外设配置
│       ├── HwIntf/              # 硬件接口层
│       ├── ros-base-control-fw/ # 底盘控制核心
│       └── rosserial_stm32/     # rosserial 库
├── hardware/                    # 硬件设计
│   └── openSTM32F4_LQFP64/
│       ├── *.sch                # KiCad 原理图
│       ├── *.kicad_pcb          # PCB 设计
│       └── assets/gerber/       # Gerber 生产文件
├── mechanical/                  # 机械设计
│   ├── *.FCStd                  # FreeCAD 设计文件
│   ├── assembly/                # 装配模型
│   └── assets/dxf/              # DXF 切割文件
├── ros/                         # ROS 工作空间
│   └── src/ros-slam-nav/        # ROS 包
└── qijiedocs/                   # 技术文档 (本目录)
```

## 文档版本

- **创建日期**: 2026-02-27
- **基于项目版本**: master 分支
- **文档作者**: AI 分析生成

## 使用说明

1. 阅读顺序建议：01 → 02 → 03/04/05/06 → 07 → 08
2. 如需深入了解 STM32 核心，重点阅读文档 08
3. 实际部署前请先初始化 Git 子模块：`git submodule update --init --recursive`
4. 各文档中包含的配置参数和代码片段仅供参考，实际使用时需根据具体环境调整

## 相关链接

- **项目仓库**: https://github.com/phonght32/PIFBot
- **OpenSTM32 仓库**: https://github.com/phonght32/openSTM32F4_LQFP64
- **固件仓库**: https://github.com/phonght32/ros-stm32-base-control
- **ROS Wiki**: http://wiki.ros.org/kinetic
- **演示视频**: https://youtu.be/UWhPCQyU918
