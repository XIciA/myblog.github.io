---
title: 机器车巡线与人群识别系统设计
toc: true
date: 2022-11-22 15:40:00
tags: 智能设计
categories: 技术
thumbnail: https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/robotcar.png
---

---


本项目完成了机器车在复杂环境的巡线以及人群识别功能。软件平台采用基于 Ubuntu16.04 的 Kinectic 版的机器人操作系统 ROS。巡线技术主要采用 HSV 划线遍历法，识别路线采用限制 HSV 通道阈值进行筛选，识别人群采用 paddlepaddle 算法库中yolov3_darknet53_pedestrian 模型进行人群定位，进而使用 HSV 模型进行颜色判别。

<!-- more -->


---
# 问题描述
机器车需要从起点位置出发，机沿途依次通过上下斜坡、单边桥、S弯道、直角弯道、U 形弯道、双边桥等地形，并在弯道区域完成人群识别任务，最终到达终点，并在终端区输出人群总数、红色系上衣人数、蓝色系上衣人数。


---
# 设计方案

## 技术基础


1. **HSV模型**：HSV（Hue, Saturation, Value）是由A. R. Smith于1978年创建的一种颜色空间，也称为六角锥体模型（Hexcone Model）。在HSV模型中，颜色由色度（Hue）、饱和度（Saturation）、明度（Value）组成。

    ![HSV模型图片](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/hsv.png)

   在使用HSV时，可以使用OpenCV中的`cv2.COLOR_RGB2HSV`将RGB模型图像转换成HSV模型图像。转换原理如下：

   假设$(r, g, b)$分别是一个颜色的红、绿和蓝坐标，它们的值是在0到1之间的实数，$max$等于$r, g, b$中的最大值，$min$等于$r, g, b$中的最小值。

   - 色度（Hue）计算如下：
     $$
     h = \begin{cases}
     0^\circ & \text{if } \max = \min \\
     60^\circ \times \frac{g-b}{\max -\min }+0^\circ, & \text{if } \max =r \text{ and } g \geq b \\
     60^\circ \times \frac{g-b}{\max -\min }+360^\circ, & \text{if } \max =r \text{ and } g<b \\
     60^\circ \times \frac{b-r}{\max -\min }+120^\circ, & \text{if } \max =g \\
     60^\circ \times \frac{r-g}{\max -\min }+240^\circ, & \text{if max }=b
     \end{cases}
     $$

   - 饱和度（Saturation）计算如下：
     $$
     s = \begin{cases}
     0, & \text{if } \max = 0 \\
     \frac{\max -\min }{\max }=1-\frac{\min }{\max }, & \text{otherwise}
     \end{cases}
     $$

   - 明度（Value）计算如下：
     $$
     v = \max
     $$

2. **yolov3_darknet53_pedestrian模型概述**：
   行人检测是利用计算机视觉技术判断图像中是否存在行人并给予精确定位，一般用矩形框表示，这也是典型的目标检测问题。该PaddleHub Module的网络为YOLOv3，其中backbone为DarkNet53，采用百度自建大规模车辆数据集训练得到，支持预测和finetune。

3. **Paddlehub的安装与调用**：
   首先安装PaddlePaddle以百度多年的深度学习技术研究和业务应用为基础，集深度学习核心训练和推理框架、基础模型库、端到端开发套件、丰富的工具组件于一体的底层环境。

4. **基于Anaconda的虚拟环境搭建**：
   Anaconda是一个具有开源、安装过程简单、高性能使用Python和R语言的高性能虚拟环境，具有各种conda包、环境管理器、1000+的开源库。

5. **工作空间和功能包**：
   Workspace是一个存放工程开发相关文件的文件夹，内容如下:
   - src: 代码空间（Source Space）
   - build: 编译空间（Build Space）
   - devel: 开发空间（Development Space）
   - install: 安装空间（Install Space）


## 巡线设计
### 设计思路
![巡线流程图](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/巡线流程图.jpg)


### 函数准备
- **质点计算函数 particle()**：在图像中取20条横线，以10为步长进行遍历。
- **可视化函数 visualization()**：在图像上画出质点的位置和相邻质点之间的连线。
- **斜率函数()**：算出相邻质点之间的斜率，其中若斜率为无穷，则输出0。
- **转换函数 transition(K)**：将斜率转换为运动参数，把图像由上至下分为a，b，c三个区域，根据斜率大小分为left、leftleft、right、rightright、forward五种状态。
- **判断函数 judge()**：根据运动参数判断小车行驶方向。

### 实现步骤
+ **第一步**：用cv2.imread读取图片。
+ **第二步**：用cv2.COLOR_BGR2HSV进行色彩空间的转化。
+ **第三步**：用HSV处理得到只含黑线部分的二值图。
+ **第四步**：找到最大轮廓，用cv2.minAreaRect和cv2.boxPoints得到最大轮廓的最小外接矩形的四个顶点。用cv2.drawContours画出最大轮廓的最小外接矩形轮廓。
+ **第五步**：执行质点计算函数 particle()。
+ **第六步**：执行可视化函数 visualization()。

+ **第七步**：执行斜率函数 slope()。
+ **第八步**：执行转换函数 transition(K)。
+ **第九步**：执行判断函数 judge()。

### 具体应用
在实际运行中，本项目把判断函数输出的运动信息做了进一步细化。该项目以修正半径 R 为唯一可控制变量，在确定线速度 v 的情况下，通过如下公式求出角速度 w：

$$
w = \frac{v}{R}
$$

通过小车位置 x 来确定修正半径 R，具体步骤如下：

首先求出偏移量 Δx：

$$
\Delta x = 320 - x
$$

然后求出偏移方向 i：

$$
i = \frac{|\Delta x|}{\Delta x}
$$

最后确定修正半径 R：

$$
R = \begin{cases}
+\infty, & 0 \leq |\Delta x| < 15 \\
6.5i, & 15 \leq |\Delta x| < 30 \\
5.5i, & 30 \leq |\Delta x| < 50 \\
4.5i, & 50 \leq |\Delta x| < 70 \\
3i, & 70 \leq |\Delta x| < 75 \\
2i, & 75 \leq |\Delta x| < 120 \\
0.65i, & 120 \leq |\Delta x| < 200 \\
0.6i, & 200 \leq |\Delta x| < 280 \\
0.5i, & 280 \leq |\Delta x| < 320 \\
\end{cases}
$$


## 人物检测设计
### 设计思路
![人物检测设计思路](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/人物检测设计思路.jpg)
### 函数准备
- **数据处理函数 data(a)**：用于处理 result 数据。处理 result 后得到的数据 a\_list 为列表，包含人物编号、最高点 y 坐标、最低点 y 坐标、最左点 x 坐标、最右点 x 坐标。
- **红蓝颜色比较函数 comparison(p)**：通过 data(a) 处理后的坐标点依次遍历检测到的人物。HSV\_red 和 HSV\_blue 为 HSV 处理后的二值图，统计两者在同一人物区域非零像素点的个数，通过比较判断出红色或蓝色。同时，通过 blue\_N 和 red\_N 统计蓝色系上衣人数和红色系上衣人数。

### 实现步骤
+ **第一步**：输入图片路径，通过 cv2.imread 读取图片，读取 BGR 格式。
+ **第二步**：使用 yolov3\_darknet53\_pedestrian 模型检测人物位置。
+ **第三步**：将图片通过 HSV 处理，分别得到只有蓝色部分的 HSV\_blue 二值化图片，分别得到只有红色部分的 HSV\_red 二值化图片。
+ **第四步**：通过 data(result) 简化数据。
+ **第五步**：用 len(result[0]['data']) 得到人物总数，使用 comparison(i) 判断每个人的颜色。
+ **第六步**：输出人群总数、蓝色系上衣人数、红色系上衣人数。

### 具体应用
在平安城市车实际运行中，本项目通过动态拍摄，并将连续拍摄的图片进行拼接，最后进行图片处理，结果鲁棒性高。



---
# 测试分析
## 测试分析

### 测试准备

1. **平安城市小车**：在测试之前，我们考虑了车的配重以及摄像头的摆放位置，对小车进行了合理改装。

2. **场地**：在参考多种环境及可能出现的不确定环境因素下，选择以室内为主体的测试环境，并用地纸覆盖室内场地进行测试。

3. **软件**：依托于 ROS-Kinetic 与 Ubuntu 系统的良好兼容性，选择了基于它们进行开发的相关程序进行测试。

4. **支持技术**：根据目前成熟的 OpenCV 技术以及其跨平台性和兼容性，依托其进行视觉识别处理。在运动方面，基于 ROS 进行环境配置和开发测试。

5. **编译环境**：本项目软件平台采用基于 Ubuntu 16.04 的 Kinetic 版的机器人操作系统ROS。

### 测试内容与分析

测试内容如下表所示：

| 实验阶段/次数 | 实验指标     | 算法改进前正确率 | 算法改进后正确率 | 升比例 |
| :--------------: | :------: | :------: | :------: | :------: |
| 3              | 人群识别     | 76%              | 100%             | 24%    |
| 5              | 巡线任务     | 71%              | 100%             | 29%    |

#### 巡线测试

执行可视化函数 visualization()，对检测到的道路进行分析，并预测小车修正轨迹，效果图如下：

![可视化结果](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/可视化结果.png)

![修正预测图](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/修正预测图.jpg)

#### 人物检测测试

通过 yolov3_darknet53_pedestrian 模型检测人物位置，效果如下图：

![人物位置检测结果](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/人物位置检测结果.png)

将图片通过 HSV 处理，分别得到只有蓝色部分的 HSV\_blue 二值化图片和只有红色部分的 HSV\_red 二值化图片，效果如下图：

![处理后只含蓝色的二值图](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/处理后只含蓝色的二值图.png)

![处理后只含红色的二值图](https://vaporfigurebed.oss-cn-beijing.aliyuncs.com//blogimg/处理后只含红色的二值图.png)

在多次实验中，经过对道路巡线和不同颜色人物识别方案的不断改进，同时根据实际运行环境进行参数调节和修正，代码在实际运用中效果提升显著，该项目完美地完成了赛题要求任务，并且鲁棒性极高。


---
# 应用前景
项目立意于推动国家平安城市建设为背景，以无人化、智能化、动态化的道 路巡逻移动机器人为助推手段，结合国内外先进视觉与多传感器融合无人驾驶技 术、基于深度学习的道路行人检测算法，可实现对城市路面多种状况的预警与联 控，如疫情时代下发烧患者在户外移动、携带可疑物品的行人或是正在实时违法 犯罪行为的罪犯们都将成为移动巡逻机器人的检测目标。 
巡逻机器人可以通过智能化的数据处理，对可能发生事件实施风险控制，通 过概率预测与结果预测，当事件损失期望达到预警阈值时，机器人可以迅速预警 通知相应管理人员、执法人员。
	
---
# 创新点
颜色识别不再单一的采用转换灰度图像，利用灰度图像进一步转换为二值化图像处理，而是采用了 HSV 范围筛选算法。该项目在实现路线检测的过程中遇到光线导致图像检测畸变，在算法调试前期大大降低了程序结果的成功率，团队通过讨论研究采用 HSV通道下不同特征图像的颜色范围进行筛选排除光纤对图像的影响以提高算法的稳定性和鲁棒性，经项目组测试，采用 HSV 范围筛选的算法检测成功率可达到 95% 以上。



