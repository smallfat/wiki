---
title: Vooltdb windows版本移植详情
tags: 
grammar_cjkRuby: true
---
## 工具
- 推荐使用msys2安装MINGW64, 可以使用pacman安装gcc工具链
- gcc工具链版本: 对应当前vooltdb版本，要求是gcc9
- gdb版本: 与gcc版本相同，要注意clion2020.1最高支持gdb 8，因此clion内无法使用gdb进行调试（不稳定）。需要在mingw终端使用gdb进行调试。


## 移植情况
#### 第三方库编译
- 第三方库指：vooltdb/contrib下面的lib
- 总体来说，大部分第三方库都支持windows系统。由于我们采用mingw编译，有些库做了一些改动，详细改动见源码及变动注释。


#### 应用模块编译
###### GatherUtils模块编译问题
- 现象 - GatherUtils模块的特点是大部分都是template代码，其特点是具现化后，会生成大量代码。
- 这中间has.cpp是最有代表性的一个编译单元。无论以什么优化级别编译has.cpp，都会生成巨量的section（100000+），且section name很长（最长name大约1K），导致.obj文件section区超长。
- 解决办法 - 首先生成汇编代码has.cpp.s，修改其中section name为短格式，如此sections虽然多，但是不会超长，然后再生成has.cpp.obj。
- 修改.s汇编代码的小工具是我用golang写的convert-asm，放在vooltdb_win/cmake/tool目录下
- 因为上述修改属于hack，因此需要一个独立编译脚本。此脚本位于src/Function/GatherUtils/cmake_has/has_lib.cmake。
```
add_custom_target(has_lib
        COMMAND ${CMAKE_CXX_COMPILER} ${CMAKE_CXX_FLAGS_SEPARATE} ${AS_BIG_OBJ} ${HAS_INCLUDES} -o ${HAS_ASM_PATH} -S ${HAS_CPP_PATH}
#        COMMAND ${ASM_OPTIMIZE_TOOL} ${HAS_ASM_PATH} ${HAS_ASM_PATH_OPTIMIZE}
#        COMMAND ${CMAKE_CXX_COMPILER} ${CMAKE_CXX_FLAGS_SEPARATE} ${AS_BIG_OBJ} ${HAS_INCLUDES} -o ${HAS_OBJ_PATH} -c ${HAS_ASM_PATH_OPTIMIZE}
)
```

###### AggerateFunction模块编译问题
- 问题 - 该模块的问题与GatherUtils模块相同，但是解决方法不同。
- 解决方法 - 将相关问题的单个编译单元拆分为多个编译单元，比如AggregateFunctionMinMaxAny.cpp拆分为AggregateFunctionMinMaxAny1.cpp /AggregateFunctionMinMaxAny2.cpp /AggregateFunctionMinMaxAny3.cpp


###### BaseDaemon重构
- 问题 - vooltdb linux版本中，Server等Application都继承自BaseDaemon。BaseDaemon实现了基于信号机制的App控制。
- 解决方法 - Windows版本中，保留了对console的Ctrl+C控制。

```

void BaseDaemon::handleKeyboardSignal(int signal_id)
{
    handleSignal(signal_id);
}

void BaseDaemon::handleSignal(int signal_id)
{
    if (signal_id == CTRL_C_EVENT ||
        signal_id == CTRL_CLOSE_EVENT ||
        signal_id == CTRL_LOGOFF_EVENT ||
        signal_id == CTRL_SHUTDOWN_EVENT)
    {
        std::unique_lock<std::mutex> lock(signal_handler_mutex);
        {
            ++terminate_signals_counter;
            sigint_signals_counter += signal_id == SIGINT;
            signal_event.notify_all();
        }

    }
}
```

###### 系统编码问题 - unicode与utf8
- 问题 - windows 系统的编码是GBK或者unicode宽字符编码，而linux的编码是utf8编码，这意味着，中文路径在原始源码中就无法使用。因此，在移植过程中，我们需要解决这个问题。
- 解决方法 - 首先需要区分数据编码与系统编码。数据编码我们还是保持不变，比如写入文件的数据内容，从文件读出的二进制流等。
- 其次，像文件路径的编码，我们需要从utf8转换为unicode。转换代码见下附源码。
- 最后，配置文件等，我们还是保持utf8编码，以保持与linux平台兼容

```

```

###### Openssl库函数重定义问题
###### Poco Network初始化问题
###### Windows对齐内存释放问题
###### "undefined reference for typeinfo to xxx"符号未生成问题


###### 其他问题


## 总结
