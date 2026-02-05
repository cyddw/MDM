**12.10**  
<details>
<summary>📖 问题记录</summary>  

    1.尝试直接使用对比学习预训练好的模型作为proxy model，也就是在采样的每一步将预测的旋转表示转为mesh输入至Encoder，对AoADFS也输入至Encoder，然后计算两者之间的infoNCE，结果显示计算出的梯度值非常小，对最终结果没有影响
    2.在1的基础上，尝试将infoNCE改为L2范数的平方(虽然梯度值较大，但是对结果也基本没有影响)
    3.对比学习终究是将两个模态在共有的潜在空间进行对齐，将其直接作为proxy好像不太合适，因此一个最简单的方法就是直接设计一个映射网络，将mesh映射至AoADFS空间，映射网络可以直接使用对比学习的Encoder
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details>  

**12.11**  
<details>
<summary>📖 问题记录</summary>  

    1.在原有的对比学习的基础上，将对比学习网络修改为映射网络，先通过一个Transformer编码mesh，然后将编码后的结果以及Rx的embedding输入decoder得到预测的不同Rx的AoADFS，然后计算两者的MSEloss，值得注意的是该映射网络的loss下降非常快
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details>  

**12.12**  
<details>
<summary>📖 问题记录</summary>  

    1.使用上述的映射网络进行eval，MPJPE = 185，性能发生显著下降
    2.可能原因：通过recover_from_ric得到的Mesh和原来的Mesh是否一致；梯度是否应该增加超参数；梯度是否应该作用于score上面；物理模型是否有用
    3.将梯度作用在score上面
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details> 

**12.13**  
<details>
<summary>📖 问题记录</summary>  

    1.将梯度作用于score上面，当scale = 1000时，输出nan；当scale = 100时，输出MPJPE = 106，性能反而下降(这里代码有点问题，reconstruction guidance被覆盖了)
    2.对上述代码进行修改之后，即reconstruction和proxy guidance同时起作用后，MPJPE = 97.588(scale = 100); MPJPE = 97.668(scale = 200); MPJPE = 98.795(scale = 400)
    3.上述实验说明physic guidance基本没有什么作用，检查通过recover_from_ric得到的Mesh和原来的Mesh是否一致：不一致 (为什么Person in wifi Dataset以及MMFi Dataset在Ours模型上表现不佳，主要原因为从旋转表示重新转为3D空间坐标表示和最原始的3D空间坐标表示是不一致的)
    4.对Feature proxy部分进行修正之后，MPJPE = 98.730(scale = 400)(修正前后并没有什么区别)
    5.在上述基础上，在梯度前面增加超参数alpha_t，其随t的增大逐渐递减，MPJPE = 97.649(scale = 400)
    6.将原本的MSE loss和scale用L2范数的平方进行更换，但是保留Feature proxy训练部分的MSE loss
    7.直接使用L2范数的平方会导致loss过大，求梯度时出现nan，因此将loss乘以scale进行约束(scale = 0.1)，还是出现nan; (scale = 0.01)，还是出现nan; 没有出现nan，但是MPJPE基本不变(scale = 0.001); 在physic_grad前面增加一项参数(w_r * sqrt_alpha_bar  / 2)，性能下降(MPJPE = 98.216)
    8.尝试对比没有reconstruction guidance以及physic guidance 和 没有reconstruction guidance但是有physic guidance的情况，来探究physic guidance是否对模型性能有提升(有physic guidance: MPJPE = 102.596，没有physic guidance：MPJPE = 101.711)
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details> 

**12.14**  
<details>
<summary>📖 问题记录</summary>  

    1.根据上述实验可知，是否有physic guidance对整个模型性能基本没有影响
    2.在没有reconstruction guidance的情况下，修改scale = 0.005，MPJPE = 106.920；若physic_grad前面没有参数，scale = 0.005，MPJPE = 103.502
    3.在没有reconstruction guidance的情况下，修改scale = 0.005，physic_grad前面没有参数，并将原本作用在ε上的改为作用在x_t上，MPJPE = 101.629; physic_grad前面增加一项alpha_t(随着去噪步的增加，逐渐增大)，MPJPE = 101.629；修改scale = 0.001，MPJPE = 101.628；physic_grad只在最后50步起作用，MPJPE = 101.639；physic_grad只在最后20步起作用，MPJPE = 101.624
    4.将A(x0)中的输入由x0改为xt，MPJPE = 101.623；physic_grad在所有去噪步都起作用，MPJPE = 101.629；physic_grad前面没有参数，MPJPE = 101.630
    5.在代码最后增加一行x = z_guided，MPJPE = 109.931(这里的x会影响最后输出的mean)；physic_grad前面增加alpha_t参数，MPJPE = 103.910；将A(x0)中的输入由xt改为x0，MPJPE = 101.387；只在最后50步起作用，MPJPE = 100.999；只在最后20步起作用，MPJPE = 99.136
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details> 

**12.15**  
<details>
<summary>📖 问题记录</summary>  

    1.scale = 0.005，MPJPE = 100.339；scale = 0.005，在最后50步起作用，MPJPE = 101.503；scale = 0.001，在最后10步起作用，MPJPE = 101.956；
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details>


**12.15**  
<details>
<summary>📖 问题记录</summary>  


</details>  

<details>
<summary>📖 实验记录</summary>

        1.重新跑一遍MMFi数据集在Our模型上
</details>

**12.30**  
<details>
<summary>📖 问题记录</summary>  
    
        1.可能需要修改的点：
        1.1使用的AoADFS数据是降采样的
        1.2没有对AoADFS数据进行归一化
        1.3没有对motion数据进行归一化
        1.4motion没有使用旋转表示
        1.5没有对输入数据的时序进行切分
        1.6loss的系数以及loss本身设置
        2.DMVAE初步重构的结果：

![sample_29](https://github.com/user-attachments/assets/820c5d9c-bead-4cf5-b388-96d6515a429e)

        两个问题：问题一：loss很快就不下降了；问题二：重构出的都是静止不动的


</details>  

<details>
<summary>📖 实验记录</summary>

        1.切断从Loss_rec到private的梯度流(和之前的结果一样)
        2.不对两个模态的share进行KL约束，只对两个模态共有的结果进行KL约束(和之前的结果一样)
        3.不考虑分离private和shared，只对其进行重构(即alpha和lamb系数置零)(还是静止不动，loss_rec还是降不下来)
</details>

**12.31**  
<details>
<summary>📖 问题记录</summary>  
    
        1.对于rotation表示，有几个问题：
        1.1rotation表示中使用的是root对y轴的积分，导致直接可视化出来的结果是旋转的
        1.2root对y轴的积分有利于模型去学习，相对于微分而言


</details>  

<details>
<summary>📖 实验记录</summary>

        1.不使用root绕y轴的积分，能否重构出较好的结果(训练过程中，loss突然增大，且可视化出来的human motion都是旋转的，没有具体动作)
</details>

**1.5**  
<details>
<summary>📖 问题记录</summary>  
    



</details>  

<details>
<summary>📖 实验记录</summary>

        实验一(探究模型参数和loss设置对重建效果的影响)
        1.只使用Human motion Encoder和Decoder进行VAE(可调节参数：lr = 1e-4(MLD),lambda_kl = 1e-5)(已完成，MPJPE收敛于300左右，同时可视化的motion是静止且旋转的)
        2.只使用Human motion Encoder和Decoder进行VAE(将lr改为1e-4)(已完成，MPJPE收敛于200左右，可视化的motion部分旋转，但基本都是静止的)
        3.只使用Human motion Encoder和Decoder进行VAE(将lr改为1e-4，lambda_kl改为1e-5)(已完成，MPJPE收敛于170左右，可视化的motion基本和GT一致，但是还是存在旋转和偏移的问题)
</details>

**1.6**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        1.只使用Human motion Encoder和Decoder进行VAE(将lr改为1e-4，lambda_kl改为1e-5，增加一项joints约束)(已完成，MPJPE收敛于80左右，可视化的motion基本和GT一致，但是想较于GT，生成的不是特别的光滑)：
![sample_58](https://github.com/user-attachments/assets/9217fd37-235b-493f-a43f-70239555ac26)
![sample_0](https://github.com/user-attachments/assets/586881d9-4fcf-47ba-bc9c-ce43bfd3cbf9)

        实验二(探究数据集设置对重建效果的影响)
        可视化Person in WiFi数据集以及MMFi数据集，在motion_representation中，若存在new_data[:, 0] = rot_ang，new_data[:, [1, 2]] = r_pos[:, [0,2]]这两项，则可视化出来的human motion是旋转的
        1.保留实验一的设置，仅对数据集进行更换，即使用旋转的MMFi数据集(已完成，MPjPE收敛于200左右，可视化的motion和GT有较大差距)：
![sample_0](https://github.com/user-attachments/assets/34091b13-794d-4293-b897-fe3da7b63540)


</details>


**1.7**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        关于rotation表示重新转为abs表示出现旋转问题的总计：(1)在motion_representation中，new_data[:, 0] = rot_ang，new_data[:, [1, 2]] = r_pos[:, [0,2]]这两行代码已经保证了旋转表示是绝对旋转 (2)在recover from ric中，若args.abs_3d=False，则认为旋转表示是相对旋转，则会使用累加形式；若args.abs_3d=True，则认为旋转表示是绝对旋转，则不对其进行累加
        1.在上个实验的基础上，仅将recover from ric中的abs_3d改为True(已完成，MPJPE收敛于40左右，且收敛速度很快，起始loss就很小，可视化结果和GT基本一致)：
![sample_58](https://github.com/user-attachments/assets/90b6e496-ad04-4bcc-b9a0-264a1c23dac6)
![sample_0](https://github.com/user-attachments/assets/17d0bdcb-d2de-41ca-9f0e-740900fb1dbc)
</details>

**1.8**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        实验三(重新探究loss设置对实验效果的影响)
        1.在上述实验的基础上，取消loss中的joint约束(已完成，MPJPE收敛于70左右，相较于有joint约束抖动和平移明显)：
![sample_58](https://github.com/user-attachments/assets/969202d6-a2ff-4551-88ff-c0c04abfe3be)
![sample_44](https://github.com/user-attachments/assets/80d50e97-158f-476e-b93b-1c505d1f7c88)

        2.将数据集中的rotation表示换成3D表示，同时loss只有3D joints约束(已完成，MPJPE收敛于30左右，去掉旋转表示反而效果更好，但是还是有抖动(可以考虑添加两帧之间的速度和加速度loss))：
![sample_0](https://github.com/user-attachments/assets/22502763-db60-4c89-9165-9a6e4dc12f3c)
![sample_58](https://github.com/user-attachments/assets/b8ce2d02-ef08-4e6c-a38a-aa37d1363e0c)

        实验四(探究模型在测试集上的泛化性能)
        1.保存上个实验的训练模型，在测试集上进行测试(已完成，MPJPE收敛于50左右)：
![sample_0](https://github.com/user-attachments/assets/3c773510-43d0-4bfb-8858-782003aa0298)
![sample_58](https://github.com/user-attachments/assets/53c4cc16-19dc-4bee-a937-acc286c29e91)


</details>

**1.9**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        1.在训练的时候保存训练的模型并在测试集上测试，以找到最优的训练轮次，避免过拟合(已完成，在epoch = 1700左右，MPJPE最小，收敛于45)

        实验五(探究对human motion进行加噪对性能的提升)
        加噪的种类：白噪声、时间平滑噪声、关节权重噪声、Depth-aware噪声；加噪的方式：整体全部加噪、关节部分加噪、时间帧部分加噪，部分sample加噪
        1.对human motion添加白噪声(整体加噪+sigma = 0.01)(已完成，训练集和测试集上基本没什么特别大的变化，测试集MPJPE最小收敛于43左右，训练集MPJPE收敛于30)，带噪的human motion：
![sample_5](https://github.com/user-attachments/assets/d8f5366f-794b-4a06-98cb-c1fc8ecafde0)

        测试集上重构的结果：
![sample_5](https://github.com/user-attachments/assets/87a921db-7445-409c-8cee-8391887860bb)
![sample_44](https://github.com/user-attachments/assets/eaf3a022-c67a-4e22-a876-ef7bb3ea54b8)
        
</details>

**1.10**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        1.对human motion添加白噪声(整体加噪+sigma = 0.02)(已完成，训练集和测试集上基本没什么特别大的变化，测试集MPJPE最小收敛于45左右，训练集MPJPE收敛于30)，带噪的human motion：
![sample_44](https://github.com/user-attachments/assets/0e19f178-5a3f-4de6-9a58-e582ccc1f588)

        测试集上重构的结果：
![sample_5](https://github.com/user-attachments/assets/91128e3a-793f-4bca-976f-ddccfa754b8a)
![sample_44](https://github.com/user-attachments/assets/4d47afbc-1094-48ea-92c2-5f7a0ae088df)


        
</details>

**1.10**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        1.对human motion添加白噪声(整体加噪+sigma = 0.005)(已完成，和sigma = 0.01基本没有区别)
</details>

**1.13**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        实验六(尝试通过对输入进行加噪的方式解决输出抖动的问题)：
        1.对human motion添加低频白噪声(白噪声通过低通)(sigma = 0.01)(已完成，还是会出现抖动的现象，且测试集上和训练集上的MPJPE和sigma = 0.01的白噪声一致)带噪的human motion:
![sample_58](https://github.com/user-attachments/assets/c802fed4-bd5f-4983-b56f-fa82deacac3a)

        测试集上的重构结果：
![sample_44](https://github.com/user-attachments/assets/4c4b8ee6-a921-411a-a551-cd51dcb03f15)
![sample_52](https://github.com/user-attachments/assets/3592c9d4-a11f-4bf0-bcaf-e9c561bf7a49)
        
</details>

**1.14**  
<details>
<summary>📖 问题记录</summary>  
    


</details>  

<details>
<summary>📖 实验记录</summary>

        1.不对human motion添加噪声，增加一项加速度loss项(已完成，测试集MPJPE最小收敛于42左右，训练集MPJPE收敛于30左右，抖动相较于之前有改善)：

![sample_5](https://github.com/user-attachments/assets/2e3d0bb3-1ace-4fd9-bb8c-788197fc4f3c)
![sample_58](https://github.com/user-attachments/assets/9a0e64a8-e4a1-4b36-818b-e5b88ee60f54)

        
</details>

**1.15**  
<details>
<summary>📖 问题记录</summary>  
    
</details>  

<details>
<summary>📖 实验记录</summary>

        1.在添加加速度loss项的基础上，对输入的human motion添加噪声(低频白噪声，sigma = 0.01)(已完成，测试集MPJPE最小收敛于42左右，训练集MPJPE收敛于30左右，可视化结果和上个实验基本没有区别)：
        测试集重构结果：
![sample_0](https://github.com/user-attachments/assets/6153b16d-c726-46f4-b02c-deeb54d23302)
![sample_58](https://github.com/user-attachments/assets/0e071912-518d-4ef0-a38c-cd4cec984a64)

</details>

**1.16**  
<details>
<summary>📖 问题记录</summary>  
    
</details>  

<details>
<summary>📖 实验记录</summary>

        实验七(扩散模型+VAE联合测试，其中spectral encoder不参与预训练，预期结果：MPJPE=150左右)
        1.在上个实验的基础上，在扩散模型中进行重构，测试集的重构结果：MPJPE =  282.8717(2000epoch)，主要是动作识别率低
![sample_8](https://github.com/user-attachments/assets/d20ee95d-6a05-4299-a4c1-7bd29809235c)
![sample_29](https://github.com/user-attachments/assets/871ef419-bc0e-4f45-9c9a-3e268e656fe1)


</details>

**1.19**  
<details>
<summary>📖 问题记录</summary>  
    
</details>  

<details>
<summary>📖 实验记录</summary>

        1.在添加加速度loss项的基础上，对输入的human motion添加噪声(低频白噪声，sigma = 0.01)，同时将输入的human motion进行Z-normalization，其中mean和std通过训练集进行计算(已完成，和没有归一化的结果没什么区别，训练集MPJPE收敛于30，测试集MPJPE最小收敛于45)，这里有一个点需要注意一下，加噪的噪声实在标准化之后添加的：
        加噪的结果可视化：
![sample_58](https://github.com/user-attachments/assets/13fce46e-6476-4b60-96b0-263f757bbd94)
![sample_52](https://github.com/user-attachments/assets/8093fd90-9357-42ce-99cb-79311facad2f)

        测试集重构结果可视化：
![sample_52](https://github.com/user-attachments/assets/a2cfe664-cb9e-4bce-bf8a-c1e64cb12839)
![sample_58](https://github.com/user-attachments/assets/cfc446e4-fe01-46c1-9dfc-e30fd4d39870)

        2.在上个实验的基础上，在扩散模型中进行重构(已完成，测试集的重构结果：MPJPE = 304.8604(2000epoch))，重构结果可视化：
![sample_8](https://github.com/user-attachments/assets/a638b146-2ca2-40e2-b02b-4045c4d043af)
![sample_15](https://github.com/user-attachments/assets/1eee0888-e750-4b01-a999-465b9c8895bd)


        3.对于VAE模型，在添加加速度loss项的基础上，不对输入的human motion进行加噪，同时将输入的human motion进行Z-normalization，其中mean和std通过训练集进行计算(已完成，训练集MPJPE收敛于35左右，测试集MPJPE最小收敛于43)，测试集重构结果可视化：
![sample_52](https://github.com/user-attachments/assets/539c16c9-9000-4e74-a387-f71ece072820)
![sample_58](https://github.com/user-attachments/assets/8b9648e6-1e55-4317-afe4-6d51cbc919e7)

        4.对于VAE模型，将Spectral Encoder加入模型进行训练(已完成，训练集MPJPE收敛于28左右(3000个epoch)，测试集MPJPE最小收敛于43)，测试集结果可视化：
![sample_52](https://github.com/user-attachments/assets/33857cd3-446e-4b1e-89fb-9cbf870d0fc0)
![sample_5](https://github.com/user-attachments/assets/371d8bab-dad4-4be2-a3d3-d64f4e8af845)

</details>

**1.20**  
<details>
<summary>📖 问题记录</summary>  
    
</details>  

<details>
<summary>📖 实验记录</summary>

        1.在上个实验的基础上，在扩散模型中进行重构，spectral_encoder_cond不进行预训练(已完成，测试集的MPJPE = 289.2039(2000epoch))，测试集结果：
![sample_8](https://github.com/user-attachments/assets/204b4d1b-569c-4646-bb66-1d2a53fbb8fc)
![sample_29](https://github.com/user-attachments/assets/a66dd39a-aff5-4b24-92cf-107a5d42228f)

        2.在扩散模型中进行重构，spectral_encoder_cond进行预训练并冻结(已完成，测试集的MPJPE = 318.8988(2000epoch))，测试集结果：
![sample_8](https://github.com/user-attachments/assets/860afbff-9d6c-47ce-80e6-9208e5ad932e)
![sample_29](https://github.com/user-attachments/assets/eee84ac6-19c9-4dc3-be8e-bc3599bdc84e)

        实验八(整体模型的优化)：
        1.对于已经训练好的Multimodal VAE，只使用Spectral部分进行human motion的重构(已完成，MPJPE = 298.658)，重构结果：
![sample_52](https://github.com/user-attachments/assets/63f2df31-e67f-410b-acb2-6782fb3e20ff)
![sample_58](https://github.com/user-attachments/assets/8073945c-7802-457d-80e3-ac70ab7ca84e)


        2.对于已经训练好的Multimodal VAE，只使用Human motion部分进行human motion的重构(已完成，MPJPE = 49.237)，重构结果：
![sample_52](https://github.com/user-attachments/assets/3451a1d2-24ab-4de1-8212-0a32cc8c0216)
![sample_58](https://github.com/user-attachments/assets/1e490742-7ed1-4458-b32b-8444984055cd)

        3.对于Multimodal VAE，loss部分增加一项对齐损失函数(beta = 0.1)(已完成，训练集MPJPE降不下去，收敛于263.016，测试集full MPJPE = 225.654)，重构结果：
![sample_52](https://github.com/user-attachments/assets/ff88e013-3c5d-417d-b4f5-3d5940e1e1a4)
![sample_58](https://github.com/user-attachments/assets/30ba8920-f2d6-4148-9b4e-0de39bc036ed)


        4.对于Multimodal VAE，增加only human motion reconstruction loss以及only spectral reconstruction loss，取消了加速度loss项(已完成，训练集full MPJPE收敛于30，测试集full MPJPE收敛于72，测试集motion MPJPE收敛于45，测试集spectral MPJPE收敛于250)
        测试集full重构结果：
![sample_58](https://github.com/user-attachments/assets/8836bb5f-7462-4300-af06-e6f4774a8781)
![sample_52](https://github.com/user-attachments/assets/f4fd1a0c-dd0d-428d-93d1-56669eec2fbb)

        测试集motion重构结果：
![sample_58](https://github.com/user-attachments/assets/ca5c07ef-fc66-42ff-a2d2-880665492fc7)
![sample_52](https://github.com/user-attachments/assets/c856ef9f-fdba-4bcf-978f-245878af44ee)

        测试集spectral重构结果：
![sample_58](https://github.com/user-attachments/assets/89418305-4b67-4134-93b6-85090042ee02)
![sample_52](https://github.com/user-attachments/assets/b094c8da-1894-43ca-949f-38f8accb3f3e)

        5.在上个实验的基础上，增加对齐项损失函数(beta = 0.1)(已完成，训练集MPJPE收敛于100，测试集full MPJPE收敛于186.711，测试集motion MPJPE收敛于123.610，测试集spectral MPJPE收敛于254.605)
        测试集full重构结果：
![sample_58](https://github.com/user-attachments/assets/c5409c78-8b2a-4d20-9cdd-d17dc033af6c)
![sample_52](https://github.com/user-attachments/assets/ee397cd5-4446-4d08-90a4-805562969af7)

        测试集motion重构结果：
![sample_52](https://github.com/user-attachments/assets/00193400-37b7-41dc-b191-78ccd68bedbd)
![sample_58](https://github.com/user-attachments/assets/45395afc-e730-48e8-846c-821a6df8ef97)

        测试集spectral重构结果：
![sample_52](https://github.com/user-attachments/assets/84b762e4-5b29-440c-bcdf-6695cb63e000)
![sample_58](https://github.com/user-attachments/assets/8fc01659-e85a-47ba-8ddc-734997020259)

</details>

**1.21**  
<details>
<summary>📖 问题记录</summary>  
    
</details>  

<details>
<summary>📖 实验记录</summary>

        1.(DMVAE2)对于Multimodal VAE，只重构PoE，loss部分增加一项对齐损失函数(beta = 0.001)(已完成，训练集MPJPE收敛于42左右，测试集full MPJPE收敛于134，motion MPJPE收敛于100，spectral MPJPE收敛于295)：
        测试集full重构结果：
![sample_58](https://github.com/user-attachments/assets/0591547e-a8be-4674-b422-06a291558625)
![sample_52](https://github.com/user-attachments/assets/8ec14904-8625-4ba8-a072-151d9deff25c)

        测试集motion重构结果：
![sample_58](https://github.com/user-attachments/assets/2fbe0d0c-5638-4e8e-a7d0-2c6f806a4951)
![sample_52](https://github.com/user-attachments/assets/1e917d1f-c856-48a9-89ff-fffec6aae511)

        测试集spectral重构结果：
![sample_58](https://github.com/user-attachments/assets/6c070644-ef3a-41ea-a1b3-3593f3c7deee)
![sample_52](https://github.com/user-attachments/assets/3544e65e-0c2e-45cc-96c1-f27a5d7bf4cc)

        2.(DMVAE4)对于Multimodal VAE，重构PoE、Spectral和Motion，取消加速度loss项，增加对齐loss(beta = 0.001)(已完成，训练集MPJPE收敛于38，测试集的full MPJPE收敛于138，测试集的motion MPJPE收敛于80，测试集的spectral MPJPE收敛于227)，测试集full重构结果：
![sample_58](https://github.com/user-attachments/assets/4cb0bcfb-38dc-4bd3-b1db-474cf2ea1f32)
![sample_52](https://github.com/user-attachments/assets/0630855f-6d22-4deb-9c81-716f77134a88)

        测试集motion重构结果：
![sample_58](https://github.com/user-attachments/assets/a28824d8-1853-4de1-8d0f-518661e5f2e3)
![sample_52](https://github.com/user-attachments/assets/06a85f9a-7dc7-4e77-afc0-df7ad477b816)

        测试集spectral重构结果：
![sample_58](https://github.com/user-attachments/assets/2cc320bd-162f-408d-92aa-8aa447f70136)
![sample_52](https://github.com/user-attachments/assets/842917b9-7e03-4c08-b9bf-cc1ed9c63564)

        3.(DMVAE3)对于Multimodal VAE，只重构Motion，保留加速度loss项，增加对齐loss(beta = 0.1)(已完成，跑了差不多1000个epoch，没有收敛的趋势，训练集的MPJPE一直在250左右震荡)
        4.(DMVAE3)对于Multimodal VAE，分成两步训练，即先训练Motion VAE，然后冻结，再训练Spectral Encoder，阶段二的损失函数为Loss_align和Loss_KL(已完成，loss下降很快，说明两者已经对齐，但是测试集的MPJPE降不下去，一直在250左右波动)，具体loss如下所示：
<img width="1119" height="713" alt="image" src="https://github.com/user-attachments/assets/dcc0fa1c-6d08-4e69-a5b3-a370b8357e9c" />

        5.(MLD2)对于上述的(DMVAE3)，在扩散模型中进行测试，Spectral_Encoder_cond冻结(已完成，训练集的loss一直在0.25以上，测试集还是收敛不了，测试集的MPJPE在350左右)，测试集的重构结果：
![sample_29](https://github.com/user-attachments/assets/f71923cc-e329-4e0d-b37c-b0f9f6a01f2a)
![sample_8](https://github.com/user-attachments/assets/599238ff-5ffc-4ede-964a-334cd4b29419)

        6.(MLD2)在上个实验的基础上，直接将输入的condition换成Motion Encoder的输出z，以验证condition diffusion的有效性(已完成，训练集的loss收敛于0.1左右，但是测试集的MPJPE一直在300左右)
        7.(MLD2)将扩散模型的条件注入方式由concatenate换成FiLM(已完成，训练集的loss收敛于0.17左右，测试集的MPJPE一直在280左右震荡)，测试集重构结果：
![sample_17](https://github.com/user-attachments/assets/1e978e5a-2a0f-4d12-b2d5-5b7f0e3cd5a4)
![sample_15](https://github.com/user-attachments/assets/807a8e42-a1b5-47fe-bcad-8b81965e8af1)

</details>

**1.22**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>
    
        1.(MLD2)重新检查了一下代码，发现测试集部分的代码没有换成Motion Encoder的输出z，对其进行修改并重跑6(已完成，训练集的loss收敛于0.1左右，测试集的MPJPE收敛于180)，测试集的重构结果：
    
![sample_17](https://github.com/user-attachments/assets/d7d6f1d2-0534-429f-8698-823cac405969)
![sample_15](https://github.com/user-attachments/assets/1550efc5-1827-4bc2-9089-ce2bc273cd9f)
    
        2.(MLD2)将扩散模型的条件注入方式由concatenate换成FiLM(已完成，训练集的loss收敛于0.17左右，测试集的MPJPE收敛于148)，测试集的重构结果：
![sample_15](https://github.com/user-attachments/assets/8cd44aef-d02e-417d-81d2-3473b9620fe4)
![sample_17](https://github.com/user-attachments/assets/0e64a871-97d0-4f02-aa1f-1d658d17669d)

        3.(MLD3)在2的基础上，将采样过程的DDIM的步数由50改为500(已完成，训练集的loss收敛于0.17左右，测试集的MPJPE收敛于142)，测试集的重构结果：
![sample_15](https://github.com/user-attachments/assets/e0a68f7f-0891-4c1d-bc9d-f507be5b8a21)
![sample_17](https://github.com/user-attachments/assets/0bab2b92-6477-4fb0-8af0-dd35cb9f71f9)

        4.(MLD4)在2的基础上，将denoiser的预测目标由噪声改为真实值(已完成，训练集的loss收敛于0.078左右，测试集的MPJPE收敛于73)测试集的重构结果：
![sample_15](https://github.com/user-attachments/assets/7e5faa57-38fc-4ab6-ada6-4cc143a9a43a)
![sample_17](https://github.com/user-attachments/assets/5e72813a-111a-42da-8c50-19d3efdc44c1)

        5.(MLD3)在上个实验的基础上，将扩散模型的条件注入方式改回concatenate，denoiser的预测目标由噪声改为真实值(已完成，可以无限逼近真实值)
        6.(MLD3)在上个实验的基础上，将条件输入由Motion Encoder的输出z换成Spectral Encoder的输出(已完成，训练集的loss收敛于0.42，测试集的MPJPE收敛于272)，测试集的重构结果：
![sample_17](https://github.com/user-attachments/assets/b8b2c7f1-dfb3-479a-8458-f2ce7384cd87)
![sample_15](https://github.com/user-attachments/assets/dea6b989-2a2d-40b7-9f0a-e2e8128ed0c4)

        7.(MLD2)在上个实验的基础上，将扩散模型的条件注入方式改回concatenate(已完成，训练集的loss收敛于0.42，测试集的MPJPE收敛于271)，测试集的重构结果：
![sample_17](https://github.com/user-attachments/assets/6b82be9c-d789-49d3-b37b-3ed3f63b11f5)
![sample_15](https://github.com/user-attachments/assets/28ea5e14-18e0-4f16-9da7-fb65188fbb47)


</details>  

**1.23**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>
    
        实验九(确定性能差的具体原因：模型问题还是数据问题)
        1.(DMVAE3)尝试提高获取的AoADFS的分辨率(32*31->64*62)(已完成，在训练1000个epoch后，训练集的loss收敛于0.5左右，测试集的MPJPE收敛于230左右)
        2.(MLD2)分辨率(64*62), 条件注入方式(concatenate)(已完成，测试集的MPJPE降不下去，一直在250左右震荡，loss也在0.42左右波动)
        3.(DMVAE4)使用Person in wifi数据集，分辨率(32*31)，取消加速度loss项(已完成，在阶段1中，训练集的MPJPE收敛于50左右，在阶段2，测试集motion的MPJPE收敛于60，测试集spectral的MPJPE收敛于400左右)
        4.(MLD2)在上个实验的基础上，测试在扩散模型中的性能(已完成，loss根本降不下来，测试集的MPJPE在400左右震荡)

</details>  

**1.24**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>
    
        综合上述实验，可以确定主要原因是使用VAE的话，压缩到latent space会导致误差被放大，同时VAE的latent空间是紧凑表示，不同动作之间的距离小
        实验十(对模型进行修正，使用AE+对比学习)
        1.(DMVAE4)使用改进的AE+对比学习的模型进行训练(loss_contra的系数为0.1)(已完成，训练集的MPJPE收敛到45左右，测试集motion的MPJPE收敛于55左右，测试集spectral的MPJPE收敛于360左右)，测试集motion重构效果：
![sample_17](https://github.com/user-attachments/assets/fe92ee41-24b1-4484-805f-79aa7299b8d3)
![sample_15](https://github.com/user-attachments/assets/29378ce0-fc30-4677-9417-0182b308eed8)

        测试集spectral重构结果：
![sample_17](https://github.com/user-attachments/assets/028da650-e06c-4acb-af24-6e41f726654c)
![sample_15](https://github.com/user-attachments/assets/97f52ffa-cb14-48d4-be96-14dc19aa412e)

        对z_motion进行t-sne可视化：
<img width="500" height="500" alt="z_motion_tsne" src="https://github.com/user-attachments/assets/a64b40ec-2393-430c-b195-1cd3058ea795" />

        2.(DMVAE5)使用改进的AE+对比学习的模型进行训练(loss_constra的系数为0.01)(和上个实验基本一样，loss收敛更快，训练集的MPJPE可以收敛到30左右，stage2的loss：从0.82收敛到0.0021左右)
        3.(DMVAE3)对MMFi数据集的z_motion进行t-sne可视化，可视化的结果：
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/535a6c29-d340-4ac7-9a52-19d203500d59" />

        4.(MLD2)对上述的(DMVAE4)进行扩散模型的验证实验(已完成，测试集的MPJPE收敛到387左右，loss下降很快，从0.91降到0.021)

</details>  


**1.25**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>
    
        1.(DMVAE4)(MLD2)在上个实验的基础上，将z_spectral对齐的对象由z_motion改为h_motion(已完成，扩散模型测试集的MPJPE在500左右，可视化结果表明，生成的动作有明显的偏离)：
![sample_29](https://github.com/user-attachments/assets/7a9a3a40-288e-4db1-be51-73b9bbeb161b)
![sample_8](https://github.com/user-attachments/assets/c2494cdd-8a19-4997-861a-831d682fbeee)

        2.(DMVAE5)(MLD2)在上个实验的基础上，将z_spectral对齐的对象改回z_motion，增加一项motion_rec_spectral和motion的loss(已完成，扩散模型测试集的MPJPE在387左右，生成的动作还是会产生偏离)：
![sample_17](https://github.com/user-attachments/assets/f776c906-f87a-42d2-a7c8-b4b5e358ee1e)
![sample_15](https://github.com/user-attachments/assets/a12243b0-7114-4b0e-9ff6-755949459a4e)

        3.(MLD2)在上个实验的基础上，将条件注入方式改为Concatenate(已完成，测试集的MPJPE收敛于400左右)
        4.(MLD2)在上个实验的基础上，将预测目标由真实值改为噪声(已完成，收敛速度显著变慢，测试集的MPJPE收敛于400)
        4.(DMVAE4)(MLD2)将输入由3D human motion换成3D rotation，同时loss处增加loss_rec和loss_rec_3d，z_spectral的对齐对象为h_motion(已完成，扩散模型测试集的MPJPE收敛至220左右，如果不对扩散模型的Spectral Encoder进行冻结的话，其MPJPE收敛至180左右)
</details>  

**1.26**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>
    
        1.(DMVAE4)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d和loss_contra(系数为0.1)，对不带噪的human motion进行重构，扩散模型不对Spectral Encoder进行冻结(DMVAE训练500个epoch，在扩散模型中最优MPJPE为167)
        2.(DMVAE5)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d和loss_contra(系数为0.1)，对不带噪的human motion进行重构，同时Spectral Encoder的输出和h_motion做对比学习，扩散模型对Spectral Encoder进行冻结(DMVAE训练500个epoch，在扩散模型中最优MPJPE为182)
        3.(DMVAE5)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d和loss_contra(系数为0.1)，对不带噪的human motion进行重构，同时Spectral Encoder的输出和z_motion做对比学习，扩散模型对Spectral Encoder进行冻结(DMVAE训练500个epoch，在扩散模型中最优MPJPE为200)
        4.(DMVAE4)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d和loss_contra(系数为0.01)，对不带噪的human motion进行重构，扩散模型不对Spectral Encoder进行冻结(DMVAE阶段1在训练集的MPJPE28左右，测试集的MPJPE在65左右，DMVAE训练2000个epoch，在扩散模型中最优MPJPE为180)
        5.(DMVAE5)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d，对不带噪的human motion进行重构，同时Spectral Encoder的输出和z_motion做对比学习，扩散模型对Spectral Encoder进行冻结((1000个epoch)DMVAE阶段1在训练集的MPJPE40左右，测试集的MPJPE62左右，DMVAE阶段2的loss收敛至2.5左右，扩散模型中最优MPJPE为171)
        6.(DMVAE5)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d，对不带噪的human motion进行重构，同时Spectral Encoder的输出和z_motion做对比学习(不对两个batch_size进行concatenate)，扩散模型对Spectral Encoder进行冻结(在扩散模型中的最优MPJPE为192)
        7.(DMVAE5)(MLD2)在5的基础上，将loss_rec的系数由1改为0.1((1000个epoch)DMVAE阶段1在训练集的MPJPE35左右，测试集的MPJPE56左右，DMVAE阶段2的loss收敛至2.76，扩散模型中最优MPJPE为174)
        8.(DMVAE4)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d，仅对带噪的human motion进行重构，同时Spectral Encoder的输出和不带噪的z_motion做对比学习(扩散模型中最优MPJPE为178)
        9.(DMVAE6)(MLD2)输入为3D rotation，loss包含loss_rec、loss_rec_3d，仅对带噪的human motion进行重构，同时Spectral Encoder的输出和带噪的z_motion做对比学习(扩散模型中最优MPJPE为202)
</details>  

**1.27**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>
    
        实验十一(提升测试集的泛化能力)
        1.(DMVAE4)(MLD2)Encoder和Decoder进行冻结，Spectral Encoder不进行冻结，对扩散模型的x进行加噪(和DMVAE加同分布的噪声，sigma = 3e-2)然后输入至Encoder(已完成，测试集MPJPE为206)
        2.(DMVAE4)(MLD3)Encoder和Decoder进行冻结，Spectral Encoder不进行冻结，对扩散模型的x进行加噪(和DMVAE加同分布的噪声，sigma = 1e-2)然后输入至Encoder(已完成，测试集MPJPE为206)
        3.(DMVAE4)(MLD2)Encoder和Decoder进行冻结，Spectral Encoder不进行冻结，对扩散模型的z进行加噪(白噪声，sigma = 1e-2)然后输入至Encoder(已完成，测试集MPJPE为206)
        4.(DMVAE6)(MLD2)对DMVAE的Encoder进行冻结，对Encoder的输出进行加噪(sigma = 0.1)，训练Decoder
        5.代码有误，上述四个实验需要重新进行
        6.(DMVAE6)(MLD2)对DMVAE的Encoder进行冻结，对Encoder的输出进行加噪(sigma = 0.1)，训练Decoder(decA epoch = 99)(已完成，扩散模型测试集MPJPE为169)
        7.(DMVAE4)(MLD3)Encoder和Decoder进行冻结，Spectral Encoder不进行冻结，对扩散模型的x进行加噪(和DMVAE加同分布的噪声，sigma = 1e-2)然后输入至Encoder(已完成，扩散模型测试集MPJPE为185)
        8.(DMVAE6)(MLD2)对DMVAE的Encoder进行冻结，对Encoder的输出进行加噪(sigma = 0.1)，训练Decoder(decA epoch = 299)(已完成，和6基本一致)
        9.(DMVAE4)(MLD3)Encoder和Decoder进行冻结，Spectral Encoder不进行冻结，对扩散模型的x进行加噪(和DMVAE加同分布的噪声，sigma = 3e-2)然后输入至Encoder(已完成，扩散模型测试集MPJPE为188)
        10.(DMVAE6)(MLD2)对DMVAE的Encoder进行冻结，对Encoder的输出进行加噪(sigma = 0.1)，训练Decoder(decA epoch = 99)，同时对扩散模型的x进行加噪(和DMVAE加同分布的噪声，sigma = 1e-2)(已完成，测试集MPJPE为200)
        11.(DMVAE4)(MLD3)Encoder和Decoder进行冻结，Spectral Encoder不进行冻结，对扩散模型的z进行加噪(白噪声，sigma = 1e-2)(已完成，扩散模型测试集MPJPE为180)
        12.(DMVAE4)(MLD2)对AE中仅增加正则化loss约束，其他不变，扩散模型不进行任何加噪操作(系数=1e-3)(DMVAE测试集MPJPE收敛于65)(DMVAE的epoch = 199，扩散模型最优MPJPE = 156；DMVAE的epoch = 399，扩散模型最优MPJPE = 166)
        13.(DMVAE4)(MLD2)对AE中仅增加正则化loss约束，其他不变，扩散模型不进行任何加噪操作(系数=1e-3)，扩散模型中也增加正则化loss约束(系数=1e-3)(DMVAE的epoch = 399，扩散模型最优MPJPE = 166)
        14.(DMVAE4)(MLD2)对AE中仅增加正则化loss约束，其他不变，扩散模型不进行任何加噪操作(系数=1e-3)，扩散模型中的denoiser的层数由9减少至7
</details>  

**1.28**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

        实验十二(Person in wifi数据集)(平衡DMVAE和diffusion的泛化能力)
        1.DMVAE部分仅对Encoder和Decoder进行训练，采用VAE方式，扩散模型进行验证(已完成，扩散模型在训练集上的loss很大，主要原因在于Encoder的输出不稳定，是随机值，这导致扩散模型的loss很难进行下降？但是Motion latent diffusion却能够得到较好的结果，可能是因为没有对Spectral Encoder进行预训练？)
        2.在上个实验的基础上，对Spectral Encoder进行预训练，这里Spectral Encoder的输出和Motion Encoder的均值进行对比学习
        3.对于DMVAE的Spectral Encoder，减少网络层数
</details>  

**1.30**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

        实验十三(对DecA进行加噪微调)
        1.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.001)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="748" height="414" alt="image" src="https://github.com/user-attachments/assets/3e0cbb50-33f0-413c-a8e5-91b97fe9e9f7" />

        对于diffusion，其测试集MPJPE：
<img width="272" height="160" alt="image" src="https://github.com/user-attachments/assets/a25ba667-c596-4d99-bf40-bad302f076ab" />

        2.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.001)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="299" height="170" alt="image" src="https://github.com/user-attachments/assets/4d60bbfd-c1c8-4c60-8711-9636430a104e" />

        3.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.002)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="1001" height="194" alt="image" src="https://github.com/user-attachments/assets/6d5c7af3-aee0-4218-85bd-d9199c0bb4a4" />

        对于diffusion，其测试集MPJPE：
<img width="260" height="159" alt="image" src="https://github.com/user-attachments/assets/13c28af7-6cd5-4ed1-9a20-ca36b6290d34" />

        4.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.002)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="281" height="170" alt="image" src="https://github.com/user-attachments/assets/4a6767e7-5dc5-4e27-be82-837ba159476b" />

        5.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.004)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="1013" height="185" alt="image" src="https://github.com/user-attachments/assets/40486083-4055-42a6-9143-fd5c78166b36" />

        对于diffusion，其测试集MPJPE：
<img width="272" height="155" alt="image" src="https://github.com/user-attachments/assets/a5b19f8a-b8e1-4a33-99bf-9ab0e169fdf1" />

        6.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.004)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="286" height="170" alt="image" src="https://github.com/user-attachments/assets/7cabc8d0-22bb-42c3-9bb2-036887fe0a12" />

        7.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.01)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="977" height="193" alt="image" src="https://github.com/user-attachments/assets/e94531b8-b979-4adb-a79e-0aece5fdfe60" />

        对于diffusion，其测试集MPJPE：
<img width="268" height="292" alt="image" src="https://github.com/user-attachments/assets/23a1726f-aecf-4445-b489-e686ec1edf36" />

        8.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.01)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="286" height="307" alt="image" src="https://github.com/user-attachments/assets/aaf55330-1905-4408-9b81-a11ea0b65324" />

        9.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.02)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="992" height="302" alt="image" src="https://github.com/user-attachments/assets/cfaced24-9aae-4fbe-a893-34460da480dc" />

        对于diffusion，其测试集MPJPE：
<img width="260" height="159" alt="image" src="https://github.com/user-attachments/assets/717af5f9-1b33-4999-ba0a-45489a58fc76" />

        10.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.02)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="281" height="170" alt="image" src="https://github.com/user-attachments/assets/5efcb121-e350-496c-b44f-092bb4889b9a" />

        11.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.04)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="515" height="98" alt="image" src="https://github.com/user-attachments/assets/3c9cfa68-9b44-4528-9894-74103a9b442c" />

        对于diffusion，其测试集MPJPE：
<img width="260" height="147" alt="image" src="https://github.com/user-attachments/assets/21c3b57a-46cd-4f44-9f53-1dc4a0cc5e9f" />

        12.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.04)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="280" height="180" alt="image" src="https://github.com/user-attachments/assets/83297677-6b6c-426a-8ed6-77dd6d5274ad" />
        
        13.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.05)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="1141" height="336" alt="image" src="https://github.com/user-attachments/assets/cd8b8f07-3498-4e42-b69b-732b93b4478b" />

        对于diffusion，其测试集MPJPE：
<img width="256" height="286" alt="image" src="https://github.com/user-attachments/assets/df0f6e0d-bd8d-4355-b234-29bfded20f9a" />

        14.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.05)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="286" height="311" alt="image" src="https://github.com/user-attachments/assets/9fc43006-669b-4892-9dcd-e2a41dc84040" />

        15.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)：
        对于AE，其加噪测试集MPJPE：
<img width="886" height="187" alt="image" src="https://github.com/user-attachments/assets/0768c9e1-7b4f-4a3c-ba4d-7bd3b81fc709" />

        对于diffusion，其测试集MPJPE：
<img width="260" height="283" alt="image" src="https://github.com/user-attachments/assets/f32354d0-e447-4c46-a27d-70b489ef37b5" />

        16.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，其测试集MPJPE:
<img width="344" height="377" alt="image" src="https://github.com/user-attachments/assets/1440872b-2704-471e-b896-ed99cd711cfb" />

        
        17.扩散模型测试集下loss_test的误差分布：
<img width="2700" height="1500" alt="image" src="https://github.com/user-attachments/assets/ff1b4e89-7b50-4150-80e6-a1466249b90c" />

        18.DecA加噪分布：
<img width="2700" height="1500" alt="image" src="https://github.com/user-attachments/assets/65a96d7a-46a8-48e5-a167-01f7dea001b7" />

        (后续思路：1.不改变加噪策略，优化其他方面的参数设置 2.不改变加噪策略，主动对diffusion的输出进行加噪)(使用训练epoch更高的epoch，扩散模型的loss_test还能进一步下降)
        
</details>  
        
**1.31**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

        实验十四(对diffusion的输出进行加噪)
        1.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，对扩散模型的输出进行加噪(sigma = 0.01):
<img width="279" height="172" alt="image" src="https://github.com/user-attachments/assets/97bbbd56-3db5-4a50-b8d4-bf2e98bcba86" />

        2.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，对扩散模型的输出进行加噪(sigma = 0.02):
<img width="263" height="160" alt="image" src="https://github.com/user-attachments/assets/a6eb92b8-e4a6-4780-8cab-abead9be55bd" />

        实验十五(调参优化)
        1.(DMVAE6)对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，stage1的epoch = 599：
<img width="881" height="210" alt="image" src="https://github.com/user-attachments/assets/29142b51-9f85-4281-83d6-0d59ede98f17" />

        2.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，stage1的epoch = 599，对于扩散模型，其DecA的训练轮次为(epoch = 399)：
<img width="290" height="172" alt="image" src="https://github.com/user-attachments/assets/eaed0090-b59b-42a6-80bb-b179f3fd51d6" />

        3.(DMVAE6)对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，stage1的epoch = 399：
<img width="883" height="150" alt="image" src="https://github.com/user-attachments/assets/207a4c6d-8d3e-440a-b50d-008a38ad4076" />

        4.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，stage1的epoch = 399，对于扩散模型，其DecA的训练轮次为(epoch = 499)：
<img width="284" height="184" alt="image" src="https://github.com/user-attachments/assets/44449bad-6d1f-4450-a1ef-890efe8594f7" />

        5.(DMVAE6)对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.04)，stage1的epoch = 399：
<img width="443" height="80" alt="image" src="https://github.com/user-attachments/assets/491fe910-7e71-4c46-8ab5-23dbad3ffd09" />

        6.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.04)，stage1的epoch = 399，对于扩散模型，其DecA的训练轮次为(epoch = 499)：
<img width="287" height="308" alt="image" src="https://github.com/user-attachments/assets/21c2c8d3-346f-4b7c-b5cb-ff57f2e9a6f3" />

        6.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，对扩散模型的输出进行加噪(sigma = 0.02)，将扩散模型的denoiser的参数改为初始值，即ff_size = 1024，num_layer = 9，dropout = 0.1:
<img width="260" height="134" alt="image" src="https://github.com/user-attachments/assets/560446fd-2045-4f59-ab29-b3d3a03a21ed" />

        7.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，将扩散模型的denoiser的参数进行修改，ff_size = 1024，num_layer = 9，dropout = 0.4：
<img width="257" height="158" alt="image" src="https://github.com/user-attachments/assets/85cdc487-4743-4f90-bd69-56d78c316c42" />

        8.(DMVAE6)(MLD(Person in wifi))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，将扩散模型的denoiser的参数进行修改，ff_size = 1024，num_layer = 9，dropout = 0.3，将spectral Encoder的dropout由0.1改为0.3：
<img width="269" height="290" alt="image" src="https://github.com/user-attachments/assets/ef550c9d-8a76-4727-8338-c8e99b9cb8c1" />

        在这之前的所有实验使用的AoADFS都是40帧
        9.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.04)，stage1的epoch = 399，对于扩散模型，其DecA的训练轮次为(epoch = 499)，对z_motion进行标准化之后再输入diffusion中，输入的AoADFS为40帧：
        训练集的训练情况：
<img width="281" height="133" alt="image" src="https://github.com/user-attachments/assets/ff1607ec-de2a-4877-8aa2-efb9b0c23852" />

        测试集的训练情况：
<img width="275" height="148" alt="image" src="https://github.com/user-attachments/assets/3af9e51d-1fb3-4203-80c9-1039b3bf4a72" />

        对测试集的z和z_ref进行denormalized，同时计算对应的mse，发现远小于0.0017，但是MPJPE并没有出现明显下降，说明DecA对于z的方向更加敏感，而不是z的数值大小
        
</details>  

**2.1**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

        1.(DMVAE6)(MLD(Person in wifi2))对于AE，stage2不训练，stage1的epoch = 399，对于扩散模型，对z_motion进行标准化之后再输入diffusion中，输入的AoADFS为40帧：
<img width="278" height="140" alt="image" src="https://github.com/user-attachments/assets/66a9c485-0886-404a-883b-6d28bec4bd6b" />

        2.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，对z_motion进行标准化，其测试集MPJPE:
<img width="275" height="140" alt="image" src="https://github.com/user-attachments/assets/bdadd1c7-e7ef-402b-bf51-5d9875b89f25" />

</details>  

**2.2**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

        实验十六(调整L2 loss系数，使得得到的latent的能量能够分散在各个维度，同时数值保持在一定范围内)
        1.(DMVAE6)(MLD(Person in wifi2))将L2 loss系数改为0.01，stage2不训练，stage1的epoch = 199，对于扩散模型，对z_motion进行标准化之后再输入diffusion中，输入的AoADFS为40帧：
<img width="281" height="171" alt="image" src="https://github.com/user-attachments/assets/39ebbc86-5a1a-4d70-98ff-071d6795a48e" />

        实验十七(调参优化)
        1.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，增加Reconstruction Guidance，
<img width="206" height="121" alt="image" src="https://github.com/user-attachments/assets/fe3545ae-2073-4de1-bda0-ab1d522065b9" />

        2.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，提高AoADFS的分辨率(32*31->181*182)
<img width="305" height="332" alt="image" src="https://github.com/user-attachments/assets/2913b9c7-1c35-4453-8741-7d561958649a" />

        3.(DMVAE6)(MLD(Person in wifi2))对于AE，不在stage2进行加噪，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中
<img width="260" height="131" alt="image" src="https://github.com/user-attachments/assets/10391b67-80cb-4050-9c98-b62d0b0e62e0" />

        4.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中
<img width="257" height="144" alt="image" src="https://github.com/user-attachments/assets/f1ed212a-661e-4446-b300-21679a39c145" />
        
</details>  

**2.4**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

         1.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中：
<img width="265" height="286" alt="image" src="https://github.com/user-attachments/assets/00b2d0c9-8fed-443e-b685-78580194d9d9" />

        2.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中，在采样过程中使用Reconstruction Guidance：
<img width="264" height="179" alt="image" src="https://github.com/user-attachments/assets/b00e42e1-1d6a-4805-8dae-84c6a2a6328d" />

        3.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中，同时对z_motion进行标准化：
<img width="278" height="123" alt="image" src="https://github.com/user-attachments/assets/6a1b6942-cf64-4cf9-bfd6-e90af669ec18" />


        4.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance：
<img width="277" height="137" alt="image" src="https://github.com/user-attachments/assets/553e3682-cba8-473d-8aac-ae651d5eff7e" />

        5.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 399)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance：
<img width="272" height="113" alt="image" src="https://github.com/user-attachments/assets/97332ab8-cdc9-4e2a-bcea-8e97fef690ea" />

        6.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance(N = 1000)：
<img width="279" height="313" alt="image" src="https://github.com/user-attachments/assets/2656ccc6-0ee9-46ac-b1e6-41c4a2e4ccc0" />


</details>  

**2.5**  
<details>
<summary>📖 问题记录</summary>  
    

</details>  

<details>
<summary>📖 实验记录</summary>

        1.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance：
<img width="274" height="310" alt="image" src="https://github.com/user-attachments/assets/28120360-c4f7-43c9-bac6-20436a4f9455" />

        2.(DMVAE6)(MLD(Person in wifi3))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，AoADFS只取后30帧，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance：
<img width="289" height="179" alt="image" src="https://github.com/user-attachments/assets/ca8d799e-99fb-47d0-8ca6-340843a49e7a" />

        3.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.1)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance:
<img width="273" height="82" alt="image" src="https://github.com/user-attachments/assets/c33101e0-c35c-4f61-850c-44576c2e38f3" />


        4.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance，提高AoADFS的时间分辨率(40->80):
<img width="281" height="307" alt="image" src="https://github.com/user-attachments/assets/bf493e51-33bf-44f5-b2b3-aa24b777cb3e" />

        5.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance，提高AoADFS的时间分辨率(40->80)(截取80个中的后60个):
<img width="280" height="167" alt="image" src="https://github.com/user-attachments/assets/c88c3ff3-86a1-48ca-8359-17064920cf59" />

        6.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance，提高AoADFS的时间分辨率(40->80)(截取80个中的后60个)，learning rate设置为2e-5:
<img width="282" height="185" alt="image" src="https://github.com/user-attachments/assets/595ff7f5-b69e-45ba-ac30-44807c57f5e0" />

        7.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance，提高AoADFS的时间分辨率(40->80)(截取80个中的后60个)，learning rate设置为5e-5:
<img width="274" height="139" alt="image" src="https://github.com/user-attachments/assets/5c4f7266-2ef0-4bde-8aeb-75482726f027" />


        8.(DMVAE6)(MLD(Person in wifi2))对于AE，在stage2对DecA进行训练，其中对EncA的输出z进行加噪(sigma = 0.07)，对于扩散模型，其DecA的训练轮次为(epoch = 199)，对human motion进行标准化后输入模型中，同时对AoADFS进行标准化输入至Spectral Encoder中(将原本的Per-location改为Channel-wise)，同时对z_motion进行标准化，采样过程中使用Reconstruction Guidance，提高AoADFS的时间分辨率(40->80)(截取80个中的后60个)，learning rate设置为2e-4:






</details>  







