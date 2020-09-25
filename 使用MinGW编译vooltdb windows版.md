---
title: 使用MinGW编译vooltdb windows版
tags: 
grammar_cjkRuby: true
---

# 准备工作
- 安装cmake/MinGW64/Clion等必备软件，并做好配置，无需多说
- 推荐使用msys2安装MINGW64, 可以使用pacman安装gcc工具链
- gcc版本: 要求是gcc9
- gdb版本: 与gcc版本相同，要注意clion 2020.1最高支持gdb 8，因此clion内无法使用gdb进行调试。需要在mingw客户端使用gdb console进行调试。
```

# 开始编译
### 编译思路
###### vooltdb编译系统的组织
- vooltdb的编译系统是基于cmake搭建的
- vooltdb自组织了一套树状的CMakeLists.txt编译配置。需要特别提到的，对于部分第三方库，vooltdb没有使用库自带的CMakeLists.txt配置，而是自己写了新的CMakeLists.txt
- 根CMakeLists.txt位于VooltDB根目录下，读编译配置从这个配置开始

###### 编译原则
- 先编译libs，再编译application：将根CMakeLists.txt里application编译的部分暂时先屏蔽
- 保证CMake生成targets能够成功，这样可以选择编译哪些targets
- 先编译难度较高的libs
- 改动代码的过程中，尽量做到平台区分，方便之后的合并工作
- 尽量仔细思考问题代码，而不是只求编译通过

### 问题
###### 问题分类
- 总的来说，分为编译系统问题和lib库问题
- 编译系统问题：主要是OS平台相关问题 - 因为原来的CMakeLists.txt是基于Linux编写的，没有考虑windows平台的情况，需要我们去修改相应的配置
- Lib库问题：主要是OS平台相关问题；
###### 具体例子
- Example 1 - On Windows platform, null INPUT_FILE in command execute_process should be NUL instead of /dev/null
```
if (WIN32)
    execute_process(COMMAND ${CMAKE_C_COMPILER} ${EXEC_ARGS}
            OUTPUT_QUIET ERROR_QUIET
            INPUT_FILE NUL
            RESULT_VARIABLE GNUCC_TUNE_TEST)
else()
    execute_process(COMMAND ${CMAKE_C_COMPILER} ${EXEC_ARGS}
            OUTPUT_QUIET ERROR_QUIET
            INPUT_FILE /dev/null
            RESULT_VARIABLE GNUCC_TUNE_TEST)
endif ()

```

- Example 2 - 预编译中宏替换导致的问题
 ```
error: 'const class Aws::Client::AWSError<Aws::Client::CoreErrors>' has no member named 'GetMessageA'; did you mean 'GetMessage'?
```

source code:

```
auto newError = AWSError<CoreErrors>(outcome.GetError().GetErrorType(), outcome.GetError().GetExceptionName(), outcome.GetError().GetMessage(), true); 
```

原因：在某个文件中，有如下定义：
```
#define GetMessage GetMessageA
```

- Example 3 - 不同平台上需要添加不同的源文件
```
if (WIN32)
file(GLOB AWS_CORE_SOURCES
        #"${AWS_CORE_LIBRARY_DIR}/source/linux-shared/*.cpp"
        "${AWS_CORE_LIBRARY_DIR}/source/platform/windows/*.cpp"
        ${AWS_CORE_SOURCES}
        )
else()
file(GLOB AWS_CORE_SOURCES
        #"${AWS_CORE_LIBRARY_DIR}/source/linux-shared/*.cpp"
        "${AWS_CORE_LIBRARY_DIR}/source/platform/linux-shared/*.cpp"
        ${AWS_CORE_SOURCES}
        )
endif ()

```

- Example 4 - 8. 在windows中不允许创建符号链接
```
# create a symlink to include headers with <avro/...>
if (NOT OS_WIN32)
	...
	COMMAND ${CMAKE_COMMAND} -E create_symlink ${AVROCPP_ROOT_DIR}/api ${AVROCPP_ROOT_DIR}/include/avro)
endif ()
```

# 后记
- OS平台相关的issue比较多
- linux平台往windows平台迁移过程中，有个问题虽然现在还没碰到，后期整系统联调可能会遇到：字符集编码问题。linux采用utf8编码，windows采用unicode编码
- 目前编译lib还比较顺利，但编译通过不等于功能正常。编译通过后的功能正常才是重点。