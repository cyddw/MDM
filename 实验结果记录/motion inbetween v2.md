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
    2.在没有reconstruction guidance的情况下，修改scale = 0.05，MPJPE = 106.920；若physic_grad前面没有参数，scale = 0.05，MPJPE = 103.502
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details> 
