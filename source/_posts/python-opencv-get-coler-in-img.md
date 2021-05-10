---
title: python+opencv获取图片中指定颜色的部分
date: 2021-05-10 19:35:37
categories:
  - python
tags:
  - python
  - opencv
  - 图像识别
typora-root-url: ..
---

1. 将图片的色彩空间转为HSV色彩空间
2. 通过比照HSV的参考表，进行获取要提取颜色的相应范围
3. 使用inRange函数进行提取
4. 使用imShow显示

示例：

```python
import cv2
import numpy as np
src = cv2.imread("D:\\myCode\\picture\\cards.png")
cv2.namedWindow("input", cv2.WINDOW_AUTOSIZE)
cv2.imshow("input", src)
"""
提取图中的红色部分
"""
hsv = cv2.cvtColor(src, cv2.COLOR_BGR2HSV)
low_hsv = np.array([0,43,46])
high_hsv = np.array([10,255,255])
mask = cv2.inRange(hsv,lowerb=low_hsv,upperb=high_hsv)
cv2.imshow("test",mask)
cv2.waitKey(0)
cv2.destroyAllWindows()
# 这个例子为提取红色部分。通过观察下表，可以看到红色的hmin，smin，vmin分别为0，43，46； hmax，smax，vmax分别为10，255，255.
```

![Image](/images/python-opencv-get-coler-in-img.png)

