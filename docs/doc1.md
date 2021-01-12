---
id: doc1
title: 如何为安装 D435 深度相机驱动
sidebar_label: Ubuntu18.04realsense D435驱动安装 realsense D435深度相机
---

## 写在前面：亿些tips

1、此教程适用于 Ubuntu18.04 / Ubuntu20.04 系统。

2、原版安装教程（官网）：

(1)
## https://github.com/IntelRealSense/librealsense/blob/master/doc/installation.md

(2)
## https://github.com/IntelRealSense/librealsense/blob/master/doc/distribution_linux.md


3、因为原教程上面还是写的 Ubuntu14.04 ，所以对于部分还要进阶再升级的命令我没有跑，如果觉得不跑不安心的话可以回上面的原链接跑一遍（但是不排除以后会不会有坑）。



---
## 一、老规矩先更新

如果你的系统有一更新就炸的危险，建议再三考虑后再更新，或者只 update 不 upgrade 。

原链接里有更新内核和重新引导的过程，因为是针对 Ubuntu14.04 的版本的我就没跑（个人觉得没必要，当然不排除以后会有坑）。


```
sudo apt-get update

sudo apt-get upgrade
```



---

## 二、下载/克隆 librealsense github 存储库：

建议下第二条的版本


```
git clone https://github.com/IntelRealSense/librealsense.git

从master分支下载并解压缩最新的稳定版本https://github.com/IntelRealSense/librealsense/archive/master.zip
```


---
## 三、准备环境

1、记得拔摄像头

2、安装构建 librealsense 二进制文件和受影响的内核模块所需的核心软件包：

（原链接里的下面这一步我没跑，目前没有坑）


```
sudo apt-get install git libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev
```


（必须）特定于 Ubuntu18.04 的软件包：

```
sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at
```


3、从 librealsense 根目录运行 Intel RealSense 权限脚本：
```

cd /home/joyce/github/librealsense    //cd进你的librealsense目录

./scripts/setup_udev_rules.sh	    //注意最前面有个"."，下同

注意：始终可以通过运行以下命令删除权限：./scripts/setup_udev_rules.sh --uninstall
```



4、为以下应用程序构建和应用修补的内核模块：


具有 LTS 内核的 Ubuntu14/16/18 ：
```
./scripts/patch-realsense-ubuntu-lts.sh
```   
 
带有 Ubuntu 的 Intel®Joule™ 基于 Canonical Ltd. 提供的自定义内核。

这一步按道理应该不用跑，但是我跑了目前没坑：

```
./scripts/patch-realsense-ubuntu-xenial-joule.sh
```


5、TM1特定

跟踪模块需要 hid_sensor_custom 内核模块才能正常运行。由于TM1的加电顺序限制，引导期间需要加载此驱动程序，以正确初始化硬件。

为了做到这一点，司机的姓名添加hid_sensor_custom到/etc/modules文件，例如：
```
echo 'hid_sensor_custom' | sudo tee -a /etc/modules
```

---
## 四、编译 librealsense2 SDK

原链接有一个在 Ubuntu14.04 上将构建工具更新为 gcc-5 ，但是按道理 Ubuntu18.04 的 gcc 已经是 gcc-11 了，所以我觉得没必要，没跑且目前没坑。

```
mkdir build		//创建一个名为build的文件夹

cd build			//cd进去，注意这两步一定要在librealsense2目录下
```

运行cmake：

1、这一步将默认版本设置为在调试模式下生成核心共享库和单元测试二进制文件。用于 -DCMAKE_BUILD_TYPE=Release 优化构建。

```
cmake ../ 		
```


2、构建 librealsense 以及演示和教程
```
cmake ../ -DBUILD_EXAMPLES=true
```


3、这一步对于没有 OpenGL 或 X11 的系统，仅构建文本示例

```
cmake ../ -DBUILD_EXAMPLES=true -DBUILD_GRAPHICAL_EXAMPLES=false
```


4、重新编译并安装 librealsense 二进制文件：


共享对象将安装在 /usr/local/lib 中的头文件中 /usr/local/include 。

二进制演示，教程和测试文件将被复制到 /usr/local/bin 。


```
sudo make uninstall

make clean

sudo make install
```



5、make -jX 并行编译，X 代表 cpu 核心可用数量


这增强可能显著提高构建时间。但副作用是，它可能导致低端平台随机挂起。

（注意：默认情况下，Linux 构建配置当前配置为使用 V4L2 后端。

注意：如果在编译过程中遇到以下错误 gcc: internal compiler error 它可能表明您的计算机上没有足够的内存或交换空间。

尝试关闭消耗内存的应用程序，如果您正在 VM 中运行，请将可用 RAM 增加到至少 2 GB。）

```
sudo make uninstall

make clean

make -j8				//我的主机是make -j12，轻薄本是make -j4

sudo make install
```

------------------
接下来是第二条参考链接的教程，同样只跑了一部分，目前没坑。
## 五、亿些补充命令运行

1、注册公钥：

```
sudo apt-key adv --keyserver keys.gnupg.net --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
```


2、添加包仓库列表：

Ubuntu 18 LTS：
```
sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u
```


3、安装库以及开发包：
```
sudo add-apt-repository "deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main" -u
```


PS：某些电脑在安装过程中会出现 configure UEFI Secure Boot 的窗口，意思是开启了 UEFI Secure Boot ，而这会阻止你使用第三方的硬件驱动，不必更改这一模式，只需要按照窗口提示设定一个 Secure Boot 的密码确认，然后重启就会自动进入到 MOK 界面，选择第二项，yes之后输入刚才设定的密码，即可重新进入系统，此时 RealSense 的驱动也就安装好了。


此时插入摄像头，输入：
```
realsense-viewer
```
如果弹出来这样一个窗口，就说明暂时成功了。![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210108082517.png)

但是这个相机有个挺苟的问题，那就是他只能用 USB3.0 的 typeC 线，用普通的充电线是用不了的，像刚刚上面那张图就是用的普通充电线。

下面是 USB3.0 线的界面，会看到左边显示了参数：![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210108083140.png)


---
## 六、开始配置vscode

1、因为我有用到 Opencv，所以我有一个 cpp_properties.json 文件（其实就是放头文件的地方）。

我直接在原来的基础上加上了 librealsense2/ 。

建议直接在你自己原来的配置文件的 includePath 参数里加，不要直接复制下面的，下面给的仅供参考。

如果文件显示侧边栏没有（且你忘了怎么打开（划掉）：最上面的任务栏->运行（Debug）->打开配置（C）->会弹出来


```
{
    "configurations": [    
        {     
            "name": "Linux",            
            "includePath": [           
                "${workspaceFolder}/**",              
                "/usr/local/include/",              
                "/usr/local/include/opencv4/",              
                "/usr/local/include/opencv4/opencv2/",               
                "/usr/local/include/librealsense2/"	 /*深度学习相机realsense2 D435的库*                
            ],            
            "defines": [],            
            "compilerPath": "/usr/bin/gcc",            
            "cStandard": "c11",            
            "cppStandard": "c++11",            
            "intelliSenseMode": "gcc-x64"            
        }        
    ],   
    "version": 4    
}
```


2、tasks.json文件

把 librealsense2.so 文件的目录复制到 “args” 参数里。

如无例外，基本都在 /usr/local/lib 里面。

如果文件显示侧边栏没有（且你忘了怎么打开（划掉）： ctrl+shift+P ，然后输入： Task ->配置任务（中文）或者 Configure Task，一般都能找到。

```
{
    "version": "2.0.0",    
    "tasks": [    
        {       
            "label": "build",   /* 要与launch.json文件里的preLaunchTask的内容保持一致 */            
            "type": "shell",/* 定义任务是被作为进程运行还是在 shell 中作为命令运行，默认是shell，即是在终端中运行，因为终端执行的就是shell的脚本 */            
            "command": "g++",/* 这里填写你的编译器地址 */            
            "args": [            
                /* 说明整个项目所需的源文件路径(.cpp) */               
                "-g",                
                "${file}",                 
                "-std=c++11", // 静态链接               
                "-o",   /* 编译输出文件的存放路径 */               
                "${fileDirname}/${fileBasenameNoExtension}",/* 要与launch.json文件里的program的内容保持一致 */              
                "-I","${workspaceFolder}/armor_pre/",              
                //"-I","${workspaceFolder}/armor_test_avi/",               
                "-static-libgcc",                
                "-Wall",// 开启额外警告               
                /* 说明整个项目所需的头文件路径（.h）*/                
                "-I","${workspaceFolder}/",                
                "-I","/usr/local/include/",             
                "-I","/usr/local/include/opencv4/",              
                "-I","/usr/local/include/opencv4/opencv2/",             
                "/usr/local/lib/libopencv_*",   /* OpenCV的lib库 */               
                "/usr/local/lib/librealsense2.so" /*深度学习相机realsense2 D435的库*/                
            ]            
        }        
     ]     
}
```

---


测试代码：

参考链接：
## https://github.com/rcxxx/engineering_vision


人间奇迹现场：![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210112132431.png)




