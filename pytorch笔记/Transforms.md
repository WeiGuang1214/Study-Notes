主要对图片进行一些变换，

```python
from torchvision import transform
```

transforms.py 相当于一个工具箱

<img src="C:\Users\微光\AppData\Roaming\Typora\typora-user-images\image-20240804180550557.png" alt="image-20240804180550557" style="zoom: 50%;" />

### tensor数据类型->张量

通过transform.ToTensor去解决两个问题

1、transform如何使用    2、为什么需要Tensor数据类型

```python
img_path = ""
img = Image.open(img_path)
print(img)

tensor_trains = transforms.ToTensor()    //先创建一个工具的对象
tensor_img = tensor_trains(img)      //调用方法，把img转换成张量表示
print(tensor_img)
//这部分内容可以打印出图片的张量形式
```

利用图片的tensor形式在网页显示图片

```python
img_path = ""
img = Image.open(img_path)

writer = SummaryWriter("logs")

tensor_trains = transforms.ToTensor()    //先创建一个工具的对象
tensor_img = tensor_trains(img)      //调用方法，把img转换成张量表示

writer.add_image("Tensor_img",tensor_img)
//这部分内容可以把图片的tensor形式在网页显示图片
```

## 常见的ransforms

### 1、pytorch中的call，内置__call__方法可以直接传参数调用call方法

```py
clss Person:
    def __call__(self,name):
    	print("__call__"+"Hello"+name)
    def hello(self,name):
    	print("hello"+name)
person = Person()
person("zhangsan")
person.hellp("lisa")

//结果：
__call__hello zhangsan
Hellolisa
```

### 2、pytorch中的toTensor，转化为张量型的图片对象，可以放入tensorboard中展示，日志文件

```python
from PIL import Image
from tochvision import transforms
from torch.utils.tensorboard import SummaryWriter
from torchvision import transforms

writer = SummaryWriter("logs")
img = Image.open("images/pytorch.png")
print(img)

trans_totensor = transforms.ToTensor()
img_tensor = trans_totensor(img)
writer.add_image("ToTensor",img_tensor)
writer.close()
```

### 3、pytorch中的 Normalize 归一化处理，将一列数据变化到某个固定区间比如[0, 1]，可以设置变化的均值和方差

```py
from PIL import Image
from tochvision import transforms
from torch.utils.tensorboard import SummaryWriter
from torchvision import transforms

writer = SummaryWriter("logs")
img = Image.open("images/pytorch.png")
print(img)

trans_totensor = transforms.ToTensor()
img_tensor = trans_totensor(img)
writer.add_image("ToTensor",img_tensor)

#normalize
print(img_tensor[0][0][0])
trans_norm = transforms.Normalize([0.5,0.5,0.5],[0.5,0.5,0.5])//图片是三通道数据，设置mean和std
img_nrom = trans_nrom(img_tensor)
print(img_nrom[0][0][0])
writer.add_image("Normalize",img_nrom)
writer.close()
```

### 4、pytorch中的 Resize，，将一列数据变化到某个固定区间比如[0, 1]，可以设置变化的均值和方差
