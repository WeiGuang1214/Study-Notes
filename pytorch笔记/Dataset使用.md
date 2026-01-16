#### 函数

dir（）：打开，看见容器里面是什么

help（）：说明书

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727098367580.jpg)

#### 读取数据

Dataset：提供一种方式去获取数据及其label

​                如何获取每一个数据及其label

​                告诉我们总共有多少的数据

Dataloader：为后面的网络提供不同的数据形式

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727098260528.jpg)

```python
# torch对数据集处理
# 重写MyData方法，并且按照绝对路径设置文件夹，标签
# 将文件夹内图片，以列表的形式创建数据集，可以通过：名字[下标]，访问图片以及相关信息
# os.path.join方法是合并两个路径
from torch.utils.data import Dataset  # torch中的Dataset类
from PIL import Image  # PIL对图像进行处理
import os


class MyData(Dataset):

    def __init__(self, root_dir, label_dir):
        self.root_dir = root_dir
        self.label_dir = label_dir
        self.path = os.path.join(self.root_dir, self.label_dir)
        self.img_path = os.listdir(self.path)

    def __getitem__(self, idx):
        img_name = self.img_path[idx]
        img_item_path = os.path.join(self.root_dir, self.label_dir, img_name)
        img = Image.open(img_item_path)
        label = self.label_dir
        return img, label

    def __len__(self):
        return len(self.img_path)


root_dir = "F:\\Python_study\\_torchStudy\\Data\\testData\\train"
ants_label_dir = "ants_image"
bees_label_dir = "bees_image"
ants_dataset = MyData(root_dir, ants_label_dir)
bees_dataset = MyData(root_dir, bees_label_dir)
print(ants_dataset[0])
img, label = ants_dataset[0]
img.show()

img, label = bees_dataset[0]
img.show()
train_data = ants_dataset + bees_dataset
print(len(train_data))
print(len(ants_dataset))
print(len(bees_dataset))
img, label = train_data[123]
img.show()
img, label = train_data[124]
img.show()

```

