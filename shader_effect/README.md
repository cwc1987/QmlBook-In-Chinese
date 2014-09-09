# Shader Effect

**注意**

**最后一次构建：2014年1月20日下午18:00。**

**这章的源代码能够在[assetts folder](http://qmlbook.org/assets)找到。**

* http://labs.qt.nokia.com/2012/02/02/qt-graphical-effects-in-qt-labs/

* http://labs.qt.nokia.com/2011/05/03/qml-shadereffectitem-on-qgraphicsview/

* http://qt-project.org/doc/qt-4.8/declarative-shadereffects.html

* http://www.opengl.org/registry/doc/GLSLangSpec.4.20.6.clean.pdf

* http://www.khronos.org/registry/gles/specs/2.0/GLSL_ES_Specification_1.0.17.pdf

* http://www.lighthouse3d.com/opengl/glsl/

* http://wiki.delphigl.com/index.php/Tutorial_glsl

* [Qt5Doc qtquick-shaders](http://doc.qt.nokia.com/5.0-snapshot/qtquick-shaders.html)

着色器允许我们利用SceneGraph的接口直接调用在强大的GPU上运行的OpenGL来创建渲染效果。着色器使用ShaderEffect与ShaderEffectSource元素来实现。着色器本身的算法使用OpenGL Shading Language（OpenGL着色语言）来实现。

实际上这意味着你需要混合使用QML代码与着色器代码。执行时，会将着色器代码发送到GPU，并在GPU上编译执行。QML着色器元素（Shader QML Elements）允许你与OpenGL着色器程序的属性交互。

让我们首先来看看OpenGL着色器。
