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

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727096815121.jpg)

```python
writer = SummaryWriter("logs")
for i in range(100):
	writer.add_scalar("y=x",i,i)

writer.close()
```

打开事件文件：

tensorboard --logdir=事件文件所在的文件夹名

--port = 6007可以改变端口

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727097022351.jpg)



#### writer.add_image()

利用OpenCV读取numpy型图片

![](https://github.com/WeiGuang1214/Study-Notes/blob/master/images/1727098097209.jpg)

```python
writer = SummaryWriter("logs")
image_path = "data/train/ants_image/0013035.jpg"
img_PIL = Image.open(image_path)
img_array = np.array(img_PIL)
print(img_array.shape)

writer.add_image("test",image_array,1,dataformats='HWC')
```

