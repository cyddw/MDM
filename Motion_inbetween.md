- [x] 1.完成Motion_inbetween整体实验，并得到实验结果(ddl🕥 25.10.19)

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
1.为什么直射路径在AoA_DFS频谱上会产生这么大的频谱偏移，不应该是在0频附近吗(可能原因：1.对CSI进行线性插值导致的；2.对CSI进行共轭相乘导致的)  
2.为什么对CSI进行unwrapped的时候，在第5个CSI部分会产生跳跃  
**3.如何对8个receiver进行整合？**  
**4.考虑一个比较好的场景**  
**5.Diffusion部分能否再优化一下，比如t直接加是不是有点不太好**  
**6.创新点落在优化z的生成那一块**  
**7.数据集是否可以优化一下，只有6个动作显然是偏少的(参考文献：https://aiotgroup.github.io/Person-in-WiFi-3D/ 和 https://ntu-aiot-lab.github.io/mm-fi?utm_source=chatgpt.com)**  
**8.参考Mobisys的相关论文(12月份左右，侧重实验)**







