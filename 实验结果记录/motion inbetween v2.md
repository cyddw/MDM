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
