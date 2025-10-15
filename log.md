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
2.对Motion_inbetween模型中的emb和x的结合方式进行修改，改为concatenate或者自注意力机制







