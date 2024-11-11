# MoveIt Setup Assistant 配置

MoveIt 配置助手使用说明以加深自己的理解。

参考自：

> MoveIt 官方教程：https://moveit.github.io/moveit_tutorials/doc/setup_assistant/setup_assistant_tutorial.html?highlight=moveit_setup_assistant
>
> 创客智造网站教程：https://www.ncnynl.com/archives/201610/1030.html
>
> 深蓝学院 ROS 机械臂开发：从入门到实践：https://www.shenlanxueyuan.com/course/265

## 说明

MoveIt是一个用于ROS的开源运动规划框架，广泛应用于机器人运动规划和控制任务。它提供了强大的工具和库，能够处理复杂的运动控制，如路径规划、碰撞检测、逆向运动学（IK）求解、传感器融合以及可视化等功能。 

MoveIt Setup Assistant 是一个是一个图形化的用户接口，用于配置整合你的机器人使用MoveIt!其主要功能是为您的机器人生成语义机器人描述格式 (SRDF) 文件，该文件指定 MoveIt 所需的附加信息，例如规划组、末端执行器和各种运动学参数。此外，它还生成其他必要的配置文件以与 MoveIt 管道一起使用。要使用 MoveIt 设置助手，您需要拥有机器人的 URDF 文件。

## 要求

- 安装 MoveIt 和 ROS

## 配置步骤

启动 MoveIt Setup Assistant

```bash
roslaunch moveit_setup_assistant setup_assistant.launch
```

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_start.png" alt="setup_assistant_start" style="zoom:60%;" />

### Start

屏幕左边是接下来我们依次要完成的步骤，第一个步骤是 `Start` ：在这里我们可以导入或者编辑我们要配置的机器人模型也就是两个选项 **创建新的 MoveIt 配置包** 或 **编辑现有的 MoveIt 配置包**。如果自己已经做好了 功能包或者别人给的功能包我们可以选择第二个导入包进行配置，如果自己设置的机器人或者只有URDF需要点击第一个。

> [!NOTE]
>
> 注意：要导入的配置包需要我们 source 下

我们以官方教程为例，找到 `panda.urdf.xacro` 文件并导入：单击浏览按钮，导航到安装上面的 Franka 软件包时安装的*panda.urdf.xacro*文件，然后选择该文件。在 ROS Noetic 上，此文件安装在 /opt/ros/noetic/share/franka_description/robots/panda/panda.urdf.xacro 中。

单击*“加载文件”* 。设置助手将加载文件（这可能需要几秒钟）并向您显示以下屏幕：

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_100.png" alt="../../_images/setup_assistant_panda_100.png" style="zoom:60%;" />

### Self-Collisions

这里配置的是自碰撞检测：机器人在做运动的时候，机器人本身可能与自身碰撞（Link之间）。默认自碰撞矩阵生成器搜索机器人上可以安全地禁用碰撞检查的成对链接，从而减少运动规划处理时间。这一步很简单，单击左侧的 *Self-Collisions* ，然后单击 *Generate Collision Matrix* 按钮。设置助手将工作几秒钟，然后在主表中向您显示其计算结果。

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_self_collisions.png" alt="before" style="zoom:60%;" />

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_self_collisions_done.png" alt="after" style="zoom:60%;" />

### Virtual Joints

想象这样一个情况：机械臂有时会配一个底盘，底盘会跟外界环境发生移动，这个时候类似于机器人里的 world 坐标系与 机器人的 baselink 坐标系发生了位置变化。虚拟关节主要用于将机器人连接到世界。对于 Panda，我们将只定义一个虚拟关节，将 Panda 的 *panda_link0* 连接到*世界*坐标系。该虚拟关节代表机器人底座在平面上的运动。

- 单击 *“Virtual Joints”* 窗格选择器。单击 *Add Virtual Joint*
- 将关节名称设置为 “virtual_joint”
- 将子链接设置为 “panda_link0”，将父框架名称设置为 “world”。
- 将关节类型设置为 “fixed”。
- 单击 *“save”* ，您应该看到以下屏幕：

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_virtual_joints.png" alt="../../_images/setup_assistant_panda_virtual_joints.png" style="zoom:60%;" />

### Planning Groups 

规划组用于从语义上描述机器人的不同部分，例如定义手臂或末端执行器，这是十分重要的一步，因为这会影响到运动学正逆解实现的效率。用人话说规划组就是我在路径规划的时候，针对某一个组做规划，比如说我一个机器人六个关节作为一组，做规划的时候我就需要联合的考虑这六个关节的位置，在做正逆解的时候也是一样的——正逆解是针对六个关节做正逆解，这个时候我们六个关节就是一个规划组。

- 单击 *“Planning Groups”* 窗格选择器。
- 单击 *Add Group*，您应该看到以下屏幕：

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_planning_groups.png" alt="../../_images/setup_assistant_panda_planning_groups.png" style="zoom:60%;" />



#### 将机械臂本体作为一个规划组

- 我们首先将 Panda 手臂添加为规划组：
  - 输入 *Group Name* 为 **panda_arm**。
  - 选择 *kdl_kinematics_plugin/KDLKinematicsPlugin* 作为运动学解算器。
  - 让 *Kin. Search Resolution* 和 *Kin. Search Timeout* 保持默认值即可。

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_arm.png" alt="../../_images/setup_assistant_panda_arm.png" style="zoom:60%;" />

- 现在，单击 *“Add Joints”* 按钮。您将在左侧看到关节列表。您需要选择属于手臂的所有关节并将它们添加到右侧。关节按照它们存储在内部树结构中的顺序排列。这使得选择串行接头链变得容易。
  - 单击 *virtual_joint* ，按住键盘上的 `Shift` 按钮，然后单击 *panda_joint8* 。现在单击 `>` 按钮将这些关节添加到右侧选定关节的列表中

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_arm_joints.png" alt="../../_images/setup_assistant_panda_arm_joints.png" style="zoom:60%;" />

- 单击 *“save”* 以保存选定的组。

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_arm_joints_saved.png" alt="../../_images/setup_assistant_panda_arm_joints_saved.png" style="zoom:60%;" />

#### 将 gripper 作为一个规划组

- 最后我们还会将末端执行器作为一个规划组。请注意，您将使用不同的过程来执行此操作。
  - 单击 *Add Group* 按钮。
  - 在*Group name* 输入 **hand**。
  - *Kinematic Solver* 保持默认值，也就是啥都没有(**None**)。
  - 让 *Kin. Search Resolution* 和 *Kin. Search Timeout* 保持默认值。
  - 单击 *“Add Joints”* 按钮。
  - 选择 **panda_hand** 、 **panda_leftfinger** 和 **panda_rightfinger** 并将它们添加到右侧的 *Selected Links* 列表中。
  - 点击 *Save*。

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_planning_groups_gripper.png" alt="../../_images/setup_assistant_panda_planning_groups_gripper.png" style="zoom:60%;" />

### Robot Poses

主要是让我们预定义一些机器人的点位，方便后续编程的时候使用。如果您想要将机器人的某个位置定义为**起始**位置，这会很有帮助。

- 单击 *“ Robot Poses”* 窗格。

- 单击 *Add Pose*。为姿势选择一个名称。机器人将处于其 *Default* 位置，其中关节值设置为允许的关节值范围的中间范围。移动各个关节直到您满意为止，然后*保存*姿势。注意姿势如何与特定群体相关联。您可以为每个组保存单独的姿势。

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_saved_poses.png" alt="../../_images/setup_assistant_panda_saved_poses.png" style="zoom:60%;" />

### End Effectors

我们已经添加了 Panda 机器人的 gripper。现在，我们将这个组指定为一个特殊组：**End effector**。将此组指定为末端执行器允许在内部对它们进行一些特殊操作。

- 单击 *End Effectors* 窗格。
- 单击 *Add End Effector*。
- 选择 **hand** 作为夹具的 *End Effector Name*。
- 选择 **hand** 作为 *End Effector Group*。
- 选择 **panda_link8** 作为该末端执行器的 *Parent Link*。
- 将 *Parent Group* 留空。

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_end_effector_add.png" alt="../../_images/setup_assistant_panda_end_effector_add.png" style="zoom:60%;" />

### Passive Joints

被动关节指的是在运动中不需要考虑的关节。被动关节选项卡旨在允许指定机器人中可能存在的任何被动关节。这些是机器人上未驱动的关节（例如被动脚轮）。这告诉规划者他们无法（从运动学角度）规划这些关节，因为它们无法直接控制。 Panda 没有任何被动关节，因此我们将跳过这一步。

### ROS Control

目前这个选项产生配置文件有点问题，不建议进行操作，推荐在文件中进行配置，这样更加稳定。**Simulation** 选项卡同上。

### 3D Perception

3D Perception 选项卡旨在设置 YAML 配置文件的参数，以配置 3D 传感器sensors_3d.yaml 。

例如点云(point_cloud )参数：

<img src="https://moveit.github.io/moveit_tutorials/_images/setup_assistant_panda_3d_perception_point_cloud.png" alt="../../_images/setup_assistant_panda_3d_perception_point_cloud.png" style="zoom:60%;" />

如果不需要 *sensors_3d.yaml* ，请选择 *None*。

### Add Author Information

Catkin 需要作者信息以用于发布目的

- 单击 *Author* 窗格。
- 输入您的姓名和电子邮件地址。

### Generate Configuration Files

最后一步 - 生成开始使用 MoveIt 所需的所有配置文件。

- 单击 *“Configuration Files”* 窗格。为将生成的包含新配置文件集的 ROS 包选择位置和名称。单击 “browse”，选择一个合适的位置（例如，您的主目录），单击 **“创建新文件夹**”，将其命名为 “panda_moveit_config”，然后单击 **“Choose”** 。 “panda_moveit_config” 是本 wiki 上其余文档中使用的位置。该包不必位于您的 ROS 包路径中。所有生成的文件将直接进入您选择的目录。
- 单击 *Generate Package* 按钮。安装助手现在将生成一组启动和配置文件并将其写入您选择的目录中。所有生成的文件都将显示在“生成的文件/文件夹”选项卡中，您可以单击每个文件以获取其包含内容的描述。
- 恭喜！ - 您现在已完成生成 MoveIt 所需的配置文件