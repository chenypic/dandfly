@(马克飞象)

# Gabor

在这个例子中，我们将看到怎样基于Gabor滤波分类特征。Gabor滤波的频率和方向与人的视觉时间相似。

图像用不同的Gabor滤波核进行处理。滤波图像的均值和方差用于特征分类，为了简单起见，采用了最小二乘误差。

```python
from __future__ import print_function

import matplotlib.pyplot as plt
import numpy as np
from scipy import ndimage as ndi

from skimage import data
from skimage.util import img_as_float
from skimage.filters import gabor_kernel
```

```python
def compute_feats(image, kernels):
    feats = np.zeros((len(kernels), 2), dtype=np.double)
    for k, kernel in enumerate(kernels):
        filtered = ndi.convolve(image, kernel, mode='wrap')
        feats[k, 0] = filtered.mean()
        feats[k, 1] = filtered.var()
    return feats
```

>  定义函数：`compute_feats(image, kernels)`,用于计算特征
>
>  首先定义一个`0`数组`feats`, 维数为`len(kernels) x 2`：行数为`kernels`的长度，列数为`2`
>
>  然后遍历`kernels`
>
>  将图像与`kernel`进行卷积计算。
>
>  返回二维数组`feats`


```python

def match(feats, ref_feats):
    min_error = np.inf
    min_i = None
    for i in range(ref_feats.shape[0]):
        error = np.sum((feats - ref_feats[i, :])**2)
        if error < min_error:`
            min_error = error
            min_i = i
    return min_i
```

> 匹配特征：输入为`feats`和`ref_feats`。


```python

# prepare filter bank kernels
kernels = []
for theta in range(4):
    theta = theta / 4. * np.pi
    for sigma in (1, 3):
        for frequency in (0.05, 0.25):
            kernel = np.real(gabor_kernel(frequency, theta=theta,
                                          sigma_x=sigma, sigma_y=sigma))
            kernels.append(kernel)
```

> 准备滤波核：
>
> 创建列表`kernels`
>
> 角度为[0,1,3,4], sigma位(1,3), 频率为(0.05, 0.25), 共计16个滤波核。
>
> `kernel`是一个二维数组。
>
> 将16个滤波核的`kernels`增加到列表`kernels`中。


```python
shrink = (slice(0, None, 3), slice(0, None, 3))
brick = img_as_float(data.load('brick.png'))[shrink]
grass = img_as_float(data.load('grass.png'))[shrink]
wall = img_as_float(data.load('rough-wall.png'))[shrink]
image_names = ('brick', 'grass', 'wall')
images = (brick, grass, wall)
```




```python
# prepare reference features,计算3幅图像的特征
ref_feats = np.zeros((3, len(kernels), 2), dtype=np.double)
ref_feats[0, :, :] = compute_feats(brick, kernels)
ref_feats[1, :, :] = compute_feats(grass, kernels)
ref_feats[2, :, :] = compute_feats(wall, kernels)

print('Rotated images matched against references using Gabor filter banks:')

print('original: brick, rotated: 30deg, match result: ', end='')
feats = compute_feats(ndi.rotate(brick, angle=190, reshape=False), kernels)
print(image_names[match(feats, ref_feats)])

print('original: brick, rotated: 70deg, match result: ', end='')
feats = compute_feats(ndi.rotate(brick, angle=70, reshape=False), kernels)
print(image_names[match(feats, ref_feats)])

print('original: grass, rotated: 145deg, match result: ', end='')
feats = compute_feats(ndi.rotate(grass, angle=145, reshape=False), kernels)
print(image_names[match(feats, ref_feats)])
```





```python
def power(image, kernel):
    # Normalize images for better comparison.
    image = (image - image.mean()) / image.std()
    return np.sqrt(ndi.convolve(image, np.real(kernel), mode='wrap')**2 +
                   ndi.convolve(image, np.imag(kernel), mode='wrap')**2)
```

> 对图像进行正则化处理

```python
# Plot a selection of the filter bank kernels and their responses.
results = []
kernel_params = []
for theta in (0, 1):
    theta = theta / 4. * np.pi
    for frequency in (0.1, 0.4):
        kernel = gabor_kernel(frequency, theta=theta)
        params = 'theta=%d,\nfrequency=%.2f' % (theta * 180 / np.pi, frequency)
        kernel_params.append(params)
        # Save kernel and the power image for each image
        results.append((kernel, [power(img, kernel) for img in images]))
# 2个频率2个方向，4个kernel

fig, axes = plt.subplots(nrows=5, ncols=4, figsize=(5, 6))
plt.gray()
#创建图，5行4例如，像素为（5,6）*80

fig.suptitle('Image responses for Gabor filter kernels', fontsize=12) #图的标题

axes[0][0].axis('off')

# Plot original images，绘制原图像，3幅
for label, img, ax in zip(image_names, images, axes[0][1:]):
    ax.imshow(img)
    ax.set_title(label, fontsize=9)
    ax.axis('off')


for label, (kernel, powers), ax_row in zip(kernel_params, results, axes[1:]):#从第2行开始绘制
    # Plot Gabor kernel
    ax = ax_row[0]
    ax.imshow(np.real(kernel), interpolation='nearest')#interpolation可以不要
    ax.set_ylabel(label, fontsize=7)
    ax.set_xticks([])
    ax.set_yticks([])

    # Plot Gabor responses with the contrast normalized for each filter
    vmin = np.min(powers)
    vmax = np.max(powers)
    for patch, ax in zip(powers, ax_row[1:]):
        ax.imshow(patch, vmin=vmin, vmax=vmax)
        ax.axis('off')

plt.show()
```

![](http://scikit-image.org/docs/dev/_images/sphx_glr_plot_gabor_001.png)