---
title: 【Vulkan】将最终绘制结果导出并存储为图片
tags: vulkan,OpenGL
category: /personal blog/vulkan/2022-03
renderNumberedHeading: true
grammar_cjkRuby: true
---


如何将vulkan中绘制的结果导出，并且存储为文件格式？

## OpenGL中获得绘制结果的方法
GPU渲染的结果保存在显存(帧缓存)中，想要将保存在显存中的结果转存到内存，在opengl中需要用到glReadPixels这个函数。
**glReadPixels：** 把已经绘制好的像素（它可能已经被保存到显卡的显存中）读取到内存。

``` haskell
			 void glReadPixels（GLint x, 
								GLint y,       	   → 左下角坐标
								GLsizei width,
                      			GLsizei height,    → 前四个参数描述了一个矩形
                                GLenum format,
                                GLenum type,
                                GLvoid * data）;
```