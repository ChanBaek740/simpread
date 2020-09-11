\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/wojianxin/p/12519498.html)

**写文章不易，如果您觉得此文对您有所帮助，请帮忙点赞、评论、收藏，感谢您！**

**一. 仿射变换介绍：**

        请参考：图解图像仿射变换：[https://www.cnblogs.com/wojianxin/p/12518393.html](https://www.cnblogs.com/wojianxin/p/12518393.html)

**二. 仿射变换 公式：**

![](https://upload-images.jianshu.io/upload_images/17221499-6a39f26770da6e55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
仿射变换过程，(x,y) 表示原图像中的坐标，(x',y') 表示目标图像的坐标 ↑  

**三. 仿射变换——图像平移 算法：**

![](https://upload-images.jianshu.io/upload_images/17221499-20952eb6c99b5d9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
仿射变换—图像平移算法，其中 tx 为在横轴上移动的距离，ty 为在纵轴上移动的距离 ↑  

**四. python 实现仿射变换——图像平移**

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

```
1 import cv2
 2 import numpy as np
 4 # 图像仿射变换->图像平移 
 5 def affine(img, a, b, c, d, tx, ty):
 6     H, W, C = img.shape
 8     # temporary image
 9     tem = img.copy()
10     img = np.zeros((H+2, W+2, C), dtype=np.float32)
11     img\[1:H+1, 1:W+1\] = tem
13     # get new image shape
14     H\_new = np.round(H \* d).astype(np.int)
15     W\_new = np.round(W \* a).astype(np.int)
16     out = np.zeros((H\_new+1, W\_new+1, C), dtype=np.float32)
18     # get position of new image
19     x\_new = np.tile(np.arange(W\_new), (H\_new, 1))
20     y\_new = np.arange(H\_new).repeat(W\_new).reshape(H\_new, -1)
22     # get position of original image by affine
23     adbc = a \* d - b \* c
24     x = np.round((d \* x\_new  - b \* y\_new) / adbc).astype(np.int) - tx + 1
25     y = np.round((-c \* x\_new + a \* y\_new) / adbc).astype(np.int) - ty + 1
27     # 避免目标图像对应的原图像中的坐标溢出
28     x = np.minimum(np.maximum(x, 0), W+1).astype(np.int)
29     y = np.minimum(np.maximum(y, 0), H+1).astype(np.int)
31     # assgin pixcel to new image
32     out\[y\_new, x\_new\] = img\[y, x\]
34     out = out\[:H\_new, :W\_new\]
35     out = out.astype(np.uint8)
37     return out
40 # Read image
41 image1 = cv2.imread("../paojie.jpg").astype(np.float32)
43 # Affine : 平移，tx(W向)：向右30；ty(H向)：向上100
44 out = affine(image1, a=1, b=0, c=0, d=1, tx=30, ty=-100)
46 # Save result
47 cv2.imshow("result", out)
48 cv2.imwrite("out.jpg", out)
49 cv2.waitKey(0)
50 cv2.destroyAllWindows()
```

[![](http://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

**五. 代码实现过程中遇到的问题：**   

　① 原图像进行仿射变换时，原图像中的坐标可能超出了目标图像的边界，需要对原图像坐标进行截断处理。如何做呢？首先，计算目标图像坐标对应的原图像坐标，算法如下：

![](https://upload-images.jianshu.io/upload_images/17221499-18e78266eaf6aa6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
目标图像坐标反向求解原图像坐标公式 ↑  

        以下代码实现了该逆变换：

        # get position of original image by affine

        adbc = a \* d - b \* c

        x = np.round((d \* x\_new  - b \* y\_new) / adbc).astype(np.int) - tx + 1

        y = np.round((-c \* x\_new + a \* y\_new) / adbc).astype(np.int) - ty + 1

    然后对原图像坐标中溢出的坐标进行截断处理（取边界值），下面代码提供了这个功能：

        x = np.minimum(np.maximum(x, 0), W+1).astype(np.int)

        y = np.minimum(np.maximum(y, 0), H+1).astype(np.int)

        原图像范围：长 W，高 H

    ② 难点解答：

        x\_new = np.tile(np.arange(W\_new), (H\_new, 1))

        y\_new = np.arange(H\_new).repeat(W\_new).reshape(H\_new, -1)

![](https://upload-images.jianshu.io/upload_images/17221499-64dc484c595ee3a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
x\_new 矩阵横向长 W\_new，纵向长 H\_new ↑    

![](https://upload-images.jianshu.io/upload_images/17221499-1d70814cd008565c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
y\_new 矩阵横向长 W\_new，纵向长 H\_new ↑    

        造这两个矩阵是为了这一步：    

        # assgin pixcel to new image

        out\[y\_new, x\_new\] = img\[y, x\]

**六. 实验结果：**

![](https://upload-images.jianshu.io/upload_images/17221499-edadc62080fed278.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
原图 ↑  

![](https://upload-images.jianshu.io/upload_images/17221499-2e979afb0003b482.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
仿射变换—图像平移后结果 (向右 30 像素，向上 100 像素) ↑  

**七. 参考内容：**

    ①  [https://www.jianshu.com/p/1cfb3fac3798](https://www.jianshu.com/p/1cfb3fac3798)

**八. 版权声明：**

    未经作者允许，请勿随意转载抄袭，抄袭情节严重者，作者将考虑追究其法律责任，创作不易，感谢您的理解和配合！