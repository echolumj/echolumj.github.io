---
title: 【Vulkan】将最终绘制结果导出并存储为图片
tags: vulkan,OpenGL
category: /personal blog/vulkan/2022-03
grammar_abbr: true
grammar_table: true
grammar_defList: true
grammar_emoji: true
grammar_footnote: true
grammar_ins: true
grammar_mark: true
grammar_sub: true
grammar_sup: true
grammar_checkbox: true
grammar_mathjax: true
grammar_flow: true
grammar_sequence: true
grammar_code: true
grammar_highlight: true
grammar_html: true
grammar_linkify: true
grammar_typographer: true
grammar_video: true
grammar_audio: true
grammar_attachment: true
grammar_mermaid: true
grammar_classy: true
grammar_decorate: false
grammar_attrs: false
grammar_cjkEmphasis: true
grammar_cjkRuby: true
grammar_center: true
grammar_align: true
grammar_tableExtra: true
---


如何将vulkan中绘制的结果导出，并且存储为文件格式？

## OpenGL中将绘制结果存储为图片的方法
GPU渲染的结果保存在显存(帧缓存)中，想要将保存在显存中的结果转存到内存，在opengl中需要用到glReadPixels这个函数。
**glReadPixels：** 把已经绘制好的像素（它可能已经被保存到显卡的显存中）读取到内存。

``` c++
 void glReadPixels（GLint x, 
						GLint y,       	   → 左下角坐标
						GLsizei width,
						GLsizei height,    → 前四个参数描述了一个矩形范围
						GLenum format,     → 像素存储的格式
						GLenum type,   	   → 像素数据的数据类型
						GLvoid * data）;   →返回像素数据
```

**实现过程：** 
step 1：申请一块放置读取到像素的内存
```c++
	RGBColor* ColorBuffer = new RGBColor[WindowSizeX * WindowSizeY];
```
step 2：从显存中读取像素
```c++
	glReadPixels(0, 0, WindowSizeX, WindowSizeY, GL_BGR, GL_UNSIGNED_BYTE, ColorBuffer);
```
step 3：将数据写入目标图片文件

```c++
	WriteBMP("output.bmp", ColorBuffer, WindowSizeX, WindowSizeY);
```

step 4：清理申请的内存

```c++
	delete[] ColorBuffer;
```

**注：** 这个存储的过程要放在OpenGL绘制结束后，在交换缓冲之前进行。

**参考链接** 
[双缓冲区模式下读取](https://blog.csdn.net/cd_yourheart/article/details/123528957)
[将OpenGL渲染的结果保存为图片](https://blog.csdn.net/u013412391/article/details/120565095)

## Vulkan中将绘制结果存储为图片的方法
**VS opengl：** 基本过程一致，GPU中保存绘制结果的地方→分配得到的内存。但是，opengl中提供了glReadPixels函数，只需要调用这个函数就可以将绘制结果读取出来，但是vulkan中并没有提供直接的函数。
 

**基本思路：** vulkan中渲染结果放在swapchain image中，程序中往往设定当前swapchain image的数量为物理设备支持的最小swapchain image数量+1,所以要在渲染完之后提交之前，将当前 Swap Chain Image 的内容先存在一块申请的显存上，之后内存映射到内存中。

**1. Format问题** 
    当前物理设备支持的swapchain image的格式为：VK_FORMAT_B8G8R8A8_SRGB
	申请到显存的设定格式：VK_FORMAT_R8G8B8A8_SRGB（创建image以及memory的代码如下）
```c++
	createImage(WIDTH, HEIGHT, 1, VK_IMAGE_TYPE_2D, >>==VK_FORMAT_R8G8B8A8_SRGB==<<, VK_IMAGE_TILING_LINEAR, >>==VK_IMAGE_USAGE_TRANSFER_DST_BIT==<<
	VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, outputImg, outputImgMemory);
```

**2.Layout问题** 
	当前swap chain Image的布局：VK_IMAGE_LAYOUT_PRESENT_SRC_KHR
	作为transfer source的布局：VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL
	申请到显存的布局：VK_IMAGE_LAYOUT_UNDEFINED
	作为transfer destination的布局：VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL
	支持内存映射的布局：VK_IMAGE_LAYOUT_GENERAL
**3.swap chain image和output image之间布局的转换** 
<mark>左图为swap chain Image的布局转换；右图为申请到的显存的布局转换</mark>
```mermaid!
	graph TD;
    VK_IMAGE_LAYOUT_PRESENT_SRC_KHR-->VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL;
    VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL-->VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
	
	VK_IMAGE_LAYOUT_UNDEFINED-->VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL-->VK_IMAGE_LAYOUT_GENERAL
```
**4.swap chain image和output image之间内容的copy** 
	 step 1 → Device 是否支持 blitting from optimal tiled images？
``` c++
bool supportsBlit = true;
VkFormatProperties formatProperties;
vkGetPhysicalDeviceFormatProperties(physicalDevice, swapChainImageFormat, &formatProperties);

if (!(formatProperties.optimalTilingFeatures &  >>==VK_FORMAT_FEATURE_BLIT_SRC_BIT==<<))
{
	std::cerr << "Device does not support blitting from optimal tiled images, using copy instead of blit!" << std::endl;
	supportsBlit = false;
}

if (!(formatProperties.linearTilingFeatures & >>==VK_FORMAT_FEATURE_BLIT_DST_BIT==<<)) {
	std::cerr << "Device does not support blitting to linear tiled images, using copy instead of blit!" << std::endl;
	supportsBlit = false;
}
```
step 2 → Device 支持 blitting from optimal tiled images→<mark>vkCmdBlitImage</mark>
 
```c++
VkOffset3D blitSize;
blitSize.x = width;
blitSize.y = height;
blitSize.z = 1;
VkImageBlit imageBlitRegion{};
imageBlitRegion.srcSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
imageBlitRegion.srcSubresource.layerCount = 1;
imageBlitRegion.srcOffsets[1] = blitSize;
imageBlitRegion.dstSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
imageBlitRegion.dstSubresource.layerCount = 1;
imageBlitRegion.dstOffsets[1] = blitSize;

// Issue the blit command
vkCmdBlitImage(
	commandBuffer,
	srcImage, VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL,
	outputImg, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL,
	1,
	&imageBlitRegion,
	VK_FILTER_NEAREST);
```
 step 3 → Device 不支持 blitting from optimal tiled images→<mark>vkCmdCopyImage</mark>

```c++
VkImageCopy imageCopyRegion{};
imageCopyRegion.srcSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
imageCopyRegion.srcSubresource.layerCount = 1;
imageCopyRegion.dstSubresource.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
imageCopyRegion.dstSubresource.layerCount = 1;
imageCopyRegion.extent.width = width;
imageCopyRegion.extent.height = height;
imageCopyRegion.extent.depth = 1;

// Issue the copy command
vkCmdCopyImage(
	commandBuffer,
	srcImage, VK_IMAGE_LAYOUT_TRANSFER_SRC_OPTIMAL,
	outputImg, VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL,
	1,
	&imageCopyRegion);
```

**5. 进行内存映射，将显存的内容映射到内存**
	step 1 → 找到subresource 图像数据的起始 offset
```c++
VkImageSubresource subResource{ VK_IMAGE_ASPECT_COLOR_BIT, 0, 0 };
VkSubresourceLayout subResourceLayout;
vkGetImageSubresourceLayout(logicalDevice, outputImg, &subResource, &subResourceLayout);

// Map image memory so we can start copying from it
const char* data;
>>==vkMapMemory==<<(logicalDevice, outputImgMemory, 0, VK_WHOLE_SIZE, 0, (void**)&data);
data += subResourceLayout.offset;
```
**6. 将data数据以ppm格式存入**
<mark>注意点：</mark>
  若swapchain image格式为BGR，目标存储格式为RGB，则需要手动转换color components
  
```c++
if (colorSwizzle
	{
		auto a = (char*)row;
		auto b = (char*)(row+1);
		auto c = (char*)(row+2);
		file.write((char*)row + 2, 1);
		file.write((char*)row + 1, 1);
		file.write((char*)row, 1);
	}
```

**参考链接：**
[截屏原理](https://gavinkg.github.io/ILearnVulkanFromScratch-CN/mdroot/Vulkan%20%E8%BF%9B%E9%98%B6/%E6%88%AA%E5%8F%96%E5%B1%8F%E5%B9%95/%E5%8E%9F%E7%90%86.html)
[代码参考](https://github.com/SaschaWillems/VulkanCapsViewer/blob/master/vulkancapsviewer.cpp)
