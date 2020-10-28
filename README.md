对GPUImage的学习记录  
# 概览
![image](https://github.com/daliang0101/DL_gpuimage/blob/main/images/summary.png)  
# 简介 
* iOS平台图像处理库  

* 对静态图像、摄像头图像/视频，做滤镜处理  

* 基于OpenGL ES 2.0，即使用了GPU硬件加速处理，效率远高于使用CPU处理  

* 内置上百种滤镜，且使用其GPUImageFilter接口可以很方便添加自定义滤镜  
  
* 支持视频/图像后期滤镜处理、编码，生成新文件，使用GPUImageMovie接口  

# 设计思想  
* 节点：对一个处理阶段的抽象  
  
* 链式处理：把整个处理过程中的每个节点，按一定的逻辑关系链接起来  
  
* input / output模型：把节点分成input、output类型  

   input型：依赖上一级节点的处理结果，等待上一级节点通知  
   
   output型：拥有待通知的下一级目标对象，把自身的处理结果通知给目标对象  
   
   input、output型：既依赖上一级节点的处理结果，也拥有待通知的下一级目标对象  
   
* input / output 模型的实现：   
  
   input协议：遵循该协议的节点对象，都能成为上一级节点的目标对象，接收它的处理结果  
   
   output父类：继承该类的子类，都可以把下一级节点对象添加到它的目标列表中，处理完成后遍历目标列表，把结果逐个通知给目标对象
   
# 主结构类、协议简介
## GLProgram：对 OpenGL ES 顶点、片段着色器程序的抽象封装  
*   创建、编译shader对象；  

*   创建shaderProgram对象，为shaderProgram对象附加具体的shader对象；  
   
*   链接、激活、销毁shaderProgram对象；  
   
*   绑定、获取shaderProgram的Attribute变量位置；
   
*   获取Uniform变量位置；
   
## GPUImageFramebuffer：对 OpenGL ES 帧缓冲区相关数据、行为的抽象封装
* 创建、绑定、销毁FBO；
  
* 创建、绑定、销毁纹理对象，指定纹理过滤模式，为FBO绑定纹理对象；  

* 引用计数，为0时，会被cache回收；提供lock、unlock接口增加、减少计数值； 

* 需要lock的位置：从Cache中获取到一个FrameBuffer时，把frameBuffer传递给下一级节点时，usingNextFrameForImageCapture == YES时；

* 需要unlock的位置：当前节点渲染结束时，需要释放从上个节点传递过来的frameBuffer的引用；通知下一级节点开始之后，把当前节点渲染使用的frameBuffer释放；

## GPUImageFramebufferCache：缓存GPUImageFramebuffer、GLProgram对象
* 生成FBO需要占用一定的内存或者显存，滤镜链上可能需要很多FBO；

* 该类对GPUImageFramebuffer、GLProgram提供复用机制；

* 以buffer size、texture parametres生成一个字符串作为key，把GPUImageFramebuffer、GLProgram对象存入NSMutableDictionary；

## GPUImageContext：对OpenGL ES  上下问对象的封装
* 管理GL ES上下文  

* 管理GPUImageFramebufferCache


# 结构层次
## 基础模块
![image](https://github.com/daliang0101/DL_gpuimage/blob/main/images/base_cls.png)

## Sources模块
![image](https://github.com/daliang0101/DL_gpuimage/blob/main/images/sources_jc.png)  

## Filters模块
![image](https://github.com/daliang0101/DL_gpuimage/blob/main/images/filter_jc.png)  

## Filters与基础模块之间的关系
![image](https://github.com/daliang0101/DL_gpuimage/blob/main/images/filter_baseCls.png)  
   
## GPUImageView结构
![image](https://github.com/daliang0101/DL_gpuimage/blob/main/images/glview.png)  



# 滤镜链内幕
* 继承GPUImageOutput的对象有一个outputFrameBuffer成员，其渲染结果就存储在这里；

* 渲染结束之后，变量目标对象列表，把目标对象的outputFrameBuffer传递给目标对象的inputFrameBuffer；

* 目标对象把inputFrameBuffer中的纹理绑定到自身的outputFrameBuffer中，也相当于把上一级处理结果图像作为当前待处理的图像；

















