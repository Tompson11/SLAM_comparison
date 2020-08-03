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

  ![]()

  x轴和y轴的加速度ax，ay和角速度wx,wy有明显的周期性，周期为雷达的扫描周期。而经过进一步的DFT分析，可以发现ax，ay，wx，wy，wz在10Hz和40Hz的频谱分量最高，说明雷达振动的基波和4次谐波影响较大。为了去除振动影响，我暂时采取了时域拟合的方法，利用基波和4次谐波的组合来拟合测量信号，使得总误差最小，然后将拟合出的振动信号从测量信号中减去，将所得信号的均值和方差作为IMU的零偏和噪声方差。

  

#### 1. museum_out

- **采集路线**：从艺术博物馆停车场出发，环绕外围一周后来到伟清楼旁空地。

  ![](https://github.com/Tompson11/SLAM_comparison/tree/master/image/result/museum_out/2D.png)

- **结果**：

  <img src="https://github.com/Tompson11/SLAM_comparison/tree/master/image/result/museum_out/2D.png" alt="image-20200729103215355" style="zoom:200%;" />

<img src="/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729103258838.png" alt="image-20200729103258838" style="zoom:200%;" />

<img src="/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729103340010.png" alt="image-20200729103340010" style="zoom:200%;" />

- **结论**：可以看到，各种算法在xy的估计上比较接近，分歧主要在z的估计上。根据实际情况，起点和终点的高度差$ \Delta Z $并不是很大，而使用了IMU紧耦合的算法最终的$ \Delta Z $都较大，而未使用IMU的`ALOAM`和IMU松耦合的`LeGo-LOAM`的$ \Delta Z $都较小，说明IMU并未能很好地提供位姿约束。

#### 2. museum_in

- **采集路线**：从艺术博物馆水池出发，在空地内行走后返回原点：

  ![image-20200729110254867](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110254867.png)

- **结果**：![image-20200729105149641](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729105149641.png)

![image-20200729110337023](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110337023.png)

![image-20200729110353576](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110353576.png)

- **结论**：该数据集实际走了一个闭环，可以看到，`ALOAM`很好地实现了闭合，这是由于实验场地面特征丰富带来的结果。而`LIO-SAM`和`LeGo-LOAM`的性能也还不错，接近闭合。`LINS`和`LIOM`则不太理想

  总的来看，这艺术博物馆采集的这两段数据中`ALOAM`和`LeGo-LOAM`的性能最佳，而其它IMU紧耦合的方案则稍次，说明在几何特征丰富的情况下，Lidar已经能够提供很好的精度，而IMU的加入反而是“画蛇添足”了。为了寻求IMU带来的性能提升，我将IMU的频率设置为了`LINS`,`LIOM`和`LIO-SAM`推荐的高频400Hz，并且又采集了以下3段数据：

#### 3. outdoor3

- 采集地点：从伟清楼出发，环绕附近建筑一周后返回原点，全长约482m

  ![image-20200729114636545](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729114636545.png)

- 结果：

  ![image-20200729114731358](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729114731358.png)

![image-20200729114755920](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729114755920.png)

![image-20200729114813804](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729114813804.png)

- 结论：该数据集实际走了一个闭环，可以看到，所有方法都出现了明显的漂移，这是由于采集路线上存在一段公路，其几何特征较少。而使用了IMU紧耦合的三种方案`LINS`,`LIOM`和`LIO-SAM`漂移较少，说明高频IMU起到了一定的约束作用，而`LeGo-LOAM`产生了相当严重的漂移，可能是由于地面点缺乏导致。

#### 4. outdoor4

- 采集地点：从伟清楼出发，与outdoor3方向相反地环绕附近建筑一周后返回原点，全长约550m![image-20200729121332169](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729121332169.png)

- 结果：

![image-20200729115800229](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729115800229.png)

![image-20200729115816313](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729115816313.png)

![image-20200729115828416](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729115828416.png)

结论：该数据集实际走了一个闭环，可以看到，只有`LIO-SAM`很好地实现了闭合，但其在闭合回环时将误差分散到了环路中的其它部分，因而轨迹看起来比起其它方法有个旋转。并且，`LIO-SAM`对高度有很好的约束，其轨迹估计的z起伏在合理范围内。

#### 5. aggressive



