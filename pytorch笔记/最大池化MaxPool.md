### 1、MaxPool2d，最大池化是指取结果的最大值输出，默认步长stride是kernel的大小

### ceil_model指定是否插入一行或者列，保留当前值里的最大值；

例子：

```python
import torch
from torch import nn
from torch.nn import MaxPool2d
from torch.utils.tensorboard import SummaryWriter

dataset = torchvision.datasets.CIFAR10("../data",train=False,download=True,
                                       			           
                                  transform=torchvision.transforms.ToTensor())

dataloader = DataLoader(dataset,batch_size=64)

input = torch.tensor([[1,2,0,3,1],
                      [0,1,2,3,1],
                      [1,2,1,0,0],
                      [5,2,3,1,1],
                      [2,1,0,1,1]],dtype=torch.float32)
input = torch.reshape(input,(-1,1,5,5))
print(input.shape)

class Demo1(nn.module):
    def __init__(self):
        super(Demo1,self).__init__()
        self.maxpool1 = MaxPool2d(kernel_size=3,ceil_mode=True)
    def forward(self,input):
        output = self.maxpool1(input)
        return output
demo1 = Demo1()

writer = SummaryWriter("logs_maxpool")
step=0
for data in dataloader:
    imgs,targets = data
    writer.add_images("input",imgs,step)
    output = demo1(input)  # 池化不会改变通道数
    writer.add_images("output",imgs,step)
    step=step+1

writer.close()
output = demo1(input)
print(output)
```
