
# Contextual ensemble network for semantic segmentation(CENet)

## 引言
### 贡献

1. CENet 引入了一种==新颖的编码器-解码器架构==，通过**集成反卷积**捕获多尺度上下文。==堆叠的特征图相互补充==，使我们能够==充分探索图像中嵌入的多尺度上下文信息==。
	1. CENet introduces a novel encoder-decoder architecture to capture multi-scale context via **ensemble deconvolution**. ==The stacked feature maps are complemented each othe==r, allowing us to ==fully explore multiple scale contextual information embedded in images==.
2. 


![](assets/Pasted%20image%2020231125170028.png)

> 摘自别的文章的相关工作
> 
> Zhou等人（2022）提出了上下文集成网络（CENet），其中上下文线索通过将编码器层的特征密集上采样到解码器层的特征来聚合。这使得CENet能够捕获多尺度上下文信息。虽然UNet++和CENet比UNet产生更高的性能，但它通过引入密集的跳过连接来实现，这导致参数和计算成本的巨大增加。


![](assets/Pasted%20image%2020231125170047.png)

## 1x1卷积的作用

编码器中堆叠从stage1到stage4的卷积特征而产生的。如果直接连接特征，则使用 RseNet-101 作为主干，通道数将为 64 + 256 + 512 + 1024 = 1856。训练如此复杂的网络是一项艰巨的工作，并且可能会==限制泛化能力==并导致我们的==模型过度拟合==。

采用 1 × 1 卷积来减少特征维度。为了计算方便，==特征通道数与编码器中对应的卷积层数保持一致==


# ADS_UNet: A nested UNet for histopathology image segmentation

![](assets/Pasted%20image%2020231125165024.png)