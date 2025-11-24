- [x] 1.完成Motion_inbetween整体实验，并得到实验结果(ddl🕥 25.10.19)
- [ ] 2.完成实验部分的剩余内容和初稿撰写(Mobisys abstract ddl🕥 25.11.29)
<details>
<summary>📖 10.11 - 11.05</summary>  

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
~~**6.创新点落在优化z的生成那一块**~~  
**7.数据集是否可以优化一下，只有6个动作显然是偏少的(参考文献：https://aiotgroup.github.io/Person-in-WiFi-3D/ 和 https://ntu-aiot-lab.github.io/mm-fi?utm_source=chatgpt.com)**  
**8.参考Mobisys的相关论文(12月份左右，侧重实验)**  
~~**9.实验结果可视化得用blender美化一下**~~  
~~**10.对Encoder进行消融实验，结果表明对比学习那块并没有提升，思考一下为什么**~~  
~~**11.对于子载波的处理，能否直接截取没有跳变部分的子载波？**~~

**10.22**  
1.Person-in-wifi-3d: metric：**Mean Per Joint Dimension Location Error (MPJDLE)** 和 **Mean Per Joint Position Error (MPJPE)**  
2.MDM and Motion inbetween: **FID、R-precision、Diversity、Multimodality**  
3.Wi-Mesh: **Per Vertex Error(PVE)和MPJPE**  
4.mmMesh: **Average Joint Rotation Error**
4.WiFi sensing of Human Activity Recognition using Continuous AoA-ToF Maps  
**5.得到val数据集的metric，流程：运行edit.py文件；将lauch.json文件中的num_sample更改为对应的val数据集中的总数据；运行eval/evaluation.py的文件**(对于135个数据，总的eval时间为20min左右)  

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
1.为什么消融实验没有用(从本质上来讲，预训练的Encoder在参数更新时，最终收敛的方向和无预训练是一样的)：Motion Encoder过于简单，只是简单的线性层；数据集问题，csi数据集处理不好，比如含噪声过多/没有对齐(主要表现在对比学习在测试集上效果很差)；由于预训练数据集和训练数据集是相同的，是否不用进行微调；正负对的构建其实也是有问题的，是不是应该把相同的动作作为正对，不同的动作作为负对  
2.进行对比学习的目的：p(z|x)，我们希望能够将csi信号编码为z，也就是真实的动作，再将z作为输入通过扩散模型q(y|z)输出得到mesh，这里p可以使用对比学习的方法将z和x两个不同的空间尽可能拉近，这里其实有一点问题：就是我们使用对比学习的时候，我们正负对的设计其实是增加了模型的收敛难度，同时正负对的引入对我们的期望其实是不相关的，我们只是想要两个空间尽可能接近，但不要求两个空间内的元素距离尽可能远离，**所以我在想能否对loss部分进行修正**  
3.针对上述不进行微调的假设进行探究：  
Motion_CLIP(随机采样，batch_size = 128，epoch = 2400) + Motion_inbetween(Motion_CLIP参数不可更新，emb和x在dim=2采用concatenate)训练和采样部分AoA_Dfs乘以5，epoch = 1000:  

<img width="382" height="110" alt="image" src="https://github.com/user-attachments/assets/717888fc-f6a2-4c52-96f4-3ad538bfdbd0" />

对20个样本进行识别，识别率在90%

**10.28**  
1.对于blender中的mesh设置流程总结：  
**背景透明问题：图片保存格式改为PNG，颜色RGBA，胶片选择透明**  
**物体中心设置问题：物体->设置原点->(原点->几何中心)**  
**物体对齐问题：shift选中对齐的物体->条目->Align tools->align location**  
2.对于AoA_DFS的处理方面，需要考虑DFS生成的延迟问题，因此接下来对DFS的补偿窗口的选择进行探究：  
1-1-1-1 Rx5：15(频偏)(明显)  
2-1-1-1 Rx5：6(频偏) 26(明显)  
3-1-1-1 Rx5: 9(频偏) 25(明显)  
4-1-1-1 Rx5：7(频偏)(明显)  
5-1-1-1 Rx5：11(频偏) 19(明显)
6-1-1-1 Rx5：13(频偏)(明显)
6-1-1-2 Rx5：6(频偏)(明显)
6-1-1-3 Rx5：10(频偏)(明显)
6-1-1-4 Rx5：14(频偏)(明显)
6-1-1-5 Rx5：10(频偏)(明显)  
大约在10左右，因此选择补偿窗口的长度为10  

**10.29**  
1.使用更新后的AoA_Dfs数据集重新进行对比学习预训练：  
收敛速度和趋势与之前的数据集基本没有区别，还是很慢
2.使用更新后的AoA_Dfs数据集重新进行消融实验，Motion_inbetween(不做CLIP，也就是做ablate，emb和x在dim=2采用concatenate)训练和采样部分AoA_Dfs乘以5，epoch = 1000：  

<img width="367" height="116" alt="image" src="https://github.com/user-attachments/assets/4984db94-4b8b-4f44-9ced-4ea06d8b3939" />

$${\color{red}问题}$$  
$${\color{red}1.对于MotionCLIPmodified3/reference(random)(rolling load).py中的代码还可以继续优化训练速度：提前将Motion和AoA_Dfs进行归一化处理}$$  
3.关于分段式对比学习的实验记录：  
**将对比学习改为分段式训练，分段帧数为20帧，同时对Motion Encoder和AoA Encoder进行更改：Motion Encoder(Transformer Encoder)、AoA Encoder(Transformer Encoder)，其中batch_size = 128**  

<img width="348" height="441" alt="屏幕截图 2025-10-29 160055" src="https://github.com/user-attachments/assets/01596a95-b36c-46e8-bbce-d2e15d9eb040" />

相较于之前的逐帧对比学习，收敛速度有了很大提升，但是收敛速度还是不够快  

**在上述的基础上，将AoA Encoder部分的AoA embedding改为cnn，发现当cnn比较复杂时，模型收敛速度慢，于是将cnn简化处理，增大感受野和池化跨度，其中batch_size = 64**  

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/faace4ca-c42f-40de-93a4-287a8f9cee0e" />  

**增大learning rate至5e-5：**  

<img width="161" height="113" alt="image" src="https://github.com/user-attachments/assets/f4c22ca8-26ee-4add-9e5c-107c08edb0de" />  

观察loss，貌似收敛不了  

**增大learning rate至1e-4：**  

跟上述lr = 5e-5一样，没法收敛  

**增大learning rate至1e-5：**  

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/59d36b6c-88fd-4594-8514-06d5fcd0cf35" />  

收敛速度变快  

**将CNN改回简单的线性层，同时将batch_size改为64，learning rate = 1e-5**  

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/ad0295dc-9d93-4599-9098-b9343c1b4b45" />  

收敛速度比cnn还快  

**将CNN改回简单的线性层，同时将batch_size改为64，learning rate = 1e-4**

无法收敛  

**对数据集进行更换(30 nsub)，重新对上述进行实验(简单的线性层，同时将batch_size改为64，learning rate = 1e-5)**  

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/ad254cd3-6b83-46c4-9ff6-8da07342549c" />

对于之前的数据集，更换数据集之后的收敛速度有提升  

**10.30**  
1.使用nsub = 30的数据集(eigen filter)进行分段对比学习，其loss下降情况记录如下： 

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/64d392a9-831a-448f-ab26-1104599aac95" />

2.使用nsub = 30的数据集(eigen filter)进行消融实验，并对AoA_Encoder进行修改，同时注意到输出的AoA和x量级一致，因此将参数5取消  

<img width="517" height="184" alt="image" src="https://github.com/user-attachments/assets/94a26aa8-f8f4-4241-b8fc-88132abc0835" />  

**10.31**  
1.使用nsub = 30的数据集(eigen filter)进行分段扩散(frozen)，其loss下降情况记录如下(/home/newdisk/hch/diffusion motion inbetween(per segment frozen)/save/l4gughhb)：  

<img width="846" height="545" alt="image" src="https://github.com/user-attachments/assets/3ae2ccfe-7af9-45cc-b4ad-e87e470b5f85" />  

采样20个测试集样本，并进行可视化，结果显示模型识别率在60%左右，部分样本出现部分正确部分错误的情况

2.使用nsub = 30的数据集(eigen filter)进行分段扩散(消融实验)，其loss下降情况记录如下(/home/newdisk/hch/diffusion motion inbetween(per segment ablation)/save/mw4g59nz)：  

<img width="855" height="547" alt="image" src="https://github.com/user-attachments/assets/c594a6d7-a198-4841-b4e1-b9178d7b34f4" />  

采样20个测试集样本，并进行可视化，结果显示模型的识别率很低，基本上无法识别

<img width="377" height="155" alt="image" src="https://github.com/user-attachments/assets/6ea648de-dc74-4880-b42e-707a28298a16" />  

由上图可知，消融实验证明了预训练确实是能够提升模型的性能  

3.使用nsub = 30的数据集(eigen filter)进行分段扩散(微调实验，lr=0.01lr)，其loss下降情况记录如下(/home/newdisk/hch/diffusion motion inbetween(per segment finetune)/save/orojedp5)：  

<img width="846" height="545" alt="image" src="https://github.com/user-attachments/assets/80d9a4af-22d8-4da7-a6a0-7a1b4e8a26ce" />

采样20个测试样本，并进行可视化，结果显示模型识别率在65%左右，样本出现部分正确部分错误的情况

4.针对模型识别率不高的可能原因：模型的Encoder需要微调(可能性不大，因为之前的逐帧实验说明微调对结果影响并不大)；模型segment的间隔太大(可能性比较大，因为per frame的识别率是比较高的)，因此，将模型的segment缩短至10，重新对对比学习部分进行预训练：

~~4.重新看了一下上述两个实验的20个采样结果，消融实验模型的识别率虽然很低，但是不会出现部分正确，部分错误的情况，因此我觉得还是得进行微调，将预训练部分的lr改为0.1lr：~~

~~<img width="846" height="545" alt="image" src="https://github.com/user-attachments/assets/3a5b133e-cc9b-4b1b-b0f9-7d82a89ab23a" />~~

~~最后得到的结果和消融实验的结果基本一致，虽然没有出现部分错误的情况，但是整体识别率基本为0~~

~~**对diffusion输出视频进行可视化过程：1.建议运行一遍sample/edit.py 2.在render_mesh.py文件中修改sample_i为对应的希望进行可视化的样本的idx 3.在launch.json文件中修改input_path为.mp4文件对应的路径(注意：每次只能得到一个动作的可视化结果，若要得到多个，需对代码进行修改)**~~

**11.1**  

1.对于segment = 10的情况进行消融、frozen和微调实验(其中，微调实验的lr改为0.01lr)：  

frozen(其中预训练的模型是segment = 20):  

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/4930a6c3-22a5-4c02-8b78-9c906db692eb" />  

将segment更改为10之后，识别率显著提高至90%，不会出现部分错误部分正确的情况(对应的地址diffusion motion inbetween(per segment frozen)/save/rxmull21/model000030030.pt)

消融(segement = 10，同时将归一化方法改为sum_normalization，对应的地址：/home/newdisk/hch/diffusion motion inbetween(per segment ablation)/save/ut6m2jsp)：  

识别率35%左右

微调(其中预训练的模型是segment = 10, segment = 10, lr = 0.01lr, 对应的地址：diffusion motion inbetween(per segment finetune)/save/yevckbqf)：  

<img width="846" height="547" alt="image" src="https://github.com/user-attachments/assets/7fd5ec09-bef4-48b6-83bb-2f4e8bab8d56" />

识别率65%，且出现明显的抖动现象，也没有部分正确部分错误的情况

2.对于nsub = 234的情况进行消融、frozen和微调实验，以验证nsub = 30的有效性：  

**11.2**  

**对上述实验做个总结：**  

**实验一(数据集：逐帧对比学习，24个子载波，AoA_DFS处理方式：高通滤波)**  

<img width="617" height="146" alt="image" src="https://github.com/user-attachments/assets/1e1e23f3-0c44-4f58-943d-8ed86afdcaaf" />

finetune、frozen和ablation实验的结果均差不多，识别率均较高  

**实验二(数据集：逐段对比学习(20)，30个子载波，AoA_DFS处理方式：高通滤波+特征空间滤波)**  

<img width="607" height="142" alt="image" src="https://github.com/user-attachments/assets/112960c5-8dfa-4742-a313-955c42ebf2cb" />

上述实验证明消融实验是有效的，但是finetune的frozen的实验结果相一致，没什么变化，识别率均不高，且出现部分正确部分错误的情况；而ablation基本都是错的，虽然不会出现部分错误的情况  

**实验三(数据集：逐段对比学习(10)，30个子载波，AoA_DFS处理方式：高通滤波+特征空间滤波)**  

<img width="614" height="143" alt="image" src="https://github.com/user-attachments/assets/c82f9e7a-1e63-4d8f-afc8-5ffc1c4cfae2" />

识别率显著提升，但是frozen不会出现部分正确的情况  

1.关于输出结果出现nan问题：  
1.1检查encode_image中self.AoA_seqTransEncoder的初始权重：模型权重已经默认初始化  
1.2检查训练之后的encode_image中self.AoA_seqTransEncoder的权重(是否存在nan和inf)：没有nan和inf  
1.3检查是否是数据预处理的问题，比如归一化之后出现数值过大的情况：在测试集中出现了归一化之后数值较大的值，0.3左右，这些值输入网络后输出为nan；训练集也有归一化之后数值较大的值，0.2左右  
1.4将归一化改为标准化：还是有nan  
1.5检查归一化代码是否有误：已验证代码是正确的  
1.6代码本身有梯度裁剪  
1.7将segment由10改回20，看是否存在该问题：将segment改为20，没有出现nan的问题  
1.8有没有可能是segment造成的？尝试微调部分的segment为10：将segment改为10，也没有出现nan的问题  
1.9将归一化改为sum normalization，输出没有nan

</details>

**11.5**  
<details>
<summary>📖 问题记录</summary>  
1.将对比学习的归一化方法改为sum normalize后无法收敛  
  
  1.1尝试修改学习率lr = 1e-3 1e-4 1e-5 1e-6 5e-6 5e-5  
  1.2尝试修改motion的归一化方法为sum normalize  
  1.3尝试增大normalize的值，即对normalize后的值均乘以10 100(收敛) 1000(收敛速度更快)    
  1.4尝试先行sum_normalize，再列sum_normalize  
  1.5尝试先列sum_normalize，再行sum_normalize  
2.扩散模型测试集输出nan的问题：  
  对AoA_Dfs进行编码的Encoder那层输出的值太大，对输出加一层归一化层
</details>  

<details>
<summary>📖 实验记录</summary>   
1.训练nsub = 234，segment = 10的对比学习(已完成)  
  
2.训练nsub = 234，segment = 10的ablation(已完成)(./diffusion motion inbetween(per segment ablation)/save/ksphz7zi/model000030030.pt)
</details>

**11.6**  
<details>
<summary>📖 问题记录</summary>  

</details>  

<details>
<summary>📖 实验记录</summary>   
1.训练nsub = 234，segment = 10的frozen(预训练 epoch = 200, segment = 10)(已完成)(./diffusion motion inbetween(per segment frozen)/save/arwyb1lr/model000030030.pt)  
  
2.训练nsub = 234，segment = 10的finetune(预训练 epoch = 200, segment = 10, lr = 0.01LR)(已完成)(./diffusion motion inbetween(per segment finetune)/save/6om8n8ca/model000030030.pt)  
3.Person in wifi 3d 数据集的处理流程：  
  3.1将原有的数据划分为多人和单人场景(./csi_process/Person-in-wifi-3d/data split.py)  
  3.2将单人场景下的数据进行40帧合并(./csi_process/Person-in-wifi-3d/data prc.py)  
  3.3添加根节点坐标(./csi_process/Person-in-wifi-3d/keypoint_prc.py)  
4.训练nsub = 30，segment = 10的对比学习(已完成)
</details>

**11.7**  
<details>
<summary>📖 问题记录</summary>  

</details>  

<details>
<summary>📖 实验记录</summary>   
1.训练nsub = 30，segment = 20的对比学习(已完成)  

2.训练nsub = 30，segment = 10的frozen(预训练 epoch = 200, segment = 10)(已完成)(./diffusion motion inbetween(per segment frozen)/save/i1k352ew/model000030030.pt)  
3.训练nsub = 30，segment = 10的finetune(预训练 epoch = 200, segment = 10， lr = 0.01LR)(已完成)(./diffusion motion inbetween(per segment finetune)/save/xd51zww7/model000030030.pt)  
4.训练nsub =30，segment = 10的ablation(已完成)(./diffusion motion inbetween(per segment ablation)/save/4z3l1y2y/model000030030.pt)  
5.训练nsub = 234，segment = 20的对比学习(已完成)  
6.训练nsub = 30，segment = 10的frozen(预训练 epoch = 200, segment = 20)(已完成)(./diffusion motion inbetween(per segment frozen)/save/ny78lc5b/model000030030.pt)    


</details>  

**11.8**  
<details>
<summary>📖 问题记录</summary>  

1.Person in wifi 3d关节对应示意图(14)：  

<img width="118" height="358" alt="image" src="https://github.com/user-attachments/assets/62c5629e-2715-486e-91b6-f3bf40358ac6" />

2.Human motion in inbetween关节对应示意图(22)：  

<img width="118" height="329" alt="image" src="https://github.com/user-attachments/assets/9d1d84e9-c6f9-495f-93bd-c636feb44a22" />

3.MMFi关节对应示意图(17)：

<img width="110" height="319" alt="image" src="https://github.com/user-attachments/assets/cb174d25-107a-47e3-8f05-7e4424504641" />

4.对Person in wifi 3d数据集的14个关节点添加根节点，根节点 = 左右髋部的中点  

</details>  

<details>
<summary>📖 实验记录</summary>   
1.训练nsub = 30，segment = 10的finetune(预训练 epoch = 200, segment = 20， lr = 0.01LR)(已完成)(./diffusion motion inbetween(per segment finetune)/save/3nq8hxk2/model000030030.pt)  

2.训练nsub = 30，segment = 20的ablation(已完成)(./diffusion motion inbetween(per segment ablation)/save/dkcry6jt/model000030030.pt)  
3.训练nsub = 234，segment = 10的frozen(预训练 epoch = 200, segment = 20)(已完成)(./diffusion motion inbetween(per segment frozen)/save/o3tvxkgb/model000030030.pt)

</details>

**11.9**  
<details>
<summary>📖 问题记录</summary>  
1.从关节空间坐标转为旋转表示的过程：  

  {Part 1 将不同的人体形状统一到同一标准模板(输入 njoints * 3，输出 njoints * 3)}  
  1.1计算tgt中各关节与其父关节之间的L2范数，并乘以初始偏移量  
  1.2通过逆运动学算法将关节坐标位置转为旋转参数(四元数):通过肩关节等计算得到前向方向向量；计算从前向方向向量旋转为目标方向向量[0,0,1]的四元数；通过预先定义的运动学关节链计算各子关节相对于父关节的局部旋转四元数；最终得到根节点全局旋转四元数和子关节局部旋转四元数  
  1.3通过前向运动学算法将关节的局部旋转表示应用到标准模板上得到标准模板各关节的空间坐标表示  
  {Part 2 处理数据同时通过多个角度对mesh进行描述}  
  1.1将mesh最低点移动到地面、根关节的平移到XZ平面的原点(0,0)  
  1.2获取足部接触状态(foot关节)(4)  
  1.3获取除根关节的6D旋转表示((njoits - 1) * 6)(通过预设定的模板得到旋转表示)  
  1.4获取根关节绕y轴的旋转速度和XZ平面的线速度(1 + 2)  
  1.5获取根关节的y轴坐标(1)    
  1.6获取除根关节的空间坐标((njoints - 1) * 3)  
  1.7获取所有关节的速度(njoints * 3)  
  {Part 3 数据构成}  
  根关节绕y轴旋转速度+根关节XZ平面线速度+根关节高度+关节空间坐标+关节旋转表示+关节线速度+足部接触状态
</details>  

<details>
<summary>📖 实验记录</summary>   
1.训练nsub = 234，segment = 10的finetune(预训练 epoch = 200, segment = 20, lr = 0.01LR)(已完成)(./diffusion motion inbetween(per segment finetune)/save/q4mcb67m/model000030030.pt)    

2.训练nsub = 30，segment = 20的frozen(预训练 epoch = 200，segment = 20)(已完成)(./diffusion motion inbetween(per segment frozen)/save/wo15n85g/model000030030.pt)  
3.训练nsub = 30，segment = 20的finetune(预训练 epoch = 200，segment = 20, lr = 0.01LR)(已完成)(./diffusion motion inbetween(per segment finetune)/save/cl3xmh0s/model000030030.pt)
</details>  


**11.11**  
<details>
<summary>📖 问题记录</summary>  

</details>  

<details>
<summary>📖 实验记录</summary>   
1.训练Person in wifi数据集的ablation(测试，尝试跑通)(已完成)     

</details>  

**11.12**  
<details>
<summary>📖 问题记录</summary>  
1.Person in wifi数据集可以跑通，但是可视化结果很差： 

  https://github.com/user-attachments/assets/67dd51ae-0646-45be-8566-32151b01eb51

  1.1先不考虑Output Motion，只考虑Ground Truth  
  1.2修改"edit.py/第150行代码"  
  1.3对原有的joints的空间坐标进行可视化发现其对应的关节和Person in wifi 3d代码中定义的不一致(可视化对应代码：./diffusion motion inbetween(person in wifi dataset per segment ablation)/visual test.py)：  

  <img width="118" height="327" alt="image" src="https://github.com/user-attachments/assets/7f2bf2f5-d9a7-4519-8fe9-3b5b77be9c0a" />  

  1.4可视化结果正常，但是空间坐标轴还需要进行调整
  
  https://github.com/user-attachments/assets/31224f1b-2209-4904-bc75-3036c4f2fb1a

  1.5交换关节在xyz空间坐标中的yz两个维度(./csi_process/Person-in-wifi-3d/dim swap.py)，同时对y坐标添加负号，最终得到正确的骨架可视化结果：

  https://github.com/user-attachments/assets/48399d89-fbc1-4cc0-9fef-7d6403e50549

  1.6对MMFi数据集的关节进行空间坐标可视化，发现其与之前的17个关节的定义不一致，需要修正：

  <img width="118" height="335" alt="image" src="https://github.com/user-attachments/assets/09ed6d2d-a476-4476-aa52-d5f8522c64c5" />


</details>  

<details>
<summary>📖 实验记录</summary>        

</details>  

**11.13**  
<details>
<summary>📖 问题记录</summary>  
  
1.对于MMFi数据集中的CSI相位不理想的问题，可能原因：噪声太大或者硬件导致的频偏不是稳定的  
  
  <img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/89356425-6f12-45be-afc5-0a54bbdb248f" />

虽然CSI相位去噪结果不理想，但是最后得到的AoA_DFS结果还是理想的：(去噪前)——>(去噪后)  

<img width="800" height="300" alt="image" src="https://github.com/user-attachments/assets/34bc0734-01fe-48a2-8eb6-acffb4f8da89" />  

<img width="800" height="300" alt="image" src="https://github.com/user-attachments/assets/9498383a-8996-485b-95fa-01fc514e0f07" />


</details>  

<details>
<summary>📖 实验记录</summary>   

</details>  

**11.15**  
<details>
<summary>📖 问题记录</summary>  
  
  1.MMFi数据集预处理：  
    1.1帧数划分、异常值检测("./csi_process/MMFi/data prc.py")  
    1.2删除鼻子的关节坐标("./csi_process/MMFi/keypoint_prc.py")  
    1.3对训练、测试和验证数据集进行划分("./csi_process/MMFi/train val test.py")
</details>  

<details>
<summary>📖 实验记录</summary>   
  
  1.训练MMFi数据集在MetaFi模型(已完成)  
  2.训练Person in wifi数据集在MetaFi模型(已完成)
</details>  

**11.17**  
<details>
<summary>📖 问题记录</summary>  

</details>  

<details>
<summary>📖 实验记录</summary>   
  
  1.训练Person in wifi数据集在Person in wifi模型(**注意：需要对Person in wifi数据集中的val.txt和test.txt进行交换**)  
  2.训练Person in wifi数据集的对比学习(已完成)  
  3.训练Person in wifi数据集在Ours模型(已完成)，得到的骨架相比于真实值存在旋转问题，导致MPJPE过大：  

https://github.com/user-attachments/assets/c550a7ff-2189-4cb4-830b-c12d8c7adcca


</details>  

**11.18**  
<details>
<summary>📖 问题记录</summary>  
  
  1.Person in wifi数据集在Ours模型中产生的旋转问题：  
  1.1是否是因为root绕y轴的旋转速度与实际偏差过大导致的，尝试将真实的root绕y轴的旋转速度代替预测的，看是否还存在旋转问题(修改之后确实没有旋转问题)  
  
https://github.com/user-attachments/assets/7be4ad9b-2863-4475-aa31-6661818ab155

  重新对其MPJPE进行评估，相较于之间的有显著下降，但是还是不如Person in wifi的结果：  
  
  <img width="464" height="16" alt="image" src="https://github.com/user-attachments/assets/eb3aa5fa-52b3-43cc-be2f-9015f33108f3" />

  可能原因，生成的motion和真实的motion之间存在明显的时延，因此对生成的AoA_DFS进行时延补偿，并重新进行实验
  
</details>  

<details>
<summary>📖 实验记录</summary>   
  
    1.训练Person in wifi数据集的对比学习(增加时延补偿)(已完成)   
    2.训练Person in wifi数据集在Ours模型(已完成)  
    3.训练MMFi数据集在Person in wifi模型(已完成)  
  
</details>  

**11.19**  
<details>
<summary>📖 问题记录</summary>  
  
    1.Person in wifi数据集在Ours模型中会产生旋转问题，导致误差上升，而且即使使用GT的root，性能也不如Person in wifi模型   
    (可能原因：关节代码部分处理有问题、使用multi-head造成、数据集问题)
    2.Person in wifi数据集在MetaFi模型中的误差过大(是否跟接收机的选取有关)  
    3.MMFi数据集在Person in wifi数据集中的训练时间过长(只能截取部分关键帧，例如120帧截取其中的12帧)  
</details>  

<details>
<summary>📖 实验记录</summary>
  
    1.重新对MetaFi模型进行修正，取消Resize操作，并增加一项池化操作，重新训练Person in wifi数据集(已完成，修改后的效果反而更差，主要是loss基本不怎么收敛)  
    2.尝试不使用multi-head，对Person in wifi数据集在Ours模型重新进行训练(已完成，旋转问题仍然存在，且相较于multi-head，性能反而下降)  
    3.尝试使用CSI相位代替CSI幅值，对MetaFi模型进行修正，重新训练Person in wifis数据集(已完成，效果与CSI幅值基本没有区别，loss还是不收敛)  
    4.尝试使用Person in wifi数据集的第二和第三个接收机的数据在MetaFi模型上重新进行训练(已完成，loss还是不收敛)  
    
</details>  

**11.20**  
<details>
<summary>📖 问题记录</summary>  
  
</details>  

<details>
<summary>📖 实验记录</summary>
  
    1.只对MMFi数据集中的关键帧进行截取，重新训练Person in wifi模型  
    2.将对比学习和扩散模型的Segment均修改为10，重新训练Person in wifi数据集在Ours模型(已完成，还是存在旋转问题)  
    3.尝试减小每个segment中的帧数，改为(5和3)，重新训练Person in wifi数据集在Ours模型(已完成，还是存在旋转问题，且使用真实根关节的绕y轴旋转速度，得到的MPJPE = 147(single rx)，若完全使用真实的根关节，即前4个均使用真实值，则精度可以提升至119)
    
</details>  

**11.21**  
<details>
<summary>📖 问题记录</summary>  
  
</details>  

<details>
<summary>📖 实验记录</summary>
  
    1.将上述的Person in wifi数据集在Ours模型的single rx改回multi-rx，同时root使用真实值，测量其MPJPE是否能够超过Person in wifi模型(已完成，得到的MPJPE为81.974，优于Person in wifi模型)  
    
    
</details>  








