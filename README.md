# HDRmaster
This project is HDR convolutional neural network(CNN) based on pytorch aiming at HDR reconstrction from a single LDR
--------------------------------------------------------------------------------------------------------------------
[HDR预测网络结构.docx](https://github.com/guanlnny/HDRmaster/files/7024788/HDR.docx)

--------------------------------------------------------------------------------------------------------------------
#工具与依赖包：
Python3、pytorch、GPU(如没有GPU提速，可以使用CPU版本，大概会有1-3倍的速度差异)；
其他依赖包详见requirements.txt，值得注意的是，如果您使用GPU版本，请确认电脑cuda版本，cuda版本越高，所需的pytorch和torchvision的版本越高（我们的版本为python3.6.13，cuda11.0，torch1.7.0，torchvision0.8.0）。

#网络架构
卷积神经网络的架构由三个类似的块组成，如图1所示。第一个块是特征提取块(FEB, Feature extraction block)，然后是特征密集块(FDB, Feature dense block)和HDR重建块(HRB, HDR reconstruction block)。其中，使用全局残差跳过连接，绕过LDR的低级特征以指导最终层的HDR重构块。

![image](https://user-images.githubusercontent.com/74043204/130309328-2ffa0fb8-9588-41fb-b36b-850ce4c71ba2.png)
图1 卷积神经网络结构

值得一提的是，特征密集块FDB的结构如图2所示，其基本单元为膨胀密集块(DDB, Dilated Dense Block)。扩张的卷积有助于增加网络的接受域，DDB有助于利用输入的所有层次结构特性。除了两个1×1卷积层用于特征压缩外，每个DDB包含四个3×3膨胀的卷积层，每个卷积层都使用了之前所有层的信息，并且使用了密集的跳跃连接，如图3所示。由于密集的前向连接，这种特性的重用允许减少网络参数并提高学习能力。三个这样的DDB一起形成了这个网络的特征密集块。

![image](https://user-images.githubusercontent.com/74043204/130308961-e100da37-4bf1-4182-8584-52cd2e7749a4.png)
图2 特征密集块FDB结构

![image](https://user-images.githubusercontent.com/74043204/130308957-738b74cd-0b66-4884-ab44-0f9a61858e5b.png)
图3 膨胀密集块DDB结构

网络中的所有卷积层(conv2d)为3×3内核，然后是ReLU激活函数，除非另有说明。FEB有2个conv2d层，FDB有20(6×3+2)个conv2d层，HRB有2个conv2d层。

--------------------------------------------------------------------------------------------------------------------
#数据集Dataset
数据集由LDR(输入)和HDR(真实值)图像对组成。训练网络学习从LDR图像到对应的HDR真值对应的映射。
数据集应该遵循以下文件夹结构-
```
> dataset
    > train
        > LDR
            > ldr_image_1.jpg/png
            > ldr_image_2.jpg/png
            .
            .
        > HDR
            > hdr_image_1.hdr/exr
            > hdr_image_2.hdr/exr
            .
            .
    > test
	同理
```
HDR图像对网络开源资料丰富，附上一个资源链接：https://drive.google.com/open?id=1tv8kdeoT12AJL2iMnQkNUfgY2RjirNp9

--------------------------------------------------------------------------------------------------------------------
#训练：
我们在Geforce RTX 2080 GPU上训练网络，训练集和测试集的批次大小均为1。采用Adam优化器[14]，动量参数β1 = 0.5， β2 = 0.999。训练集中所有图像对都经过200代的训练，前100代的初始学习率为2e-4，在后100代中线性衰减。通过GPU加速训练，训练400对图像所需的时间约为6个小时。

--------------------------------------------------------------------------------------------------------------------
#测试：
我们给出了我们本地的训练结果文件HDRmaster.bin，方便您直接进行测试；
调用测试指令：python test.py --ckpt_path HDRmaster.bin --log_scores
