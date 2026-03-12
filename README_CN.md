# LingkBot: A Hierarchical Embodied AI Orchestration System for ROS 2, Nav2, RTAB-Map, YOLO-World, RAG and VLM Robotics

[![ROS 2 Jazzy](https://img.shields.io/badge/ROS2-Jazzy-blue)](https://docs.ros.org/en/jazzy/)
![Hardware](https://img.shields.io/badge/Robot-LeoRover-orange)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Medium Deep Dive](https://img.shields.io/badge/Deep%20Dive-Medium-black)](https://medium.com/@qq693180059lrf/00-from-classical-robotics-to-embodied-agents-49b8b247e253)

LingkBot is a `ROS 2 Jazzy` robotics system integration project centered on embodied AI, semantic navigation, mobile manipulation, robot perception, and sim2real robustness. It is built around `LeoRover + myCobot 280 Pi + Intel RealSense D435 + RPLidar`, and connects `SLAM Toolbox`, `Cartographer`, `Nav2`, `RTAB-Map`, `YOLO-World`, `RGB-D 3D projection`, `MoveIt 2`, `MoveIt Task Constructor (MTC)`, `RAG memory pipeline`, `VLM orchestrator`, `ROS 2 Components`, and real-robot watchdog layers into one hierarchical embodied agent workflow.

一个基于 `ROS 2 Jazzy` 的个人级移动操作与具身机器人系统集成项目，聚焦 `LeoRover + myCobot + Intel RealSense D435 + RPLidar` 这一套机器人平台。项目覆盖 `SLAM Toolbox`、`Cartographer`、`Nav2`、`RTAB-Map`、`YOLO-World`、`RGB-D 3D projection`、`MoveIt 2`、`MTC`、`RAG memory pipeline`、`VLM orchestrator`、`ROS 2 Components` 和 `sim2real robustness` 等关键链路。

绝大部分能力先在仿真中打通，再逐步推进到真机 bringup、watchdog、map jump protection 和 kidnapped robot recovery。  
这一页是整套系列文章的总入口，目标不是展示某个单点模型，而是系统化记录一条从 classical robotics 到 hierarchical embodied agent 的演化路径。

**视频展示：** [YouTube](https://www.youtube.com/watch?v=-muKKy4nznA)

> **开源计划：** 项目计划在 **2026 年 9 月之后** 开源，系列文章里也会对开源相关安排和范围做更多说明。

**Keywords:** `Embodied AI`, `ROS 2`, `Nav2`, `SLAM Toolbox`, `Cartographer`, `RTAB-Map`, `YOLO-World`, `MoveIt 2`, `MoveIt Task Constructor`, `MTC`, `RAG`, `VLM Orchestrator`, `Semantic Navigation`, `Mobile Manipulation`, `LeoRover`, `myCobot`, `RealSense D435`, `RPLidar`, `Robot System Integration`, `Sim2Real`, `Behavior Tree`, `ROS 2 Components`

重点不是单点算法，而是把：

`仿真 -> 建图/定位 -> 自主探索 -> 视觉识别 -> 3D目标恢复 -> 机械臂抓取 -> 语义地图 -> 场景记忆 -> 语言导航 -> VLM具身编排 -> 真机保护层`

尽量压进同一套能运行、能演示、能继续扩展的机器人系统里。

## System Stack
| 模块 | 组成 |
| --- | --- |
| **机器人平台** | LeoRover, myCobot 280 Pi, RPLidar, Intel RealSense D435, Intel NUC |
| **仿真与可视化** | ROS 2 Jazzy, Gazebo, RViz2 |
| **定位、建图与导航** | SLAM Toolbox, Cartographer, Nav2, RTAB-Map, Frontier Exploration, Behavior Trees |
| **感知与空间理解** | YOLOv8 / YOLOv11-Seg, YOLO-World, RGB-D depth fusion, TF2 2D-to-3D projection, Mediapipe Hands |
| **移动操作与抓取** | MoveIt 2, MoveIt Task Constructor (MTC), myCobot manipulation pipeline |
| **具身决策与记忆** | Gemini 2.5 Flash, VLM Orchestrator, RAG memory pipeline, semantic navigation |
| **系统架构与工程化** | ROS 2 Components, Composition, lifecycle orchestration, lightweight BT mission flow |
| **真机鲁棒性** | cmd_vel guard, map jump watchdog, map snapshot manager, sim2real bringup |
| **3D 表征扩展** | ROSplat, 3D Gaussian Splatting (3DGS) |

- 中文文档：[`README_CN.md`](README_CN.md)
- English version: [`README.md`](README.md)

## Quick Navigation

- `00` 总序：从经典机器人到具身代理
- `01-02` 基础闭环：SLAM、Nav2、自主探索、移动操作、抓取
- `03-05` 语义导航：3D 重定位、YOLO-World 对象地图、RAG 场景记忆
- `06` 分层具身代理：VLM 在线控制权分配与模块化编排
- `07-08` 真机与鲁棒性：sim2real、watchdog、跳变保护、稳定地图补丁
- `09-10` 支线扩展：ROSplat / 3DGS、手势控制机械臂
- `11` 终章：整套路线上层总结、比较与定位

---

## 项目到底实现了什么

### 1. 经典机器人底座
- `slam_toolbox / cartographer` 建图
<p align="center">
  <img src="assets/readme/slam.gif" alt="ROS 2 SLAM Toolbox mapping demo on LeoRover" width="48%" />
  <img src="assets/readme/cartographer.gif" alt="ROS 2 Cartographer mapping demo on LeoRover" width="48%" />
</p>
- `Nav2` 定位与导航
- 基于 frontier 的自主探索（自定义 `pink_ball_explorer`）（不展示Explore-Lite了）

<p align="center">
  <img src="assets/readme/showcase_demo_01_frontier_exploration.gif" alt="Frontier exploration demo with Nav2 and pink_ball_explorer" width="78%" />
</p>

对应文章：
- [`01 经典路径：从零跑通 SLAM 建图与自主探索`](01_the_physical_foundation/01_slam_nav2_exploration.md)

### 2. 移动操作闭环
- 机器人在巡逻/探索中发现目标
- `YOLO + RGB-D + TF` 把 2D 检测结果压成可抓取的 3D 坐标
- `myCobot + MoveIt2 + MTC` 完成顶抓链路
- 支持探索、巡逻、视觉接近、人工确认、抓取、恢复这一整套状态切换

<p align="center">
  <img src="assets/readme/showcase_demo_02_mobile_manipulation.gif" alt="Mobile manipulation demo with YOLO RGB-D TF2 MoveIt 2 and MTC" width="78%" />
</p>

对应文章：
- [`02 移动操作：导航、识别、抓取全链路闭环`](01_the_physical_foundation/02_full_workflow_vision_grasping.md)

### 3. 三条语义导航路线
- `RTAB-Map` 3D 重定位与 kidnapped robot problem 兜底
<p align="center">
  <img src="assets/readme/showcase_demo_03_semantic_mapping.gif" alt="Semantic mapping and RTAB-Map 3D reconstruction demo" width="48%" />
  <img src="assets/readme/03_kidnapped_robot_problem_demo_01.gif" alt="Kidnapped robot recovery demo using RTAB-Map relocalization" width="48%" />
</p>
- Route A: `YOLO-World` 物体级在线打标导航
- Route B: `LLM/VLM + 关键帧 + RAG` 场景记忆导航
<p align="center">
  <img src="assets/readme/showcase_demo_04_scene_memory_navigation.gif" alt="Scene memory navigation demo with keyframes RAG and grounding" width="48%" />
</p>
- Route C: `VLM orchestrator` 在线高层语义决策与具身控制接管
<p align="center">
  <img src="assets/readme/showcase_demo_05_vlm_orchestrator.gif" alt="VLM orchestrator demo for online semantic decision making" width="48%" />
</p>

对应文章：
- [`03 绑架问题：当 2D 导航失忆时，我让 3D 地图出来兜底`](02_semantic_navigation_paradigms/03_kidnapped_robot_problem.md)
- [`04 路线 A：基于 YOLO-World 的物体级在线打标导航`](02_semantic_navigation_paradigms/04_yolo_world_semantic_mapping.md)
- [`05 路线 B：基于 LLM 关键帧分析与 RAG 检索的场景记忆导航`](02_semantic_navigation_paradigms/05_llm_rag_scene_memory_navigation.md)

### 4. 分层具身代理
- `VLM orchestrator` 不再只是做“看图回答问题”，而是在线进入控制环，直接参与探索、锁定、对齐、接近与抓取阶段的控制权分配
- 机器人会结合当前视觉上下文、任务目标和局部状态，动态决定下一步该由谁接管身体，这已经不是简单的 AI 功能包，而是一套真正在线运行的 hierarchical embodied agent
- 这条路线最值钱的地方，不是“单模型端到端”神话，而是把高层多模态理解、经典导航、局部视觉闭环和抓取执行，真正组织成了一条连续的具身执行链

<p align="center">
  <img src="assets/readme/showcase_demo_06_vlm_orchestrator_v1.gif" alt="Embodied agent demo version 1 with online VLM control handoff" width="48%" />
  <img src="assets/readme/showcase_demo_07_vlm_orchestrator_v2.gif" alt="Embodied agent demo version 2 with hierarchical orchestration" width="48%" />
</p>

进一步采用 `Component + Composition + 轻量 BT mission flow` 做系统级解耦，把 `Perception / Reasoning / Navigation / Alignment / Grasp / Supervision` 拆成职责清晰的模块，让这套系统从“一个能打的 demo”升级成“可扩展、可维护、可接管的 embodied architecture”。
<p align="center">
  <img src="assets/readme/06_zero_shot_vlm_orchestrator_figure_01.png" alt="Hierarchical embodied architecture with ROS 2 Components and composition" width="48%" />
  <img src="assets/readme/showcase_demo_08_bt_mission_flow.gif" alt="BT任务流演示" width="48%" />
</p>

对应文章：
- [`06 VLM 在线接管的分层 Embodied Agent`](03_the_embodied_agent/06_zero_shot_vlm_orchestrator.md)
- [`07 跨越虚实：LeoRover 真机耦合，不是把仿真搬过去`](04_sim2real_and_robustness/07_sim2real_hardware_coupling.md)
- [`08 真机不死之身：Watchdog、跳变保护与稳定地图补丁`](04_sim2real_and_robustness/08_watchdog_and_safety_patches.md)

### 5. 探索扩展
- `ROSplat / 3DGS` 并行表征层

<p align="center">
  <img src="assets/readme/showcase_demo_09_rosplat_visualization.gif" alt="ROSplat and 3D Gaussian Splatting visualization demo" width="48%" />
</p>

- `Mediapipe` 手势控制机械臂

<p align="center">
  <img src="assets/readme/showcase_demo_10_gesture_control_arm.gif" alt="Gesture controlled robot arm demo with Mediapipe Hands" width="48%" />
</p>

对应文章：
- [`09 ROSplat 与 3DGS：我为什么还是想把高斯地图接进来`](05_side_quests/09_3dgs_rosplat_visualization.md)
- [`10 手势控制机械臂：看到网上喷漆机器人以后，我也手痒了`](05_side_quests/10_gesture_control_arm.md)

### 6. 总结-可以从这里倒叙开始阅读
- 这不是一个新算法项目
- 这是一部个人级具身系统集成进化史
- 也是一个典型的“缝合怪为什么稀缺”的案例

对应文章：
- [`11 这不是三篇技术路线总结，而是一部平民具身系统的进化史`](06_epilogue/11_semantic_navigation_paradigm_comparison.md)

---

## 这个项目不是什么

- 不是单篇论文式的新算法项目
- 不是工业级产品
- 不是一个已经完全真机闭环量产化的系统

## 这个项目是什么

- 一个个人级机器人系统集成项目
- 一条从经典机器人到具身代理的演化路线
- 一个尽量把“移动机器人 + 机械臂 + 视觉 + 语义 + 语言 + 真机保护层”压进同一系统的尝试

## Search-Friendly Summary

如果你在搜索下面这些方向，这个系列基本都覆盖了可运行案例、架构思路或踩坑记录：

- `ROS 2 embodied AI project`
- `LeoRover Nav2 SLAM Toolbox RTAB-Map`
- `YOLO-World semantic navigation robot`
- `RAG memory navigation robotics`
- `MoveIt 2 MTC mobile manipulation`
- `VLM orchestrator robot control`
- `kidnapped robot recovery RTAB-Map`
- `sim2real robotics watchdog map jump protection`
- `ROS 2 Components composition behavior tree robot architecture`

如果想看这套项目背后的完整进化史，建议直接从这里开始：

- [`00 从经典机器人到具身代理`](00_preface_and_structure/00_from_classical_robotics_to_embodied_agents.md)

或者直接看总结
- [`11 这不是三篇技术路线总结，而是一部平民具身系统的进化史`](06_epilogue/11_semantic_navigation_paradigm_comparison.md)

---

## 推荐阅读顺序

1. [`00 从经典机器人到具身代理`](00_preface_and_structure/00_from_classical_robotics_to_embodied_agents.md)
2. [`01 经典路径：从零跑通 SLAM 建图与自主探索`](01_the_physical_foundation/01_slam_nav2_exploration.md)
3. [`02 移动操作：导航、识别、抓取全链路闭环`](01_the_physical_foundation/02_full_workflow_vision_grasping.md)
4. [`03 绑架问题：当 2D 导航失忆时，我让 3D 地图出来兜底`](02_semantic_navigation_paradigms/03_kidnapped_robot_problem.md)
5. [`04 路线 A：基于 YOLO-World 的物体级在线打标导航`](02_semantic_navigation_paradigms/04_yolo_world_semantic_mapping.md)
6. [`05 路线 B：基于 LLM 关键帧分析与 RAG 检索的场景记忆导航`](02_semantic_navigation_paradigms/05_llm_rag_scene_memory_navigation.md)
7. [`06 VLM 在线接管的分层 Embodied Agent`](03_the_embodied_agent/06_zero_shot_vlm_orchestrator.md)
8. [`07 跨越虚实：LeoRover 真机耦合，不是把仿真搬过去`](04_sim2real_and_robustness/07_sim2real_hardware_coupling.md)
9. [`08 真机不死之身：Watchdog、跳变保护与稳定地图补丁`](04_sim2real_and_robustness/08_watchdog_and_safety_patches.md)
10. [`09 ROSplat 与 3DGS：我为什么还是想把高斯地图接进来`](05_side_quests/09_3dgs_rosplat_visualization.md)
11. [`10 手势控制机械臂：看到网上喷漆机器人以后，我也手痒了`](05_side_quests/10_gesture_control_arm.md)
12. [`11 这不是三篇技术路线总结，而是一部平民具身系统的进化史`](06_epilogue/11_semantic_navigation_paradigm_comparison.md)

联系方式：qq693180059lrf@gmail.com  
领英：www.linkedin.com/in/runfeng-ling-34b51238b

