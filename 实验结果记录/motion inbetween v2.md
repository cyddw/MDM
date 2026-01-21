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

        关于rotation表示重新转为abs表示出现旋转问题的总计：(1)在motion_representation中，new_data[:, 0] = rot_ang，new_data[:, [1, 2]] = r_pos[:, [0,2]]这两行代码已经保证了旋转表示是绝对旋转 (2)在recover from ric中，若args.abs_3d=False，则认为旋转表示是相对旋转，则会使用累加形式；若args.abs_3d=False，则认为旋转表示是绝对旋转，则不对其进行累加
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

        6.(MLD2)在上个实验的基础上，直接将输入的condition换成Motion Encoder的输出z，以验证condition diffusion的有效性
</details>
