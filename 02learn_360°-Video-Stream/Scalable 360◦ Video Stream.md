[TOC]
# **Scalable 360° Video Stream**

## 1. Introduction

###  (1) What makes 360° video rise up?
| 兴起方面                          | 举例                         |
| --------------------------------- | ---------------------------- |
| VR，AR硬件设备                    | Oculus Rift & HTC Vive       |
| 全景相机的可用性                  | GoPro OmniAll & Insta360 One |
| 社交 / 视频平台支持全景视频的发布 | YouTube..                    |

### (2) The difficulties of the 360° video.
| 难点           | 具体                         |
| -------------- | ---------------------------- |
| 带宽要求高     | **25 Mb/s **--> **400 Mb/s** |
| 更大的存储空间 | 90mins视频大概270GB空间      |
| 低时延         | 应该不超过几十毫秒           |
| 复杂的视图适应 | 适应头部移动所需视频视角     |

其中存储空间：全景视频需要
- 一个场景的多个视图 

- 为了适应不同客户端需要不同格式和分辨率的视频文件 

- 需要自适应率来支持自适应率ABR比特流

  ![image-20210111104240367](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210111104240367.png)

## 2. 概念综述

### (1) 捕获视频，编码，存储的数据模型

**捕获**：视频是由一个**中心摄像机系统**捕获记录的。

- k个分布式的2-D摄像机环境生成k个独立的MPEG-DASH视频流。
- 同时还可以将视频设备配置在一起后**缝合内容**。

![捕获视频](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210108090449538.png)

**编码**：已有的**全向视频编码技术**和**分层编码技术**可以适应视角的变化。

【待补充两种编码技术的可行性】

**存储**：以视角分块存储。

![视角分块](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210108090434128.png)

- MPD清单文件。

- MPEG指定了一组**360°视频投影二维平面**的方案。此外，它指定了在ISO基本媒体的**全景视频的存储格式**，以及**在DASH上的封装和指令**，以及可以变成全景视频的标准。

  **投影方法**：

  【待补充哪个更好用】
  
  | 种类     | 矩形 | 立方          | 金字塔型        |
  | -------- | ---- | ------------- | --------------- |
  | 空间占用 | 最大 | 比第一种少25% | 减少80%文件大小 |
  | 其他要求 | \    | \             | 需要GPU处理     |


### (2) 全景视频的架构

高质量的全景视频架构是基于**现有CDN架构**的基础上的，架构分为**创建**，**传输**和**边缘交付**。

![全景视频的架构](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210110172834085.png)

### (3) 全景视频的放映和流媒体的准备

#### 调整视频质量：

适应带宽，调整视角

<img src="https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210111110527940.png" alt="调整视频质量" style="zoom:80%;" />

#### 解决延迟的方法：

用到的最广泛

**ONE：预测运动状况**以便于提前加载

| 困难                     | 原因                                                   |
| ------------------------ | ------------------------------------------------------ |
| 解决视角预测的短期约束   | 基于时间的方法保持**较小的回放缓冲区**<br>（客户端的） |
| 需要提前3s预测           | VR的运动到光子延迟要求小于20 ms                        |
| 解决混合码率片的边界效应 | 组合图像块渲染时可见边框和质量不一致                   |

**TWO：**为每个视频块**独立创建视角自适应表示**。

预定义的视角，感兴趣区域的质量高，其余低。
| 公司      | 采用方法                           | 优点                                           | 缺点 |
| --------- | ---------------------------------- | ---------------------------------------------- | ---- |
| Facebook  | 自适应360°视频流<br>多分辨率全景图 | 用一个解码器对视频进行解码<br>降低了整体分辨率 |      |
| Pixvana   | 自适应360°视频流                   |                                                |      |
| ...待补充 |                                    |                                                |      |
| ...待补充 |                                    |                                                |      |

**THREE：传输整个球面框架的数据**，这个是有效的，但可能会增加这种方法的存储和带宽需求。

### (4) 视频传输 [重点]

#### 目标：

- **减少传输的数据量**
- **保持高质量和低延迟**

#### 目前挑战：

- **缺乏评价全景视频质量的方法**，很多文献只使用**图像质量指标QoE**来评估**观众感知的质量**
- 注重个性化和交互式，需要可扩展的360°交互系统。

#### 提高传输速度的方法：

前提：360°视频内容交付系统，视频内容是由**地点**和**事件**关联的。

**发现连贯性 / 一致性 [重要] **

- **相邻贴图**
- **轨迹预测**
- 相邻**客户端**
- **兴趣点**引发的行为近似
- 凝视检测（后面的研究说）

![兴趣点示意图](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210111112644000.png)

利用一致性可以**避免将整个全景视频片段**传输到边缘服务器上，并只**预取需要的内容**来极大地**减少流量和延迟**。

## 3. 研究结论 / 现状

### （1）ABR流媒体的流行 ---> **QoE指标提议**。

| 提出机构 / 组织/研究者 | 评价指标                                     |
| ---------------------- | -------------------------------------------- |
| ITU                    | 延迟                                         |
| ITU                    | **视频流到客户端的平均质量**                 |
| ITU                    | **在播放期间质量变化的频率**                 |
| \                      | 将QoS映射到QoE<br/>（速率自适应算法）        |
| Singla                 | **晕屏**程度/效果                            |
| Upenik                 | 全方位内容主观评价的试验台<br>仅限全静止图片 |
| \                      | 根据**电影摄影规则**提供的观看指导           |

注：评估**影响晕屏的两个场景因素**

- **HMDs观看全方位360°视频**
- **观看360°视频时突然的场景变化**

### （2）近期的相关研究结论

- 只有**一个视角**的视频及其**相邻的视频**流到客户端，这可以导致高达65%的比特率节省。

- Zhou等人的评估表明，**视角的突然变化**会导致未观看片段的检索**浪费占总下载带宽的20%**。

- Bao等人针对全向360°视频提出了一种**基于运动**的方法，他们证明了**用户对视区的选择是相关的**。这个知识可以用来发送多播流到**地理位置**上的观看者。

- **凝视检测**，定位用户的焦点，人眼仅能在**大约20度的小中心视角**下看到丰富的图像，凝视信息将作为控制信息**从客户端流向上游边缘和分布树的节点**。一种大幅度减少流量的方法是将视频编码为多个分辨率和平铺大小，类似于为单视图视频提出的方法。**小尺寸高分辨率**视频块可以作为客户的**中心视觉**，**大尺寸低分辨率**视频块可以作为客户的**周边视觉**。

  <img src="https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210111115234528.png" alt="凝视检测" style="zoom:67%;" />

## 4. 后序课题探索

目前困难：**内容创作、视频分发和QoE**

- **利用一致性**优化**360°视频传输**，使其可扩展、准确和高效。（做缓存
- 在现有CDN架构的基础上，还没有设计**新的机制来满足360°视频严格的延迟要求**。**【参考文献和目前程度 + 待找两个现有机制】**
- **预测轨迹**的算法
- 视频**兴趣点挖掘**
- 凝视检测
- **无法将电影摄影规则融入内容创作技巧**。
- 影响晕屏的因素。（**网络特性**的延迟、抖动、可用带宽、视图/贴图退化率和**流媒体特性**的延迟、质量变化、视口变化的影响）

## 5. 内容问题补充

### （1）上次要找的两个系统

已经对**多媒体存储系统**有所研究，并扩展到在单个服务器上存储相关的三维流。

此外，现有**大规模的map-reduce存储系统**，包括**Hadoop分布式文件系统**和**谷歌文件系统**。

[网上的解释](https://www.zhihu.com/question/333417513)

### （2）两种编码技术

在过去的中已经提出了一系列的**全向视频编码技术**。最近，人们提出了的**分层编码方案**，其目标是减少延迟并更快地适应视角的变化。虽然许多方法需要特殊的播放器，浏览器可以用来观看360◦视频，如果他们支持WebVR标准。

>A series of encoding techniques for such omnidirectional videos have been presented in the past . Recently, layered encoding schemes have been proposed , which have the goal of reducing stalling and adapting quicker to viewport changes. 

[有一个研究很不错的样子](https://github.com/RyanXingQL/Blog/blob/master/posts/mfqev2.md)

### （3）视频的投影贴图格式

| 种类     | 矩形 | 立方          | 金字塔型        |
| -------- | ---- | ------------- | --------------- |
| 空间占用 | 最大 | 比第一种少25% | 减少80%文件大小 |
| 其他要求 | \    | \             | 需要GPU处理     |

调研油管：

#### 矩形（Equirectangular）：

![image-20210112231601956](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210112231601956.png)

- 这种格式的优点是比较直观，并且投影是矩形的。
- 缺点也很明显：
  1. 球体的上下**两极**投影出来的**像素数很多**，而细节内容比较丰富的**赤道区域**相比来说像**素数就很少**，导致还原时清晰度比较糟糕。
  2. 另外，这种格式的画面在**未渲染的情况下扭曲比较明显**

#### 立方形（Cube Map）：

先投影到立方体后，再展开

![image-20210112231843272](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210112231843272.png)

但是像素分布仍然不均匀

#### 目前谷歌用的投影技术 - 等角度立方体贴图”（EAC）

![image-20210112232536103](https://littlefisher.oss-cn-beijing.aliyuncs.com/images/image-20210112232536103.png)

更改优化投影时的采样点位置，使得边角与中心的像素密度相等。

这样做的好处就是在相同的源视频分辨率下可以**提高细节部分的清晰度**。
[还不错的讲解](https://zhuanlan.zhihu.com/p/106922217)