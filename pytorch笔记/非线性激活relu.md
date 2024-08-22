### 1、relu激活值函数，会处理计算结果，

### nn.relu：input＞0会保留原始值，＜0的时候会取0，输入是N，batch_size，但是输出形状不限制

### nn.sigmoid()：输入是batch_size，后续形状不限制

例子：

```python
import torch
from torch import nn
from torch.nn import MaxPool2d
from torch.nn import sigmoid
from torch.utils.tensorboard import SummaryWriter

dataset = torchvision.datasets.CIFAR10("../data",train=False,download=True,
                                       			           
                                  transform=torchvision.transforms.ToTensor())

dataloader = DataLoader(dataset,batch_size=64)

input = torch.tensor([[1,-0.5],
                      [-1,3]])
input = torch.reshape(input,(-1,1,2,2))
print(input.shape)

class Demo1(nn.module):
    def __init__(self):
        super(Demo1,self).__init__()
        self.relu1 = ReLU(input,inplace=False)  # True 指定会不会把原来的结果进行替换
        self.sigmoid1 = Sigmoid()
        
    def forward(self,input):
        output = self.sigmoid1(input)
        return output
demo1 = Demo1()

writer = SummaryWriter("logs_maxpool")
step=0
for data in dataloader:
    imgs,targets = data
    writer.add_images("input",imgs,global_step=step)
    output = demo1(input)  # 池化不会改变通道数
    writer.add_images("output",imgs,global_step=step)
    step +=1

writer.close()
output = demo1(input)
print(output)
```
