# 修改pip install 默认安装依赖路径

1. 激活一个虚拟环境，比如Conda activate zhy

   ![fc1aa4f6a4cf4ef2224736b58f7533be](/home/zhy/文档/xwechat_files/wxid_8unpebk656vf12_b2d1/temp/2024-12/RWTemp/9e20f478899dc29eb19741386f9343c8/fc1aa4f6a4cf4ef2224736b58f7533be.jpg)

可以注意到，USER_BASE是默认的.local地址，USER_SITE不可用，所以这时候pip install的包会安装到系统环境下

2. 在虚拟环境下:

   `vim /home/sztu/anaconda3/envs/zhy/lib/python3.10/site.py`

可以看到函数开头的USER_SITE和USER_BASE是none

![225b411b0cd8d3a13097a5c3ad9e7828](/home/zhy/文档/xwechat_files/wxid_8unpebk656vf12_b2d1/temp/2024-12/RWTemp/9e20f478899dc29eb19741386f9343c8/225b411b0cd8d3a13097a5c3ad9e7828.jpg)

3.修改配置

`USER_SITE = '/home/sztu/anaconda3/envs/zhy/lib/python3.10/site-packages'`

`USER_BASE= '/home/sztu/anaconda3/envs/'`

**ENABLE_USER_SITE = None 改为 False，否则修改当前虚拟环境下的USER_SITE会改变envs下所有虚拟环境的配置**
![image-20241218162628560](/home/zhy/.config/Typora/typora-user-images/image-20241218162628560.png)
![image-20241218162649975](/home/zhy/.config/Typora/typora-user-images/image-20241218162649975.png)

esc, wq保存退出
此时在虚拟环境执行 python -m site

USER_BASE和USER_SITE都存在，而且不同虚拟环境之间已经做了隔离（如zhy和qynpytorch）

虚拟环境下which pip

![image-20241218162815208](/home/zhy/.config/Typora/typora-user-images/image-20241218162815208.png)
![image-20241218162826152](/home/zhy/.config/Typora/typora-user-images/image-20241218162826152.png)

4. 如果还有问题，当前路径下conda install pip即可
5. **终极解决方案，.local下的pip文件权限最高，默认使用它，base环境下pip unintall pip，删除pip，然后去各个虚拟环境重新conda install pip**
6. 怕影响环境，但是不怕麻烦版：pip install 库 --target=虚拟环境路径\\lib\\site-packages
                                                         pip install -r requirements --target=虚拟环境路径\\lib\\site-packages   

