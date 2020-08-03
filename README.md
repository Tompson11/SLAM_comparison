# 工作汇报

### 一.测试的SLAM方案

本次我共测试了github上开源的**8**种方案，按照特点可分为

| 特点             | 方案                                                         |
| ---------------- | ------------------------------------------------------------ |
| 纯Lidar          | [A-LOAM](https://github.com/HKUST-Aerial-Robotics/A-LOAM)(港科大版本的LOAM)，[hdl_graph_slam](https://github.com/koide3/hdl_graph_slam)，[BLAM](https://github.com/erik-nelson/blam) |
| Lidar与IMU松耦合 | [LeGo-LOAM](https://github.com/RobustFieldAutonomyLab/LeGO-LOAM)，[SC-LeGo-LOAM](https://github.com/irapkaist/SC-LeGO-LOAM)(在LeGo-LOAM上使用了一种新的回环检测方法) |
| Lidar与IMU紧耦合 | [LINS](https://github.com/ChaoqinRobotics/LINS---LiDAR-inertial-SLAM)，[LIO-SAM](https://github.com/ChaoqinRobotics/LINS---LiDAR-inertial-SLAM)，[LIOM](https://github.com/hyye/lio-mapping) |

**说明**

- 上述方案中除了`hdl_graph_slam`和`BLAM`外，其余方案都是基于`LOAM`或`LeGo-LOAM`
- 在实验中，`hdl_graph_slam`和`BLAM`在所有数据集上的性能均不理想，因而下面不再讨论。而`SC-LeGo-LOAM`的性能较之`LeGo-LOAM`也没有明显改善，因而下面也不再讨论。

**原生Demo**

在第二部分的实验结果展示中，为了方便比较，不同方案得到的结果形式都进行了统一化，但实际上各种方案在执行时的视觉效果是不同的，这里展示了利用各种方案的原生配置所得到的demo，日后可根据需要配置成下面的任意一种效果：

|      |      |
| :----: | :----: |
|  ![ALOAM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/demo/ALOAM_demo.gif)    |       ![LeGo_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/demo/LeGo_demo.gif) |
|   **ALOAM**   | **LeGo_LOAM**     |
|  ![LINS_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/demo/LINS_demo.gif)    |       ![LIO_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/demo/LIO-SAM_demo.gif) |
|   **LINS**   | **LIO-SAM**     |
|  ![LIOM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/demo/LIOM_demo.gif)    |      |
|   **LIOM**   ||



### 二.实验

实验共使用了**5**段在**伟清楼**附近采集的数据：

| 数据段名称 | 采集时间  | 传感器配置            | 路程 | 备注           |
| ---------- | --------- | --------------------- | ---- | -------------- |
| museum_out | 2020.7.25 | 10Hz Lidar+ 100Hz IMU | 639m | 无             |
| museum_in  | 2020.7.25 | 10Hz Lidar+ 100Hz IMU | 226m | 回环           |
| outdoor3   | 2020.7.27 | 10Hz Lidar+ 400Hz IMU | 481m | 回环           |
| outdoor4   | 2020.7.27 | 10Hz Lidar+ 400Hz IMU | 562m | 回环           |
| aggresive  | 2020.7.27 | 10Hz Lidar+ 400Hz IMU | 138m | 回环，剧烈运动 |


**说明**

- **两日的数据**：由于在7.25采集的两段数据中没有明显看出加入IMU的优势，于是在7.27我将IMU频率调高至`LINS`，`LIOM`和`LIO-SAM`所采用的400Hz后又采集了三段数据。

- **IMU的校正**：在每次采集数据前，我都将IMU的z轴与重力加速度重新对齐，方法是利用手机软件（如：水平仪，[AIDA64](https://www.aida64.com/)等）测量，寻找到一个较好的水平面，将实验设备置于其上，利用Mtmanager对IMU进行Alignment Reset，实现z轴的重置。

- **IMU初值**：IMU紧耦合算法中需要提供IMU的初始零偏（bias）和噪声方差，而每段数据在开始时的一段时间内实验设备都是静止的，因而我利用这段时间内（约5s）的IMU测量值进行了零偏和噪声的估计。需要注意的是，由于IMU是粘贴在Lidar外壳上的，而Lidar在扫描时会引起外壳周期性的振动，这也反映在了IMU的测量值中，如下图：
<div>
<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/others/imu_acc_meas.png" alt="acc_meas" width="40%" hspace="4%"/>
<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/others/imu_gyr_meas.png" alt="imu_gyr_meas" width="40%" hspace="4%"/>
</div>
  x轴和y轴的加速度ax，ay和角速度wx,wy有明显的周期性，周期为雷达的扫描周期。而经过进一步的DFT分析，可以发现ax，ay，wx，wy，wz在10Hz和40Hz的频谱分量最高，说明雷达振动的基波和4次谐波影响较大。为了去除振动影响，我暂时采取了时域拟合的方法，利用基波和4次谐波的组合来拟合测量信号，使得总误差最小，然后将拟合出的振动信号从测量信号中减去，将所得信号的均值和方差作为IMU的零偏和噪声方差。

  

#### 1. museum_out

- **采集路线**：从艺术博物馆停车场出发，环绕外围一周后来到伟清楼旁空地。
 <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/satellite/museum_out.png" alt="" align="center"/>

- **定位结果**：

   <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/museum_out/3D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/museum_out/2D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/museum_out/xyz.png" alt="" align="center"/>

- **建图结果**：

|      |      |
| :----: | :----: |
|  ![ALOAM](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_out/ALOAM.png)    |       ![LeGo_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_out/LeGo.png) |
|   **ALOAM**   | **LeGo_LOAM**     |
|  ![LINS_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_out/LINS.png)    |       ![LIO_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_out/LIO-SAM.png) |
|   **LINS**   | **LIO-SAM**     |
|  ![LIOM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_out/LIOM.png)    |      |
|   **LIOM**   ||

- **结论**：
从定位结果上看，各种算法在***x***，***y***的估计上比较接近，分歧主要在高度***z***的估计上。根据实际情况，起点和终点的高度差并不是很大，`LIO-SAM`和`LeGo-LOAM`在这点上性能较好，其余方案则略有不足。
从建图结果上看，各算法的建图效果相当，只是`LINS`和`LeGo-LOAM`的地图稍微稀疏一些。

#### 2. museum_in

- **采集路线**：从艺术博物馆水池出发，在内场游走后回到原点：
 <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/satellite/museum_in.png" alt="" align="center"/>

- **定位结果**：

   <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/museum_in/3D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/museum_in/2D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/museum_in/xyz.png" alt="" align="center"/>

- **建图结果**：

|      |      |
| :----: | :----: |
|  ![ALOAM](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_in/ALOAM.png)    |       ![LeGo_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_in/LeGo.png) |
|   **ALOAM**   | **LeGo_LOAM**     |
|  ![LINS_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_in/LINS.png)    |       ![LIO_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_in/LIO-SAM.png) |
|   **LINS**   | **LIO-SAM**     |
|  ![LIOM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/museum_in/LIOM.png)    |      |
|   **LIOM**   ||

- **结论**：
从定位结果看，`LIOM`与其他方法有较大偏移，但轨迹的形状却是相似的，这是由于`LIOM`对IMU和Lidar外参进行了校正，使得坐标系有所偏移。`ALOAM`和`LIOM`很好地闭合了回环，但`LIO-SAM`，`LINS`和`LeGo-LOAM`的性能也还不错，也接近闭合。
从建图结果上看，各算法的建图效果相当，只是`LINS`和`LeGo-LOAM`的地图稍微稀疏一些。

#### 3. outdoor3

- **采集路线**：从伟清楼出发，环绕附近建筑一周后返回原点：
 <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/satellite/outdoor3.png" alt="" align="center"/>

- **定位结果**：

   <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/outdoor3/3D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/outdoor3/2D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/outdoor3/xyz.png" alt="" align="center"/>

- **建图结果**：

|      |      |
| :----: | :----: |
|  ![ALOAM](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor3/ALOAMmerge.png)    |       ![LeGo_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor3/LeGomerge.png) |
|   **ALOAM**   | **LeGo_LOAM**     |
|  ![LINS_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor3/LINSmerge.png)    |       ![LIO_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor3/LIO-SAMmerge.png) |
|   **LINS**   | **LIO-SAM**     |
|  ![LIOM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor3/LIOMmerge.png)    |      |
|   **LIOM**   ||

- **结论**：
从定位结果来看，IMU紧耦合且存在回环检测的`LIO-SAM`成功地闭合了回环，其他方案都发生了明显漂移，其中，`LeGo-LOAM`在***z***方向产生了相当离谱的估计，这是由于点云中地面点较少，不能很好地约束住***z***方向,而使用了IMU紧耦合的另外两种`LINS`,`LIOM`较之`ALOAM`漂移较少，证明高频IMU起到了一定的约束作用。<br/>
从建图结果来看，`LIO-SAM`由于闭合了回环，建立了全局一致的地图。而其他方案建立的地图中则可以看出明显的漂移，例如图中左下角显示了起点处地图的侧视图，可以看到除`LIO-SAM`外，其他方案的地图都出现明显的分层现象，这就是***z***方向漂移的结果。

#### 4. outdoor4

- **采集路线**：从伟清楼出发，与outdoor3方向相反地环绕附近建筑一周后返回原点：
 <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/satellite/outdoor4.png" alt="" align="center"/>

- **定位结果**：

   <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/outdoor4/3D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/outdoor4/2D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/outdoor4/xyz.png" alt="" align="center"/>

- **建图结果**：

|      |      |
| :----: | :----: |
|  ![ALOAM](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor4/ALOAMmerge.png)    |       ![LeGo_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor4/LeGomerge.png) |
|   **ALOAM**   | **LeGo_LOAM**     |
|  ![LINS_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor4/LINSmerge.png)    |       ![LIO_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor4/LIO-SAMmerge.png) |
|   **LINS**   | **LIO-SAM**     |
|  ![LIOM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/outdoor4/LIOMmerge.png)    |      |
|   **LIOM**   ||

**结论**：
<br>从定位结果来看，只有`LIO-SAM`很好地闭合了回环。`LeGo-LOAM`再次发生了退化，估计效果不佳，而`LINS`和`LIOM`在z方向产生了明显漂移，`ALOAM`在y和z方向都产生了明显漂移，说明IMU还是起到了一定的约束作用。
<br>从建图结果来看，`LIO-SAM`得到了全局一致的地图，而其他方法由于漂移建图效果不太理想，例如图中的左下部分显示了起点处地图的俯视图，可以看到除`LIO-SAM`外都存在分层现象，这是z方向漂移的结果，而`ALOAM`和`LeGo-LOAM`还可以看出明显的平行相似结构（如柱子和右侧墙壁），这是y方向漂移的结果。


#### 5. aggressive
- **采集路线**：从伟清楼出发，在伟清楼，英士楼和刘卿楼围城的空地上行走，途中伴随有跑步后骤停，猛烈转弯等剧烈动作，最终回到原点：
 <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/satellite/aggre.png" alt="" align="center"/>

- **定位结果**：

   <img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/aggre/3D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/aggre/2D.png" alt="" align="center"/>

<img src="https://github.com/Tompson11/SLAM_comparison/blob/master/image/result/aggre/xyz.png" alt="" align="center"/>

- **建图结果**：

|      |      |
| :----: | :----: |
|  ![ALOAM](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/aggre/ALOAM.png)    |       ![LeGo_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/aggre/LeGo.png) |
|   **ALOAM**   | **LeGo_LOAM**     |
|  ![LINS_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/aggre/LINS.png)    |       ![LIO_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/aggre/LIO-SAM.png) |
|   **LINS**   | **LIO-SAM**     |
|  ![LIOM_demo](https://github.com/Tompson11/SLAM_comparison/blob/master/image/map/aggre/LIOM.png)    |      |
|   **LIOM**   ||

**结论**：
<br>从定位结果来看，`LIO-SAM`和`LIOM`很好地闭合了回环。`LINS`在z方向产生了明显漂移，`ALOAM`在y和z方向都产生了明显漂移，`LeGo-LOAM`则索性崩溃，这说明IMU还是起到了一定的约束作用。
<br>从建图结果来看，采用了IMU紧耦合的三种方案`LIO-SAM`和`LIOM`和`LINS`得到的地图看起来都还不错，很难看出漂移或失真痕迹，而`ALOAM`和`LeGo-LOAM`的结果就相当糟糕了！

### 三.总结
1. 各种方案的优缺点如下：

| 方案          | 优点                                                         | 不足                                                         |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **ALOAM**     | 1. 在几何特征丰富时比较稳定                                  | 1. 后期内存会出现爆炸，计算效率下降<br>2. 在几何特征较少时会产生明显漂移 |
| **LeGo-LOAM** | 1. 在地面点丰富时比较稳定<br>2. 轻量级                       | 1. 在地面点缺乏时很容易崩溃<br>2. 得到的地图比较稀疏         |
| **LINS**      | 1. 轻量级                                                    | 1. z方向漂移明显<br/>2. 得到的地图比较稀疏<br/>3. 目前的版本要求Lidar与IMU体坐标系的xy平面平行，不接受自己提供的外參 |
| **LIO-SAM**   | 1. 存在回环检测，能较好地闭合回环<br/>2. 稳定性强<br/>3. Demo看起来比较舒服 | 1. 在几何特征丰富的情况下可能不如**ALOAM**                   |
| **LIOM**      | 1. 存在重力加速度的校正和IMU初始状态估计                     | 1. 稳定性不好，有时性能好，有时又不行，可能与其初始化环节的性能有关<br/>2. 内存占用大，时间性能较差 |
2. 在进行IMU校正后，融合高频IMU确实能够提升SLAM性能，尤其是在几何特征缺乏或者剧烈运动的情况下。
3. `LIO-SAM`在定位和建图方面做的都不错，比较建议使用。
