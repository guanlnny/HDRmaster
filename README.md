# HDRmaster
This project is HDR convolutional neural network(CNN) based on pytorch aiming at HDR reconstrction from a single LDR

网络架构
卷积神经网络的架构由三个类似的块[9]组成，如图4所示。第一个块是特征提取块(FEB, Feature extraction block)，然后是特征密集块(FDB, Feature dense block)和Mask重建块(MRB, Mask reconstruction block)。受[10]的启发，我们使用全局残差跳过连接，绕过LDR的低级特征以指导最终层的Mask重构块。

图4 卷积神经网络结构
FEB负责从输入图像中提取低级特征信息。
	(1)
其中，代表FEB的操作。
紧接着，输入到FDB，用以提取图像中的高级特征信息：
	(2)
其中，代表FDB的操作。
	同时，使用全局残差跳跃连接将FEB中第一卷积层的低层LDR特征添加到FDB的输出中，如下所示：
	(3)
其中，表示网络学习到的全局残差特征映射。
传递给MRB生成mask如下所示:
	(4)
其中，表示MRB的操作。 
值得一提的是，特征密集块FDB的结构如图5所示，其基本单元为膨胀密集块(DDB, Dilated Dense Block)，它是对[7]中提出的稠密块的改进。扩张的卷积有助于增加网络的接受域[11]，DDB有助于利用输入的所有层次结构特性。除了两个1×1卷积层用于特征压缩外，每个DDB包含四个3×3膨胀的卷积层，每个卷积层都使用了之前所有层的信息，并且使用了密集的跳跃连接，如图6所示。由于密集的前向连接，这种特性的重用允许减少网络参数并提高学习能力。三个这样的DDB一起形成了这个网络的特征密集块。


图5 特征密集块FDB结构

图6 膨胀密集块DDB结构
网络中的所有卷积层(conv2d)为3×3内核，然后是ReLU激活函数，除非另有说明。FEB有2个conv2d层，FDB有20(6×3+2)个conv2d层，HRB有2个conv2d层。
