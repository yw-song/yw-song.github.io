# 手眼标定配置

手眼标定的目的是获取机器人坐标系和相机坐标系的关系，最后将视觉识别的结果反馈到机器人坐标系下。在手眼标定中，机器人坐标系通常指的是**末端执行器的坐标系**。这样，标定得到的变换关系可以直接将相机识别的物体位置从相机坐标系转换到末端执行器坐标系，从而使机器人末端精确地对目标物体进行操作。

手眼标定分为眼在手上 (eye in hand) 和眼在手外 (eye to hand)

<img src="D:\BLOG\img\手眼标定\手眼标定分类" alt="image-20241105225859429" style="zoom:70%;" />

**眼在手上标定：**眼在手上固定，眼与基底的相对位置保持不动，机器人带着相机在不同位置拍照，在拍照过程中不断地记录图片以及示教器上的世界坐标（该世界坐标指的是末端执行器坐标系到基底坐标系的 TF 变换）标定过程中有这样四个坐标系：

1. 基底坐标系
2. 末端执行器的坐标系（也就是示教器上的六个值）
3. 相机坐标系（相机成像中心为原点，主光轴为 Z 轴的坐标系）
4. 标定板的坐标系

**眼在手外标定：**标定板固定在机械臂法兰盘上，保证标定板与机械臂法兰盘保持不变，相机与机械臂保持不变，机器人带着标定板运动，不断拍图片记录数据。

下面以眼在手上为例讲解手眼标定过程：

## 获取末端执行器到基底的变化矩阵 $^{b}T_{g}$ 

在示教器中会有 笛卡尔空间坐标 $(x,y,z)$ 以及三个欧拉角的数值，根据这个数值能求出来变换矩阵。

<img src="D:\BLOG\img\手眼标定\示教器" alt="image-20241105230728253" style="zoom:80%;" />

若空间中有一点 $P$ 在基底坐标系和夹爪坐标系下分别表示为 $P_{b}$ 和 $P_{g}$ , 则有：
$$
P_b=R_zR_yR_xP_g+{}^bt_g
$$
其中，
$$

R_{x} =\begin{bmatrix}1&0&0\\0&\cos\theta&-\sin\theta\\0&\sin\theta&\cos\theta\end{bmatrix} 
R_{y} =\begin{bmatrix}0&\cos\theta&-\sin\theta\\0&1&0\\0&\sin\theta&\cos\theta\end{bmatrix} 
R_{z} =\begin{bmatrix}0&\cos\theta&\sin\theta\\0&-\sin\theta&\cos\theta\\0&0&1\end{bmatrix} 
$$
$^{b}T_{g}$ 就能得到了....这个变换矩阵指的是 *Transform Matrix Gripper to Base*

## 获取标定板到相机的变换矩阵 $^{c}T_{t}$

利用 `OpenCV` 的 `solvePnP()` 估计标定板的姿势得到每张标定板图片的 $R$ 和 $t$ 

## 求解相机到夹爪的变换矩阵 $^{g}T_{c}$ 

再次利用 `OpenCV` 中的函数可以直接求解：

```c++
void cv::calibrateHandEye(InputArrayOfArrays	R_gripper2base,
						  InputArrayofArrays	t_gripper2base,
                          InputArrayOfArrays	R_target2cam,
                          InputArrayOfArrays	t_target2cam,
                          OutputArray			R_cam2gripper,
                          OutputArray			t_cam2gripper,
                          HandEyeCalibrationMethod method = CALIB_HAND_EVE_TSAI
                          )
```



<img src="D:\BLOG\img\手眼标定\眼在手上" alt="image-20241105232634228" style="zoom:80%;" />

通过两次不同位置的标定可以得到矩阵：
$$
^b\mathrm{T}_g^{(1)}·{}^g\mathrm{T}_c·{}^c\mathrm{T}_t^{(1)}={}^b\mathrm{T}_g^{(2)}·{}^g\mathrm{T}_c·{}^c\mathrm{T}_t^{(2)}
$$
求解方程可以得到 $^{g}T_{c}$ , 也就是说以后得到相机坐标系下的点 $P_{c}$ , 可以使用 $P_b={}^bT_g·{}^gT_c ·P_c$ 得到对应的基底坐标系下的点。

OK 这样就得到 *camera to gripper* 的转化关系，以后相机看到一个物品可以得到 *target to camera* 的信息，然后就能得到 *target to gripper* 的关系了。

我们可以对这个式子进行进一步的数学抽象：

问题可以描述为 $\mathbf{AX}=\mathbf{XB}$ 模型

1. 对于眼在手上有：

$$
\begin{gathered}
{}^{b}{\mathrm{T}_{g}}^{(1)}·{}^g{\mathrm{T}_{c}}·^{c}{\mathrm{T}_{t}}^{(1)} ={}^b\mathrm{T}_g{}^{(2)}·{}^g\mathrm{T}_c·{}^c\mathrm{T}_t{}^{(2)} \\
(^b{\mathrm{T}_g}^{(2)})^{-1}{}·^b{\mathrm{T}_g}^{(1)}{}^g{\mathrm{T}_c} ={}^g\mathrm{T}_c·{}^c\mathrm{T}_t{}^{(2)}·({}^c\mathrm{T}_t{}^{(1)})^{-1} \\
\mathrm{A}_{i}\mathrm{X} =\mathrm{XB}_i 
\end{gathered}
$$

2. 对眼在手外有：

$$
\begin{gathered}
^g{\mathrm{T}_b}^{(1)}·{}^b{\mathrm{T}_c}·{}^c{\mathrm{T}_t}^{(1)} ={}^g\mathrm{T}_b{}^{(2) }·{}^b\mathrm{T}_c·{}^c\mathrm{T}_t{}^{(2)} \\
(^g{\mathrm{T}_b}^{(2)})^{-1}·{}^g{\mathrm{T}_b}^{(1)}·{}^b{\mathrm{T}_c} ={}^b\mathrm{T}_c·{}^c\mathrm{T}_t{}^{(2)}·({}^c\mathrm{T}_t{}^{(1)})^{-1} \\
\mathrm{A}_{i}\mathrm{X} =\mathrm{XB}_i 
\end{gathered}
$$

<img src="D:\BLOG\img\手眼标定\眼在手外" alt="image-20241106141610218" style="zoom:80%;" />