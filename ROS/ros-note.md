---
title: ROS笔记
date: 2019-07-30 09:38:43
categories: ROS
tags:
---
# ROS工作空间
## 刷新工作环境
编译后刷新工作环境，在终端可以找到新编译的功能包。
<!--more-->
```
cd ~/tutorial_ws
catkin_make
source ~/tutorial_ws/devel/setup.bash  #刷新环境  方法一
rospack profile #刷新环境   方法二
```
- 可以在~/.bashrc中加入```source ~/tutorial_ws/devel/setup.bash```，每次打开终端都会自动添加~/tutorial_ws下的功能包。

## 工作空间的覆盖
- 查看工作空间```env | grep ros```
- 工作空间的路径依次在ROS_PACKAGE_PATH环境变量中记录
- 新设置的路径在ROS_PACKAGE_PATH中会自动放置在最前端
- 运行时，ROS会优先查找最前端的工作路径中是否存在指定的功能包，如果不存在，就顺序向后查找其他工作空间。

# catkin编译的工作流程
1. 首先在工作空间`catkin_ws/src/`下递归的查找其中每一个ROS的package。
   - 可以把几个源代码包放到同一个文件夹下
![](../images/ros-note/catkin_ws.jpg)
2. package中会有`package.xml`和`CMakeLists.txt`文件，Catkin(CMake)编译系统依据`CMakeLists.txt`文件,从而生成`makefiles`(放在`catkin_ws/build/`)。
3. 然后`make`刚刚生成的`makefiles`等文件，编译链接生成可执行文件(放在`catkin_ws/devel`)。
![](../images/ros-note/catkin_flow.jpg)

# 常用命令
## rosnode命令

|    命令    | 作用 |
| :------:   | :------:           |
| `rosnode list`               |   列出当前运行的node信息 |
| `rosnode info node_name`   |  显示出node的详细信息  |
| `rosnode kill node_name`   |  结束某个node |
| `rosnode ping`    |   测试连接节点 |
| `rosnode machine `     |  列出在特定机器或列表机器上运行的节点 |
| `rosnode cleanup`| 清除不可到达节点的注册信息|

以上命令中常用的为前三个  
可以通过`rosnode help`来查看`rosnode`命令的用法。

## rostopic命令
|   命令    | 作用 |
| :------:   | :------:           |
| `rostopic list`               |   列出当前所有的topic |
| `rostopic info topic_name`   |  显示某个topic的属性信息  |
| `rostopic echo topic_name`   |  显示某个topic的内容 |
| `rostopic pub topic_name ...`    |  向某个topic发布内容|
| `rostopic bw topic_name`    |  查看某个topic的带宽|
| `rostopic hz topic_name`    |  查看某个topic的频率|
| `rostopic find topic_type`    |  查找某个类型的topic|
| `rostopic type topic_name`    |  查看某个topic的类型(msg)|

可以通过`rostopic help`或`rostopic command -h`查看具体用法。

## rosmsg命令
|    命令    | 作用 |
| :------:   | :------:           |
| `rosmsg list`               |   列出系统上所有的msg |
| `rosmsg show msg_name`   |  显示某个msg的内容  |

## rosservice命令
| 命令 | 作用 |
| :---: | :---: |
| `rosservice list` | 显示服务列表 |
| `rosservice info` | 打印服务信息 |
| `rosservice type` | 打印服务类型 |
| `rosservice uri` | 打印服务ROSRPC uri |
| `rosservice find` | 按服务类型查找服务 |
| `rosservice call` | 使用所提供的args调用服务 |
| `rosservice args` | 打印服务参数 |

## rossrv命令
|    命令    | 作用 |
| :------:   | :------:           |
| `rossrv show`  |  显示服务描述|
| `rossrv list`   | 列出所有服务  |
| `rossrv md5`   |  显示服务md5sum |
| `rossrv package `    |  列出包中的服务|
|`rossrv packages`    |  列出包含服务的包|

## rosparam命令
|    命令    | 作用 |
| :------:   | :------:           |
| `rosparam set param_key param_value`  |  设置参数|
| `rosparam get param_key`   | 显示参数  |
| `rosparam load file_name `   | 从文件加载参数|
| `rosparam dump file_name `    |  保存参数到文件|
|`rosparam delete`    |  删除参数|
|`rosparam list`|列出参数名称|


# 通信
## topic
![](../images/ros-note/话题通信模型.png)
![](../images/ros-note/话题通信_自定义msg.png)
![](../images/ros-note/话题通信_发布者.png)
![](../images/ros-note/话题通信_订阅者.png)
CMakeLists.txt编译:  
```bash
add_executable(talker src/talker.cpp)#可执行文件    源文件
target_link_libraries(talker ${catkin_LIBRARIES})#可执行文件  固定
add_dependencies(talker ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})#可执行文件  固定  固定
```
## service
![](../images/ros-note/服务通信模型.png)
![](../images/ros-note/服务通信_自定义srv.png)
![](../images/ros-note/服务通信_服务器.png)
![](../images/ros-note/服务通信_客户端.png)
CMakeLists.txt编译:  
```bash
add_executable(server src/server.cpp)#可执行文件    源文件
target_link_libraries(server ${catkin_LIBRARIES})#可执行文件  固定
add_dependencies(server ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})#可执行文件  固定  固定
```
## **topic VS service**
| 名称 | Topic | Service |
| :---: | :---: | :---: |
| 通信方式 | 异步通信 | 同步通信 |
| 实现原理 | TCP/IP | TCP/IP |
| 通信模型 | Publish-Subscribe | Request-Reply |
| 映射关系 | Publish-Subscribe(多对多) | Request-Reply（多对一） |
| 特点 | 接受者收到数据会回调（Callback） | 远程过程调用（RPC）服务器端的服务 |
| 应用场景 | 连续、高频的数据发布 | 偶尔使用的功能/具体的任务 |
| 举例 | 激光雷达、里程计发布数据 | 开关传感器、拍照、逆解计算 |

**注意**：远程过程调用(Remote Procedure Call，RPC),可以简单通俗的理解为在一个进程里调用另一个进程的函数。

## Action
![](../images/ros-note/动作通信.png)
![](../images/ros-note/动作通信_自定义动作.png)
![](../images/ros-note/动作通信_服务器.png)
![](../images/ros-note/动作通信_客户端.png)
```bash
add_executable(DoDishes src/DoDishes.cpp)#可执行文件    源文件
target_link_libraries(DoDishes ${catkin_LIBRARIES})#可执行文件  固定
add_dependencies(DoDishes ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})#可执行文件  固定  固定
```

# roscpp函数说明
## 回调函数与spin()方法
回调函数作为参数被传入到了另一个函数中，在未来某个时刻，就会立即执行。
- 以topic Subscriber为例，回调函数作为参数被传入到了另一个函数中，当有新的message到达，立即执行。（在action server中还有其他变量作为参数传入另一个函数）
- Subscriber接收到消息，实际上是先把消息放到一个**队列**中去，如图所示。队列的长度在Subscriber构建的时候设置好了。当有spin函数执行，就会去处理消息队列中队首的消息。
![](../images/ros-note/cb_queue.png)

## sleep()
```cpp
//休眠一次
ros::Duration(0.5).sleep(); //用Duration对象的sleep方法休眠

//多次休眠
ros::Rate r(10); //10HZ
while(ros::ok())
{
    r.sleep();//定义好sleep的频率，Rate对象会自动让整个循环以10hz休眠，即使有任务执行占用了时间
}
```