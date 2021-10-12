<div align="center">
  <img src="https://upload-images.jianshu.io/upload_images/14008578-f6155ba56122d76b.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp"/>
</div>

<h></h>

# CIFAR10:一个用于识别普适物体的数据集

CIFAR-10 数据集由 10 个类别的 60000 张 32x32 彩色图像组成，每个类别 6000 张图像。有 50000 张训练图像和 10000 张测试图像。CIFAR-10标注一共包含10个类别：airplane(飞机)，automobile（汽车），bird（鸟），cat（猫），deer（鹿），dog（狗），frog（青蛙），horse（马），ship（船）和truck（卡车）。

数据集分为五个训练批次和一个测试批次，每个批次有 10000 张图像。测试批次包含从每个类别中随机选择的 1000 张图像。训练批次包含随机顺序的剩余图像，但一些训练批次可能包含比另一个类别更多的图像。在它们之间，训练批次包含来自每个类的 5000 张图像。

<h></h>
<h></h>

目录
=================
* [模型选择](#模型选择)
* [模型配置](#模型配置)
* [实验内容](#实验内容)


## 模型选择

常见用于CIFAR-10分类的模型有VGG16、ResNet18、ResNet50、ResNet101、DLA等，在本实验中选取VGG16进行训练和分类任务。

VGG16共包含：

* 13个卷积层（Convolutional Layer），分别用conv3-XXX表示
* 3个全连接层（Fully connected Layer）,分别用FC-XXXX表示
* 5个池化层（Pool layer）,分别用maxpool表示

其中，卷积层和全连接层具有权重系数，因此也被称为权重层，总数目为13+3=16，这是VGG16中16的来源。

<div align="center">
  <img src="https://img2018.cnblogs.com/blog/1365470/201903/1365470-20190307201517501-751836953.png"/>
</div>

VGG16具有如此之大的参数数目，可以预期它具有很高的拟合能力；但同时缺点也很明显：

* 即训练时间过长，调参难度大。
* 需要的存储容量大，不利于部署。例如存储VGG16权重值文件的大小为500多MB，不利于安装到嵌入式系统中。

## 模型配置

```
conda create pytorch
conda activate pytorch
conda install torch
conda install torchvision
conda install tensorboard
```

## 实验内容
### 步骤1：检查初始loss

![image](https://user-images.githubusercontent.com/82877577/136906251-7bdfb0e8-1329-4dd5-ba67-12f11178db92.png)

正常初始loss的值为2.4左右，此时初始loss值为2.4347，接近2.4，故初始loss结果验证正确。

### 步骤2: 拟合一个小样例

在一个小样例上尝试将训练的准确率提升至100%。

![image](https://user-images.githubusercontent.com/82877577/136908459-a9a184d1-09e3-4a50-9fa7-c24422817c25.png)

训练到第180个epoch时，在数据集上的准确率到达100%，证明该模型对于该数据集有效。

### 步骤3: 找到使得梯度下降最快的学习率

假设选定的学习率有1e-1,1e-2,1e-3,1e-4。在一个epoch上对所有batch进行训练，并记录loss。

![image](https://user-images.githubusercontent.com/82877577/136916663-aedcca14-3323-4172-b0f3-5137ada6f198.png)

通过实验可以看出，随着学习率逐渐变小，loss下降的趋势更加明显。但学习率小于0.001时，loss下降趋势趋于缓和。因此，当学习率设置为0.001时，loss的下降趋势最明显。

### 步骤4：根据不同参数，训练1-5个epoch

| 学习率 | 权重衰减 |   ACC  
| ----------------- | ----------- | ----------- |
| 1e-1 |  1e-4  | 37.696% |
| 1e-1 |  1e-5  | 34.060% |
| 1e-1 |   0    | 34.324% |
| 1e-2 | 1e-4 | 77.234% |
| 1e-2 | 1e-5 | 78.320% |
| 1e-2 |  0 |   76.902% |
| 1e-3 | 1e-4 | 76.900% |
| 1e-3 | 1e-5 | 75.750% |
| 1e-3 | 0 |    76.516% |
| 1e-4 | 1e-4 | 59.800% |
| 1e-4 | 1e-5 | 60.122% |
| 1e-4 | 0 |  60.082%   |

经过多次实验，在学习率为1e-2,权重衰减为1e-4时，训练5个epoch的情况下，训练集的准确率最高，为78.320%。

### 步骤5：重新定义参数值，训练10-20个epoch

在学习率为1e-2,权重衰减为1e-4的前提下，分别用10个epcoh与20个epoch对数据集进行训练。

当训练10个epoch时， ACC为：

![image](https://user-images.githubusercontent.com/82877577/136914934-7ee51404-ebe7-4e54-b628-76e14d8b5de0.png)

当训练20个peoch时， ACC为：

![image](https://user-images.githubusercontent.com/82877577/136917187-885a0870-a85a-42fd-8fbb-7bfa56d29278.png)

### 步骤6：绘制损失与准确率曲线

在训练集上训练20个epoch，在训练集上Loss下降为0.212，ACC达到大约92%的准确率。
#### ACC：
![image](https://user-images.githubusercontent.com/82877577/136947985-bb928e88-651c-4f73-a931-e45e8884b5db.png)

#### Loss：
![image](https://user-images.githubusercontent.com/82877577/136948620-3a79bc38-591d-469d-88a8-e5384a87703b.png)

在训练集与验证集上训练20个epoch，训练集的准确率与验证集的准确率曲线较为拟合。

![image](https://user-images.githubusercontent.com/82877577/136956416-354dcd5e-67fa-4a37-92d1-5e351819c326.png)
