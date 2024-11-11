# TF 变换与 URDF

ROS 中机器人模型包含大量的部件，这些部件统称之为 link，每一个 link 上面对应着一个 frame。Link 和 frame概念是绑定在一起的。维护各个坐标系之间的关系，就要靠着 `tf tree` 来处理，维护着各个坐标系之间的联通。`tf tree` 就是一个维护各个部件坐标关系的数据结构，每个坐标系都有一个父坐标系，利用这个父子关系来找出转换的方式.每一个圆圈代表一个frame，对应着机器人上的一个link，任意的两个frame之间都必须是联通的，如果出现某一环节的断裂，就会引发error系统报错。所以完整的 `tf tree` 不能有任何断层的地方，这样我们才能查清楚任意两个 frame 之间的关系。这一点从TF树的含义就可以看出，因为每个坐标系都需要和自己的父坐标系建立联系才可以转换，而兄弟坐标系间想要转换就只能经由父坐标系，推广到更大范围，想要整个 `tf tree` 上的节点都可以进行坐标转换，就只能让整个树结构联通，只有这样才能进行转换。换一种方式去思考，将这个 `tf tree` 看做一个无向图，两个节点之间联通，意味着存在一个转换路径让彼此坐标系建立联系。

<img src="D:\BLOG\img\TF变换与URDF\TFtree" alt="image-20241109143332480" style="zoom:80%;" />

## TF Tools

使用 tf 工具来查看 tf 在幕后做了什么。

### view_frames

`view_frames` 创建 tf 通过 ROS 广播的帧的图表。

```bash
$ rosrun tf view_frames
```

输出结果为：

```bash
Transform Listener initing
Listening to /tf for 5.000000 seconds
Done Listening
dot - Graphviz version 2.16 (Fri Feb  8 12:52:03 UTC 2008)

Detected dot version 2.16
frames.pdf generated
```

---

如果出现报错：

```bash
$ rosrun tf view_frames
Listening to /tf for 5.0 seconds
Done Listening
b'dot - graphviz version 2.43.0 (0)\n'
Traceback (most recent call last):
  File "/opt/ros/noetic/lib/tf/view_frames", line 119, in <module>
    generate(dot_graph)
  File "/opt/ros/noetic/lib/tf/view_frames", line 89, in generate
    m = r.search(vstr)
TypeError: cannot use a string pattern on a bytes-like object
```

这个错误是因为在 `view_frames` 脚本中，处理的是字节对象（bytes-like object），但却用作字符串模式进行了正则表达式匹配，导致了 `TypeError`。

处理方式：打开终端，并使用以下命令修改 `/opt/ros/noetic/lib/tf/view_frames` 文件中的代码

```bash
sudo sed -i "89s/vstr/vstr.decode('utf-8')/" /opt/ros/noetic/lib/tf/view_frames
```

---

在这里，tf 侦听器正在侦听通过 ROS 广播的 frames，并绘制帧如何连接的树。要查看树：

```bash
$ evince frames.pdf
```

正如该博客的第一篇图：一共有 10 个坐标系，我们还可以看到 `base_link` 是 `base_plant` 和 `shoulder_link` 帧的父级。出于调试目的， `view_frames` 还会报告一些诊断信息，比如：

1. **Broadcaster**：
   - 广播者（Broadcaster）是TF系统中的一个组件，负责发布（广播）坐标帧之间的变换信息。任何想要告知其他节点坐标帧之间如何相互变换的节点都可以成为一个广播者。例如，一个机器人的底盘坐标系相对于世界坐标系的变换信息可以由一个传感器节点广播。

2. **Average rate**：
   - 平均速率（Average rate）通常指的是广播变换信息的频率。在ROS的TF监控工具（如`rqt_tf_tree`）中，这个值可以显示每个变换的平均更新频率，即每秒变换信息被广播的次数。这个值可以帮助你了解系统的实时性和性能。

3. **Most recent transform**：
   - 最近的变换（Most recent transform）指的是在TF系统中，对于任意两个坐标帧之间的变换关系，系统会存储最新的变换信息。当你请求一个坐标帧到另一个坐标帧的变换时，TF会提供最新的变换数据，这个数据就是“最近的变换”。

4. **Buffer length**：
   - 缓冲区长度（Buffer length）是指TF系统为每个坐标帧之间的变换信息保留的历史数据的长度。这个缓冲区允许你查询过去某个时间点的坐标帧变换信息。缓冲区长度设置得越大，可以回溯的历史信息就越多，但同时也会增加内存的使用量。

### rqt_tf_tree 

[rqt_tf_tree](https://wiki.ros.org/rqt_tf_tree) 是一个实时工具，用于可视化通过 ROS 广播的帧树。您只需通过图左上角的刷新底部来刷新树。

 用法：

```bash
$ rosrun rqt_tf_tree rqt_tf_tree
```

或者更简单的用法：

```bash
$ rqt &
```

从 `Plugins` 选项卡中选择 `TF_tree` 

### tf_echo

`tf_echo` 是 ROS 中的一个工具，用于显示两个坐标系（frames）之间的变换关系（transform），前提是这些坐标系的变换已在 ROS 网络中广播。

用法：

```bash
$ rosrun tf tf_echo [reference_frame] [target_frame]
```

让我们看一下turtle2 坐标系相对于turtle1 坐标系的变换，它相当于从 turtle1 到世界坐标系的变换乘以从世界到 turtle2 坐标系的变换的乘积。

```bash
$ rosrun tf tf_echo turtle1 turtle2
```

## URDF

URDF中文名称统一机器人描述格式，使用XML格式描述机器人文件。主要是用来描述一个机器人的结构组成，利用xml文件的结构关系，去表示机器人的结构关系。



我现在在 URDF 中有两个连杆分别是 wrist2_Link 和 wrist3_Link 通过 j6 相连接：

```
<joint name="j6" type="revolute">
    <origin xyz="0 0 0.102" rpy="-1.570796 2.268928 0" />
    <parent link="wrist2_Link" />
    <child link="wrist3_Link" />
    <axis xyz="0 0 1" />
    <limit lower="-3.0543261" upper="3.0543261" effort="28" velocity="3.2" />
    <calibration rising="0" falling="0" />
    <dynamics damping="0" friction="0" />
    <safety_controller soft_upper_limit="3.0543261" soft_lower_limit="-3.0543261" k_position="100" k_velocity="40" />
 </joint>
```

我们主要关注两者之间的位姿关系：`<origin xyz="0 0 0.102" rpy="-1.570796 2.268928 0" />` 

现在位置对了，但是 wrist3 的姿态不对，应该再绕着 wrist3 当然的坐标系的 z 轴旋转 130 度，所以这个 origin 行代码该怎么写？





在 moveit 的 api 中：

```python
move_group.go_to_pose_goal(req.p_x, req.p_y, req.p_z, req.o_x, req.o_y, req.o_z, req.o_w)
```

传入参数有位置坐标与姿态，然后机械臂会移动到相应的位姿。问题在于是哪个坐标系相对于哪个坐标系的位置与姿态？我如果想修改这个坐标系在哪里可以修改？
