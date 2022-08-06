---
title: 在 WPF Image 控件中显示 Bitmap 数据
tags:
  - WPF
  - .Net
  - 流媒体
categories: 
  - 400-编程
  - .Net
date: 2017-03-05 00:19:06
---
## 方案一： 使用 MemoryStream

先将 Bitmap 储存成 MemoryStream ，然后指定给 BitmapImage 。

```csharp
    void UpdateImageByMemoryStream()
    {

        using (MemoryStream memory = new MemoryStream())
        {

            int w = 0, h = 0;

            MyDll.GetBitmapSize(ref w, ref h);
            System.Drawing.Bitmap bitmap;

            bitmap = new System.Drawing.Bitmap(w, h, System.Drawing.Imaging.PixelFormat.Format32bppArgb);


            System.Drawing.Rectangle rect = new System.Drawing.Rectangle(0, 0, bitmap.Width, bitmap.Height);

            while (true)
            {
                System.Drawing.Imaging.BitmapData bmpData =
                bitmap.LockBits(rect, System.Drawing.Imaging.ImageLockMode.ReadWrite, bitmap.PixelFormat);

                // Get the address of the first line.
                IntPtr ptr = bmpData.Scan0;

                var status = MyDll.GetBitmapData(ptr) >= 0 ? true : false;

                // Unlock the bits.
                bitmap.UnlockBits(bmpData);

                if (status)
                {
				    bitmap.Save(memory, System.Drawing.Imaging.ImageFormat.Bmp);
                    UpdateUI(() =>
                    {
                        memory.Position = 0;
                        BitmapImage bitmapimage = new BitmapImage();
                        bitmapimage.BeginInit();
                        bitmapimage.StreamSource = memory;
                        bitmapimage.CacheOption = BitmapCacheOption.OnLoad;
                        bitmapimage.EndInit();

                        imgPreview.Source = bitmapimage;
                    });
                }
            }
        }
    }
```

## 方案二： 调用 Imaging.CreateBitmapSourceFromHBitmap


```csharp
    void UpdateImageByBitmap()
    {


        int w = 0, h = 0;

        MyDll.GetBitmapSize(ref w, ref h);
        var bitmap = new System.Drawing.Bitmap(w, h, System.Drawing.Imaging.PixelFormat.Format32bppArgb);

        System.Drawing.Rectangle rect = new System.Drawing.Rectangle(0, 0, bitmap.Width, bitmap.Height);
        while (true)
        {
            System.Drawing.Imaging.BitmapData bmpData =
                bitmap.LockBits(rect, System.Drawing.Imaging.ImageLockMode.ReadWrite, bitmap.PixelFormat);

            // Get the address of the first line.
            IntPtr ptr = bmpData.Scan0;

            var status = MyDll.GetBitmapData(ptr) >= 0 ? true : false;

            // Unlock the bits.
            bitmap.UnlockBits(bmpData);

            if (status)
                UpdateUI(() =>
                {
                    imgPreview.Source = BitmapHelper.BitmapToBitmapSource(bitmap);
                });

        }
    }
```

```csharp
    public class BitmapHelper
    {
        [DllImport("gdi32.dll")]
        private static extern bool DeleteObject(IntPtr hObject);
        public static BitmapSource BitmapToBitmapSource(System.Drawing.Bitmap bitmap)
        {
            IntPtr ptr = bitmap.GetHbitmap();
            BitmapSource result = Imaging.CreateBitmapSourceFromHBitmap(
                ptr, IntPtr.Zero, Int32Rect.Empty, 
                BitmapSizeOptions.FromEmptyOptions());
            //release resource
            DeleteObject(ptr);

            return result;
        }
    }
```


## 方案三： 使用 WriteableBitmap

注：这个方法会长时间占用 UI 线程。

```csharp
    void UpdateImageByWriteBitmap()
    {
        int w = 0, h = 0;

        MyDll.GetBitmapSize(ref w, ref h);
        var rect = new System.Windows.Int32Rect(0, 0, w, h);

        UpdateUI(() =>
        {
            imgPreview.Source =
                writeBitmap = new WriteableBitmap(rect.Width, rect.Height, 96, 96, PixelFormats.Bgra32, null);
        });
        while (true)
        {
            UpdateUI(() =>
            {
                writeBitmap.Lock();
                if (MyDll.GetBitmapData(writeBitmap.BackBuffer) >= 0)
                    writeBitmap.AddDirtyRect(rect);
                writeBitmap.Unlock();
            });
        }
    }
```

<!-- more -->

## MainWindow.xaml.cs

```csharp
using System;
using System.IO;
using System.Runtime.InteropServices;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Interop;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace WpfDemo
{
	public partial class MainWindow : Window
	{
		Image imgPreview;
		WriteableBitmap writeBitmap;

		public MainWindow()
		{
			InitializeComponent();
		}

		void UpdateUI(Action x)
		{
			if (imgPreview.Dispatcher.CheckAccess())
			{
				x.Invoke();
			}
			else
			{
				imgPreview.Dispatcher.Invoke(x);
			}
		}

		// ...
	}
}
```

## 调用 DLL 获取图片数据

```csharp
    public class MyDll
    {
        // 获取图片大小
        [DllImport("xxx.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.Cdecl)]
        public static extern void GetBitmapSize(ref int width, ref int height);

        // 获取图片数据，返回值大于 0 代表成功
        [DllImport("xxx.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.Cdecl)]
        public static extern int GetBitmapData(IntPtr desFrameData);
    }
```