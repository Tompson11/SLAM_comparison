# 工作汇报

### 一.测试的SLAM方案

本次我共测试了github上开源的8种方案，按照特点可分为

| 特点             | 方案                                                         |
| ---------------- | ------------------------------------------------------------ |
| 纯Lidar          | [A-LOAM](https://github.com/HKUST-Aerial-Robotics/A-LOAM)(港科大版本的LOAM)，[hdl_graph_slam](https://github.com/koide3/hdl_graph_slam)，[BLAM](https://github.com/erik-nelson/blam) |
| Lidar与IMU松耦合 | [LeGo-LOAM](https://github.com/RobustFieldAutonomyLab/LeGO-LOAM)，[SC-LeGo-LOAM](https://github.com/irapkaist/SC-LeGO-LOAM)(在LeGo-LOAM上使用了一种新的回环检测方法) |
| Lidar与IMU紧耦合 | [LINS](https://github.com/ChaoqinRobotics/LINS---LiDAR-inertial-SLAM)，[LIO-SAM](https://github.com/ChaoqinRobotics/LINS---LiDAR-inertial-SLAM)，[LIOM](https://github.com/hyye/lio-mapping) |

说明

- 上述方案中除了`hdl_graph_slam`和`BLAM`外，其余方案都是基于`LOAM`或`LeGo-LOAM`
- 在实验中，`hdl_graph_slam`和`BLAM`在所有数据集上的性能均不理想，因而下面不再讨论。而`SC-LeGo-LOAM`的性能较之`LeGo-LOAM`也没有明显改善，因而下面也不再讨论。



### 二.实验

实验共使用了5段在伟清楼附近采集的数据集，其中每一段数据的Lidar频率均为10Hz。首先，我将IMU频率设置为KITTI数据集的100Hz，采集了两段数据museum_in和museum_out：

#### 1. museum_out

- 采集路线：从艺术博物馆停车场出发，环绕外围一周后来到伟清楼旁空地，全长约630m

  ![image-20200729110105349](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110105349.png)

- 结果：

  

  <img src="/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729103215355.png" alt="image-20200729103215355" style="zoom:200%;" />

<img src="/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729103258838.png" alt="image-20200729103258838" style="zoom:200%;" />

<img src="/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729103340010.png" alt="image-20200729103340010" style="zoom:200%;" />

- 结论：可以看到，各种算法在xy的估计上比较接近，分歧主要在z的估计上。根据实际情况，起点和终点的高度差$ \Delta Z $并不是很大，而使用了IMU紧耦合的算法最终的$ \Delta Z $都达到了1m以上，而未使用IMU的`ALOAM`和IMU松耦合的`LeGo-LOAM`的$ \Delta Z $都较小，说明IMU并未能很好地提供位姿约束。

#### 2. museum_in

- 采集路线：从艺术博物馆水池出发，在空地内行走后返回水池，全长约224m

  ![image-20200729110254867](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110254867.png)

- 结果：![image-20200729105149641](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729105149641.png)

![image-20200729110337023](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110337023.png)

![image-20200729110353576](/home/zhangzhuo/.config/Typora/typora-user-images/image-20200729110353576.png)

- 结论：该数据集实际走了一个闭环，可以看到，`ALOAM`很好地实现了闭合，这是由于实验场地面特征丰富带来的结果。而`LIO-SAM`和`LeGo-LOAM`的性能也还不错，接近闭合。`LINS`和`LIOM`则不太理想

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



