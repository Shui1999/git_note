<h1 style="text-align:center;">CMake&MakefileNote</h1>

##  gcc

### gcc编译

源文件(.cpp)编译生成目标文件(.o)

```sh
gcc -c main.cpp -o main.o
```

### gcc链接

目标文件(.o)链接生成可执行文件

```sh
gcc main.o -o main  # 有误
# 修复
gcc main.o -o main -lstdc++  #静态链接c++标准库
    
# 上述命令合并
gcc main.cpp -o main -lstdc++
 
# 可以使用g++编译
gcc main.cpp -o main
```

### c++编译流程

源文件(.cpp) ——》预处理(.i)——》编译(.s)——》汇编(.o)——》链接(.exe)

#### 预处理

将#include及宏等插入程序中，得到**.i**文件

```sh
g++ -E main.cpp -o main.i
```

#### 编译

将文本文件(**.i**)编译成**.s**的汇编文件

```sh
g++ -S main.i -o main.s
```

#### 汇编

将.s文件翻译成机器语言的二进制指令，并打包成一种称为**可重定位目标程序**的格式，并将结果保存为**.o**文件

```sh
g++ -c main.s -o main.o
```

#### 链接

将合并所有的**.o**文件，得到可执行文件

```sh
g++ main.o -o main
```

### 源文件包含头文件

可以进行类型检查（类型检查只在编译过程，链接过程不做）

### 头文件搜索路径(I)

```sh
g++ -I . xx.cpp main.cpp -o main
```

## Makefile

### 第一种写法

```makefile
target:
	g++ -I . xx.cpp yy.cpp main.cpp -o main

```

注意：

* target为目标名称，可以随意命名，可以包含多个目标，"make 目标名称"，不声明时默认执行第一个目标
* g++ 前必须tab缩进

### 第二种写法

```makefile
target: main
main:
	g++ -I . xx.cpp yy.cpp main.cpp -o main
%.o: %.cpp
	g++ -I . -c $< -o $@
	
# %.o: %.cpp表示所有的.o文件由.cpp文件生成
# <: 表示所有依赖的挨个值
# @: 表示所有目标的挨个值，均为"自动化变量"
```

```makefile
OBIS := xx.o yy.o main.o
target: $(OBJS) main
main:
	g++ -I . $(OBJS) -o main
	
%.o: %.cpp
	g++ -I . -c $< -o $@

# 删除文件
clean:
	rm -rf $(OBJS) main
```

注意

* "target: main"表示target目标依赖main目标
* 引用变量，加小、大括号，便于安全
* =是最基本的赋值
* ：=是覆盖之前的值
* ？=是如果没有被赋值就赋予等号之后的值
* +=是添加等号之后的值

## CMake

### CMake流程

* configure: 读取、解析并执行cmake语言
* generate: 生成makefiles或project files
* build： 编译、链接、测试生成可执行文件

指令

```sh
cmake -B build -G "MinGW Makefiles"
cmake --build build
    
# windows环境下cmake编译，无需创建buil文件夹
cmake -B build
cmake --build build
"以绝对路径执行exe文件"
    
"或者"
mkdir build
cd  build
cmake -G "MinGW Makefiles" ..
make
```

### 基础编译指令

```cmake
cmake_minimum_required(VERSION 3.20.0)
project(HelloWorld)
set(CMAKE_CXX_STANDARD 11)

# gdb调试
# set(CMAKE_BUILD_TYPE "Debug")
# set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
# set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")

# 设置编译输出路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)


# 生成静态库并编译
# include_directories(${PROJECT_SOURCE_DIR}/include)
# file(GLOB SRCS ${PROJECT_SOURCE_DIR}/src/*.cpp)
# set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/a)
# add_library(hello STATIC ${SRCS})
# add_executable(main main.cpp ${SRCS})

# 生成动态库并编译

include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRCS ${PROJECT_SOURCE_DIR}/src/*.cpp)
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/so)
add_library(hello SHARED ${SRCS})
add_executable(main main.cpp ${SRCS})

# 链接静态库并编译
# include_directories(${PROJECT_SOURCE_DIR}/include)
# link_directories(${PROJECT_SOURCE_DIR}/a)
# link_libraries(hello)
# add_executable(main main.cpp)

# 链接动态库并编译
# include_directories(${PROJECT_SOURCE_DIR}/include)
# link_directories(${PROJECT_SOURCE_DIR}/so)
# add_executable(main main.cpp)
# target_link_libraries(main PUBLIC hello)
# 注意在windows生成的动态库需和执行文件处于同一目录方可链接
```

### windows下vscode配置（launch.json和tasks.json）

```yaml
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++.exe - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/bin/main.exe",  //待调试的可执行文件路径
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "E:\\Learning\\MinGW\\bin\\gdb.exe",  //用以调试的gdb路径
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "Build"  //调试前执行的任务
        }
    ]
}
```

```yaml

{
	"version": "2.0.0",
	"options": {
		"cwd": "${workspaceFolder}/build"
	},
	"tasks": [
		{
			"type": "shell",
			"label": "cmake",
			"command": "cmake",
			"args": [
			 "-G",
			 "MinGW Makefiles",
			 ".."
			],
		},
		{
			"label": "make",
			"group": {
				"kind": "build",
				"isDefault": true
			},
			"command": "make",
			"args": [
				
			],
		},
		{
			"label": "Build",
			"dependsOn": ["cmake", 
			"make" ],
		},
	],
}

```

### linux下vscode配置

```yaml
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        
        {
            "name": "g++.exe - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/bin/main",  //待调试的可执行文件路径
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "/usr/bin/gdb",  //用以调试的gdb路径
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "Build"  //调试前执行的任务
        }
    ]
}
```

```yaml

{
	"version": "2.0.0",
	"options": {
		"cwd": "${workspaceFolder}/build"
	},
	"tasks": [
		{
			"type": "shell",
			"label": "cmake",
			"command": "cmake",
			"args": [
			 ".."
			],
		},
		{
			"label": "make",
			"group": {
				"kind": "build",
				"isDefault": true
			},
			"command": "make",
			"args": [
				
			],
		},
		{
			"label": "Build",
			"dependsOn": ["cmake", 
			"make" ],
		},
	],
}

```

### CMake常用命令

```cmake
aux_source_directory(${PROJECT_SOURCE_DIR} SRC)  第一个参数传递文件目录，会扫描所有的源文件，赋值到第二个变量中

set(CMAKE_EXE_LINKER_FLAGS "-static")  # 设置全局静态库链接

include_directories(${PYTHONHOME}/include) #包含头文件目录,但范围内有效，最好放置在 add_subdirectory() 之前便于传递
    
link_directories(${PYTHONHOME}/libs) # 包含链接库目录，同上
    
add_library(${PROJECT_NAME} SHARED process.cpp) # 生成库，STATIC表示静态库  
    
list(APPEND CMAKE_MODULE_PATH "${EIGEN3_INCLUDE_DIR}//cmake")
    
file(GLOB SRC "${PROJECT_SOURCE_DIR}/*.cpp") #如果GLOB换成GLOB_RECURSE，将递归搜索目录而不仅是当前层级

add_definitions() # 添加宏定义

    
target_link_libraries(cv_plot_demo ${OpenCV_LIBS} Python3::Python Python3::NumPy) 使用PUBLIC或PRIVATE进行权限控制保证搜索路径不导出
    
execute_process(COMMAND git clone https://github.comxxxx.git
               WORKING_DIRECTORY ${DIR}) //执行外部操作，如git

option 用于快速设置定义变量并赋值为对应的bool值 option(变量名 描述 值)
    

# 包模块管理，使用FetchContentmok
    
include(FetchContent)  #引入功能模块

FetchContent_Declare(
				google_test  //项目名称
    			GIT_REPOSITORY https://github.com/xxx.git  //仓库地址
    			GIT_TAG v.16.2  //仓库版本
    			GIT_SHALLOW TRUE  //是否指拉取最新的记录
)

FetchContent_MakeAvailable(google_test) 
#直接链接
add_executable(main main.cpp)
target_link_libraries(main google_test)
```



## C++调用Python

将python作为一个库来调用，检测python安装目录下是否已经包含头文件和库文件

* 头文件：Python.h;                 路径：/Python3.x/include
* 库文件：python3x_d.lib;       路径：/Python3x/libs

验证环境代码

```c++
#include "Python.h"
#include "iostream"
int main() {
    Py_SetPythonHome(L"E:\\Learning\\Anaconda");
    Py_Initialize(); //初始化
    if(!Py_IsInitialized())
    {
        std::cout<< "Python init failed!"<<std::endl;
        return -1;
    }
    PyRun_SimpleString("print('Hello Python!')");
    Py_Finalize(); //释放资源
    return 0;
}

```

```cmake
cmake_minimum_required(VERSION 3.27)
project(CxxAndPython)

set(CMAKE_CXX_STANDARD 11)

set(PYTHONHOME "E:\\Learning\\Anaconda")
include_directories(E:\\Learning\\Anaconda\\include)

link_libraries(E:\\Learning\\Anaconda\\libs)


find_package(Python3 COMPONENTS Interpreter Development REQUIRED)

add_executable(CxxAndPython main.cpp)
target_link_libraries(CxxAndPython Python3::Python)
```

### c++调用python无参函数

1. 初始化函数接口（Py_Initialize)
2. 初始化python系统文件路径（PyRun_SimpleString)
3. 调用python文件名（PyImport_ImportModule)
4. 获取函数对象 (PyObject_GetAttrString)
5. 调用函数对象（Pyobject_CallObject）
6. 结束python接口初始化（Py_Finalize)

**example**

```c++
/*
script/test_1.py
def say():
    print("Hello Python!")
*/


#include "Python.h"
#include "iostream"
int main() {
    Py_SetPythonHome(L"E:\\Learning\\Anaconda");
    //初始化python接口
    Py_Initialize(); //初始化
    if(!Py_IsInitialized())
    {
        std::cout<< "Python init failed!"<<std::endl;
        return -1;
    }

    //初始化python系统化文件路径
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('../script')");

    //调用python文件名，不用写后缀
    PyObject* module = PyImport_ImportModule("test_1");
    if (module == nullptr)
    {
        std::cout<<"module not found!"<<std::endl;
        return -1;
    }

    //获取函数对象
    PyObject* func = PyObject_GetAttrString(module, "say");
    if (!func || !PyCallable_Check(func))
    {
        std::cout<<"function not found!"<<std::endl;
        return -1;
    }

    // 调用函数
    PyObject_CallObject(func, nullptr);

    Py_Finalize(); //释放资源
    return 0;
}

```

### c++调用python有参函数

1.  初始化函数接口（Py_Initialize)
2. 初始化python系统文件路径（PyRun_SimpleString)
3. 调用python文件名（PyImport_ImportModule)
4. 获取函数对象 (P yObject_GetAttrString)
5. 传递参数（PyObject_New, Py——BuildValue)
6. 调用函数对象（PyObject_CallObject)
7. 接收函数返回值（PyArg_Parse)
8. 结束python接口初始化（Py_Finalize)

**example**

```c++
#include "Python.h"
#include "iostream"
int main() {
    Py_SetPythonHome(L"E:\\Learning\\Anaconda");
    Py_Initialize(); //初始化
    if(!Py_IsInitialized())
    {
        std::cout<< "Python init failed!"<<std::endl;
        return -1;
    }

    //初始化python系统化文件路径
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('../script')");

    //调用python文件名，不用写后缀
    PyObject* module = PyImport_ImportModule("test_2");
    if (module == nullptr)
    {
        std::cout<<"module not found!"<<std::endl;
        return -1;
    }

    //获取函数对象
    PyObject* func = PyObject_GetAttrString(module, "add");
    if (!func || !PyCallable_Check(func))
    {
        std::cout<<"function not found!"<<std::endl;
        return -1;
    }

    //给python函数传递参数
    //函数调用的参数传递均是以元组形式打包，2表示参数个数
    //如果函数参数只有一个时，写1即可
    PyObject* args = PyTuple_New(2);

    //0：第一个参数传入 int 类型的值为 1
    //1：第二个参数传入 int 类型的值为 2
    PyTuple_SetItem(args, 0, Py_BuildValue("i", 1));
    PyTuple_SetItem(args, 1, Py_BuildValue("i", 2));

    //使用c++的python接口调用函数
    PyObject* ret = PyObject_CallObject(func, args);

    //接收python计算好的返回值
    int result;

    // i 表示 int 类型
    //需要注意：PyArg_Parse的最后一个参数，必须加上“&”符号
    PyArg_Parse(ret, "i", &result);

    std::cout<<"The result is "<<result<<" !"<<std::endl;
   

    // 调用函数
//    PyObject_CallObject(func, nullptr);

    Py_Finalize(); //释放资源
    return 0;
}

```

### c++调用python类

**example**

```c++
#include "Python.h"
#include "iostream"
int main() {
    Py_SetPythonHome(L"E:\\Learning\\Anaconda");
    Py_Initialize(); //初始化
    if(!Py_IsInitialized())
    {
        std::cout<< "Python init failed!"<<std::endl;
        return -1;
    }

    //初始化python系统化文件路径
    PyRun_SimpleString("import sys");
    PyRun_SimpleString("sys.path.append('../script')");

    //调用python文件名，不用写后缀
    PyObject* module = PyImport_ImportModule("test_3");
    if (module == nullptr)
    {
        std::cout<<"module not found!"<<std::endl;
        return -1;
    }

    //获取类
    PyObject* cls = PyObject_GetAttrString(module, "Person");
    if (!cls)
    {
        std::cout<<"class not found!"<<std::endl;
        return -1;
    }

    //给类初始化传递参数
    //函数调用的参数传递均是以元组形式打包，2表示参数个数
    //如果函数参数只有一个时，写1即可
    PyObject* args = PyTuple_New(2);

    //0：第一个参数传入 int 类型的值为 1
    //1：第二个参数传入 int 类型的值为 2
    PyTuple_SetItem(args, 0, Py_BuildValue("s", "jack"));
    PyTuple_SetItem(args, 1, Py_BuildValue("i", 18));

    //根据类名实例化对象
    PyObject* obj = PyObject_CallObject(cls, args);

    //根据对象得到成员函数
    PyObject* cls_func = PyObject_GetAttrString(obj, "foo");
    if (!cls_func || !PyCallable_Check(cls_func))
    {
        std::cout<<"function not found!"<<std::endl;
        return -1;
    }

    PyObject_CallObject(cls_func, nullptr);



    Py_Finalize(); //释放资源
    return 0;
}

```





