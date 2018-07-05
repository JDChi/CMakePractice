# 写在前面
对CMake Practice这本书里的例子进行实践，因为书中有部分内容可能因时间原因写法上有了调整，故不符合现在。

# 例子1（内部构建）
CMakeLists.txt
```
# 指定工程名称
PROJECT (HELLO)
# 显示定义变量，如果有多个源文件也可写成SET(SRC_LIST main.c t1.c t2.c)
SET(SRC_LIST main.c)
# HELLO就是工程的名称，建议还是用PROJECT_BINARY_DIR，这个比较通用
MESSAGE(STATUS "This is BINARY dir" ${HELLO_BINARY_DIR})
# MESSAGE指令包含SEND_ERROR（产生错误，生成过程被跳过） SATUS（输出前缀为-的信息） FATAL_ERROR（立即终止所有cmake的过程）三种类型
MESSAGE(STATUS "This is SOURCE dir " ${HELLO_SOURCE_DIR})

# 书中SRC_LIST没有添加引用符号，导致出错 
# 定义这个工程会生成一个文件名为hello的可执行文件
ADD_EXECUTABLE(hello ${SRC_LIST}) 
```

main.c
```c
#include <stdio.c>
int main(){
	printf("Hello World from test1 Main!\n");
	return 0;
}
```

执行```cmake . ```后输出
> -- The C compiler identification is AppleClang 9.1.0.9020039  
> -- The CXX compiler identification is AppleClang 9.1.0.9020039  
> -- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc  
> -- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc -- works  
> -- Detecting C compiler ABI info  
> -- Detecting C compiler ABI info - done  
> -- Detecting C compile features  
> -- Detecting C compile features - done  
> -- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++  
> -- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++ -- works  
> -- Detecting CXX compiler ABI info  
> -- Detecting CXX compiler ABI info - done  
> -- Detecting CXX compile features  
> -- Detecting CXX compile features - done  
> -- This is BINARY dir /Users/jdnew/develop_workspace/cmake-practice/test1  
> -- This is SOURCE dir /Users/jdnew/develop_workspace/cmake-practice/test1  
> -- Configuring done  
> -- Generating done  
> -- Build files have been written to: /Users/jdnew/develop_workspace/cmake-practice/test1

## 目录结构

```
test1
|    main.c
|    CMakeLists.txt
```

# 例子2（外部构建）

CMakeLists.txt
```
PROJECT (HELLO)

# ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL]) 
# 用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置。
ADD_SUBDIRECTORY(src bin)

INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake/test2 )
INSTALL(PROGRAMS runhello.sh DESTINATION bin)
INSTALL(DIRECTORY doc/ DESTINATION share/doc/cmake/test2)
```

src/CMakeLists.txt
```
ADD_EXECUTABLE(hello main.c)
```

## 目录结构
```
test2
|    build
|    src
|    |- main.c
|    |- CMakeLists.txt
|    CMakeLists.txt
|    doc
|    |- hello.txt
|    README
|    COPYRIGHT
|    runhello.sh
```

# 例子3（静态库与动态库构建）
## 目录结构
```
test3
|    CMakeLists.txt
|    lib
|    |- CMakeLists.txt
|    |- hello.c
|    |- hello.h
|    build
```

src/CMakeLists.txt
```
SET(LIBHELLO_SRC hello.c)
# 在Mac上编译出来的共享库是以dylib结尾，Linux上才是so
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
# ADD_LIBRARY方法下的target是唯一，所以不能同名
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
# 所以要使用SET_TARGET_PROPERTIES方法里的OUTPUT_NAME属性来设置输出后的名字
SET_TARGET_PROPERTIES(hello_static PROPERTIES OUTPUT_NAME "hello")

SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)
# 添加版本号 VERSION指代动态库版本，SOVERSION指代API版本
SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)
# 安装共享库和头文件 静态库使用ARCHIVE关键字
INSTALL(TARGETS hello hello_static
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
INSTALL(FILES hello.h DESTINATION include/hello)
```

```cmake -DCMAKE_INSTALL_PREFIX=/usr ..```命令要改成```cmake -DCMAKE_INSTALL_PREFIX=/usr/local ..```，因为Mac上后来更改了权限，对于开发者而言的权限是在/usr/local目录下

# 例子4
## 目录结构
```
test4
|    CMakeLists.txt
|    src
|    |- main.c
|    |- CMakelist.txt
|    build
```

src/CMakeLists.txt
```
ADD_EXECUTABLE(main main.c)
# 添加头文件搜索路径
INCLUDE_DIRECTORIES(/usr/local/include/hello)
# 链接共享库
TARGET_LINK_LIBRARIES(main hello)
```