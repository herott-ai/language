# hero

## **什么是cmake**

你或许听过好几种 Make 工具，例如 GNU Make ，QT 的 qmake ，微软的 MSnmake，BSD Make（pmake），Makepp，等等。这些 Make 工具遵循着不同的规范和标准，所执行的 Makefile 格式也千差万别。这样就带来了一个严峻的问题：如果软件想跨平台，必须要保证能够在不同平台编译。而如果使用上面的 Make 工具，就得为每一种标准写一次 Makefile ，这将是一件让人抓狂的工作。
CMake 就是针对上面问题所设计的工具：**它首先允许开发者编写一种平台无关的 CMakeList.txt 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 工程。从而做到“Write once, run everywhere”**。显然，CMake 是一个比上述几种 make 更高级的编译配置工具。一些使用 CMake 作为项目架构系统的知名开源项目有 VTK、ITK、KDE、OpenCV、OSG 等。

在 linux 平台下使用 CMake 生成 Makefile 并编译的流程如下：

1. 编写 CMake 配置文件 CMakeLists.txt 。
2. 执行命令 cmake PATH 或者 ccmake PATH 生成 Makefile。其中， PATH 是 CMakeLists.txt 所在的目录。（ccmake 和 cmake 的区别在于前者提供了一个交互式的界面）
3. 使用 make 命令进行编译。

## cmake入门

### 编译单个源文件

main.cpp所在目录是`/hero`。

```c++
#include<iostream>
using namespace std;
int main()
{
        int a,b;
        cin>>a>>b;
        cout<<a+b<<endl;
}
```

CMakeLists.txt 文件与 main.cpp 源文件同个目录下。

```cmake
Cmake_minimum_required(VERSION 3.1)
project (hero)
add_executable(ryy main.cpp)
```

> 命令大小写是无所谓的，但是参数必须按照指定的规则，比如VERSION必须大写。

然后在当前目录执行`cmake .`，然后再执行`make`即可。

### 编译多个源文件

.
├── add.cpp
├── CMakeLists.txt
└── main.cpp

此时我们将两数相加这一操作封装成一个函数add.cpp。

`main.cpp`

```c++
#include<iostream>
using namespace std;
int add(int,int);    //这个声明是必须有的，否则make时报错
int main()
{
        int a,b;
        cin>>a>>b;
        cout<<add(a,b)<<endl;
}
```

此时CMakeLists.txt的改动也是不大的，只需要将add.cpp添加到add_executable命令中即可。

`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.1)
project (hero)
add_executable(ryy main.cpp add.cpp)
```

但是如果有大量源文件，我们要手动把每一个源文件都加入进去嘛？

更省事的方法是使用 `aux_source_directory` 命令，该命令会**查找指定目录下的所有源文件**，**然后将结果存进指定变量名**。其语法如下：

```cmake
aux_source_directory(<dir> <variable>)
```

新的`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.1)
project (hero)
aux_source_directory(. RES)
add_executable(ryy ${RES})
```

> 花括号是必须有的。并且不能换成()。

这样，CMake 会将当前目录所有源文件的文件名赋值给变量 RES ，再指示变量 RES 中的源文件需要编译成一个名称为 ryy 的可执行文件。

### **多个目录，多个源文件**

.
├── CMakeLists.txt
├── main.cpp
└── math
    ├── add.cpp
    └── add.hpp

现在将add函数放入子目录math中，并在add.hpp中书写函数声明。



对于这种情况，需要分别在项目根目录 /hero 和 /hero/math 目录里各编写一个 CMakeLists.txt 文件。为了方便，我们可以先将 math 目录里的文件编译成静态库再由 main 函数调用。

`MakeLists.txt `

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 3.1)
# 项目信息
project (hero)
# 查找当前目录下的所有源文件
# 并将名称保存到 RES 变量
aux_source_directory(. RES)
# 添加 math 子目录
add_subdirectory(math)
# 指定生成目标
add_executable(ryy ${RES})
# 添加链接库
target_link_libraries(ryy Math)
```

该文件添加了下面的内容: 第4行，使用命令 add_subdirectory 指明本项目包含一个子目录 math，**这样 math 目录下的 CMakeLists.txt 文件`和`源代码也会被处理** 。**第6行，使用命令 target_link_libraries 指明可执行文件 ryy 需要链接一个名为 Math 的链接库** 。



子目录中的 `CMakeLists.txt`：

```cmake
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_LIB_SRCS 变量
aux_source_directory(. DIR_LIB_SRCS)
# 生成链接库
add_library (Math ${DIR_LIB_SRCS})
```

在该文件中使用命令 `add_library` 将 math 目录中的源文件编译为静态链接库。



`main.cpp`

```cpp
#include <iostream>
using namespace std;
#include "./math/add.hpp"
int main()
{
        int a,b;
        cin>>a>>b;
        cout<<add(a,b)<<endl;
}
```

必须写全路径`#include "./math/add.hpp"`。为什么以后再研究。

### **自定义编译选项**

CMake **允许为项目增加编译选项，从而可以根据用户的环境和需求选择最合适的编译方案**。

例如，可以将 Math 库设为一个可选的库，如果该选项为 ON ，就使用该库定义的数学函数来进行运算。否则就调用标准库中的数学函数库。
修改 CMakeLists 文件，我们要做的第一步是在顶层的 CMakeLists.txt 文件中添加该选项：

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 3.1)
# 项目信息
project (hero)
# 加入一个配置头文件，用于处理 CMake 对源码的设置
configure_file (
"${PROJECT_SOURCE_DIR}/config.h.in"
"${PROJECT_BINARY_DIR}/config.h"
)
# 是否使用自己的 Math 库
option (USE_MYMATH
"Use provided math implementation" ON)
# 是否加入 Math 库
if (USE_MYMATH)
include_directories ("${PROJECT_SOURCE_DIR}/math")
add_subdirectory (math)
set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)
# 查找当前目录下的所有源文件
# 并将名称保存到 DIR_SRCS 变量
aux_source_directory(. DIR_SRCS)
# 指定生成目标
add_executable(Demo ${DIR_SRCS})
target_link_libraries (Demo ${EXTRA_LIBS})
```



