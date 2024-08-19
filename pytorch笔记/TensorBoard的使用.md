## 1、Tensorboard的使用

通过loss查看训练的过程是不是正常

```python
from torch.utils.tensorboard import SummaryWriter
```

ctrl按住包可以检查方法的作用

### add_scalar()

```python
writer = SummaryWriter("logs")
writer.add_scalar()

writer.close()
```

添加一个标量，标题、数值、步长

x轴global_step、y轴scalar_value

<img src="C:\Users\微光\AppData\Roaming\Typora\typora-user-images\image-20240804165100081.png" alt="image-20240804165100081" style="zoom:67%;" />

```python
writer = SummaryWriter("logs")
for i in range(100):
	writer.add_scalar("y=x",i,i)

writer.close()
```

打开事件文件：

tensorboard --logdir=事件文件所在的文件夹名

--port = 6007可以改变端口

![image-20240804165708515](C:\Users\微光\AppData\Roaming\Typora\typora-user-images\image-20240804165708515.png)



#### writer.add_image()

利用OpenCV读取numpy型图片

<img src="C:\Users\微光\AppData\Roaming\Typora\typora-user-images\image-20231208171852548.png" alt="image-20231208171852548" style="zoom: 80%;" />

```python
writer = SummaryWriter("logs")
image_path = "data/train/ants_image/0013035.jpg"
img_PIL = Image.open(image_path)
img_array = np.array(img_PIL)
print(img_array.shape)

writer.add_image("test",image_array,1,dataformats='HWC')
```

