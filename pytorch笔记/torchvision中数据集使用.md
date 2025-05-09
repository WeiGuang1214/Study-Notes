### 1、从torch中下载并且加载数据集,并且利用tensorboard可视化数据集

```python
import torchvision
dataset_transform = torchvision.transforms.Compose([
    torchvision.transforms.Totensor()
    
])
train_set = torchvision.datasets.CIFAR10(root="./dataset",train=True,download=True)
test_set = torchvision.datasets.CIFAR10(root="./dataset",train=False,download=True)
#通过print可以打印数据集内容
# print(test_set[0])
#print(test_set.calsses) #打印图片的类型
#img,target = test_set[0] #分别获得图片还有标签
#print(img)
#print(target)
#priny(test_set.classes[target])
#img.show()#查看图片

writer = SummaryWriter("p10")  #p10是文件夹名字，里面存的是tensorboard文档
for i in range(10):
    img,target =  test_set[i]
    writer.add_image("tets_set",img,i)
writer.close()

```

