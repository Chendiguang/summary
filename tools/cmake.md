# 记录在学习CMake的经历

## 入门

学习时遇到一篇比较棒的[CMake入门实践](https://www.hahack.com/codes/cmake/)文章

## 提高

* windows下使用cmake

    事先准备：安装cmake，MinGW，配置相应的环境变量。此外在vscode编辑器中开发调试时，需要安装相应的插件

    不同于Linux，直接在CMakeList文件目录下直接执行```cmake .```，在windows下需要执行```cmake -G "MinGW Makefiles" .```

* [参考例子](https://github.com/wzpan/cmake-demo/blob/master/Demo8/CMakeLists.txt)
