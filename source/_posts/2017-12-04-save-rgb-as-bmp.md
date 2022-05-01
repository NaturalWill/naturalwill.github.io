---
title: 如何将 RGB 数据保存为 BMP 图片？
date: 2017-12-04 14:25:07
tags:
  - 流媒体
  - 图像
categories: 
  - 400-编程
  - C/C++
---


直接上代码，一个函数搞定。

```C
#include <stdio.h>
#include <tchar.h>

void SaveAsBMP(char *filename, unsigned char* image, int width, int height, int bpp)
{
	BITMAPFILEHEADER bmpheader;
	BITMAPINFOHEADER bmpinfo;
	FILE *fp;

	if ((fp = fopen(filename, "wb+")) == NULL) {
		printf("open file failed!\n");
		return;
	}
	// 4 Byte aligned
	int stride = (((width * bpp) + 31) >> 5) << 2;  // (( BPP * Width ) / 32) * 4
	int img_data_size = stride * height;


	bmpheader.bfType = 0x4d42;
	bmpheader.bfReserved1 = 0;
	bmpheader.bfReserved2 = 0;
	bmpheader.bfOffBits = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER);
	bmpheader.bfSize = bmpheader.bfOffBits + img_data_size;

	bmpinfo.biSize = sizeof(BITMAPINFOHEADER);
	bmpinfo.biWidth = width;
	bmpinfo.biHeight = height;
	bmpinfo.biPlanes = 1;
	bmpinfo.biBitCount = bpp;
	bmpinfo.biCompression = BI_RGB;
	bmpinfo.biSizeImage = img_data_size; //  it can be 0 when biCompression is BI_RGB.
	bmpinfo.biXPelsPerMeter = 3780;  // 96 * 39.370079 ~= 3780
	bmpinfo.biYPelsPerMeter = 3780;
	bmpinfo.biClrUsed = 0;
	bmpinfo.biClrImportant = 0;

	fwrite(&bmpheader, sizeof(bmpheader), 1, fp);
	fwrite(&bmpinfo, sizeof(bmpinfo), 1, fp);
	
	unsigned char *buffer = new unsigned char[img_data_size];

	unsigned char* pImage = image;
	unsigned char* pDest = buffer;

	int pixel_w_size = width * bpp / 8;
	pImage += pixel_w_size*(height - 1);
	
	for (int i = 0; i < height; i++) {
		memcpy(pDest, pImage, pixel_w_size);
		pDest += stride;

		//fwrite(pImage, pixel_w_size, 1, fp);
		pImage -= pixel_w_size;
	}

	pImage = nullptr;

	fwrite(buffer, img_data_size, 1, fp);
	fclose(fp);

	delete[] buffer;
	buffer = nullptr;
}
```

<!-- more -->

## 拓展

BMP文件格式，又称为Bitmap（位图）或是DIB(Device-Independent Device，设备无关位图)，是Windows系统中广泛使用的图像文件格式。由于它可以不作任何变换地保存图像像素域的数据，因此成为我们取得RAW数据的重要来源。Windows的图形用户界面（graphical user interfaces）也在它的内建图像子系统GDI中对BMP格式提供了支持。

BMP文件的数据按照从文件头开始的先后顺序分为四个部分：
* bmp文件头(bmp file header)：提供文件的格式、大小等信息
* 位图信息头(bitmap information)：提供图像数据的尺寸、位平面数、压缩方式、颜色索引等信息
* 调色板(color palette)：可选，如使用索引来表示图像，调色板就是索引与其对应的颜色的映射表
* 位图数据(bitmap data)：就是图像数据啦^_^

| 块名称      |  对应Windows结构体定义  |  大小（Byte） |
| ---- | ---- | ---- |
| 文件信息头  |  BITMAPFILEHEADER    |  14 |
| 位图信息头  |  BITMAPINFOHEADER     |  40 |
| 调色版      |                      |  由颜色索引数决定 |
| RGB颜色阵列  |  BYTE*             |  由图像长宽尺寸决定 |


### BMP文件头（14字节）
```C
typedef struct tagBITMAPFILEHEADER {
  WORD  bfType;
  DWORD bfSize;
  WORD  bfReserved1;
  WORD  bfReserved2;
  DWORD bfOffBits;
} BITMAPFILEHEADER, *PBITMAPFILEHEADER;
```
* bfType
The file type; must be BM. 即 0x4d42 
* bfSize
The size, in bytes, of the bitmap file. 该位图文件的大小，用字节为单位
* bfReserved1
Reserved; must be zero.
* bfReserved2
Reserved; must be zero.
* bfOffBits
The offset, in bytes, from the beginning of the BITMAPFILEHEADER structure to the bitmap bits.
从文件头开始到实际的图象数据之间的字节的偏移量。这个参数是非常有用的，因为位图信息头和调色板的长度会根据不同情况而变化，所以你可以用这个偏移值迅速的从文件中读取到位数据。

### 位图信息头（40字节）

```C
typedef struct tagBITMAPINFOHEADER {
  DWORD biSize;
  LONG  biWidth;
  LONG  biHeight;
  WORD  biPlanes;
  WORD  biBitCount;
  DWORD biCompression;
  DWORD biSizeImage;
  LONG  biXPelsPerMeter;
  LONG  biYPelsPerMeter;
  DWORD biClrUsed;
  DWORD biClrImportant;
} BITMAPINFOHEADER, *PBITMAPINFOHEADER;
```

* biSize
The number of bytes required by the structure.
* biWidth
The width of the bitmap, in pixels.
If biCompression is *BI_JPEG* or *BI_PNG*, the biWidth member specifies the width of the decompressed JPEG or PNG image file, respectively.
* biHeight
The height of the bitmap, in pixels. If biHeight is positive, the bitmap is a bottom-up DIB and its origin is the lower-left corner. If biHeight is negative, the bitmap is a top-down DIB and its origin is the upper-left corner.
If biHeight is negative, indicating a top-down DIB, biCompression must be either BI_RGB or BI_BITFIELDS. Top-down DIBs cannot be compressed.
If biCompression is *BI_JPEG* or *BI_PNG*, the biHeight member specifies the height of the decompressed JPEG or PNG image file, respectively.
说明图象的高度，以象素为单位。注：这个值除了用于描述图像的高度之外，它还有另一个用处，就是指明该图像是倒向的位图，还是正向的位图。
如果该值是一个正数，说明图像是倒向的，如果该值是一个负数，则说明图像是正向的。大多数的BMP文件都是倒向的位图，也就是时，高度值是一个正数。
* biPlanes
The number of planes for the target device. This value must be set to 1.
为目标设备说明位面数，其值将总是被设为1。
* biBitCount
The number of bits-per-pixel. The biBitCount member of the BITMAPINFOHEADER structure determines the number of bits that define each pixel and the maximum number of colors in the bitmap. This member must be one of the following values.
说明比特数/象素，其值为1、4、8、16、24、或32。但是由于我们平时用到的图像绝大部分是24位和32位的。
* biCompression
The type of compression for a compressed bottom-up bitmap (top-down DIBs cannot be compressed). This member can be one of the following values.
* biSizeImage
The size, in bytes, of the image. This may be set to zero for BI_RGB bitmaps.
If biCompression is BI_JPEG or BI_PNG, biSizeImage indicates the size of the JPEG or PNG image buffer, respectively.
说明图象的大小，以字节为单位。当用BI_RGB格式时，可设置为0。
* biXPelsPerMeter
The horizontal resolution, in pixels-per-meter, of the target device for the bitmap. An application can use this value to select a bitmap from a resource group that best matches the characteristics of the current device.
说明水平分辨率，用象素/米表示。
* biYPelsPerMeter
The vertical resolution, in pixels-per-meter, of the target device for the bitmap.
* biClrUsed
The number of color indexes in the color table that are actually used by the bitmap. If this value is zero, the bitmap uses the maximum number of colors corresponding to the value of the biBitCount member for the compression mode specified by biCompression.
If biClrUsed is nonzero and the biBitCount member is less than 16, the biClrUsed member specifies the actual number of colors the graphics engine or device driver accesses. If biBitCount is 16 or greater, the biClrUsed member specifies the size of the color table used to optimize performance of the system color palettes. If biBitCount equals 16 or 32, the optimal color palette starts immediately following the three DWORD masks.
When the bitmap array immediately follows the BITMAPINFO structure, it is a packed bitmap. Packed bitmaps are referenced by a single pointer. Packed bitmaps require that the biClrUsed member must be either zero or the actual size of the color table.
说明位图实际使用的彩色表中的颜色索引数（设为0的话，则说明使用所有调色板项）。
* biClrImportant
The number of color indexes that are required for displaying the bitmap. If this value is zero, all colors are required.
说明对图象显示有重要影响的颜色索引的数目，如果是0，表示都重要。


### 位图数据

位图数据记录了位图的每一个像素值，记录顺序是在扫描行内是从左到右，扫描行之间是从下到上。

无论是磁盘上的位图文件还是内存中的位图图像，像素都由一组位（英语：bit）表示。

位图的一个像素值所占的字节数：
* 每像素占1位（色深为1位，1bpp）的格式支持2种不同颜色。像素值直接对应一个位的值，最左像素对应第一个字节的最高位。使用该位的值用来对色表的索引：为0表示色表中的第一项，为1表示色表中的第二项（即最后一项）。
* 每像素占2位（色深为2位，2bpp）的格式支持4种不同颜色。每个字节对应4个像素，最左像素为最高的两位（仅在Windows CE中有效）。需要使用像素值来对一张含有4个颜色值的色表进行索引。
* 每像素占4位（色深为4位，4bpp）的格式支持16种不同的颜色。每个字节对应2个像素，最左像素为最高的四位。需要使用像素值来对一张含有16个颜色值的色表进行索引。
* 每像素占8位（色深为8位，8bpp）的格式支持256种不同的颜色。每个字节对应1个像素。需要使用像素值来对一张含有256个颜色值的色表进行索引。
* 每像素占16位（色深为16位，16bpp）的格式支持65536种不同的颜色，每2个字节（byte）对应一个像素。该像素的不透明度（英语：alpha）、红、绿、蓝采样值即存储在该2个字节中。
* 每像素占24位（色深为24位，24bpp）的格式支持16777216种不同的颜色，每3个字节对应一个像素。按顺序分别为B,G,R。
* 每像素占32位（色深为32位，32bpp）的格式支持4294967296种不同的颜色，每4个字节对应一个像素。按顺序分别为B,G,R,A。

为了区分一个颜色值中的哪些位表示哪种采样值，DIB头给出了一套默认规则，同时也允许使用BITFIELDS将某组位指定为像素中的某个通道。

### 对齐规则

由于Windows在进行行扫描的时候最小的单位为4个字节，如果数据对齐满足这个值的话对于数据的获取速度等都是有很大的增益的。
因此，BMP图像顺应了这个要求，要求每行的数据的长度必须是4的倍数，如果不够需要进行比特填充（以0填充），这样可以达到按行的快速存取。
这时，位图数据区的大小就未必是图片宽×每像素字节数×图片高能表示的了，因为每行可能还需要进行比特填充。

填充后的每行的字节数为：

	RowSize = 4 * (( BPP * Width ) / 32)

其中BPP（Bits Per Pixel）为每像素的比特数。

在程序中，我们可以表示为：

	int stride = (((Width * BPP) + 31) >> 5) << 2;

注意： int 的除法。结果还是int，会舍掉小数点。所以，我们加上31，再除以32，就可以防止字节数变少。

这样，位图数据区的大小为：

	img_data_size = stride * Height;


## 相关资料

* [BMP文件格式详解](https://www.cnblogs.com/Matrix_Yao/archive/2009/12/02/1615295.html )
* [位图文件（BMP）格式分析以及程序实现](http://blog.csdn.net/yyfzy/article/details/785945 )
* [BMP file format](https://en.wikipedia.org/wiki/BMP_file_format )
* [BITMAPFILEHEADER structure][BITMAPFILEHEADER]
* [BITMAPINFOHEADER structure][BITMAPINFOHEADER]
* [BMP格式图像文件详析](http://www.thethirdmedia.com/pc/200407/20040722117029.shtm )
* [ffmpeg(7)：将h264编码的视频流保存为BMP或者JPEG图片](http://blog.csdn.net/oldmtn/article/details/46742555 )



[BITMAPFILEHEADER]: https://msdn.microsoft.com/query/dev15.query?appId=Dev15IDEF1&l=ZH-CN&k=k(WINGDI%2FBITMAPFILEHEADER);k(BITMAPFILEHEADER);k(DevLang-C%2B%2B);k(TargetOS-Windows)&rd=true
[BITMAPINFOHEADER]: https://msdn.microsoft.com/query/dev15.query?appId=Dev15IDEF1&l=ZH-CN&k=k(WINGDI%2FBITMAPINFOHEADER);k(BITMAPINFOHEADER);k(DevLang-C%2B%2B);k(TargetOS-Windows)&rd=true