**12.10**  
<details>
<summary>📖 问题记录</summary>  

    1.尝试直接使用对比学习预训练好的模型作为proxy model，也就是在采样的每一步将预测的旋转表示转为mesh输入至Encoder，对AoADFS也输入至Encoder，然后计算两者之间的infoNCE，结果显示计算出的梯度值非常小，对最终结果没有影响
    2.在1的基础上，尝试将infoNCE改为L2范数的平方
</details>  

<details>
<summary>📖 实验记录</summary>
  
</details>  
