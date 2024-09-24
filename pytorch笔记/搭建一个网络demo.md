### 1、Sequential函数，就是网络结构的一个序列，只需要输入input就可以，最终返回output

例子：

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727189216399.jpg)

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727189376083.jpg)



```python
import torch
from torch import nn
from torch.nn import MaxPool2d
from torch.nn import sigmoid
from torch.utils.tensorboard import SummaryWriter
from torch.nn import Conv2d, MaxPool2d,Flatten,Linear,Sequential

dataset = torchvision.datasets.CIFAR10("../data",train=False,download=True,
                                       			           
                                  transform=torchvision.transforms.ToTensor())

dataloader = DataLoader(dataset,batch_size=64)

input = torch.tensor([[1,-0.5],
                      [-1,3]])
input = torch.reshape(input,(-1,1,2,2))
print(input.shape)

class Tudui(nn.module):
    def __init__(self):
        super(Tudui,self).__init__()
        self.conv1 = Conv2d(3,32,5,padding=2)
        self.maxpool1 = MaxPool2d(2)
        self.conv2 = Conv2d(32,32,5,padding=2)
        self.maxpool2 = MaxPool2d(2)
        self.conv3 = Conv2d(32,64,5,padding=2)
        self.maxpool3 = MaxPool2d(2)
        self.flatten = Flatten()
        self.Linear1 = Linear(1024,64)
        self.Linear2 = Linear(64,10)
        
        self.model1 = Sequential(
        	Conv2d(3,32,5,padding=2),
            MaxPool2d(2),
            Conv2d(32,32,5,padding=2),
            MaxPool2d(2),
            Conv2d(32,64,5,padding=2),
            MaxPool2d(2),
            Flatten(),
            Linear(1024,64),
            Linear(64,10)
        )
        
    def forward(self,x):
        x=self.conv1(x)
        x=self.maxpool1(x)
        x=self.conv2(x)
        x=self.maxpool2(x)
        x=self.conv3(x)
        x=self.maxpool3(x)
        x=self.flatten(x)
        x=self.linear1(x)
        x=self.linear2(x)
        return x
    
        //forward就可以写成：
        x=self.model(x)
        return x

tudui = Tudui()
print(tudui)
input = torch.ones((64,3,32,32))
output = tudui(input)
print(output.shape)

```
