# CMake

CMake是由Kitware创造的工具。由于它们的3D可视化软件VTK使得Kitware家喻户晓，当然这也有CMake这个跨平台makefile生成器的功劳。它使用一系列的```CMakeLists.txt```文件来生成平台指定的makefile。CMake被KDE项目所使用，它与Qt社区有一种特殊的关系。

```CMakeLists.txt```文件存储了项目配置。一个简单的hello world使用QtCore的项目如下：

```
// ensure cmake version is at least 3.0
cmake_minimum_required(VERSION 3.0)
// adds the source and build location to the include path
set(CMAKE_INCLUDE_CURRENT_DIR ON)
// Qt's MOC tool shall be automatically invoked
set(CMAKE_AUTOMOC ON)
// using the Qt5Core module
find_package(Qt5Core)
// create excutable helloworld using main.cpp
add_executable(helloworld main.cpp)
// helloworld links against Qt5Core
target_link_libraries(helloworld Qt5::Core)
```

这将使用main.cpp编译一个可执行的helloworld应用程序，并与额外的Qt5Core库链接。编译文件通常会被修改：

```
// sets the PROJECT_NAME variable
project(helloworld)
cmake_minimum_required(VERSION 3.0)
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
find_package(Qt5Core)

// creates a SRC_LIST variable with main.cpp as single entry
set(SRC_LIST main.cpp)
// add an executable based on the project name and source list
add_executable(${PROJECT_NAME} ${SRC_LIST})
// links Qt5Core to the project executable
target_link_libraries(${PROJECT_NAME} Qt5::Core)
```

CMake十分强大。需要一些时间来适应语法。通常CMake更加适合大型和复杂的项目。

**引用**

[CMake Help](http://www.cmake.org/documentation/) - CMake在线帮助文档

[Running CMake](http://www.cmake.org/runningcmake/)

[KDE CMake Tutorial](https://techbase.kde.org/Development/Tutorials/CMake)

[CMake Book](http://www.kitware.com/products/books/CMakeBook.html)

[CMake and Qt](http://www.cmake.org/cmake/help/v3.0/manual/cmake-qt.7.html)


