### 1、nn.module，基本的一个神经网络框架

```python
import torch.nn as nn    #导入nn包
import torch. nn . functional as F

class Model (nn. Module) :#继承nn类
	def__ init__ (self): # 初始化
		super(Model, self).__ init . () # 调用父类的初始化函数
		self.conv1 = nn. Conv2d(1, 20， 5)
		self.conv2 = nn. Conv2d(20，20，5)
	def forward(self, x): # 前向传播
		x = F.relu(self. conv1(x)) # 一层卷积 激活
		return F . relu(self.conv2(x)) 	# 二层卷积 激活

```

![image-20240821132446519](C:\Users\微光\AppData\Roaming\Typora\typora-user-images\image-20240821132446519.png)

### 一个简单的例子

```py
import torch.nn as nn

class Demo1(nn.Module):
    def __init__(self):
        super().__init__()
    def forward(self,input):
        output = input +1
        return output
    
demo1 = Demo1() # 创建一个网络
x = torch.tensor(1,0) # 创建一个输入，张量类型的x
output = demo1(x)
print(output)
```

### 卷积

```python
import torch
import torch.nn.functional as F
input = torch.tensor([[1,2,0,3,1],
                      [0,1,2,3,1],
                      [1,2,1,0,0],
                      [5,2,3,1,1],
                      [2,1,0,1,1]])
kernel = torch.tensor([[1,2,1],
                       [0,1,0],
                       [2,1,0]])
input =  torch.reshape(input,(1,1,5,5)) # reshape可以变换尺寸
kernel = torch.reshape(kernel,(1,1,3,3)) #  1 个批次维度，1 个通道维度,3*3的张量

# stride是步长，指卷积计算移动的步长
# padding是填充，在矩阵四周填充padding()内的数值

output3 = F.conv2d(input,kernel,stride=1,padding=1)

# print(input.shape)
# print(kernel.shape)

#卷积
output = F.conv2d(input,kernel,stride=1)
print(output)
```

## 卷积层

![image-20240821171948214](C:\Users\微光\AppData\Roaming\Typora\typora-user-images\image-20240821171948214.png)

channel指的是通道数，多一个通道数相当于多一个卷积核进行计算，最终的计算结果有几个通道数就有几个卷积结果

```py
import torch
import torchvision

dataset = torchvision.datasets.CIFAR10("路径",train=False,transform=torchvision.transforms.Totensor())
dataloader = DataLoader(dataset,batch_size=64)

class Demo1(nn.module):
    def __init__(self):
        super(demo1,self).__init__()
        slef.conv1 = Conv2d(in_channel=3,out_channel=6,kernel_size=3,stride=1,padding=0)
    def forward(self,x):
        x = self.conv1(x)
        return x
demo1 = Demo1()
print(demo1)

writer = SummaryWriter("../logs")
step=0
for data in dataloader:
    imgs,targets = data
    output = Demo(imgs)
    print(imgs.shape)
    print(output.shape)
    writer.add_images("input",imgs,step)
    
    output = torch.reshape(output,(-1,3,30,30)) # -1是默认设置批次
    writer.add_images("output",output,step) # 因为add_images的默认3通道，所以要修改output的通道数
    step=step+1
 writer.close()
```

