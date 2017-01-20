# 1.数据表示

本章介绍ITK中表示数据的基本的类。最常用的类为`itk::Image` 、`itk::PointSet` 和`itk::PointSet`.

## 1.1. Image

`itk::Image `类遵循了泛型编程的思想，类型与算法分离。ITK支持任意类型的像素和空间维度。

### 1.1.1. 创建一个Image

本部分的源代码为`Image1.cxx` .

例程演示了怎么样去手动创建一个` itk::Image`类。下面是实例化、声明和创建一个Image类的最小的代码。

首先，头文件必须要包含。

```c++
#include "itkImage.h"
```

然后我们必须觉得用什么类型表示像素，以及图像的维度。当这两个参数确定后，我们可以就可以实例化`Image`类。现在我们创建一个3D、无符号短字符数据类型的图像。

```c++
typedef itk::Image< unsigned short, 3 > ImageType;
```

然后就可以调用`New()`操作创建图像，将其分配给`itk::SmartPointer`.

```c++
ImageType::Pointer image = ImageType::New();
```

在ITK中，图像以一个或多个区域的组合形式存在。一个区域是图像的子集，有可能是系统中其他类处理的图像的一部分。最常见的区域是`LargestPossibleRegion`， 其定义了一个完整的图像。另外一种重要的区域是`BufferedRegion`, 是内存中图像的一部分，以及`RequestedRegion`，是滤波器或其他类要求的一部分。

在ITK中，手动创建一幅图像需要对图像进行实例化，并将其与区域描述结合起来。

一个区域由两个类来定义: `itk::Index`和`itk::Size`. 区域的原点由`Index`来定义。区域的延伸，或者大小，由`Size`来定义。当一幅图像被创建，用户有责任定义图像的`size`和图像的起始位置的`Insex`。这两个参数使处理选择的区域变得可能。
`Index`由一个n维数组表示，每一个元素表示图像的最初像素值。
```c++
ImageType::IndexType start;
start[0] = 0; // first index on X
start[1] = 0; // first index on Y
start[2] = 0; // first index on Z
```
区域大小由一个相同维数的数组表示（利用`itk::Size`类）.数组中的元素是无符号整型，表示图像像素在各个方向上的延伸。
```c++
ImageType::SizeType size;
size[0] = 200; // size along X
size[1] = 200; // size along Y
size[2] = 200; // size along Z
```
当定义了起始`index`和图像`size`, 就可以创建一个`itk::ImageRegion`对象
```c++
ImageType::RegionType region;
region.SetSize( size );
region.SetIndex( start );
```
最终，这个区域传递给`Image`类来定义延伸和原点。`SetRegions`方法同时设置了`LargestPossibleRegion`，`BufferedRegion`, 和`RequestedRegion`. 注意：没有进行任何操作给图像像素数据分配内存。调用`Allocate()`方法来分配内存。`Allocate`不需要任何变量，直到直到分配了足够的内存。
```c++
image->SetRegions( region );
image->Allocate();
```
实际上很少直接地去分配和初始化一幅图像。通常图像从一个源文件读取。下面的例子说明如何从一个文件读取图像。
### 1.1.2. 从文件读取图像
源代码为`Image2.cxx`.
首要任务是包含头`itk::ImageFileReader`类的头文件。
```c++
#include "itkImageFileReader.h"
```
Then, the image type should be defined by specifying the type used to represent pixels and the dimensions of the image.
然后，图像类型根据指定表示像素和维数的类型来定义图像。
```c++
typedef unsigned char PixelType;
const unsigned int Dimension = 3;
typedef itk::Image< PixelType, Dimension > ImageType;
```
利用图像类型，可以来实例化一个图像读取类。图像类型被用作模板参数，来表示加载到内存中的数据。这个类型不需要和文件中存储的类型完全一致。使用了一个基于C-style的一个转换器，用来表示硬盘上的数据的类型应该精确的表示出来。用户不需要对像素数据做任何转换，除了将文件的像素类型转换为`ImageFileReader`的像素类型。下面展示`ImageFileReader`类型的一个典型实例。

```c++
typedef itk::ImageFileReader< ImageType > ReaderType;
```
现在`reader`类型可以用来创建一个`reader`对象。一个`itk::SmartPointer`用来接收新创建的`ewader`的引用。调用`New()`方法创建一个图像`reader`的实例。
```c++
ReaderType::Pointer reader = ReaderType::New();
```
最小信息需求是加载到内存中的文件名，可以通过`SetFileName()`方法来实现。文件格式由文件扩展名来推断得知。用户同样需要使用`itk::ImageIOBase`类明确地指明数据格式。
```c++
const char * filename = argv[1];
reader->SetFileName( filename );
```
`Reader`被称为管道流对象，对应于管道更新需求，和初始化管道流。管道更新机制保证用`reader`仅在得到了一个数据请求，但是还没有读取数据时执行。在当前的例子中，我们明确地调用`Update()`方法，因为`reader`的输出没有连接到其他的滤波器。在正常的应用中，`reader`的输出被连接到一个滤波器的输入，滤波器上的更新调用引发一个`reader`的一个更新。下面的例子说明`reader`上的一个调用明确地更新。

```c++
reader->Update();
```

使用`GetOutput()`方法访问新读取的图像。这个方法也可以在更新需求之前被调用。直到`reader`实际执行之前，即使图像是空的，这个对图像的引用仍然是有效的。

```c++
ImageType::Pointer image = reader->GetOutput();
```

在`reader`执行之前，任何的访问图像将获得一个没有像素数据的图片。正如一个图像没有被合适的初始化将产生一个程序崩溃一样。

### 1.1.3. 访问像素数据

源代码为`Image3.cxx`.

This example illustrates the use of the SetPixel() and GetPixel() methods. These two methods provide direct access to the pixel data contained in the image. Note that these two methods are relatively slow and should not be used in situations where high-performance access is required.
Image iterators are the appropriatemechanism to efficiently access image pixel data. (See Chapter 6 on page 141 for information about image iterators.)

这个例子展示了`SetPixel() `和`GetPixel()`方法的使用。





