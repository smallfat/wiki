---
title: 文档
grammar_cjkRuby: true
---

# vooltdb当前所有OS编译环境的情况
### linux
- 编译方式：直接编译
- 编译环境地址：root@192.168.1.144:/home/vooltdb_package/linux
- 编译脚本地址：root@192.168.1.144:/home/vooltdb_package/linux/auto_tool/pkg.sh
- 上述脚本加入了cronjob，设为每天零时执行
- 取包地址：root@192.168.1.144:/home/vooltdb_package/linux/code/build_vd_minirel/programs/vooltdb

### macos
- 编译方式：交叉编译
- 编译环境地址：root@192.168.1.144:/home/vooltdb_package/macos
- 编译脚本地址：root@192.168.1.144:/home/vooltdb_package/macos/build/scripts/build.sh
- 取包地址：root@192.168.1.144:/home/vooltdb_package/macos/build/scripts/build/programs/clickhouse
- macos SDK和compiling tools地址:root@192.168.1.144:/home/vooltdb_package/macos/tools
- 注意：代码中把v8的部分都注释了，这部分需要v8编译对应OS的静态库，否则会报链接错误；因而脚本中也没有加入自动更新代码的部分

### aarch
- 编译方式：交叉编译
- 编译环境地址：root@192.168.1.144:/home/vooltdb_package/aarch
- 编译脚本地址：root@192.168.1.144:/home/vooltdb_package/aarch/build/scripts/build.sh
- 取包地址：root@192.168.1.144:/home/vooltdb_package/aarch/build/scripts/build/programs/clickhouse
- aarch SDK和compiling tools地址:root@192.168.1.144:/home/vooltdb_package/aarch/tools/arm-sdk/
- 注意：代码中把v8的部分都注释了，这部分需要v8编译对应OS的静态库，否则会报链接错误；因而脚本中也没有加入自动更新代码的部分

### freebsd
- 编译方式：freebsd虚拟机上进行编译，需要用下述目录处的虚拟机文件创建一个virtualbox freebsd虚拟机，然后在虚拟机内进行编译
- 虚拟机文件地址：root@192.168.1.144:/home/vooltdb_package/freebsd/FreeBSD-12.0-RELEASE-amd64.vhd
- freebsd虚拟机账户：/root/passwd
- 编译环境地址：/root/vooltdb_build/build
- 编译脚本地址：/root/vooltdb_build/build/build.sh
- 取包地址：/root/vooltdb_build/build/programs/clickhouse
- 注意：代码中把v8的部分都注释了，这部分需要v8编译对应OS的静态库，否则会报链接错误；因而脚本中也没有加入自动更新代码的部分

### windows
- 编译方式：mingw64
- 编译环境地址：192.168.0.141:E:\vooltdb_win_build\
- 编译脚本地址：192.168.0.141:E:\vooltdb_win_build\build\build.bat
- 取包地址：192.168.0.141:E:\vooltdb_win_build\build\package\vooltdb.exe

### 其他信息
- 在上述编译系统中，若用到了git账户作为自动更新用途，统一使用的是jasonliu/AAAAAAAA


# Windows移植
### 环境配置
- 当前选择使用MINGW64来对vooltdb进行windows版本的移植
- 推荐使用msys2安装MINGW64, 可以使用pacman安装gcc工具链
- gcc工具链版本: 对应当前vooltdb版本，要求是gcc9。可以从pacman历史版本库中找到安装包
- gdb版本: 与gcc版本相同，要注意clion2020.1最高支持gdb 8，因此clion内无法使用gdb进行调试（不稳定）。需要在mingw终端使用gdb进行调试。使用gdb9以上高版本的clion应该可以在IDE内进行调试
- 安装完mingw和pacman后，需要在pacman内安装libopenssl/libsystree等库，这些库在编译的过程中会用到

### 移植中碰到的问题
##### 第三方库编译
- 第三方库指：vooltdb/contrib下面的lib
- 总体来说，大部分第三方库都支持windows系统。由于我们采用mingw编译，有些库做了一些改动，详细改动见源码及变动注释。


##### 应用模块编译
###### GatherUtils模块编译问题
- 现象 - GatherUtils模块的特点是大部分都是template代码，其特点是具现化后，会生成大量代码。
- 这中间has.cpp是最有代表性的一个编译单元。无论以什么优化级别编译has.cpp，都会生成巨量的section（100000+），且section name很长（最长name大约1K），导致.obj文件section区超长。
- 解决办法 - 首先生成汇编代码has.cpp.s，修改其中section name为短格式，如此sections虽然多，但是不会超长，然后再生成has.cpp.obj。
- 修改.s汇编代码的小工具是我用golang写的convert-asm，源码与可执行文件均放在vooltdb_win/cmake/tool目录下
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


WriteBufferFromFile::WriteBufferFromFile(
    const std::string & file_name_,
    size_t buf_size,
    int flags,
    mode_t mode,
    char * existing_memory,
    size_t alignment)
    : WriteBufferFromFileDescriptor(-1, buf_size, existing_memory, alignment), file_name(file_name_)
{
#ifdef OS_WIN32
    UNUSED(mode);
    Poco::UnicodeConverter::convert(file_name, file_name_w);
#endif

#ifndef OS_WIN32
    fd = ::open(file_name.c_str(), flags == -1 ? O_WRONLY | O_TRUNC | O_CREAT | O_CLOEXEC : flags | O_CLOEXEC, mode);
#else
    fd = ::_wopen(file_name_w.c_str(), flags == -1 ? _O_BINARY | O_WRONLY | O_TRUNC | O_CREAT | O_NOINHERIT : flags | O_NOINHERIT, _S_IREAD | _S_IWRITE);
#endif

    if (-1 == fd)
        throwFromErrnoWithPath("Cannot open file ", file_name,
                               errno == ENOENT ? ErrorCodes::FILE_DOESNT_EXIST : ErrorCodes::CANNOT_OPEN_FILE);

}
```

###### Openssl库函数重定义问题
- 问题 - 在最后链接阶段，报“openssl 函数重定义”错误
- 解决方法 - 原因在于，系统中第三方库有openssl库，mingw64也自带openssl库。因此，必须指定一方为单一提供方。修改cmake脚本如下vooltdb_win/cmake/find./ssl.make：

```
option(ENABLE_SSL "Enable ssl" ${ENABLE_LIBRARIES})

if(ENABLE_SSL)
option(USE_INTERNAL_SSL_LIBRARY "Set to FALSE to use system *ssl library instead of bundled" FALSE)
set(USE_INTERNAL_SSL_LIBRARY 0)

```


###### Poco Network初始化问题
- 问题 - 整个系统编译及链接完成后，运行程序报"网络运行失败"错误
- 原因是，Poco库使用全局静态变量来进行网络初始化。这个方法在linux下是有效的。但是在mingw64/gcc编译后，该全局静态变量符号被优化掉了，因此在最后的链接中没有被链接到可执行程序内。
- 解决方法 - 直接在server::main内显式调用Poco::initializeNetwork()来初始化网络

```
  int Server::main(const std::vector<std::string> & /*args*/) {
        // call poco init network explicitly since global static NetworkInitializer do not work in Poco net lib
        // by jasonliu
        Poco::Net::initializeNetwork();

        Logger *log = &logger();
```


###### Windows对齐内存释放问题
- 问题 - 系统在编译链接完成后运行，直接crash
- 解决方法 - 经过分析代码和调试，原因是windows经过对齐分配的内存，不能直接简单free，而需要使用align_free()

```

  static Status ReallocateAligned(int64_t old_size, int64_t new_size, uint8_t** ptr) {
    uint8_t* previous_ptr = *ptr;
    if (previous_ptr == zero_size_area) {
      DCHECK_EQ(old_size, 0);
      return AllocateAligned(new_size, ptr);
    }
    if (new_size == 0) {
      DeallocateAligned(previous_ptr, old_size);
      *ptr = zero_size_area;
      return Status::OK();
    }
    // Note: We cannot use realloc() here as it doesn't guarantee alignment.

    // Allocate new chunk
    uint8_t* out = nullptr;
    RETURN_NOT_OK(AllocateAligned(new_size, &out));
    DCHECK(out);
    // Copy contents and release old memory chunk
    memcpy(out, *ptr, static_cast<size_t>(std::min(new_size, old_size)));
#ifdef _WIN32
    _aligned_free(*ptr);
#else
    free(*ptr);
#endif  // defined(_WIN32)
    *ptr = out;
    return Status::OK();
  }
```

###### Decimal128类型内存分配对齐问题
- 问题：MALLOC_MIN_ALIGNMENT的值默认是8，所以Decimal128分配内存时不一定以16字节对齐，而导致crash
- 解决方法：将MALLOC_MIN_ALIGNMENT的值设为16

```
void * Allocator<clear_memory_, mmap_populate>::realloc(void * buf, size_t old_size, size_t new_size, size_t alignment)
    {
        if (old_size == new_size)
        {
            /// nothing to do.
            /// BTW, it's not possible to change alignment while doing realloc.
        }
        else if (old_size < MMAP_THRESHOLD && new_size < MMAP_THRESHOLD
                 && alignment <= MALLOC_MIN_ALIGNMENT)
        {
            /// Resize malloc'd memory region with no special alignment requirement.
            CurrentMemoryTracker::realloc(old_size, new_size);

#ifdef OS_WIN32
            // here we force Allocator to allocate aligned memory on windows
            // cause we don't know how to free the mem : free or _aligned_free? It's not like linux
            // by jasonliu
            void * new_buf = _aligned_realloc(buf, new_size, MALLOC_MIN_ALIGNMENT);
......

``` 

# 使用Vooltdb自带的TA测试vooltdb windows版本
- 测试方法：linux 端运行测试脚本，向windows server发起请求
- 测试集位置
${vooltdb_project_root}/tests/queries/0_stateless
- 测试命令
	- 首先设置好环境变量 CLICKHOUSE_HOST/CLICKHOUSE_PORT_TCP，意思是目标主机vooltdb的IP/Port
	- 然后执行下述命令
 ```
 cd ${vooltdb_project_root}; 
 
 ./tests/clickhouse-test --timeout=3000 -j --no-zookeeper --no-shard --queries ./tests/queries --order=asc 
 ```
 
# Feature 9.2.1 Prestudy
### 在windows上自动识别时区
- vooltdb使用cctz库来处理日期，cctz库使用时区文件(zoneinfo)来处理时区。在windows上，系统没有维护当前哪个时区文件正在使用的信息。这与linux不同
- 可以使用GetTimeZoneInformation API取得当前时区（如中国是8），再将此数字转换为“Asia/Shanghai”
- 实验性代码已经commit到identify-timezone-auto branch

### 当config.xml中interserver-http-port/interserver-https-port为-1时，不启用此端口
可考虑在下述代码前判断当前config中interserver_http_port/interserver-https-port选项的值，若不为-1才执行。

```
        /// Server.cpp
		/// Interserver IO HTTP
            create_server("interserver_http_port", [&](UInt16 port)
            {
                Poco::Net::ServerSocket socket;
                auto address = socket_bind_listen(socket, listen_host, port);
                socket.setReceiveTimeout(settings.http_receive_timeout);
                socket.setSendTimeout(settings.http_send_timeout);
                servers.emplace_back(std::make_unique<Poco::Net::HTTPServer>(
                    createHandlerFactory(*this, async_metrics, "InterserverIOHTTPHandler-factory"), server_pool, socket, http_params));

                LOG_INFO(log, "Listening for replica communication (interserver): http://" + address.toString());
            });

            create_server("interserver_https_port", [&](UInt16 port)
            {
#if USE_POCO_NETSSL
                Poco::Net::SecureServerSocket socket;
                auto address = socket_bind_listen(socket, listen_host, port, /* secure = */ true);
                socket.setReceiveTimeout(settings.http_receive_timeout);
                socket.setSendTimeout(settings.http_send_timeout);
                servers.emplace_back(std::make_unique<Poco::Net::HTTPServer>(
                    createHandlerFactory(*this, async_metrics, "InterserverIOHTTPSHandler-factory"), server_pool, socket, http_params));

                LOG_INFO(log, "Listening for secure replica communication (interserver): https://" + address.toString());
#else
                UNUSED(port);
                throw Exception{"SSL support for TCP protocol is disabled because Poco library was built without NetSSL support.",
                        ErrorCodes::SUPPORT_IS_DISABLED};
#endif
            });
```


#  Feature - 支持1970年之前的日期
- 原理：在DateTime lookup 表中，将时间窗口从1970年向左滑动至1900年，滑动距离为2208988800秒
- 代码：
 
  ```
        //DateLUTImpl.cpp
	
	// feature 9.2: the date before 1970-01-01 should be supported - we move the time window left to 1900-01-01
	
    // by jasonliu
    cctz::civil_day date{1900, 1, 1};

    do
    {
        cctz::time_zone::civil_lookup lookup = cctz_time_zone.lookup(date);

        start_of_day = std::chrono::system_clock::to_time_t(lookup.pre) + TIME_WINDOW_OFFSET;    /// Ambiguity is possible.

        Values & values = lut[i];
        values.year = date.year();
        values.month = date.month();
        values.day_of_month = date.day();
        values.day_of_week = getDayOfWeek(date);
        values.date = start_of_day;


       // DataTypeDateTime.cpp
  
  void DataTypeDateTime::deserializeTextQuoted(IColumn & column, ReadBuffer & istr, const FormatSettings & settings) const {
    time_t x;
    if (checkChar('\'', istr)) /// Cases: '2017-08-31 18:36:48' or '1504193808'
    {
        ::readText(x, istr, settings, time_zone, utc_time_zone);
        assertChar('\'', istr);
    }
    else /// Just 1504193808 or 01504193808
    {
        readIntText(x, istr);
        x = DateLUTImpl::toTransformedTime(x);
    }
    assert_cast<ColumnType &>(column).getData().push_back(x);    /// It's important to do this at the end - for exception safety.}

```


