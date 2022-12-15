*-wise* 后缀表示   ...的方法

prior estimations 先验估计 就是有标签，已知是啥，送入模型拟合



remote photoplethysmography (rPPG)

real-time 实时的

multi-patch 多发面的

blood volume pulse (BVP)   血流量

Beat Per Minute  (BPM)   每分钟节拍数

region of interest (ROI)  感兴趣区

multimodal 多模态的


Blind signal separation (BSS)：

盲信号分离指仅从观测的混合信号（通常是多个传感器的输出）中恢复独立的源信号。









***principle of rPPG:***

*HR measurement via rPPG is based on the principle of **optical absorption by the skin varies periodically with the blood volume pulse (BVP)**.* 

*Human skin is usually treated as a three-layer model: subcutis, dermis, and epidermis from inner to surface.* 

*The **hemoglobin** in the blood of dermis and subcutis layers, and **melanin** in the epidermis layer are the major chromatophores of human skin.* 

*The changes of hemoglobin content **during a cardiac cycle** would cause tiny color variations in the skin.* 

*Although **the color changes** are invisible to human eyes, they **can be captured by visible sensors**, which makes it possible to measure HR remotely.*



**challengse：**

- 虽然受控环境下rppg足够精准，但是心率测量不仅需要精确度，还需要**快速响应**

- 并且，一次评估只应当使用**很少个视频帧**

- rPPG信号会被人脸运动和光照变化影响
- 心率在50到90小范围浮动，当强烈情感和剧烈运动会有变化所以，单次心率测量在健康情感监测状态下有限制，需要持续的心率测量



**continuous HR measurement:**

1.  基于脸部标记和皮肤分割计算 multi-patch ROIs ，这样针对脸部运动问题有更好的鲁棒性
2.  把心动周期信号从 RGB空间 映射至 色度空间，以减少各个颜色通道间的巨大差异，再用滤除白噪声和不感兴趣的噪声
3.  考虑到心率信号的前后相关性，学习一个基于历史心率信号的心率**分布**，并用它完成心率检测



**如何消除光照与运动的影响：**

*Li  proposed a framework that achieved the stateof-the-art HR estimation accuracy on the public-domainMAHNOB-HCI database. **They used facial landmarksto locate the area of face. The influence of illumination wasremoved by comparing with the background, and the influenceof non-grid face motion was suppressed by statisticalanalysis and a few temporal filters.***





#### *3.1. ROI Selection and Processing*

***1. choose which area as ROI?***

*the most informative facial part containing color changes due to heart rhythms is the **cheek area**.The cheek area contains much less non-rigid motions due**to smiling and talking than the other areas.*

![image-20221104100057295](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104100057295.png) 

> *why not others?*
>
> *All of these ROIchoices contain some irrelevant areas, and may introducenon-grid motions. At the same time, all of ROI choices averageover a large face region, which may contain differentpatterns of local variations and lose the local consistency.*



***2. How to process the ROI?***

*After obtaining a ROI, we use a piece-wise linear wrapping method to wrap the cheek area into a M × N rectanglefor the convenience of computing.*

![image-20221104101309051](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104101309051.png) 

*Furthermore, in order purify the raw color signals,irrelevant pixels are removed from the rectangle ROI by using skin segmentation.*

![image-20221104101554414](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104101554414.png) 



#### *3.2. Local Chrominance Features Generation and Temporal Filtering*

***1. Divide the processed ROI into smaller ones***

*we divide the whole rectangle ROI (M × N) into K smaller ones considering that the face is not a perfect lambertian surface, and smaller ROIs should have better consistency than a larger one.*

![image-20221104103019078](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104103019078.png) 

***2. Average Pooling***

<img src="C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104102936505.png" alt="image-20221104102936505" style="zoom: 67%;" /> 

![image-20221104103001219](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104103001219.png) 

> 简而言之，
> 把大ROI分割成K个小ROI，
> 每个小ROI的R通道avgpooling成1个值（小ROI R值之和除以小ROI的像素个数）
> G，B通道也一样，那么对于一个小ROI，就获得了3个值
> 一直观测，就获得了3个signal，即![image-20221104103810910](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104103810910.png) 
>
> k个ROI就获得了K*3个signal

***3. Transform from RGB space to chrominance space***

*Given these raw rPPG signals extracted from multipleROIs, we then transform the signals from RGB to chrominance space.The chrominance features are found to bemore robust to motion and illumination variations*. 

***S** is the chrominance signal.*

![0](C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104110941890.png) 



***4. filtering***

*After converting the rPPG signals into chrominance space, we use several additional filters to remove various artifacts.*

1. *We first use a **Gaussian smoothing filter** with a window size of 5 frames to **reduce noises introduced by ROI average pooling**.*
2. *Then, a 4th order **butterworth bandpass filter** with the transmission band of **[0.7, 4] Hz (corresponding***
   ***to [42, 240] bpm)** is used to **eliminate the frequencies that are less likely to be HR distributions.***



#### *3.3 HR Distribution*

*we propose to use the HR distribution to model the temporal relationship, and use it to modulate the succeeding HR measurement.*

***1. Gaussian distribution***

*it is reasonable to assume that the pulse frequency distribution of individual subjects follows a Gaussian distribution*<img src="C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104120948013.png" alt="image-20221104120948013" style="zoom:50%;" /> *where μ and σ are the mean and standard deviation of HR distribution, respectively*

***2. learn the $\mu$ and $\sigma$ (train the model)***

*For continuous HR measurement of a subject, we first learn μ~HR~ and σ~HR~. We need a period of time T to estimate HR without the help of HR distribution.Then, the parameters (μHR and σHR) canbe easily estimated by using the mean and standard deviation of prior estimations.*  *（The HR distribution models the subject’s HR within a recent period, and it **can be used as a constraint to remove outlier HR estimations**）*

*Specifically, we use prior estimations HR1,HR2, · · · ,HRT to compute HR distribution, and get the parameters μHR, σHR.*

***3. given a new sequence signal (predict)***

<img src="C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104123426739.png" alt="image-20221104123426739" style="zoom: 67%;" /> 

<img src="C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104123658136.png" alt="image-20221104123658136" style="zoom:67%;" /> 

 <img src="C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104123739019.png" alt="image-20221104123739019" style="zoom: 67%;" />



***3.4. Fusion of Estimates from Multi-ROIs***

*we have divided the ROI into k small regions and get k HR estimations for a unit signal sequence by using the methods above.*

*A simple average of the k estimations is not enough to get an accurate HR estimation, because we notice that some of the estimations are extreme high or low.:exclamation:*

*In order to reduce the influence of these extreme errors, we use a **median** HR estimation of the K estimates.*

*Specifically, we firstly sort all the k estimations, and we get {hr1 hr2, · · · hrk}. Then we choose the median 2l+1 estimations {hr[k/2]−l hr[k/2]−l+1, · · · hr[k/2]+l} as the stable estimations. Finally, we compute the HR estimation as* <img src="C:\Users\anbang\AppData\Roaming\Typora\typora-user-images\image-20221104170618132.png" alt="image-20221104170618132" style="zoom:50%;" /> 

（就是取中位数左右的部分求平均）



