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

### 4、pytorch中的 Resize，调整tensor的尺寸

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

#Resize
print(img.size)
trans_resize = transforms.Resize((512,512))#设置变换的大小，只输入一个数的时候改变长和宽中最长的那个

img_resize = trans_resize(img)#对img对象进行变换
img_resize = trans_totensor(img_resize)#缩放后转换为tenser数据类型

print(img_resize)

writer.close()
```

### 5、pytorch中的 Compose，参数是一个列表，（[transforms参数1，transforms参数2]），会连续完成一串的操作组合,输入输出的类型要匹配

```py
# Compose - resize - 2
trans_ resize_2 = transforms.Resize(512)
trans_ compose = transforms.Compose([trans_resize_2，trans_ totensor])
img_ resize_2 = trans_compose( img)
writer .add_ image( "Resize", img_resize. 2, 1)
```

### 剩余的函数需要查看输入的参数以及输出，就可以，主要是对张量进行操作
