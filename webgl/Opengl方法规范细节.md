由于webgl使用的是opengl es3的规范，所以大部分方法对于webgl来说同样适用

# glReadPixels

> 在threejs中对应的方法是 renderer.readRenderTargetPixels

> 在threejs中的签名是  renderTarget : WebGLRenderTarget, x : Float, y : Float, width : Float, height : Float, buffer : TypedArray, activeCubeFaceIndex : Integer


函数签名为:
```c
void glReadPixels(	GLint x,
 	GLint y,
 	GLsizei width,
 	GLsizei height,
 	GLenum format,
 	GLenum type,
 	void * data);


void glReadnPixels(	GLint x,
 	GLint y,
 	GLsizei width,
 	GLsizei height,
 	GLenum format,
 	GLenum type,
 	GLsizei bufSize,
 	void *data);
```

主要看看参数解释:

- x,y
	指定从帧缓冲区读取的第一个像素的窗口坐标。这个位置是一个矩形像素块的左下角。
- width,height
	指定像素矩形的尺寸。宽度和高度为一对应一个像素。
- format
	指定像素数据的格式。接受以下符号值：GL_STENCIL_INDEX，GL_DEPTH_COMPONENT，GL_DEPTH_STENCIL，GL_RED，GL_GREEN，GL_BLUE，GL_RGB，GL_BGR，GL_RGBA 和 GL_BGRA。
- type
	指定像素数据的数据类型。必须是 GL_UNSIGNED_BYTE，GL_BYTE，GL_UNSIGNED_SHORT，GL_SHORT，GL_UNSIGNED_INT，GL_INT，GL_HALF_FLOAT，GL_FLOAT，GL_UNSIGNED_BYTE_3_3_2，GL_UNSIGNED_BYTE_2_3_3_REV，GL_UNSIGNED_SHORT_5_6_5，GL_UNSIGNED_SHORT_5_6_5_REV，GL_UNSIGNED_SHORT_4_4_4_4，GL_UNSIGNED_SHORT_4_4_4_4_REV，GL_UNSIGNED_SHORT_5_5_5_1，GL_UNSIGNED_SHORT_1_5_5_5_REV，GL_UNSIGNED_INT_8_8_8_8，GL_UNSIGNED_INT_8_8_8_8_REV，GL_UNSIGNED_INT_10_10_10_2，GL_UNSIGNED_INT_2_10_10_10_REV，GL_UNSIGNED_INT_24_8，GL_UNSIGNED_INT_10F_11F_11F_REV，GL_UNSIGNED_INT_5_9_9_9_REV 或 GL_FLOAT_32_UNSIGNED_INT_24_8_REV 其中之一。
- bufSize
	指定 glReadnPixels 函数的缓冲区数据的大小。
- data
	返回像素数据

函数从坐下角开始从左往右读取，从低往高读取

> 需要注意的是读取像素数据是从左下角开始的，而对于HTML Canvas来说坐标系原点在左上角

在THREEJs中存在setViewOffset函数，这个函数的x,y偏移都是以左上角为原点作偏移的。setViewOffset函数通常和readRenderTargetPixels一起来实现GPU拾取的功能