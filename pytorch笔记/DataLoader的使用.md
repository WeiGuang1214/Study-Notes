### 1、torch.utils.data.DataLoader(dataset,batch_size=1,shuffle[[=Flase,sample=None]])；dataset是数据集；batch_size是一次处理的数据量，批次；shuffle指定抽取数据是否按照相同的顺序；drop_last是指定舍弃最后一个数据

```python
import torchvision
from torch.utils.data import DataLoader
from torch.utils.tensorboard import SummaryWriter

test_data = torchvision.datasets.CIFAR10("./dataset",train=False,transform=tochvision.transforms.ToTensor())

test_loader = DataLoader(dataset=test_data,batch_size=64,shuffle=True,num_workers=0,drop_last=False)
#测试数据集第一张图片
img,target = test_data[0]
print(img.shape)
print(target)

writer = SummaryWriter("dataloader")
step=0

#等价于
for epoch in range(4):
    step=0
	for data in test_loader:#对所有的测试集数据进行4批次取data
   		 imgs,targets = data
  	 	 #print(imgs.shape)
  	 	 #print(targets)
	     writer.add_images("Epoch:{}".format(epoch),imgs,step)
   		 step=step+1
writer.close()

```

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727098476206.jpg)

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/image.png)
