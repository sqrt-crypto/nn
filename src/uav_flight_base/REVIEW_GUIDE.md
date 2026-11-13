# 审核指南（给 reviewers）

> 本文件用于帮助评审同学在 **5 分钟内**判断本模块是否可以合并。
> 模块：`uav_flight_base` —— 基于 ROS Noetic + Gazebo 11 的无人机仿真模块（任务 1~4）

## 一、这个 PR 做了什么

| 任务 | 内容 | 关键文件 |
|---|---|---|
| 任务 1 | 键盘遥控 + 一阶惯性运动学仿真 | `scripts/uav_teleop_key.py`、`scripts/uav_mock_sim.py` |
| 任务 2 | 激光雷达反应式避障 / 相机颜色检测 + 视觉伺服 | `scripts/uav_scan_avoider.py`、`scripts/uav_color_detector.py`、`scripts/uav_visual_servo.py` |
| 任务 3 | 栅格建图 + A* 规划 + 纯追踪 | `scripts/uav_grid_nav.py` |
| 任务 4 | 端到端神经网络（图像/扫描 → 速度指令） | `scripts/uav_expert_collect.py`、`scripts/uav_train_np.py`、`scripts/uav_np_driver.py` |

统一入口为 `main.py`，所有功能均可通过 launch 启动。

## 二、5 分钟复现步骤

```bash
# 1. 编译
cp -r src/uav_flight_base ~/catkin_ws/src/
cd ~/catkin_ws && catkin_make && source devel/setup.bash

# 2. 看有哪些任务
python3 ~/catkin_ws/src/uav_flight_base/main.py --list

# 3. 跑任务 2（激光避障，最省资源，无需显示器）
roslaunch uav_flight_base avoid.launch gui:=false

# 4. 另一个终端确认话题
rostopic hz /uav/scan        # 应约 10 Hz
rostopic echo /cmd_vel       # 应能看到非零速度指令
```

若在无图形界面的机器上跑相机相关任务，用虚拟显示：

```bash
xvfb-run -a -s "-screen 0 1024x768x24" python3 main.py --task 2 --mode vision
```

## 三、请重点检查这 4 点

1. **能否编译通过**：`catkin_make` 无报错
2. **launch 能否启动**：`roslaunch uav_flight_base avoid.launch` 后 `rosnode list` 能看到 `uav_scan_avoider`
3. **话题是否正常**：`/uav/scan` 有数据、`/cmd_vel` 有输出、`/odom` 在更新
4. **代码风格**：Python 文件为 UTF-8、节点均有 `rospy.init_node`、参数走 rosparam

## 四、已知限制（不算缺陷，已在文档中说明）

- 无人机模型使用 `planar_move` 插件直接设定位姿，**不包含真实气动力学**，用于验证算法链路而非飞行品质
- 端到端策略在训练分布（开阔场地）表现良好，在密集障碍场景会趋于保守停滞，属于行为克隆的分布漂移，报告第 4.3 节有分析
- 相机类任务需要图形环境或 `xvfb-run`，这是 Gazebo 相机渲染的固有限制

## 五、如何提交审核

1. 打开 PR → 切到 **Files changed**
2. 右上角 **Review changes**
3. 选择 **Approve**（或 Comment 后写明已验证内容）
4. **Submit review**

建议在评语中写一句实测结论，例如：

> 已在 Ubuntu 20.04 + ROS Noetic 编译通过，`avoid.launch` 启动正常，`/uav/scan` 约 10 Hz，`/cmd_vel` 有速度输出。

## 六、说明

本模块对应课程要求的四个任务，报告中给出了各任务的实测数据与多场景性能评价。
若发现问题，欢迎在 PR 中直接留言，作者会补充修改。
