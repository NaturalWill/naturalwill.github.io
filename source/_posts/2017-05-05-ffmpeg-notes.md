---
title: ffmpeg 获取摄像头、麦克风选项
date: 2017-05-05 11:44:09
tags:
  - ffmpeg
  - Notes
categories: 
  - 400-软件使用
  - Notes
---


简单记录一下 ffmpeg 获取 DirectShow 设备数据的方法。本文所述的方法主要是对应Windows平台的。（由于 ffmpeg 输出编码为 UTF-8, 所以这里使用 Git Bash ）

## 列出设备

	$ ./ffmpeg -list_devices true -f dshow -i dummy
	ffmpeg version 3.2.4 Copyright (c) 2000-2017 the FFmpeg developers
	  built with gcc 6.3.0 (GCC)
	[dshow @ 00000000001c23c0] DirectShow video devices (some may be both video and audio devices)
	[dshow @ 00000000001c23c0]  "Integrated Webcam"
	[dshow @ 00000000001c23c0]     Alternative name "@device_pnp_\\?\usb#vid_0c45&pid_6448&mi_00#7&2645f9e4&0&0000#{65e8773d-8f56-11d0-a3b9-00a0c9223196}\global"
	[dshow @ 00000000001c23c0]  "screen-capture-recorder"
	[dshow @ 00000000001c23c0]     Alternative name "@device_sw_{860BB310-5D01-11D0-BD3B-00A0C911CE86}\{4EA69364-2C8A-4AE6-A561-56E4B5044439}"
	[dshow @ 00000000001c23c0] DirectShow audio devices
	[dshow @ 00000000001c23c0]  "麦克风 (High Definition Audio 设备)"
	[dshow @ 00000000001c23c0]     Alternative name "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{E26AD2BE-B335-41EC-BED5-40F982F21FF7}"
	[dshow @ 00000000001c23c0]  "virtual-audio-capturer"
	[dshow @ 00000000001c23c0]     Alternative name "@device_sw_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\{8E146464-DB61-4309-AFA1-3578E927E935}"
	dummy: Immediate exit requested



## 查看设备选项

video

	$ ./ffmpeg -list_options true -f dshow -i video="Integrated Webcam"
	ffmpeg version 3.2.4 Copyright (c) 2000-2017 the FFmpeg developers
	  built with gcc 6.3.0 (GCC)
	[dshow @ 0000000000ea23e0] DirectShow video device options (from video devices)
	[dshow @ 0000000000ea23e0]  Pin "捕获" (alternative pin name "0")
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=640x480 fps=10 max s=640x480 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=640x480 fps=10 max s=640x480 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=352x288 fps=10 max s=352x288 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=352x288 fps=10 max s=352x288 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=320x240 fps=10 max s=320x240 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=320x240 fps=10 max s=320x240 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=176x144 fps=10 max s=176x144 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=176x144 fps=10 max s=176x144 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=160x120 fps=10 max s=160x120 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=160x120 fps=10 max s=160x120 fps=30
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=1280x720 fps=11 max s=1280x720 fps=11
	[dshow @ 0000000000ea23e0]   pixel_format=yuyv422  min s=1280x720 fps=11 max s=1280x720 fps=11
	[dshow @ 0000000000ea23e0]   vcodec=mjpeg  min s=1280x720 fps=10 max s=1280x720 fps=30
	[dshow @ 0000000000ea23e0]   vcodec=mjpeg  min s=1280x720 fps=10 max s=1280x720 fps=30
	video=Integrated Webcam: Immediate exit requested

audio

	$ ./ffmpeg -list_options true -f dshow -i audio="麦克风 (High Definition Audio 设备)"
	ffmpeg version 3.2.4 Copyright (c) 2000-2017 the FFmpeg developers
	  built with gcc 6.3.0 (GCC)
	[dshow @ 00000000024d24a0] DirectShow audio only device options (from audio devices)
	[dshow @ 00000000024d24a0]  Pin "Capture" (alternative pin name "Capture")
	[dshow @ 00000000024d24a0]   min ch=1 bits=8 rate= 11025 max ch=2 bits=16 rate= 44100
		Last message repeated 22 times
	audio=麦克风 (High Definition Audio 设备): Immediate exit requested
