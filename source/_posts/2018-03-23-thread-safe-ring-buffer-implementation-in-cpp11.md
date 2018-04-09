---
title: C++ 11 实现线程安全的环形缓冲区
date: 2018-03-23 17:44:26
tags:

categories: 
  - 计算机
  - 编程
  - C/C++
---


环形缓冲区（[Ring Buffer](https://zh.wikipedia.org/wiki/%E7%92%B0%E5%BD%A2%E7%B7%A9%E8%A1%9D%E5%8D%80 )），是一种用于表示一个固定尺寸、头尾相连的缓冲区的数据结构，适合缓存数据流。

环形缓冲区的一个有用特性是：当一个数据元素被用掉后，其余数据元素不需要移动其存储位置。换句话说，环形缓冲区适合实现先进先出缓冲区。

<link href="/style/ringbuffer.css" rel="stylesheet" type="text/css" />
<div style="text-align: center"><div class="lds-ring"><div></div><div></div><div></div><div></div></div></div>

> 由于计算机内存是线性地址空间，因此环形缓冲区需要特别的设计才可以从逻辑上实现。
> 一般的，环形缓冲区需要4个指针：
> * 在内存中实际开始位置；
> * 在内存中实际结束位置，也可以用缓冲区长度代替；
> * 存储在缓冲区中的有效数据的开始位置（读指针）；
> * 存储在缓冲区中的有效数据的结尾位置（写指针）。

> 缓冲区是满、或是空，都有可能出现读指针与写指针指向同一位置，有多种策略用于检测缓冲区是满、或是空：
> * 总是保持一个存储单元为空
> * 使用数据计数
> * 镜像指示位
> * 读/写 计数
> * 记录最后的操作


<!--more -->

以下是使用数据计数策略用于检测缓冲区是满、或是空，使用自旋锁保证线程安全的高效环形缓冲区。

```cpp
/*
* ring_buffer_s.h
*/
#pragma once
#include <mutex>
#include "spin_mutex.h"

/**
* \brief 线程安全的环形缓冲区
*/
class ring_buffer_s
{
public:
	explicit ring_buffer_s(size_t capacity);
	ring_buffer_s(ring_buffer_s &&) = delete;
	ring_buffer_s& operator=(ring_buffer_s&& other) = delete;

	ring_buffer_s(const ring_buffer_s &) = delete;
	ring_buffer_s& operator=(const ring_buffer_s& other) = delete;

	~ring_buffer_s();

	/**
	 * \brief 获取缓冲区已用大小
	 */
	size_t size() const;
		
	/**
	 * \brief 获取缓冲区容量
	 */
	size_t capacity() const;
	
	/**
	 * \brief 写入数据
	 * \param data 要写入的数据
	 * \param bytes 要写入的数据的大小
	 * \return 最终写入的数据的大小
	 */
	size_t write(const void *data, size_t bytes);


	/**
	 * \brief 读取数据
	 * \param data 用来存放读取数据的buffer
	 * \param bytes 要读取的数据大小
	 * \return 最终获取到的数据的大小
	 */
	size_t read(void *data, size_t bytes);

private:
	size_t front_, rear_, size_, capacity_;
	uint8_t *data_;
	mutable spin_mutex mut_;
	mutable std::mutex mut_read_;
	mutable std::mutex mut_write_;

};
```


```cpp
/*
* ring_buffer_s.cpp
*/

#include <algorithm>
#include "ring_buffer_s.h"

inline ring_buffer_s::ring_buffer_s(const size_t capacity)
	: front_(0)
	, rear_(0)
	, size_(0)
	, capacity_(capacity)
{
	data_ = new uint8_t[capacity];
}

inline ring_buffer_s::~ring_buffer_s()
{
	delete[] data_;
}

inline size_t ring_buffer_s::size() const
{
	return size_;
}

inline size_t ring_buffer_s::capacity() const
{
	return capacity_;
}

inline size_t ring_buffer_s::write(const void *data, const size_t bytes)
{
	if (bytes == 0) return 0;

	// 通过互斥量保证任意时刻，至多只有一个线程在写数据。
	std::lock_guard<std::mutex>lk_write(mut_write_);
	const auto capacity = capacity_;
	const auto bytes_to_write = std::min(bytes, capacity - size_);

	if (bytes_to_write == 0) return 0;
	
	// 一次性写入
	if (bytes_to_write <= capacity - rear_)
	{
		memcpy(data_ + rear_, data, bytes_to_write);
		rear_ += bytes_to_write;
		if (rear_ == capacity) rear_ = 0;
	}
	// 分两步进行
	else
	{
		const auto size_1 = capacity - rear_;
		memcpy(data_ + rear_, data, size_1);

		const auto size_2 = bytes_to_write - size_1;
		memcpy(data_, static_cast<const uint8_t*>(data) + size_1, size_2);
		rear_ = size_2;
	}

	// 通过自旋锁保证任意时刻，至多只有一个线程在改变 size_ .
	std::lock_guard<spin_mutex>lk(mut_);
	size_ += bytes_to_write;
	return bytes_to_write;
}

inline size_t ring_buffer_s::read(void *data, const size_t bytes)
{
	if (bytes == 0) return 0;

	// 通过互斥量保证任意时刻，至多只有一个线程在读数据。
	std::lock_guard<std::mutex>lk_read(mut_read_);

	const auto capacity = capacity_;
	const auto bytes_to_read = std::min(bytes, size_);
	
	if (bytes_to_read == 0) return 0;

	// 一次性读取
	if (bytes_to_read <= capacity - front_)
	{
		memcpy(data, data_ + front_, bytes_to_read);
		front_ += bytes_to_read;
		if (front_ == capacity) front_ = 0;
	}
	// 分两步进行
	else
	{
		const auto size_1 = capacity - front_;
		memcpy(data, data_ + front_, size_1);
		const auto size_2 = bytes_to_read - size_1;
		memcpy(static_cast<uint8_t*>(data) + size_1, data_, size_2);
		front_ = size_2;
	}

	// 通过自旋锁保证任意时刻，至多只有一个线程在改变 size_ .
	std::lock_guard<spin_mutex>lk(mut_);
	size_ -= bytes_to_read;
	return bytes_to_read;
}
```


```cpp
/*
* spin_mutex.h
*/
#pragma once
#include <atomic>

class spin_mutex {
	std::atomic_flag flag_ = ATOMIC_FLAG_INIT;
public:
	spin_mutex() = default;
	~spin_mutex() = default;
	spin_mutex(const spin_mutex&) = delete;
	spin_mutex& operator= (const spin_mutex&) = delete;
	spin_mutex(spin_mutex&&) = delete;
	spin_mutex& operator= (spin_mutex&&) = delete;

	void lock() {
		while (flag_.test_and_set(std::memory_order_acquire))
			;
	}
	void unlock() {
		flag_.clear(std::memory_order_release);
	}
};
```