- [x] 1.完成Motion_inbetween整体实验，并得到实验结果(ddl🕥 25.10.19)
- [ ] 2.完成实验部分的剩余内容和初稿撰写(Mobisys abstract ddl🕥 25.11.29)

**10.11**  
1.MDM_inbetween训练800个epoch，使用的Motion_CLIP为2700个数据集，间隔4个点采样(实际上只对前90个数据进行了训练，因为代码写错了)，其测试集的效果：  

https://github.com/user-attachments/assets/a7bcdeda-91f2-4349-aa02-fdd17fcf3f8e

这个结果很有意思：从某种意义上来讲，我们的condition在编码后并没有起到很大的指导作用，或者说只能对整体起到一个非常微弱的引导，无法做到每帧的精确识别  
2.增加两项实验：Motion_CLIP随机采样，其中batch_size设置为64，epoch = 400；Motion_CLIP随机采样，其中batch_size设置为256，epoch = 1000  
$${\color{red}问题}$$  
$${\color{red}1.若包含6个人以及8个位置，那么数据集规模差不多有2TB左右，如何进行处理？}$$  
**10.12**  
- [x] 电磁大作业：paper阅读
- [x] 电磁大作业：paper阅读 

1.由于服务器内存原因，上述两项实验只能单独去跑  
2.Motion_CLIP(batch_size = 30，顺序采样，epoch = 300，训练时间 = 20h)测试效果：从loss曲线可知，仍未收敛还可以继续训练  

<img width="846" height="547" alt="7a1866f9-5589-4b44-96fc-4caadcb72872" src="https://github.com/user-attachments/assets/40c7e31f-8002-4ba6-8301-e5e8096c2d71" />  

训练集batch_size = 30效果：大部分能达到0.9以上，但由于没收敛，所以效果会差一点  
测试集batch_size = 30效果：没有丝毫效果  
3.将上述Motion_CLIP作为Motion_inbetween的encoder，epoch = 800，其测试集的效果：

https://github.com/user-attachments/assets/7f6b9492-f734-4807-aab8-9a270d3c0018

与sample001相比，两者好像基本没有区别，但是Motioin_CLIP差别很大，说明Motion_CLIP在模型中没起到作用，有没有可能是因为encoder输出值过小(好像也还好，Motion和emb基本在一个数量级上)  
**10.13**  
- [x] 电磁大作业：paper阅读
- [x] 电磁大作业：报告撰写
- [x] 电磁大作业：报告撰写

1.Motion_CLIP(batch_size = 64，随机采样，epoch = 400，训练时间 = 12h)测试效果：从loss曲线可知，仍未收敛还可以继续训练

<img width="846" height="547" alt="2bc941ee-f9e9-4d99-ae42-fb182e58aa31" src="https://github.com/user-attachments/assets/e0e26161-e1f6-4157-8d8d-49f606f83fde" />

训练集batch_size = 64效果：虽然loss还可以继续收敛，但是已经比较小了，结果没有丝毫效果  
测试集batch_size = 64效果：没有丝毫效果  

2.将上述Motion_CLIP作为Motion_inbetween的encoder，epoch = 800，其测试集的效果：  

https://github.com/user-attachments/assets/dd4d4c13-cb22-4a9e-bb10-d520fae17ffd

结果在预料之中，和之前的两个结果完全一致  
3.Motion_CLIP随机采样，其中batch_size设置为128，epoch = 10000，每间隔200个epoch保存一次模型  

**10.14**  
$${\color{red}问题}$$  
$${\color{red}1.直接加的效果不好，能否将其改为concatenate或者交叉注意力？}$$  
$${\color{red}2.MotionCLIP还是得进行微调}$$
- [x] 电磁大作业：代码仿真
- [x] 电磁大作业：代码仿真(基本上已经实现定位，下一步：完善代码)

**10.15**  
1.对Motion_inbetween模型进行微调，使得Encoder_image在模型训练过程中可以被更新  
2.对Motion_inbetween模型中的emb和x的结合方式进行修改，改为concatenate或者交叉注意力(将其更改为concatenate，在dim=0的维度上进行cat)  
3.对epoch = 2200的Motion_CLIP模型进行测试(batch_size = 128, 训练时长：57h)：

<img width="833" height="547" alt="6f475a9c-883d-4c35-83cf-bbab769c5122" src="https://github.com/user-attachments/assets/6e241dbb-077b-42b7-b982-085e8292ffc5" />

训练集batch_size = 128效果：大部分都能达到0.9以上  
测试集batch_size = 128效果：没有效果，但是极少数能达到0.9

**10.16**  
1.Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)，其测试集的效果和之间一致，即Ground_truth和Out_Motion不匹配  
2.Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新，emb和x在dim=0采用concatenate)，其测试集的效果和之前一致，即Ground_truth和Out_Motion不匹配  
3.思考1：依据上述结果，怀疑是否是input_motion和AoA_Dfs是不匹配的，后续经过代码检查，已确认两者是匹配的  
&nbsp;&nbsp;思考2：依据上述结果，怀疑是否是learning_rate太小，导致Motion_CLIP无法被更新，于是将lr由1e-4修改为1e-3，结果发现刚开始训练时Motion_CLIP参数变化很快，但是迭代几步之后就不动了；而1e-4虽然更新慢，但在足够多的步数后还是与初始参数相差较大  
4.由于没有得到理想的结果，于是将num_sample设置为20，观察每个模型总体的识别成功率，结果发现Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)的识别率在60%左右；Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新，emb和x在dim=0采用concatenate)的识别率在50%左右；Motion_CLIP(顺序采样，数据集 = 90，batch_size = 30)的识别率在50%左右：上述实验结果显然说明一个问题：Motion_CLIP并没有起到任何的作用，而且emb和x如何结合对结果也没有影响  
5.最后推测最可能的原因是AoA_Dfs数据的量级太小，于是将Motion_CLIP(随机采样，betch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)采样部分AoA_Dfs乘以3，训练部分不改变，发现识别率上升至80%；接下来将针对这一部分，进行以下实验，并记录结果：  
Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新，emb和x在dim=0采用concatenate)采样部分AoA_Dfs乘以3，训练部分不改变：识别率上升至60%(相较于之前的50%，变化不大)  
Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)采样部分AoA_Dfs乘以10，训练部分不改变：输出结果直接失效  
Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)采样部分AoA_Dfs乘以5，训练部分不改变：部分输出结果失效  
Motion_CLIP(顺序采样，数据集 = 90，batch_size = 30)采样部分AoA_Dfs乘以3，训练部分不改变：识别率基本不变  
Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)采样部分AoA_Dfs乘以3，训练部分不改变，重复采样5次：结果明显改善，不会出现出现不同的动作

https://github.com/user-attachments/assets/ae102335-1d23-47d9-bc3e-b9e7d15479db

6.增加后续实验：  
Motion_CLIP(随机采样，batch_size = 128，epoch = 2400) + Motion_inbetween(Motion_CLIP参数可更新)训练和采样部分AoA_Dfs乘以3，epoch = 1000：识别率在85%左右，和下面的实验结果一致  
Motion_CLIP(随机采样，batch_size = 64) + Motion_inbetween(Motion_CLIP参数可更新)训练和采样部分AoA_Dfs乘以3，epoch = 1000：识别率在85%左右，和下面的实验结果一致  
Motion_CLIP(随机采样，batch_size = 128，epoch = 2400) + Motion_inbetween(Motion_CLIP参数可更新)训练和采样部分AoA_Dfs乘以7，epoch = 1000：识别率在85%左右  
Motion_CLIP(随机采样，batch_size = 128，epoch = 2400) + Motion_inbetween(Motion_CLIP参数可更新)训练部分AoA_Dfs乘以7，采样部分AoA_Dfs乘以9，epoch = 1000：识别率在85%左右  

**10.17**  
1.如何继续提升模型的识别率？上述实验其实已经从侧面证明了不管Motion_CLIP预训练效果如何，其最终都会收敛至同一结果；将模型改为concatenate，dim=2是否能提升其识别率？  
2.增加后续实验：  
Motion_CLIP(随机采样，batch_size = 128，epoch = 2400) + Motion_inbetween(不做CLIP，也就是做ablate，emb和x在dim=2采用concatenate)训练和采样部分AoA_Dfs乘以5，epoch = 1000： 
模型识别率相较于下面部分识别率有所下降，85%左右
Motion_CLIP(随机采样，batch_size = 128，epoch = 2400) + Motion_inbetween(Motion_CLIP参数可更新，emb和x在dim=2采用concatenate)训练和采样部分AoA_Dfs乘以5，epoch = 1000：  
结果证明使用concatenate之后的识别率上升至95%左右  

**10.21**  
工作进展2总结：  
~~1.为什么直射路径在AoA_DFS频谱上会产生这么大的频谱偏移，不应该是在0频附近吗(可能原因：1.对CSI进行线性插值导致的)~~  
~~2.为什么对CSI进行unwrapped的时候，在第5个CSI部分会产生跳跃~~  
**3.如何对8个receiver进行整合？**  
**4.考虑一个比较好的场景**  
**5.Diffusion部分能否再优化一下，比如t直接加是不是有点不太好**  
**6.创新点落在优化z的生成那一块**  
**7.数据集是否可以优化一下，只有6个动作显然是偏少的(参考文献：https://aiotgroup.github.io/Person-in-WiFi-3D/ 和 https://ntu-aiot-lab.github.io/mm-fi?utm_source=chatgpt.com)**  
**8.参考Mobisys的相关论文(12月份左右，侧重实验)**  
**9.实验结果可视化得用blender美化一下**  
**10.对Encoder进行消融实验，结果表明对比学习那块并没有提升，思考一下为什么**

**10.22**  
1.Person-in-wifi-3d: metric：**Mean Per Joint Dimension Location Error (MPJDLE)** 和 **Mean Per Joint Position Error (MPJPE)**  
2.MDM and Motion inbetween: **FID、R-precision、Diversity、Multimodality**  
3.Wi-Mesh: **Per Vertex Error(PVE)和MPJPE**  
4.mmMesh: **Average Joint Rotation Error**
4.WiFi sensing of Human Activity Recognition using Continuous AoA-ToF Maps  
**5.得到val数据集的metric，流程：将lauch.json文件中的num_sample更改为对应的val数据集中的总数据；运行eval/evaluation.py的文件**(对于135个数据，总的eval时间为20min左右)  

**10.23**  
1.WiFi sensing of Human Activity Recognition using Continuous AoA-ToF Maps：先通过**共轭相乘代替线性相位去噪**，然后通过**PDP识别LOS并滤除**  
2.Person-in-wifi数据集没有出现**csi相位突变**的情况，但是通过AoA_DFS算法得到的结果也没有能做到精确识别(例如多人场景的识别)  
3.对于Person-in-wifi数据集，我们假设线性相位去噪操作是没有问题的(因为相位没有发生突变)，先不可视化其AoA_DFS联合估计谱，而只对其MUSIC算法以及Capon算法进行探究，可视化其AoA  
场景(human = 1，线性相位去噪+无滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/c66e3af7-f665-4356-bff1-dfb2276a1c17" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/5e020aba-1ab1-4b1b-b783-7834fe6f1d2b" />  

场景(human = 1，线性相位去噪+静态滤波(无共轭相乘)) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/b4257fbc-9a50-4dec-a7ca-0d229e37318c" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/f1115b0d-06d3-4c07-8567-804a553e0262" />  

场景(human = 1，线性相位去噪+静态滤波(有共轭相乘, ref=0)) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/341493cc-93bf-4637-8de3-dcb51d2c504c" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/07900202-ed65-4473-8af1-c745e97727af" />

场景(human = 1，线性相位去噪+静态滤波(有共轭相乘, ref=1)) MUSIC 和 Capon 

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/103c7a11-3434-40a3-9557-73f69b49c839" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/dc0f8c46-fecd-4ce8-b19f-c092b9eec0a4" />

场景(human = 1，线性相位去噪+高通滤波(无共轭相乘，Hz = 2)) MUSIC 和 Capon 

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/6bcf4f94-3572-4b79-8487-5ae1150d1010" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/f5ed0435-d49f-46f3-bdb5-e9c6b00cb63d" />

场景(human = 1，线性相位去噪+高通滤波(无共轭相乘，Hz = 20)) MUSIC 和 Capon 

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/cb1aaf05-5e84-46f5-9a01-ec9c8119b475" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/dd1bc6a1-344d-4b9c-9edd-f61997e59368" />

场景(human = 1，线性相位去噪+高通滤波(共轭相乘，Hz = 2)) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/c5a91dfb-93e2-4ea1-bc60-aab604b7ff01" /> <img width="500" height="417" alt="image" src="https://github.com/user-attachments/assets/1266fe53-126f-400c-812a-658228a9f0c0" />

场景(human = 1，无线性相位去噪+无滤波) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/e614d05c-b17c-40d1-8aee-d5abf53c6702" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/a59f1abd-4852-4375-a0ff-1b9fcadaaa0d" />

场景(human = 1，无线性相位去噪+无滤波(共轭相乘)) MUSIC 和 Capon

<img width="500" height="416" alt="image" src="https://github.com/user-attachments/assets/c1ee9cc5-339d-4362-ab15-6f583a1fcec9" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/9bc39ab2-791a-4fef-b8f5-7bac9ca06b0e" />

Person-in-wifi-3d的数据集已经经过线性相位去噪

场景(human = 2，线性相位去噪+无滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/2851f03d-97c6-4aeb-b7d3-df6df40f2645" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/e9cabc1e-e9bb-4930-91de-8a1a70ffc3e8" />

场景(human = 2，线性相位去噪+静态滤波(无共轭相乘)) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/28d1d2a6-d456-43de-9acd-c57e18e503a4" /> <img width="500" height="416" alt="image" src="https://github.com/user-attachments/assets/992ead00-acc5-429f-98fa-b5e27d480987" />

场景(human = 2，线性相位去噪+静态滤波(有共轭相乘, ref=0)) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/e1e117a8-49b0-4880-96df-71df714ab8f6" /> <img width="500" height="414" alt="image" src="https://github.com/user-attachments/assets/58f6d12b-34e3-44aa-95b9-c7a4e464d212" />

场景(human = 2，线性相位去噪+静态滤波(有共轭相乘, ref=1)) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/972e6da3-e2fb-4bbe-aad2-abc65aa02340" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/21dd29aa-b34f-458d-bce2-7549a1bb7ad0" />

场景(human = 2，线性相位去噪+高通滤波(无共轭相乘，Hz = 2)) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/ef520a7e-0c7a-4786-a7b5-9f726f4bf2b9" /> <img width="500" height="415" alt="image" src="https://github.com/user-attachments/assets/567af455-04fe-40e1-8466-7c8c5ef07be6" />

场景(human = 2，线性相位去噪+高通滤波(无共轭相乘，Hz = 20)) MUSIC 和 Capon 

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/c0d30ea6-f96b-4cf1-ab2d-2c9bb3acbe95" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/a0432001-b430-4891-bec2-4c6e76078c11" />

场景(human = 2，线性相位去噪+高通滤波(共轭相乘，Hz = 2)) MUSIC 和 Capon 

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/18c9cbea-61f6-4039-9cc0-03f32d3c0f7a" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/33461a4a-2fac-4f54-a40d-e30f9885cb36" />

**10.24**  
1.重新看了一遍MUSIC和Capon算法的代码，发现一个问题：原来的算法将子载波视为快拍数，从而得到多条时间维度的MUSIC和Capon；对上述算法进行改变，改为将时间视为快拍数，从而得到多条子载波维度的MUSIC和Capon(AoA_my) 结果显示，得到的结果基本还是一致的，可能有些许偏差  
2.基于10.23的几个实验，可以得到以下结论：person-in-wifi-3d数据集是经过**线性相位去噪**的；无论是高通滤波还是静态滤波，好像都**没有去除掉直射路径分量**  
3.基于上述问题，尝试使用新的滤波算法：**特征空间滤波**(发现如果只是对最大特征值进行滤除，效果不好，于是将其阈值设置为2，即对前两个特征值进行滤除)  

**10.25**  
1.对于之前提到的将时间视为快拍数，那怎么针对每一帧都得到对应的AoA_DFS联合谱估计？所以还是得将子载波数视为快拍数  
2.对于**特征空间滤波的有效性进行验证(阈值设置为1)**  

场景(场景1，human = 1，线性相位去噪+无特征空间滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/a748a521-6b09-42ac-b57d-efa0c0e890ed" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/bb8fbf58-04bf-418c-9d97-0c999f0a5a534" />

场景(场景1，human = 1，线性相位去噪+特征空间滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/f0dfe988-1ff6-4491-b456-5d40f65ae88d" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/df9af048-1071-4bc7-a501-88497d7dc296" />

场景(场景2，human = 1，线性相位去噪+无特征空间滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/94e6ff9b-e759-4d58-871a-a32b19ff22d7" /> <img width="500" height="414" alt="image" src="https://github.com/user-attachments/assets/5f70571b-ac69-4c62-9fc3-91f9c48122c9" />

场景(场景2，human = 1，线性相位去噪+特征空间滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/917f4888-d187-487f-a20a-0e764d2ac48f" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/eabb4959-8370-4504-90bc-a5abf336e06d" />

场景(场景1，human = 2，线性相位去噪+无特征空间滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/f03d8bf3-d902-4ae9-bd0a-298bfeb33ba4" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/584459ae-d6f7-4128-9958-9c58bddcd25b" />

场景(场景1，human = 2，线性相位去噪+特征空间滤波) MUSIC 和 Capon  

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/bbee3f4c-5641-41fc-9410-e2589a22a65b" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/30ffb98d-c905-45c9-b9d4-1d8fb912b97b" />

场景(场景2，human = 2，线性相位去噪+无特征空间滤波) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/57ba1867-ed1a-40df-abf4-351532f5d4cb" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/b8744af4-0b4d-4bee-8bed-84a43ef569aa" />

场景(场景2，human = 2，线性相位去噪+特征空间滤波) MUSIC 和 Capon

<img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/1b275e72-8b69-4075-8996-bdfc3ff3bc4e" /> <img width="500" height="413" alt="image" src="https://github.com/user-attachments/assets/0da3354e-ad4b-4d27-a5ee-31a503f0ba29" />

3.最后选择线性相位去噪+特征空间滤波(阈值设置为1)+基于MUSIC的AoA_DFS联合谱估计

**10.27**  
1.为什么消融实验没有用(从本质上来讲，预训练的Encoder在参数更新时，最终收敛的方向和无预训练是一样的)：Motion Encoder过于简单，只是简单的线性层；数据集问题，csi数据集处理不好，比如含噪声过多/没有对齐(主要表现在对比学习在测试集上效果很差)；  
2.进行对比学习的目的：p(z|x)，我们希望能够将csi信号编码为z，也就是真实的动作，再将z作为输入通过扩散模型q(y|z)输出得到mesh，这里p可以使用对比学习的方法将z和x两个不同的空间尽可能拉近，这里其实有一点问题：就是我们使用对比学习的时候，我们正负对的设计其实是增加了模型的收敛难度，同时正负对的引入对我们的期望其实是不相关的，我们只是想要两个空间尽可能接近，但不要求两个空间内的元素距离尽可能远离，**所以我在想能否对loss部分进行修正**













