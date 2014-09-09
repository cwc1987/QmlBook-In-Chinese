# OpenGL着色器（OpenGL Shader）

OpenGL的渲染管线分为几个步骤。一个简单的OpenGL渲染管线将包含一个顶点着色器和一个片段着色器。

![](http://qmlbook.org/_images/openglpipeline.png)

顶点着色器接收顶点数据，并且在程序最后赋值给gl_Position。然后，顶点将会被裁剪，转换和栅格化后作为像素输出。
片段（像素）进入片段着色器，进一步对片段操作并将结果的颜色赋值给gl_FragColor。顶点着色器调用多边形每个角的点（顶点=3D中的点），负责这些点的3D处理。片段（片度=像素）着色器调用每个像素并决定这个像素的颜色。
