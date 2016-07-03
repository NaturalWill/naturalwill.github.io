---
layout: post
title: Linux下文件的压缩与解压缩
tags: [Linux, tar, zip, Java]
date: 2016/07/03
---



### tar

	# 创建压缩包
	tar -cvf name-of-archive.tar /path/to/directory-or-file	
	tar -cvf name-of-archive.tar /home/arch --exclude=/home/arch/.cache --exclude=/home/arch/Downloads
	tar -cvf name-of-archive.tar /home/arch --exclude=*.jpg
	
	tar -czvf name-of-archive.tar.gz /path/to/directory-or-file	
	tar -cJvf name-of-archive.tar.xz /path/to/directory-or-file	
	tar -cjvf name-of-archive.tar.bz2 /path/to/directory-or-file	
	tar -cZvf name-of-archive.tar.Z /path/to/directory-or-file	
	
	
	# 查看压缩包内容
	tar -tf archive.tar
	tar -tvf archive.tar
	tar -tzf archive.tar.gz
	tar -tzvf archive.tar.gz
	
	
	# 解压文件
	tar -xvf archive.tar
	tar -xvf archive.tar -C /tmp
	
	tar -xzvf archive.tar.gz
	tar -xJvf archive.tar.xz
	tar -xjvf archive.tar.bz2
	tar -xZvf archive.tar.Z

参数解释
	* -c: 创建压缩包（create）
	* -t: 列出压缩包内容（list）
	* -x: 提取文件（extract）
	* -v: 显示详细信息（verbose）
	* -f: 用于指定文件名（filename），f后必须接文件名。
	* -C: 解压到指定目录
	* --exclude: 忽略文件或目录
	* -z: 使用 gzip 算法来创建压缩包或提取文件。
	* -J: 使用 xz 算法创建压缩包或提取文件。
	* -j: 使用 bzip2 算法创建压缩包或提取文件。
	* -Z: 使用 ncompress 算法创建压缩包或提取文件。
	
	
### 其他

	# 7z
	7z a archive.7z /path/to/directory-or-file
	7z x archive.7z
	7z x archive.rar
	
	# zip
	zip -r archive.zip /path/to/directory-or-file
	unzip archive.zip

	# gzip
	gzip archive.tar
	gunzip archive.tar.gz
	
	# xz
	xz archive.tar
	unxz archive.tar.xz
	
	# bzip2
	bzip2 archive.tar
	bunzip2 archive.tar.bz2
	
	# Z
	compress archive.tar
	uncompress archive.tar.Z


### jar

	jar -cvf archive.jar /path/to/directory-or-file
	jar -xvf archive.jar

注：如果是打包的是Java类库，并且该类库中存在主类，那么需要写一个 META-INF/MANIFEST.MF 配置文件，内容如下：

	Manifest-Version: 1.0
	Created-By: 1.6.0_27 (Sun Microsystems Inc.)
	Main-class: the_name_of_the_main_class_should_be_put_here

然后用如下命令打包：

	jar -cvfm name-of-archive.jar META-INF/MANIFEST.MF /path/to/directory-or-file

这样以后就能用 `java -jar name-of-archive.jar` 命令直接运行主类中的 `public static void main` 方法了。