**12.6**
- [x] 1.完成模型的基本框架确定以及Introduction and Motivation的完成

**12.7**
- [x] 1.完成Introduction部分的撰写

**12.8**
- [x] 1.完成数字信号处理PPT制作

**12.9**
- [x] 1.确定模型需要进一步修改的部分

**12.10**
- [x] 1.完成physic guidance部分的代码

**12.11**
- [x] 1.实现feature proxy部分的代码

**12.13**
- [x] 1.完成related work第一部分的内容撰写(仅为内容和逻辑)

**12.14**
- [x] 1.完成related work第二部分的内容撰写(仅为内容和逻辑)
- [x] 2.完成related work第三部分的内容撰写(仅为内容和逻辑)

**12.16**
- [x] 1.做一个简单的验证实验：在MMFi数据集上，将对比学习的Human mesh换成使用motion representation转换后的mesh，看性能是否提升

**12.17**
- [ ] 1.做一个简单的验证实验：在Person in WiFi数据集上，将对比学习的Human mesh换成使用motion representation转换后的mesh，看性能是否提升
- [x] 2.搜索相关的contrastive learning的工作，如何对齐两个带噪模态的特征

**12.18**
- [x] 1.确定整个模型需要修改的部分

**12.20**
- [x] 1.对进度整理5的ppt进行制作，同时梳理整个模型的架构，推敲是否合理

**12.21**
- [x] 1.对两篇论文的代码进行查看，确定每个架构的细节

**12.22**
- [x] 1.确定模型的loss

**12.23**
- [x] 1.弄懂DMVAE的代码部分
- [x] 2.尝试跑通MLD的VAE部分

**12.24**
- [x] 1.完成进度整理部分的problem formulation

**12.25**
- [x] 1.完成multi-VAE部分的代码

**12.27**
- [x] 1.完成数据集部分的代码
- [x] 2.完成数据集部分和multi-VAE部分的桥接
- [x] 3.完成loss部分的代码
- [x] 4.调试、检查整体multi-VAE部分的代码，同时标出需要进行调试和可能需要修改的部分

**12.28**
- [x] 1.完成进度整理5ppt

**12.30**
- [x] 1.调试DMVAE代码，确保重构出来的human motion尽可能精确
- [x] 2.将DMVAE改为使用rotation表示进行重构

**12.31**
- [ ] 1.重新修改MMFi数据集的3D旋转rotation表示的代码(8、10、12有问题)

**1.4**
- [x] 1.(stage1)：只使用Motion Encoder和Motion Decoder重构human motion
- [x] 2.(stage2): 在stage1的基础上对输入的motion进行加噪，能够重构出clean的human motion
- [ ] 3.(stage3): 在stage2的基础上引入CSI数据，能够提高重构的性能

**1.13**
- [x] 1.完成latent diffusion model部分的代码修改

**1.18**
- [x] 1.修改MLD中读取的human motion为前110帧

**3.5**
- [x] 1.完成baseline：GraphPoseFi的代码编写

**3.7**
- [x] 1.完成PA-MPJPE和MPJPE对于所有joints的评估

**3.8**
- [x] 1.不对latent z进行加噪，对输入数据进行加噪，加噪的强度为0,0.1,0.2,0.3,0.4,0.5，同时在latent motion diffusion上进行测试(这里AE一阶段和二阶段的epoch均为100)

**3.9**
- [ ] 1.对latent z进行加噪，加噪的强度为0,0.05,0.1,0.15,0.2,0.25，这里的stage1的epoch为100，加噪强度为0.1

**3.10**
- [x] 1.将模型输出的3D keypoint转为3D human mesh

**3.12-3.15**
- [x] 1.补做三个实验：latent z加噪的实验，去掉Motion Encoder和Decoder的实验，增加一个HPE-Li作为baseline
- [x] 2.完成实验部分的内容

**3.16-3.19**
- [x] 1.完成Problem Formulation和System Model部分的内容

**3.19-3.22**
- [x] 1.完成剩余所有部分的内容
