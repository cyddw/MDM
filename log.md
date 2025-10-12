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
2.Motion_CLIP(batch_size = 30，顺序采样，epoch = 30，训练时间 = 20h)测试效果：从loss曲线可知，仍未收敛还可以继续训练  

<img width="846" height="547" alt="7a1866f9-5589-4b44-96fc-4caadcb72872" src="https://github.com/user-attachments/assets/40c7e31f-8002-4ba6-8301-e5e8096c2d71" />  

训练集batch_size = 30效果：大部分能达到0.9以上，但由于没收敛，所以效果会差一点  
测试集batch_size = 30效果：没有丝毫效果  
将上述Motion_CLIP作为Motion_inbetween的encoder，epoch = 800
