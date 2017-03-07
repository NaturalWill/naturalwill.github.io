---
title: 在 WPF 中调用 SDL2 播放 RGB/YUV
date: 2017-03-07 23:40:50
tags:
  - ".Net"
---

## SDL简介

SDL（Simple DirectMedia Layer）是一套开放源代码的跨平台多媒体开发库，使用C语言写成。SDL提供了数种控制图像、声音、输出入的函数，让开发者只要用相同或是相似的代码就可以开发出跨多个平台（Linux、Windows、Mac OS X等）的应用软件。目前SDL多用于开发游戏、模拟器、媒体播放器等多媒体应用领域。

<!-- more -->

## SDL播放视频的流程

* 初始化
    * 初始化SDL
    * 创建窗口（Window）
    * 基于窗口创建渲染器（Render）
    * 创建纹理（Texture）
* 循环显示画面
    * 设置纹理的数据
    * 纹理复制给渲染目标
    * 显示
	

## C 语言部分

```C
// libMyPlayer.h
#pragma once

#define EXPORT_API __declspec(dllexport)

#include <stdio.h>
#include <memory.h>

#include <Windows.h>


extern "C"
{
    #include "sdl/SDL.h"
    FILE __iob_func[3] = { *stdin,*stdout,*stderr };  // 使用 VS2015 编译时，需加入这一句
}

extern "C" {
    EXPORT_API int SdlSetWin(HWND handler, int win_w, int win_h);
    EXPORT_API int SdlInit(int pix_w, int pix_h);
    EXPORT_API int SdlRender(unsigned char* buffer);
}
```


注意： 
使用 VS2015 编译时，需加入 `extern "C" FILE __iob_func[3] = { *stdin,*stdout,*stderr };` 这一句，同时在 `链接器->附加依赖项` 里加入 `legacy_stdio_definitions.lib`


```C
// libMyPlayer.c
#include "libMyPlayer.h"

//set '1' to choose a type of file to play
#define LOAD_BGRA    1
#define LOAD_RGB24   0
#define LOAD_BGR24   0
#define LOAD_YUV420P 0

//Bit per Pixel
#if LOAD_BGRA
const int bpp = 32;
#elif LOAD_RGB24|LOAD_BGR24
const int bpp = 24;
#elif LOAD_YUV420P
const int bpp = 12;
#endif

int screen_w = 500, screen_h = 500;
int pixel_w = 320, pixel_h = 180;

unsigned char * buffer = new unsigned char[pixel_w*pixel_h*bpp / 8];
//BPP=32
unsigned char * buffer_convert = new unsigned char[pixel_w*pixel_h * 4];

//Convert RGB24/BGR24 to RGB32/BGR32
//And change Endian if needed
void CONVERT_24to32(unsigned char *image_in, unsigned char *image_out, int w, int h) {
    for (int i = 0;i < h;i++)
        for (int j = 0;j < w;j++) {
            //Big Endian or Small Endian?
            //"ARGB" order:high bit -> low bit.
            //ARGB Format Big Endian (low address save high MSB, here is A) in memory : A|R|G|B
            //ARGB Format Little Endian (low address save low MSB, here is B) in memory : B|G|R|A
            if (SDL_BYTEORDER == SDL_LIL_ENDIAN) {
                //Little Endian (x86): R|G|B --> B|G|R|A
                image_out[(i*w + j) * 4 + 0] = image_in[(i*w + j) * 3 + 2];
                image_out[(i*w + j) * 4 + 1] = image_in[(i*w + j) * 3 + 1];
                image_out[(i*w + j) * 4 + 2] = image_in[(i*w + j) * 3];
                image_out[(i*w + j) * 4 + 3] = '0';
            }
            else {
                //Big Endian: R|G|B --> A|R|G|B
                image_out[(i*w + j) * 4] = '0';
                memcpy(image_out + (i*w + j) * 4 + 1, image_in + (i*w + j) * 3, 3);
            }
        }
}


//Refresh Event
#define REFRESH_EVENT  (SDL_USEREVENT + 1)
//Break
#define BREAK_EVENT  (SDL_USEREVENT + 2)

int thread_exit = 0;

int refresh_video(void *opaque) {
    thread_exit = 0;
    while (!thread_exit) {
        SDL_Event event;
        event.type = REFRESH_EVENT;
        SDL_PushEvent(&event);
        SDL_Delay(40);
    }
    thread_exit = 0;
    //Break
    SDL_Event event;
    event.type = BREAK_EVENT;
    SDL_PushEvent(&event);
    return 0;
}

SDL_Window* screen;
SDL_Renderer* sdlRenderer;
SDL_Texture* sdlTexture;

SDL_Rect sdlRect;


SDL_Event event;
HWND winhandler;

int SdlSetWin(HWND handler, const int win_w, const int win_h)
{
    winhandler = handler;
    screen_w = win_w;
    screen_h = win_h;
    return 0;
}

int SdlInit(const int pix_w, const int pix_h)
{
    pixel_w = pix_w;
    pixel_h = pix_h;

    buffer = new unsigned char[pixel_w*pixel_h*bpp / 8];
    buffer_convert = new unsigned char[pixel_w*pixel_h * 4];


    if (SDL_Init(SDL_INIT_VIDEO)) {
        printf("Could not initialize SDL - %s\n", SDL_GetError());

        return -1;
    }

    //SDL 2.0 Support for multiple windows
    //screen = SDL_CreateWindow("Simplest Video Play SDL2", SDL_WINDOWPOS_UNDEFINED, SDL_WINDOWPOS_UNDEFINED,
    //  screen_w, screen_h, SDL_WINDOW_OPENGL | SDL_WINDOW_RESIZABLE);
    screen = SDL_CreateWindowFrom(winhandler);

    if (!screen) {
        printf("SDL: could not create window - exiting:%s\n", SDL_GetError());

        return -1;
    }
    sdlRenderer = SDL_CreateRenderer(screen, -1, 0);

    Uint32 pixformat = 0;
#if LOAD_BGRA
    //Note: ARGB8888 in "Little Endian" system stores as B|G|R|A
    pixformat = SDL_PIXELFORMAT_ARGB8888;
#elif LOAD_RGB24
    pixformat = SDL_PIXELFORMAT_RGB888;
#elif LOAD_BGR24
    pixformat = SDL_PIXELFORMAT_BGR888;
#elif LOAD_YUV420P
    //IYUV: Y + U + V  (3 planes)
    //YV12: Y + V + U  (3 planes)
    pixformat = SDL_PIXELFORMAT_IYUV;
#endif

    sdlTexture = SDL_CreateTexture(sdlRenderer, pixformat, SDL_TEXTUREACCESS_STREAMING, pixel_w, pixel_h);
    return 0;
}

int SdlRender(unsigned char* buffer)
{


#if LOAD_BGRA
    //We don't need to change Endian
    //Because input BGRA pixel data(B|G|R|A) is same as ARGB8888 in Little Endian (B|G|R|A)
    SDL_UpdateTexture(sdlTexture, NULL, buffer, pixel_w * 4);
#elif LOAD_RGB24|LOAD_BGR24
    //change 24bit to 32 bit
    //and in Windows we need to change Endian
    CONVERT_24to32(buffer, buffer_convert, pixel_w, pixel_h);
    SDL_UpdateTexture(sdlTexture, NULL, buffer_convert, pixel_w * 4);
#elif LOAD_YUV420P
    SDL_UpdateTexture(sdlTexture, NULL, buffer, pixel_w);
#endif
    //FIX: If window is resize
    sdlRect.x = 0;
    sdlRect.y = 0;
    sdlRect.w = screen_w;
    sdlRect.h = screen_h;

    SDL_RenderClear(sdlRenderer);
    SDL_RenderCopy(sdlRenderer, sdlTexture, NULL, &sdlRect);
    SDL_RenderPresent(sdlRenderer);

    return 0;
}

```

## WPF 播放 RGB 例子

```C#
    // 在 UI 线程中新建窗口，并将窗口句柄传入 DLL 中
    RunAtUI(() =>
    {
        win = new MyPlayerWindow();
        win.Show();

        IntPtr hwnd = new WindowInteropHelper(win).Handle;
        if (LibCrPlayerHelper.SdlSetWin(hwnd, (int)win.Width, (int)win.Height) < 0)
            throw new Exception("Error: SdlSetWin.");
    });

    // 获取图片大小，设置图片大小，并初始化
    int w = 0, h = 0;
    MyDll.GetBitmapSize(ref w, ref h);
    bitmap = new System.Drawing.Bitmap(w, h, System.Drawing.Imaging.PixelFormat.Format32bppArgb);
    rect = new System.Drawing.Rectangle(0, 0, bitmap.Width, bitmap.Height);
    LibCrPlayerHelper.SdlInit(bitmap.Width, bitmap.Height);

    // 循环显示画面
    while (true)
    {
        var bmpData = bitmap.LockBits(rect, System.Drawing.Imaging.ImageLockMode.ReadWrite, bitmap.PixelFormat);

        // Get the address of the first line.
        IntPtr ptr = bmpData.Scan0;

        if (MyDll.GetBitmapData(ptr) >= 0)
            LibCrPlayerHelper.SdlRender(ptr);

        // Unlock the bits.
        bitmap.UnlockBits(bmpData);
    }
```

## MainWindow.xaml.cs

```C#
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

        private Window win;
        private System.Drawing.Bitmap bitmap;
        private System.Drawing.Rectangle rect;

        public MainWindow()
        {
            InitializeComponent();
        }
        void RunAtUI(Action x)
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

## Helper

```C#
public class LibMyPlayerHelper
{
    // 设置窗口句柄，窗口大小
    [DllImport("libMyPlayer.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.Cdecl)]
    public static extern int SdlSetWin(IntPtr handler, int win_w, int win_h);
    // 设置图片大小，并开始初始化
    [DllImport("libMyPlayer.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.Cdecl)]
    public static extern int SdlInit(int pix_w, int pix_h);
    // 传入图片数据，并开始渲染
    [DllImport("libMyPlayer.dll", CharSet = CharSet.Auto, CallingConvention = CallingConvention.Cdecl)]
    public static extern int SdlRender(IntPtr ptr);

}
```

```C#
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


## 可能遇到的问题

如果给 SDL 传入 WPF 某个控件的句柄， SDL 也会直接渲染整个窗口。

    IntPtr hwnd = ((HwndSource)PresentationSource.FromVisual(uielement)).Handle;
	if (LibCrPlayerHelper.SdlSetWin(hwnd, (int)win.Width, (int)win.Height) < 0)
		throw new Exception("Error: SdlSetWin.");