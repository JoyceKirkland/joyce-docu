---
id: doc1
title: ubuntu18.04 ( 20 通用 )安装 REALSENSE D435 深度相机驱动
sidebar_label: ubuntu18.04realsense D435驱动安装 realsense D435深度相机
---

## 亿些tips
原版安装教程（官网）：

https://github.com/IntelRealSense/librealsense/blob/master/doc/installation.md

因为原教程上面还是写的 ubuntu14.04 ，所以对于部分还要进阶再升级的命令我并没有跑，如果觉得不跑不安心的话可以回上面的原链接跑一遍（因为没跑也是能用的，但是不排除以后会不会有坑）
这一篇是为了避免再次花费时间去研究原教程而写，因为要结合自身实际嘛。
在跑完原教程后我还参考这个跑了一部分命令：

https://blog.csdn.net/sinat_36502563/article/details/89174282

因为我在跑完原链接之后完全不知道要怎么开始，因此去找到了第二个教程，命令有些类似，因此无法判断第二个链接的内容是不是必须跑的（我认为是必须），但是至少第二条跑完一部分之后我看到了一些结果。
欢迎大佬指正！


---
## 一、老规矩先更新

如果你的系统有一更新就炸的危险，建议再三考虑后再更新，或者只 update 不 upgrade 。



```
sudo apt-get update

sudo apt-get upgrade
```


原链接里有更新内核和重新引导的过程，因为是针对 ubuntu14.04 的版本的我就没跑（个人觉得没必要，当然不排除以后会有坑）

---

## 二、下载/克隆 librealsense github 存储库：

这里我只跑了第一条orz没看到第二条上面写的最新版本，这里建议下第二条的版本。


```
git clone https://github.com/IntelRealSense/librealsense.git

从master分支下载并解压缩最新的稳定版本https://github.com/IntelRealSense/librealsense/archive/master.zip
```


---
## 三、准备环境

1、记得拔摄像头

2、安装构建 librealsense 二进制文件和受影响的内核模块所需的核心软件包：

（原链接里的下面这一步我没跑，我直接跑了特定 18 的版本去了，目前没有坑）


```
sudo apt-get install git libssl-dev libusb-1.0-0-dev pkg-config libgtk-3-dev
```


下面这一步必须跑，这是特定于 ubuntu18 的软件包：

```
sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at
```



原链接里有个 cmake 注意，同样没跑这一步目前没坑

cmake 注意：某些 librealsense CMAKE 标志（例如 CUDA ）要求版本 3.8+ ，而该版本目前无法通过 apt Manager 获得 Ubuntu LTS 的支持。

转到官方CMake网站下载并安装该应用程序

原链接原话注意：关于图形子系统利用率的注意事项：如果您计划构建 SDK 的启用 OpenGL 的示例，则需要 glfw3,mesa 和 gtk 软件包。该 librealsense 核心库和一系列演示/工具是专为无头环境中进行部署。



3、从 librealsense 根目录运行 Intel Realsense 权限脚本：
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

上面的脚本将下载，修补和构建受真实感影响的内核模块（驱动程序）。

然后它将尝试插入打补丁的模块，而不是活动模块。

如果失败，将还原原始 uvc 模块。

原链接有一个基于 Arch 的发行版还有一个带有 ubuntu16.04 的一个 Odroid XU4 图像，同样没跑目前没坑



5、TM1特定

跟踪模块需要 hid_sensor_custom 内核模块才能正常运行。由于TM1的加电顺序限制，引导期间需要加载此驱动程序，以正确初始化硬件。

为了做到这一点，司机的姓名添加hid_sensor_custom到/etc/modules文件，例如：
```
echo 'hid_sensor_custom' | sudo tee -a /etc/modules
```

---
## 四、编译 librealsense2 SDK

原链接有一个在 ubuntu14.04 上将构建工具更新为 gcc-5 ，但是按道理 ubuntu18.04 的 gcc 已经是 gcc-11 了，所以我觉得没必要，没跑且目前没坑。

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

原链接内容：

（这增强可能显著提高构建时间。但副作用是，它可能导致低端平台随机挂起。

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


csdn 链接原话：至此，驱动包安装完成，使用以下命令可以打开 RealSense 的预览插件，连接相机即可查看设备输出。

PS：某些电脑在安装过程中会出现 configure UEFI Secure Boot 的窗口，意思是开启了 UEFI Secure Boot ，而这会阻止你使用第三方的硬件驱动，不必更改这一模式，只需要按照窗口提示设定一个 Secure Boot 的密码确认，然后重启就会自动进入到 MOK 界面，选择第二项，yes之后输入刚才设定的密码，即可重新进入系统，此时 RealSense 的驱动也就安装好了。


此时插入摄像头
```
realsense-viewer
```
如果弹出来这样一个窗口，就说明暂时成功了。![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210108082517.png)

但是这个相机有个挺苟的问题，那就是他只能用 USB3.0 的 typeC 线，用普通的充电线是用不了的，像刚刚上面那张图就是用的普通充电线。

下面是 USB3.0 线的界面，会看到左边显示了参数：![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210108083140.png)


---
## 六、开始配置vscode

1、因为我有用到 opencv，所以我有一个 cpp_properties.json 文件（其实就是放头文件的地方）

我直接在原来的基础上加上了 librealsense2/

建议直接在你自己原来的配置文件中加，不要直接复制下面的，我下面给的只是我自己的，仅供参考

下面就是我后面自己加上的这个 librealsense2 的库，其他的就是我原本的设置，后文的 task.json 也一样

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

找到 librealsense2.so 文件的目录，同样复制进去就行

如无例外，应该差不多是都在 /usr/local/lib 里面。

如果文件显示侧边栏没有（且你忘了怎么打开（划掉）：ctrl+shift+P，然后输入：Task->配置任务（中文）或者Configure Task，一般都能找到。

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


然后下面是测试代码（后面更新一个短一点又能显示出来的

参考的是这条csdn链接里的参考链接：https://blog.csdn.net/weixin_43793181/article/details/103186041?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161000787716780258076340%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161000787716780258076340&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v1~rank_blog_v1-2-103186041.pc_v1_rank_blog_v1&utm_term=%E7%9B%B8%E6%9C%BA&spm=1018.2226.3001.4450
```
{

#include<iostream> 
#include<sstream> 
#include<iostream> 
#include<fstream>
#include<algorithm> 
#include<string> 
#include<opencv2/imgproc/imgproc.hpp>
#include<opencv2/core/core.hpp>
#include<opencv2/highgui/highgui.hpp>
#include<librealsense2/rs.hpp>
#include<librealsense2/rsutil.h>
using namespace cv;
using namespace std;
using namespace rs2;

//获取深度像素对应长度单位（米）的换算比例

float get_depth_scale(device dev)
{
    for (sensor& sensor : dev.query_sensors()) //检查设备的传感器    
    {    
        if (depth_sensor dpt = sensor.as<depth_sensor>()) //检查传感器是否为深度传感器        
        {        
            return dpt.get_depth_scale();            
        }        
    }
    throw runtime_error("Device does not have a depth sensor");    
}

//深度图对齐到彩色图函数
Mat align_Depth2Color(Mat depth,Mat color,pipeline_profile profile)
{
    //声明数据流   
    auto depth_stream=profile.get_stream(RS2_STREAM_DEPTH).as<video_stream_profile>();    
    auto color_stream=profile.get_stream(RS2_STREAM_COLOR).as<video_stream_profile>();
    
    //获取内参   
    const auto intrinDepth=depth_stream.get_intrinsics();    
    const auto intrinColor=color_stream.get_intrinsics();
    
    //直接获取从深度摄像头坐标系到彩色摄像头坐标系的欧式变换矩阵    
    rs2_extrinsics  extrinDepth2Color;    
    rs2_error *error;    
    rs2_get_extrinsics(depth_stream,color_stream,&extrinDepth2Color,&error);    
    float pd_uv[2],pc_uv[2];   //平面点定义   
    float Pdc3[3], Pcc3[3];     //空间点定义    
    float depth_scale = get_depth_scale(profile.get_device());//获取深度像素与现实单位比例（D435默认1毫米）    
    int y=0,x=0;    
    Mat result=Mat(color.rows,color.cols,CV_16U,Scalar(0));//初始化结果
    
    //对深度图像遍历    
    for(int row=0;row<depth.rows;row++)    
    {    
        for(int col=0;col<depth.cols;col++)        
        {       
            //将当前的(x,y)放入数组pd_uv，表示当前深度图的点            
            pd_uv[0]=col;           
            pd_uv[1]=row;
            
            //取当前点对应的深度值            
            uint16_t depth_value=depth.at<uint16_t>(row,col);            
            float depth_m=depth_value*depth_scale;           
            rs2_deproject_pixel_to_point(Pdc3,&intrinDepth,pd_uv,depth_m);  //将深度图的像素点根据内参转换到深度摄像头坐标系下的三维点            
            rs2_transform_point_to_point(Pcc3,&extrinDepth2Color,Pdc3);     //将深度摄像头坐标系的三维点转化到彩色摄像头坐标系下            
            rs2_project_point_to_pixel(pc_uv,&intrinColor,Pcc3);            //将彩色摄像头坐标系下的深度三维点映射到二维平面上
            
            //取得映射后的（u,v)            
            x=(int)pc_uv[0];          
            y=(int)pc_uv[1];
            
            //最值限定            
            x=x<0? 0:x;          
            x=x>depth.cols-1 ? depth.cols-1:x;            
            y=y<0? 0:y;            
            y=y>depth.rows-1 ? depth.rows-1:y;            
            result.at<uint16_t>(y,x)=depth_value;            
        }        
    }
    return result;//返回一个与彩色图对齐了的深度信息图像    
}

void measure_distance(Mat &color,Mat depth,Size range,pipeline_profile profile)
{ 

    float depth_scale = get_depth_scale(profile.get_device()); //获取深度像素与现实单位比例（D435默认1毫米）
    Point center(color.cols/2,color.rows/2);                   //自定义图像中心点    
    Rect RectRange(center.x-range.width/2,center.y-range.height/2,    
                   range.width,range.height);                  //自定义计算距离的范围
                   
    //遍历该范围
    float distance_sum=0;    
    int effective_pixel=0;    
    for(int y=RectRange.y;y<RectRange.y+RectRange.height;y++)
    {    
        for(int x=RectRange.x;x<RectRange.x+RectRange.width;x++)
        {
            //如果深度图下该点像素不为0，表示有距离信息
            if(depth.at<uint16_t>(y,x))
            {
                distance_sum+=depth_scale*depth.at<uint16_t>(y,x);
                effective_pixel++;
            }
        }
    }
    cout<<"遍历完成，有效像素点:"<<effective_pixel<<endl;    
    float effective_distance=(distance_sum/effective_pixel)*1000;    
    cout<<"目标距离："<<effective_distance<<" mm"<<endl;   
    char distance_str[30];   
    sprintf(distance_str,"the distance is:%f mm",effective_distance);    
    rectangle(color,RectRange,Scalar(0,0,255),2,8);    
    putText(color,(string)distance_str,Point(color.cols*0.02,color.rows*0.05),    
    FONT_HERSHEY_PLAIN,2,Scalar(0,255,0),2,8);    
}

int main()
{
    colorizer color_map;   // 帮助着色深度图像    
    pipeline pipe;         //创建数据管道    
    config pipe_config;    
    pipe_config.enable_stream(RS2_STREAM_DEPTH,640,480,RS2_FORMAT_Z16,30);    
    pipe_config.enable_stream(RS2_STREAM_COLOR,640,480,RS2_FORMAT_BGR8,30);    
    pipeline_profile profile = pipe.start(pipe_config); //start()函数返回数据管道的profile    
    while (1)    
    {
    
        frameset frameset = pipe.wait_for_frames();  //堵塞程序直到新的一帧捕获        
        //取深度图和彩色图        
        frame color_frame = frameset.get_color_frame();       
        frame depth_frame = frameset.get_depth_frame();        
        frame depth_frame_1 = frameset.get_depth_frame().apply_filter(color_map);
        
        //获取宽高        
        const int depth_w=depth_frame.as<video_frame>().get_width();       
        const int depth_h=depth_frame.as<video_frame>().get_height();        
        const int color_w=color_frame.as<video_frame>().get_width();        
        const int color_h=color_frame.as<video_frame>().get_height();
        
        //创建OPENCV类型 并传入数据        
        Mat depth_image(Size(depth_w,depth_h),        
                        CV_16U,(void*)depth_frame.get_data(),Mat::AUTO_STEP);
                        
        Mat depth_image_1(Size(depth_w,depth_h),        
                          CV_8UC3,(void*)depth_frame_1.get_data(),Mat::AUTO_STEP);
                          
        Mat color_image(Size(color_w,color_h),        
                        CV_8UC3,(void*)color_frame.get_data(),Mat::AUTO_STEP);
                        
        //实现深度图对齐到彩色图
        Mat result=align_Depth2Color(depth_image,color_image,profile);        
        measure_distance(color_image,result,Size(40,40),profile);           
        //自定义窗口大小
        //显示        
        imshow("depth_image",depth_image_1);        
        imshow("color_image",color_image);        
        //imshow("result",result);        
        int key = waitKey(1);       
        if(char(key) == 27)break;        
    }
    return 0;    
}

}
```


人间奇迹现场：![](https://image-up-1304421499.cos.ap-guangzhou.myqcloud.com/img/20210108082635.png)




