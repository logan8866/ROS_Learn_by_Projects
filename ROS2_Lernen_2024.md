# ROS2 Foxy 学习
- [ROS2 Foxy 学习](#ros2-foxy-学习)
  - [1.初级（熟悉基本命令行）](#1初级熟悉基本命令行)
    - [1.1 环境配置](#11-环境配置)
    - [1.2 使用小海龟和rqt](#12-使用小海龟和rqt)
    - [1.3理解ROS节点](#13理解ros节点)
    - [1.4 理解Topic](#14-理解topic)
    - [1.5 理解services](#15-理解services)
    - [1.6 理解Parameter](#16-理解parameter)
    - [1.7 理解action](#17-理解action)
    - [1.8 启动节点](#18-启动节点)
  - [2.初级（熟悉接口调用）](#2初级熟悉接口调用)
    - [2.1 使用colcon](#21-使用colcon)
    - [2.2 创建worksapce](#22-创建worksapce)
    - [2.3 创建package](#23-创建package)
    - [2.4 创建publisher和subscriber（c++）](#24-创建publisher和subscriberc)
    - [2.5 创建publisher和subscriber（python）](#25-创建publisher和subscriberpython)
    - [2.6 service和cleint接口调用（C++）](#26-service和cleint接口调用c)
    - [2.7 service和cleint接口调用（Python）](#27-service和cleint接口调用python)
    - [2.8 使用自定义的msg与srv文件](#28-使用自定义的msg与srv文件)
    - [2.9 自定义的msg与publisher在同package下运行](#29-自定义的msg与publisher在同package下运行)
    - [2.10 参数的使用（C++）](#210-参数的使用c)
    - [2.11 参数的使用（Python）](#211-参数的使用python)
  - [3.中级](#3中级)
    - [3.1 rosdep的使用](#31-rosdep的使用)
    - [3.2 创建一个action](#32-创建一个action)
    - [3.3 编写action server与client（c++）](#33-编写action-server与clientc)
    - [3.3 编写action server与client（python）](#33-编写action-server与clientpython)
    - [3.4 在一个程序里面运行多个节点](#34-在一个程序里面运行多个节点)
    - [3.5 launch file](#35-launch-file)
      - [3.5.1 创建launch文件](#351-创建launch文件)

## 1.初级（熟悉基本命令行）
### 1.1 环境配置
先配置打开的shell默认有ROS2的环境，则要运行：
``` shell
echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
```

在 ROS 2 中，多个节点可以组成一个通信域（Domain），而每个通信域都有一个唯一的标识符（Domain ID）。默认情况下，所有的节点都属于相同的通信域，通信域的 ID 为 0。

设置 ROS_DOMAIN_ID 环境变量可以让你将节点分组到不同的通信域中以避免它们之间的通信干扰或者限制通信范围非常有用。
```shell
export ROS_DOMAIN_ID=<your_domain_id>
```
设置ROS只与本地节点通信：
```shell
export ROS_LOCALHOST_ONLY=1
```
### 1.2 使用小海龟和rqt
启动海龟节点：
```shell
ros2 run turtlesim turtlesim_node
```

启动键盘控制海龟：
```shell
ros2 run turtlesim turtle_teleop_key
```

查看当前的node，topic，service，action列表：
```shell
ros2 node list
ros2 topic list
ros2 service list
ros2 action list
```
使用rqt：
```shell
rqt
```
在plugins里面添加topic，service，action
例如：设置小乌龟画笔的参数
![alt](https://docs.ros.org/en/foxy/_images/set_pen.png)

把宽度设置为5，然后点击call发送service，然后就看到画笔变粗了
![alt](https://docs.ros.org/en/foxy/_images/new_pen.png)

Remapping的使用方法：
```shell
ros2 run turtlesim turtle_teleop_key --ros-args --remap turtle1/cmd_vel:=turtle2/cmd_vel
```
这意味着 turtle_teleop_key 节点在启动后会订阅 /turtle2/cmd_vel 主题而不是默认的 /turtle1/cmd_vel 主题。

### 1.3理解ROS节点
每个节点都可以通过主题、服务、操作或参数从其他节点发送和接收数据。
![alt](https://docs.ros.org/en/foxy/_images/Nodes-TopicandService.gif)

启动节点：
```shell
ros2 run <package_name> <executable_name>
```

理解Remapping：重新映射允许将节点名称、主题名称、服务名称等默认节点属性重新指定为自定义值。

把当前的/turtlesim映射到/my_turtle:
```shell
ros2 run turtlesim turtlesim_node --ros-args --remap __node:=my_turtle
```

查看节点的信息（action，service，topic）：
```shell
ros2 node info <node_name>
```
### 1.4 理解Topic
节点可以一对多
![alt](https://docs.ros.org/en/foxy/_images/Topic-MultiplePublisherandMultipleSubscriber.gif)

监听节点：
```shell
ros2 topic echo <topic_name>
```

查看节点信息：
```shell
ros2 topic info /turtle1/cmd_vel
```
```shell
Type: geometry_msgs/msg/Twist
Publisher count: 1
Subscription count: 2
```

节点通过消息在主题上发送数据。发布者和订阅者必须发送和接收相同类型的消息才能进行通信。
可以运行
```shell
ros2 interface show <msg type>
``` 
来了解消息的数据结构。
例如
```shell
ros2 interface show geometry_msgs/msg/Twist
```
结果是
```shell
# This expresses velocity in free space broken into its linear and angular parts.

    Vector3  linear
    Vector3  angular
```

通过终端发topic消息
```shell
ros2 topic pub <topic_name> <msg_type> '<args>'
```
\<args>必须是YAML格式
例如：
```shell
ros2 topic pub --once /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```
以一定的速率发布topic消息：
```shell
ros2 topic pub --rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"
```
查看数据的发送速度：
```shell
ros2 topic hz /turtle1/pose
```
### 1.5 理解services
services是半双工一对多的消息传递机制
![alt](https://docs.ros.org/en/foxy/_images/Service-MultipleServiceClient.gif)
查看消息的类型
```shell
ros2 service type <service_name>
```

通过消息的类型查找服务：
```shell
ros2 service find <type_name>
```
例如查看什么服务使用的Empty消息
```shell
ros2 service find std_srvs/srv/Empty
```
结果返回：
```shell
/clear
/reset
```
查看service消息的数据结构：
```shell
ros2 interface show <type_name>.srv
```
---上面的是发送的数据，下面是返回的数据
例如让小乌龟产卵：
```shell
ros2 interface show turtlesim/srv/Spawn
```
结果是：
```shell
float32 x
float32 y
float32 theta
string name # Optional.  A unique name will be created and returned if this is empty
---
string name
```
在终端call service：
```shell
ros2 service call <service_name> <service_type> <arguments>
```
例如：
```shell
ros2 service call /spawn turtlesim/srv/Spawn "{x: 2, y: 2, theta: 0.2, name: ''}"
```
### 1.6 理解Parameter
得到参数：
```shell
ros2 param get <node_name> <parameter_name>
```
设置参数：
```shell
ros2 param set <node_name> <parameter_name> <value>
```
保存参数到文件：
```shell
ros2 param dump <node_name>
```
这个节点的所有参数都被保存在了*.yaml文件里面
从yaml文件导入参数:
```shell
ros2 param load <node_name> <parameter_file>
```
在启动节点的时候从文件导入参数：
```shell
ros2 run <package_name> <executable_name> --ros-args --params-file <file_name>
```
### 1.7 理解action
action是ROS2中的通信类型之一，适用于长时间运行的任务。由三部分组成：目标、反馈和结果。

action建立在topic和service的基础上。功能与service类似，但action是可抢占的（执行中可以取消）。它们还提供稳定的反馈，而service只返回一个响应。

动作客户端向动作服务器发送一个目标，动作服务器确认目标并返回反馈流和结果。
![alt](https://docs.ros.org/en/foxy/_images/Action-SingleActionClient.gif)

获取action的信息（action的名字，客户端和服务器）：
```shell
ros2 action info /turtle1/rotate_absolute
```
结果是：
```shell
Action: /turtle1/rotate_absolute
Action clients: 1
    /teleop_turtle
Action servers: 1
    /turtlesim
```
根据action查询数据结构：
```shell
ros2 interface show turtlesim/action/RotateAbsolute
```
结果是：
```shell
# The desired heading in radians
float32 theta
---
# The angular displacement in radians to the starting position
float32 delta
---
# The remaining rotation in radians
float32 remaining
```
第一个是目标，第二个是结果，第三个是实时反馈

通过终端发送目标数据：
```shell
ros2 action send_goal <action_name> <action_type> <values>
```
例如：
```shell
ros2 action send_goal /turtle1/rotate_absolute turtlesim/action/RotateAbsolute "{theta: 1.57}"
```
返回结果：
```shell
Waiting for an action server to become available...
Sending goal:
   theta: 1.57

Goal accepted with ID: f8db8f44410849eaa93d3feb747dd444

Result:
  delta: -1.568000316619873

Goal finished with status: SUCCEEDED
```
### 1.8 启动节点
通过启动文件启动多个节点：
```shell
ros2 launch turtlesim multisim.launch.py
```
文件里面写的是：
```shell
# turtlesim/launch/multisim.launch.py

from launch import LaunchDescription
import launch_ros.actions

def generate_launch_description():
    return LaunchDescription([
        launch_ros.actions.Node(
            namespace= "turtlesim1", package='turtlesim', executable='turtlesim_node', output='screen'),
        launch_ros.actions.Node(
            namespace= "turtlesim2", package='turtlesim', executable='turtlesim_node', output='screen'),
    ])
```
## 2.初级（熟悉接口调用）
### 2.1 使用colcon
在src中保存着ROS源码，colcon会对源码进行编译。
创建一个worksapce：
```shell
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws
```
从github上复制一个ROS的例子：
```shell
git clone https://github.com/ros2/examples src/examples -b foxy
```
在工作区的根目录下运行 colcon build进行编译：
```shell
colcon build --symlink-install
```
编译完成之后，要更新一下环境：
```shell
source install/setup.bash
```
然后试着跑一下demo：
```shell
ros2 run examples_rclcpp_minimal_subscriber subscriber_member_function
```
### 2.2 创建worksapce
首先先在终端source一下ROS2的环境：
```shell
source /opt/ros/foxy/setup.bash
```
然后创建一个目录：
```shell
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```
然后从github复制一个例子：
```shell
git clone https://github.com/ros/ros_tutorials.git -b foxy-devel
```
然后执行命令添加依赖：
```shell
# cd if you're still in the ``src`` directory with the ``ros_tutorials`` clone
cd ..
rosdep install -i --from-path src --rosdistro foxy -y
```
然后执行命令来创建worksapce：
```shell
colcon build
```
然后得到了三个目录（src需要自己创建）：
```shell
build  install  log  src
```
编译后source一下环境就可以运行了：
```shell
source install/local_setup.bash
ros2 run turtlesim turtlesim_node
```
可以修改 turtlesim 窗口的标题栏，找到 ~/ros2_ws/src/ros_tutorials/turtlesim/src 中的 turtle_frame.cpp文件。

在第 52 行，找到函数 setWindowTitle("TurtleSim");。将 "TurtleSim "改为 "MyTurtleSim"，然后保存文件。

运行 colcon build
```shell
ros2 run turtlesim turtlesim_node
```
![alt](https://docs.ros.org/en/foxy/_images/overlay.png)
可以看到标题栏更改了。
### 2.3 创建package
package的结构：

CMakeLists.txt 文件，说明如何在软件包内构建代码

include/<package_name> 目录，包含软件包的公共头文件

package.xml 包含软件包的信息

包含软件包源代码的 src 目录
```shell
my_package/
     CMakeLists.txt
     include/my_package/
     package.xml
     src/
```
一个worksapce可以包含很多package。

创建package首先要进入worksapce的src目录：
```shell
cd ~/ros2_ws/src
```
然后执行命令创建package：
```shell
ros2 pkg create --build-type ament_cmake <package_name>
```
举个例子：
```shell
ros2 pkg create --build-type ament_cmake --node-name my_node my_package
```
单独编译某一个package：
```shell
colcon build --packages-select my_package
```
运行package里面的节点：
```shell
ros2 run my_package my_node
```
> 运行前别忘了source环境

在ros2_ws/src/my_package中的package.xml里面包含package描述（作者信息） 
```xml
<?xml version="1.0"?>
<?xml-model
   href="http://download.ros.org/schema/package_format3.xsd"
   schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
 <name>my_package</name>
 <version>0.0.0</version>
 <description>TODO: Package description</description>
 <maintainer email="user@todo.todo">user</maintainer>
 <license>TODO: License declaration</license>

 <buildtool_depend>ament_cmake</buildtool_depend>

 <test_depend>ament_lint_auto</test_depend>
 <test_depend>ament_lint_common</test_depend>

 <export>
   <build_type>ament_cmake</build_type>
 </export>
</package>
```
```xml
<description>Beginner client libraries tutorials practice package</description>
```
### 2.4 创建publisher和subscriber（c++）
在workspace的src下面创建一个package：
```shell
ros2 pkg create --build-type ament_cmake cpp_pubsub
```
然后在package里面的的src目录创建**publisher_member_function.cpp**
里面的内容是：
```C++
#include <chrono>
#include <functional>
#include <memory>
#include <string>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"

using namespace std::chrono_literals;

/* This example creates a subclass of Node and uses std::bind() to register a
* member function as a callback from the timer. */

class MinimalPublisher : public rclcpp::Node
{
  public:
    MinimalPublisher()
    : Node("minimal_publisher"), count_(0)
    {
      publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
      timer_ = this->create_wall_timer(
      500ms, std::bind(&MinimalPublisher::timer_callback, this));
    }

  private:
    void timer_callback()
    {
      auto message = std_msgs::msg::String();
      message.data = "Hello, world! " + std::to_string(count_++);
      RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
      publisher_->publish(message);
    }
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
    size_t count_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```
rclcpp/rclcpp.hpp包含了常见的ROS2接口，std_msgs/msg/string.hpp包含了消息类型。
MinimalPublisher是ROS节点的子类
```c++
public:
  MinimalPublisher()
  : Node("minimal_publisher"), count_(0)
  {
    publisher_ = this->create_publisher<std_msgs::msg::String>("topic", 10);
    timer_ = this->create_wall_timer(
    500ms, std::bind(&MinimalPublisher::timer_callback, this));
  }
```
在构造函数里面创造一个topic的publisher，队列长度为10，然后再创造一个timer定时器，每隔500ms执行一次回调函数

>std::bind的使用方法:
```c++
#include <iostream>
#include <functional>

void printMessage(const std::string &message) {
    std::cout << "Message: " << message << std::endl;
}

class MyClass {
public:
    void printNumber(int number) {
        std::cout << "Number: " << number << std::endl;
    }
};

int main() {
    // 绑定全局函数
    auto print = std::bind(printMessage, "Hello, world!");
    print();  // 调用 printMessage("Hello, world!");

    // 绑定成员函数
    MyClass obj;
    auto printNum = std::bind(&MyClass::printNumber, &obj, 42);
    printNum();  // 调用 obj.printNumber(42);

    return 0;
}

```
节点的回调函数：
```c++
private:
  void timer_callback()
  {
    auto message = std_msgs::msg::String();
    message.data = "Hello, world! " + std::to_string(count_++);
    RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
    publisher_->publish(message);
  }
```
先创建一个消息实例，用publisher发布消息

>里面涉及了嵌套命名空间的使用：
```c++
#include <iostream>

namespace OuterNamespace {
    int outerVariable = 10;

    namespace InnerNamespace {
        int innerVariable = 20;
    }  // end of InnerNamespace
}  // end of OuterNamespace

int main() {
    std::cout << "OuterNamespace::outerVariable: " << OuterNamespace::outerVariable << std::endl;
    std::cout << "OuterNamespace::InnerNamespace::innerVariable: " << OuterNamespace::InnerNamespace::innerVariable << std::endl;

    return 0;
}

```
```shell
rclcpp::TimerBase::SharedPtr timer_;
rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
size_t count_;
```
创建成员指针

```shell
int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalPublisher>());
  rclcpp::shutdown();
  return 0;
}
```
首先初始化ROS2，然后运行节点，运行完节点之后就关闭ROS2.
>共享指针的使用，共享指针不需要手动释放内存

在CMakeLists.txt 和 package.xml中添加依赖：

在package.xml中添加：
```xml
<depend>rclcpp</depend>
<depend>std_msgs</depend>
```
在CMakeLists.txt中添加：
```c++
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

add_executable(talker src/publisher_member_function.cpp)
ament_target_dependencies(talker rclcpp std_msgs)

install(TARGETS
  talker
  DESTINATION lib/${PROJECT_NAME})
```
最后是这样的：
```c++
cmake_minimum_required(VERSION 3.5)
project(cpp_pubsub)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

add_executable(talker src/publisher_member_function.cpp)
ament_target_dependencies(talker rclcpp std_msgs)

install(TARGETS
  talker
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

在创建一个subscriber，建立一个c++文件***subscriber_member_function.cpp***
内容是：
```c++
#include <memory>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
using std::placeholders::_1;

class MinimalSubscriber : public rclcpp::Node
{
  public:
    MinimalSubscriber()
    : Node("minimal_subscriber")
    {
      subscription_ = this->create_subscription<std_msgs::msg::String>(
      "topic", 10, std::bind(&MinimalSubscriber::topic_callback, this, _1));
    }

  private:
    void topic_callback(const std_msgs::msg::String::SharedPtr msg) const
    {
      RCLCPP_INFO(this->get_logger(), "I heard: '%s'", msg->data.c_str());
    }
    rclcpp::Subscription<std_msgs::msg::String>::SharedPtr subscription_;
};

int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalSubscriber>());
  rclcpp::shutdown();
  return 0;
}
```
在CMakeLists.txt中添加依赖：
```c++
add_executable(listener src/subscriber_member_function.cpp)
ament_target_dependencies(listener rclcpp std_msgs)

install(TARGETS
  talker
  listener
  DESTINATION lib/${PROJECT_NAME})
```
编译并且运行：
```shell
#编译
colcon build --packages-select cpp_pubsub
#环境
. install/setup.bash
#运行节点
ros2 run cpp_pubsub talker
ros2 run cpp_pubsub listener
```
### 2.5 创建publisher和subscriber（python）
创建一个package：
```shell
ros2 pkg create --build-type ament_python py_pubsub
```
在这个目录下ros2_ws/src/py_pubsub/py_pubsub，建立文件publisher_member_function.py
内容是：
```python
import rclpy
from rclpy.node import Node

from std_msgs.msg import String


class MinimalPublisher(Node):

    def __init__(self):
        super().__init__('minimal_publisher')
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        timer_period = 0.5  # seconds
        self.timer = self.create_timer(timer_period, self.timer_callback)
        self.i = 0

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello World: %d' % self.i
        self.publisher_.publish(msg)
        self.get_logger().info('Publishing: "%s"' % msg.data)
        self.i += 1


def main(args=None):
    rclpy.init(args=args)

    minimal_publisher = MinimalPublisher()

    rclpy.spin(minimal_publisher)

    # Destroy the node explicitly
    # (optional - otherwise it will be done automatically
    # when the garbage collector destroys the node object)
    minimal_publisher.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```
添加依赖：
在ros2_ws/src/py_pubsub目录下有setup.py, setup.cfg, 和 package.xml，在package.xml中添加：
```xml
<exec_depend>rclpy</exec_depend>
<exec_depend>std_msgs</exec_depend>
```
在setup.py中添加：
```python
entry_points={
        'console_scripts': [
                'talker = py_pubsub.publisher_member_function:main',
        ],
},
```
在setup.cfg中修改：
```cfg
[develop]
script-dir=$base/lib/py_pubsub
[install]
install-scripts=$base/lib/py_pubsub
```
再写subscriber,在ros2_ws/src/py_pubsub/py_pubsub下面建立subscriber_member_function.py

内容是：
```python
import rclpy
from rclpy.node import Node

from std_msgs.msg import String


class MinimalSubscriber(Node):

    def __init__(self):
        super().__init__('minimal_subscriber')
        self.subscription = self.create_subscription(
            String,
            'topic',
            self.listener_callback,
            10)
        self.subscription  # prevent unused variable warning

    def listener_callback(self, msg):
        self.get_logger().info('I heard: "%s"' % msg.data)


def main(args=None):
    rclpy.init(args=args)

    minimal_subscriber = MinimalSubscriber()

    rclpy.spin(minimal_subscriber)

    # Destroy the node explicitly
    # (optional - otherwise it will be done automatically
    # when the garbage collector destroys the node object)
    minimal_subscriber.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```
在setup.py中添加：
```python
entry_points={
        'console_scripts': [
                'talker = py_pubsub.publisher_member_function:main',
                'listener = py_pubsub.subscriber_member_function:main',
        ],
},
```
编译与运行：
```shell
colcon build --packages-select py_pubsub

source install/setup.bash

ros2 run py_pubsub talker
ros2 run py_pubsub listener
```
### 2.6 service和cleint接口调用（C++）
先在workspace下面的src创建package：
```shell
ros2 pkg create --build-type ament_cmake cpp_srvcli --dependencies rclcpp example_interfaces
```
--dependencies会自动把rclcpp和example_interfaces依赖添加到package.xml 和 CMakeLists.txt中

example_interfaces中包含了srv file：
```c++
int64 a
int64 b
---
int64 sum
```
在ros2_ws/src/cpp_srvcli/src中创建add_two_ints_server.cpp，内容是：
```C++
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

#include <memory>

void add(const std::shared_ptr<example_interfaces::srv::AddTwoInts::Request> request,
          std::shared_ptr<example_interfaces::srv::AddTwoInts::Response>      response)
{
  response->sum = request->a + request->b;
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Incoming request\na: %ld" " b: %ld",
                request->a, request->b);
  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "sending back response: [%ld]", (long int)response->sum);
}

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);

  std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_server");

  rclcpp::Service<example_interfaces::srv::AddTwoInts>::SharedPtr service =
    node->create_service<example_interfaces::srv::AddTwoInts>("add_two_ints", &add);

  RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Ready to add two ints.");

  rclcpp::spin(node);
  rclcpp::shutdown();
}
```
在CMakeLists.txt中添加：
```c++
add_executable(server src/add_two_ints_server.cpp)
ament_target_dependencies(server rclcpp example_interfaces)

install(TARGETS
    server
  DESTINATION lib/${PROJECT_NAME})
```
再写client：在ros2_ws/src/cpp_srvcli/src中创建add_two_ints_client.cpp，内容是：
```c++
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/srv/add_two_ints.hpp"

#include <chrono>
#include <cstdlib>
#include <memory>

using namespace std::chrono_literals;

int main(int argc, char **argv)
{
  rclcpp::init(argc, argv);

  if (argc != 3) {
      RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "usage: add_two_ints_client X Y");
      return 1;
  }

  std::shared_ptr<rclcpp::Node> node = rclcpp::Node::make_shared("add_two_ints_client");
  rclcpp::Client<example_interfaces::srv::AddTwoInts>::SharedPtr client =
    node->create_client<example_interfaces::srv::AddTwoInts>("add_two_ints");

  auto request = std::make_shared<example_interfaces::srv::AddTwoInts::Request>();
  request->a = atoll(argv[1]);
  request->b = atoll(argv[2]);

  while (!client->wait_for_service(1s)) {
    if (!rclcpp::ok()) {
      RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Interrupted while waiting for the service. Exiting.");
      return 0;
    }
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "service not available, waiting again...");
  }

  auto result = client->async_send_request(request);
  // Wait for the result.
  if (rclcpp::spin_until_future_complete(node, result) ==
    rclcpp::FutureReturnCode::SUCCESS)
  {
    RCLCPP_INFO(rclcpp::get_logger("rclcpp"), "Sum: %ld", result.get()->sum);
  } else {
    RCLCPP_ERROR(rclcpp::get_logger("rclcpp"), "Failed to call service add_two_ints");
  }

  rclcpp::shutdown();
  return 0;
}
```
在CMakeLists.txt中修改：
```c++
cmake_minimum_required(VERSION 3.5)
project(cpp_srvcli)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(example_interfaces REQUIRED)

add_executable(server src/add_two_ints_server.cpp)
ament_target_dependencies(server rclcpp example_interfaces)

add_executable(client src/add_two_ints_client.cpp)
ament_target_dependencies(client rclcpp example_interfaces)

install(TARGETS
  server
  client
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```
编译与运行：
```shell
colcon build --packages-select cpp_srvcli

source install/setup.bash

ros2 run cpp_srvcli server
ros2 run cpp_srvcli client 2 3
```
### 2.7 service和cleint接口调用（Python）
在workspace中的src中创建package：
```shell
ros2 pkg create --build-type ament_python py_srvcli --dependencies rclpy example_interfaces
```
在ros2_ws/src/py_srvcli/py_srvcli下创建service_member_function.py
内容是：

```python
from example_interfaces.srv import AddTwoInts

import rclpy
from rclpy.node import Node


class MinimalService(Node):

    def __init__(self):
        super().__init__('minimal_service')
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_two_ints_callback)

    def add_two_ints_callback(self, request, response):
        response.sum = request.a + request.b
        self.get_logger().info('Incoming request\na: %d b: %d' % (request.a, request.b))

        return response


def main(args=None):
    rclpy.init(args=args)

    minimal_service = MinimalService()

    rclpy.spin(minimal_service)

    rclpy.shutdown()


if __name__ == '__main__':
    main()
```
在ros2_ws/src/py_srvcli/py_srvcli下创建client_member_function.py

内容是：
```python
import sys

from example_interfaces.srv import AddTwoInts
import rclpy
from rclpy.node import Node


class MinimalClientAsync(Node):

    def __init__(self):
        super().__init__('minimal_client_async')
        self.cli = self.create_client(AddTwoInts, 'add_two_ints')
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('service not available, waiting again...')
        self.req = AddTwoInts.Request()

    def send_request(self, a, b):
        self.req.a = a
        self.req.b = b
        self.future = self.cli.call_async(self.req)
        rclpy.spin_until_future_complete(self, self.future)
        return self.future.result()


def main(args=None):
    rclpy.init(args=args)

    minimal_client = MinimalClientAsync()
    response = minimal_client.send_request(int(sys.argv[1]), int(sys.argv[2]))
    minimal_client.get_logger().info(
        'Result of add_two_ints: for %d + %d = %d' %
        (int(sys.argv[1]), int(sys.argv[2]), response.sum))

    minimal_client.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```
在setup.py中添加：
```python
entry_points={
    'console_scripts': [
        'service = py_srvcli.service_member_function:main',
        'client = py_srvcli.client_member_function:main',
    ],
},
```
编译与运行：
```shell
colcon build --packages-select py_srvcli

source install/setup.bash

ros2 run py_srvcli service
ros2 run py_srvcli client 2 3
```
### 2.8 使用自定义的msg与srv文件
创建一个package做自定义消息：
```shell
ros2 pkg create --build-type ament_cmake tutorial_interfaces
```
创建文件夹：
```shell
mkdir msg srv
```
- 自定义msg
在tutorial_interfaces/msg下新建一个Num.msg，里面写：
```c++
int64 num
```
在tutorial_interfaces/msg下建立Sphere.msg，里面写：
```c++
geometry_msgs/Point center
float64 radius
```
- srv自定义
在tutorial_interfaces/srv下面建立AddThreeInts.srv，里面写：
```c++
int64 a
int64 b
int64 c
---
int64 sum
```
在CMakeLists.txt中写：
```c++
find_package(geometry_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/Num.msg"
  "msg/Sphere.msg"
  "srv/AddThreeInts.srv"
  DEPENDENCIES geometry_msgs # Add packages that above messages depend on, in this case geometry_msgs for Sphere.msg
)
```
在package.xml中写：
```xml
<depend>geometry_msgs</depend>
<buildtool_depend>rosidl_default_generators</buildtool_depend>
<exec_depend>rosidl_default_runtime</exec_depend>
<member_of_group>rosidl_interface_packages</member_of_group>
```
编译与测试：
```shell
colcon build --packages-select tutorial_interfaces

source install/setup.bash

ros2 interface show tutorial_interfaces/msg/Num
ros2 interface show tutorial_interfaces/msg/Sphere
ros2 interface show tutorial_interfaces/srv/AddThreeInts
```
- 在使用自定义msg的时候添加依赖的方法

CMakeLists.txt中写：
```c++
#...

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tutorial_interfaces REQUIRED)                         # CHANGE

add_executable(talker src/publisher_member_function.cpp)
ament_target_dependencies(talker rclcpp tutorial_interfaces)         # CHANGE

add_executable(listener src/subscriber_member_function.cpp)
ament_target_dependencies(listener rclcpp tutorial_interfaces)     # CHANGE

install(TARGETS
  talker
  listener
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```
在package.xml中写：
```xml
#c++
<depend>tutorial_interfaces</depend>
#python
<exec_depend>tutorial_interfaces</exec_depend>
```
- 在使用自定义srv的时候添加依赖的方法

在CMakeLists.txt中写：
```c++
#...

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tutorial_interfaces REQUIRED)        # CHANGE

add_executable(server src/add_two_ints_server.cpp)
ament_target_dependencies(server
  rclcpp tutorial_interfaces)                      #CHANGE

add_executable(client src/add_two_ints_client.cpp)
ament_target_dependencies(client
  rclcpp tutorial_interfaces)                      #CHANGE

install(TARGETS
  server
  client
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```
在package.xml中写：
```xml
#c++
<depend>tutorial_interfaces</depend>
#python
<exec_depend>tutorial_interfaces</exec_depend>
```
### 2.9 自定义的msg与publisher在同package下运行

先创建一个package然后在package下面创建一个msg文件夹
```shell
ros2 pkg create --build-type ament_cmake more_interfaces
mkdir more_interfaces/msg
```
在more_interfaces/msg下创建AddressBook.msg，里面编辑想要发送msg信息的结构：
```c++
uint8 PHONE_TYPE_HOME=0
uint8 PHONE_TYPE_WORK=1
uint8 PHONE_TYPE_MOBILE=2

string first_name
string last_name
string phone_number
uint8 phone_type
```
在package.xml中添加相应的依赖：
```xml
<buildtool_depend>rosidl_default_generators</buildtool_depend>

<exec_depend>rosidl_default_runtime</exec_depend>

<member_of_group>rosidl_interface_packages</member_of_group>
```
在CMakeLists.txt中添加相应的依赖：
```c++
find_package(rosidl_default_generators REQUIRED)

set(msg_files
  "msg/AddressBook.msg"
)

rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
)

ament_export_dependencies(rosidl_default_runtime)
```
在more_interfaces/src下建立publish_address_book.cpp写一个topic publisher类：
```c++
#include <chrono>
#include <memory>

#include "rclcpp/rclcpp.hpp"
#include "more_interfaces/msg/address_book.hpp"

using namespace std::chrono_literals;

class AddressBookPublisher : public rclcpp::Node
{
public:
  AddressBookPublisher()
  : Node("address_book_publisher")
  {
    address_book_publisher_ =
      this->create_publisher<more_interfaces::msg::AddressBook>("address_book", 10);

    auto publish_msg = [this]() -> void {
        auto message = more_interfaces::msg::AddressBook();

        message.first_name = "John";
        message.last_name = "Doe";
        message.phone_number = "1234567890";
        message.phone_type = message.PHONE_TYPE_MOBILE;

        std::cout << "Publishing Contact\nFirst:" << message.first_name <<
          "  Last:" << message.last_name << std::endl;

        this->address_book_publisher_->publish(message);
      };
    timer_ = this->create_wall_timer(1s, publish_msg);
  }

private:
  rclcpp::Publisher<more_interfaces::msg::AddressBook>::SharedPtr address_book_publisher_;
  rclcpp::TimerBase::SharedPtr timer_;
};


int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<AddressBookPublisher>());
  rclcpp::shutdown();

  return 0;
}
```
在CMakeLists.txt中加入依赖：
```c++
find_package(rclcpp REQUIRED)

add_executable(publish_address_book src/publish_address_book.cpp)
ament_target_dependencies(publish_address_book rclcpp)

install(TARGETS
    publish_address_book
  DESTINATION lib/${PROJECT_NAME})

rosidl_target_interfaces(publish_address_book
  ${PROJECT_NAME} "rosidl_typesupport_cpp")
```
编译与运行：
```shell
colcon build --packages-up-to more_interfaces

source install/local_setup.bash
ros2 run more_interfaces publish_address_book
```
### 2.10 参数的使用（C++）
先建立package：
```shell
ros2 pkg create --build-type ament_cmake cpp_parameters --dependencies rclcpp
```
在ros2_ws/src/cpp_parameters/src下建立
cpp_parameters_node.cpp，内容是：
```c++
#include <chrono>
#include <functional>
#include <string>

#include <rclcpp/rclcpp.hpp>

using namespace std::chrono_literals;

class MinimalParam : public rclcpp::Node
{
public:
  MinimalParam()
  : Node("minimal_param_node")
  {
    this->declare_parameter("my_parameter", "world");

    timer_ = this->create_wall_timer(
      1000ms, std::bind(&MinimalParam::timer_callback, this));
  }

  void timer_callback()
  {
    std::string my_param = this->get_parameter("my_parameter").as_string();

    RCLCPP_INFO(this->get_logger(), "Hello %s!", my_param.c_str());

    std::vector<rclcpp::Parameter> all_new_parameters{rclcpp::Parameter("my_parameter", "world")};
    this->set_parameters(all_new_parameters);
  }

private:
  rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char ** argv)
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<MinimalParam>());
  rclcpp::shutdown();
  return 0;
}
```
在CMakeLists.txt中添加依赖：
跟在find_package(rclcpp REQUIRED)后面：
```c++
add_executable(minimal_param_node src/cpp_parameters_node.cpp)
ament_target_dependencies(minimal_param_node rclcpp)

install(TARGETS
    minimal_param_node
  DESTINATION lib/${PROJECT_NAME}
)
```
编译与运行：
```shell
colcon build --packages-select cpp_parameters

source install/setup.bash

ros2 run cpp_parameters minimal_param_node
```
通过终端修改参数：
```shell
ros2 param list
ros2 param set /minimal_param_node my_parameter earth
```
也可以在launch文件中修改parameter，在ros2_ws/src/cpp_parameters/中创建cpp_parameters_launch.py
内容是：
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package="cpp_parameters",
            executable="minimal_param_node",
            name="custom_minimal_param_node",
            output="screen",
            emulate_tty=True,
            parameters=[
                {"my_parameter": "earth"}
            ]
        )
    ])
```
在CMakeLists.txt中添加：
```c++
install(
  DIRECTORY launch
  DESTINATION share/${PROJECT_NAME}
)
```
编译与运行：
```shell
colcon build --packages-select cpp_parameters

source install/setup.bash

ros2 launch cpp_parameters cpp_parameters_launch.py
```
### 2.11 参数的使用（Python）
创建一个package：
```shell
ros2 pkg create --build-type ament_python python_parameters --dependencies rclpy
```
在ros2_ws/src/python_parameters/python_parameters下创建
python_parameters_node.py，内容是：
```python
import rclpy
import rclpy.node

class MinimalParam(rclpy.node.Node):
    def __init__(self):
        super().__init__('minimal_param_node')

        self.declare_parameter('my_parameter', 'world')

        self.timer = self.create_timer(1, self.timer_callback)

    def timer_callback(self):
        my_param = self.get_parameter('my_parameter').get_parameter_value().string_value

        self.get_logger().info('Hello %s!' % my_param)

        my_new_param = rclpy.parameter.Parameter(
            'my_parameter',
            rclpy.Parameter.Type.STRING,
            'world'
        )
        all_new_parameters = [my_new_param]
        self.set_parameters(all_new_parameters)

def main():
    rclpy.init()
    node = MinimalParam()
    rclpy.spin(node)

if __name__ == '__main__':
    main()
```
在setup.py中添加依赖：
```python
entry_points={
    'console_scripts': [
        'minimal_param_node = python_parameters.python_parameters_node:main',
    ],
},
```
编译与运行：
```shell
colcon build --packages-select python_parameters

source install/setup.bash

ros2 run python_parameters minimal_param_node
```
通过launch file修改parameter的值，在ros2_ws/src/python_parameters/中建立python_parameters_launch.py，内容是：
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='python_parameters',
            executable='minimal_param_node',
            name='custom_minimal_param_node',
            output='screen',
            emulate_tty=True,
            parameters=[
                {'my_parameter': 'earth'}
            ]
        )
    ])
```
在setup.py中添加：
```python
import os
from glob import glob
# ...

setup(
  # ...
  data_files=[
      # ...
      (os.path.join('share', package_name), glob('launch/*launch.[pxy][yma]*')),
    ]
  )
```
## 3.中级
### 3.1 rosdep的使用
rosdep用来管理依赖

软件包的 package.xml 文件包含一组依赖项。该文件中的依赖项通常被称为 "rosdep keys"。它们用 <depend>、<test_depend>、<exec_depend>、<build_depend> 和 <build_export_depend> 标签表示。它们指定了在什么情况下需要使用每个依赖项。

如果依赖项只用于测试代码（如 gtest），则使用 test_depend。

对于仅在构建代码时使用的依赖项，请使用 build_depend。

对于代码导出头所需的依赖项，请使用 build_export_depend。

对于仅在运行代码时使用的依赖项，使用 exec_depend。

对于混合用途，请使用 depend，它涵盖了构建、导出和执行时的依赖项。

这些依赖项由软件包的创建者手动填充到 package.xml 文件中，并应详尽列出软件包所需的所有非构建库和软件包。
```shell
sudo rosdep init
rosdep update
rosdep install --from-paths src -y --ignore-src
```
### 3.2 创建一个action
首先先在workspace下创建一个package：
```shell
mkdir -p ros2_ws/src #you can reuse existing workspace with this naming convention
cd ros2_ws/src
ros2 pkg create action_tutorials_interfaces
```
action数据结构储存在.action文件中，在
action_tutorials_interfaces下mkdir action，在action目录下建立Fibonacci.action：
```c++
int32 order
---
int32[] sequence
---
int32[] partial_sequence
```
在CMakeLists.txt中添加相应的依赖：
```c++
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "action/Fibonacci.action"
)
```
在package.xml中添加相应的依赖：
```xml
<buildtool_depend>rosidl_default_generators</buildtool_depend>

<depend>action_msgs</depend>

<member_of_group>rosidl_interface_packages</member_of_group>
```
编译与运行：
```shell
# Change to the root of the workspace
cd ~/ros2_ws
# Build
colcon build
# Source our workspace
# On Windows: call install/setup.bat
. install/setup.bash
# Check that our action definition exists
ros2 interface show action_tutorials_interfaces/action/Fibonacci
```

### 3.3 编写action server与client（c++）
先在worksapce下src中创建package：
```shell
cd ~/ros2_ws/src
ros2 pkg create --dependencies action_tutorials_interfaces rclcpp rclcpp_action rclcpp_components -- action_tutorials_cpp
```
在action_tutorials_cpp/src/fibonacci_action_server.cpp中写：
```c++
#include <functional>
#include <memory>
#include <thread>

#include "action_tutorials_interfaces/action/fibonacci.hpp"
#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "rclcpp_components/register_node_macro.hpp"

#include "action_tutorials_cpp/visibility_control.h"

namespace action_tutorials_cpp
{
class FibonacciActionServer : public rclcpp::Node
{
public:
  using Fibonacci = action_tutorials_interfaces::action::Fibonacci;
  using GoalHandleFibonacci = rclcpp_action::ServerGoalHandle<Fibonacci>;

  ACTION_TUTORIALS_CPP_PUBLIC
  explicit FibonacciActionServer(const rclcpp::NodeOptions & options = rclcpp::NodeOptions())
  : Node("fibonacci_action_server", options)
  {
    using namespace std::placeholders;

    this->action_server_ = rclcpp_action::create_server<Fibonacci>(
      this,
      "fibonacci",
      std::bind(&FibonacciActionServer::handle_goal, this, _1, _2),
      std::bind(&FibonacciActionServer::handle_cancel, this, _1),
      std::bind(&FibonacciActionServer::handle_accepted, this, _1));
  }

private:
  rclcpp_action::Server<Fibonacci>::SharedPtr action_server_;

  rclcpp_action::GoalResponse handle_goal(
    const rclcpp_action::GoalUUID & uuid,
    std::shared_ptr<const Fibonacci::Goal> goal)
  {
    RCLCPP_INFO(this->get_logger(), "Received goal request with order %d", goal->order);
    (void)uuid;
    return rclcpp_action::GoalResponse::ACCEPT_AND_EXECUTE;
  }

  rclcpp_action::CancelResponse handle_cancel(
    const std::shared_ptr<GoalHandleFibonacci> goal_handle)
  {
    RCLCPP_INFO(this->get_logger(), "Received request to cancel goal");
    (void)goal_handle;
    return rclcpp_action::CancelResponse::ACCEPT;
  }

  void handle_accepted(const std::shared_ptr<GoalHandleFibonacci> goal_handle)
  {
    using namespace std::placeholders;
    // this needs to return quickly to avoid blocking the executor, so spin up a new thread
    std::thread{std::bind(&FibonacciActionServer::execute, this, _1), goal_handle}.detach();
  }

  void execute(const std::shared_ptr<GoalHandleFibonacci> goal_handle)
  {
    RCLCPP_INFO(this->get_logger(), "Executing goal");
    rclcpp::Rate loop_rate(1);
    const auto goal = goal_handle->get_goal();
    auto feedback = std::make_shared<Fibonacci::Feedback>();
    auto & sequence = feedback->partial_sequence;
    sequence.push_back(0);
    sequence.push_back(1);
    auto result = std::make_shared<Fibonacci::Result>();

    for (int i = 1; (i < goal->order) && rclcpp::ok(); ++i) {
      // Check if there is a cancel request
      if (goal_handle->is_canceling()) {
        result->sequence = sequence;
        goal_handle->canceled(result);
        RCLCPP_INFO(this->get_logger(), "Goal canceled");
        return;
      }
      // Update sequence
      sequence.push_back(sequence[i] + sequence[i - 1]);
      // Publish feedback
      goal_handle->publish_feedback(feedback);
      RCLCPP_INFO(this->get_logger(), "Publish feedback");

      loop_rate.sleep();
    }

    // Check if goal is done
    if (rclcpp::ok()) {
      result->sequence = sequence;
      goal_handle->succeed(result);
      RCLCPP_INFO(this->get_logger(), "Goal succeeded");
    }
  }
};  // class FibonacciActionServer

}  // namespace action_tutorials_cpp

RCLCPP_COMPONENTS_REGISTER_NODE(action_tutorials_cpp::FibonacciActionServer)
```
> using的用法：这行代码创建了一个名为 Fibonacci 的别名，它表示了 action_tutorials_interfaces::action::Fibonacci 这个类型。这个别名可以用来代替较长的类型名，使代码更具可读性和可维护性。

> using namespace std::placeholders; 的用法：
```c++
#include <iostream>
#include <functional>

void printSum(int a, int b) {
    std::cout << "Sum: " << a + b << std::endl;
}

int main() {
    using namespace std::placeholders;  // 引入 std::placeholders 命名空间

    auto addFive = std::bind(printSum, _1, 5);  // 直接使用 _1 作为占位符

    addFive(10);  // 调用 printSum(10, 5)

    return 0;
}

```
>在调用 detach() 之后，主线程不再等待分离的线程执行完成。在主线程执行完毕后，程序将立即退出。分离线程后，它会在后台独立运行，直到自身的任务完成。

在CMakeLists.txt中添加依赖：
```c++
add_library(action_server SHARED
  src/fibonacci_action_server.cpp)
target_include_directories(action_server PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_compile_definitions(action_server
  PRIVATE "ACTION_TUTORIALS_CPP_BUILDING_DLL")
ament_target_dependencies(action_server
  "action_tutorials_interfaces"
  "rclcpp"
  "rclcpp_action"
  "rclcpp_components")
rclcpp_components_register_node(action_server PLUGIN "action_tutorials_cpp::FibonacciActionServer" EXECUTABLE fibonacci_action_server)
install(TARGETS
  action_server
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)
```
再写客户端的代码，在action_tutorials_cpp/src/fibonacci_action_client.cpp中写代码：
```c++
#include <functional>
#include <future>
#include <memory>
#include <string>
#include <sstream>

#include "action_tutorials_interfaces/action/fibonacci.hpp"

#include "rclcpp/rclcpp.hpp"
#include "rclcpp_action/rclcpp_action.hpp"
#include "rclcpp_components/register_node_macro.hpp"

namespace action_tutorials_cpp
{
class FibonacciActionClient : public rclcpp::Node
{
public:
  using Fibonacci = action_tutorials_interfaces::action::Fibonacci;
  using GoalHandleFibonacci = rclcpp_action::ClientGoalHandle<Fibonacci>;

  explicit FibonacciActionClient(const rclcpp::NodeOptions & options)
  : Node("fibonacci_action_client", options)
  {
    this->client_ptr_ = rclcpp_action::create_client<Fibonacci>(
      this,
      "fibonacci");

    this->timer_ = this->create_wall_timer(
      std::chrono::milliseconds(500),
      std::bind(&FibonacciActionClient::send_goal, this));
  }

  void send_goal()
  {
    using namespace std::placeholders;

    this->timer_->cancel();

    if (!this->client_ptr_->wait_for_action_server()) {
      RCLCPP_ERROR(this->get_logger(), "Action server not available after waiting");
      rclcpp::shutdown();
    }

    auto goal_msg = Fibonacci::Goal();
    goal_msg.order = 10;

    RCLCPP_INFO(this->get_logger(), "Sending goal");

    auto send_goal_options = rclcpp_action::Client<Fibonacci>::SendGoalOptions();
    send_goal_options.goal_response_callback =
      std::bind(&FibonacciActionClient::goal_response_callback, this, _1);
    send_goal_options.feedback_callback =
      std::bind(&FibonacciActionClient::feedback_callback, this, _1, _2);
    send_goal_options.result_callback =
      std::bind(&FibonacciActionClient::result_callback, this, _1);
    this->client_ptr_->async_send_goal(goal_msg, send_goal_options);
  }

private:
  rclcpp_action::Client<Fibonacci>::SharedPtr client_ptr_;
  rclcpp::TimerBase::SharedPtr timer_;

  void goal_response_callback(std::shared_future<GoalHandleFibonacci::SharedPtr> future)
  {
    auto goal_handle = future.get();
    if (!goal_handle) {
      RCLCPP_ERROR(this->get_logger(), "Goal was rejected by server");
    } else {
      RCLCPP_INFO(this->get_logger(), "Goal accepted by server, waiting for result");
    }
  }

  void feedback_callback(
    GoalHandleFibonacci::SharedPtr,
    const std::shared_ptr<const Fibonacci::Feedback> feedback)
  {
    std::stringstream ss;
    ss << "Next number in sequence received: ";
    for (auto number : feedback->partial_sequence) {
      ss << number << " ";
    }
    RCLCPP_INFO(this->get_logger(), ss.str().c_str());
  }

  void result_callback(const GoalHandleFibonacci::WrappedResult & result)
  {
    switch (result.code) {
      case rclcpp_action::ResultCode::SUCCEEDED:
        break;
      case rclcpp_action::ResultCode::ABORTED:
        RCLCPP_ERROR(this->get_logger(), "Goal was aborted");
        return;
      case rclcpp_action::ResultCode::CANCELED:
        RCLCPP_ERROR(this->get_logger(), "Goal was canceled");
        return;
      default:
        RCLCPP_ERROR(this->get_logger(), "Unknown result code");
        return;
    }
    std::stringstream ss;
    ss << "Result received: ";
    for (auto number : result.result->sequence) {
      ss << number << " ";
    }
    RCLCPP_INFO(this->get_logger(), ss.str().c_str());
    rclcpp::shutdown();
  }
};  // class FibonacciActionClient

}  // namespace action_tutorials_cpp

RCLCPP_COMPONENTS_REGISTER_NODE(action_tutorials_cpp::FibonacciActionClient)
```
在CMakeLists.txt中添加依赖：
```c++
add_library(action_client SHARED
  src/fibonacci_action_client.cpp)
target_include_directories(action_client PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_compile_definitions(action_client
  PRIVATE "ACTION_TUTORIALS_CPP_BUILDING_DLL")
ament_target_dependencies(action_client
  "action_tutorials_interfaces"
  "rclcpp"
  "rclcpp_action"
  "rclcpp_components")
rclcpp_components_register_node(action_client PLUGIN "action_tutorials_cpp::FibonacciActionClient" EXECUTABLE fibonacci_action_client)
install(TARGETS
  action_client
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)
```
运行：
```shell
ros2 run action_tutorials_cpp fibonacci_action_client
```
### 3.3 编写action server与client（python）
在 fibonacci_action_server.py 中写：
```python
import time


import rclpy
from rclpy.action import ActionServer
from rclpy.node import Node

from action_tutorials_interfaces.action import Fibonacci


class FibonacciActionServer(Node):

    def __init__(self):
        super().__init__('fibonacci_action_server')
        self._action_server = ActionServer(
            self,
            Fibonacci,
            'fibonacci',
            self.execute_callback)

    def execute_callback(self, goal_handle):
        self.get_logger().info('Executing goal...')


        feedback_msg = Fibonacci.Feedback()

        feedback_msg.partial_sequence = [0, 1]


        for i in range(1, goal_handle.request.order):

            feedback_msg.partial_sequence.append(

                feedback_msg.partial_sequence[i] + feedback_msg.partial_sequence[i-1])

            self.get_logger().info('Feedback: {0}'.format(feedback_msg.partial_sequence))

            goal_handle.publish_feedback(feedback_msg)

            time.sleep(1)


        goal_handle.succeed()

        result = Fibonacci.Result()

        result.sequence = feedback_msg.partial_sequence

        return result


def main(args=None):
    rclpy.init(args=args)

    fibonacci_action_server = FibonacciActionServer()

    rclpy.spin(fibonacci_action_server)


if __name__ == '__main__':
    main()
```
在终端中测试：
```shell
ros2 action send_goal --feedback fibonacci action_tutorials_interfaces/action/Fibonacci "{order: 5}"
```
在 fibonacci_action_client.py 中写：

```python
import rclpy
from rclpy.action import ActionClient
from rclpy.node import Node

from action_tutorials_interfaces.action import Fibonacci


class FibonacciActionClient(Node):

    def __init__(self):
        super().__init__('fibonacci_action_client')
        self._action_client = ActionClient(self, Fibonacci, 'fibonacci')

    def send_goal(self, order):
        goal_msg = Fibonacci.Goal()
        goal_msg.order = order

        self._action_client.wait_for_server()

        self._send_goal_future = self._action_client.send_goal_async(goal_msg, feedback_callback=self.feedback_callback)

        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('Goal rejected :(')
            return

        self.get_logger().info('Goal accepted :)')

        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.get_result_callback)

    def get_result_callback(self, future):
        result = future.result().result
        self.get_logger().info('Result: {0}'.format(result.sequence))
        rclpy.shutdown()

    def feedback_callback(self, feedback_msg):
        feedback = feedback_msg.feedback
        self.get_logger().info('Received feedback: {0}'.format(feedback.partial_sequence))


def main(args=None):
    rclpy.init(args=args)

    action_client = FibonacciActionClient()

    action_client.send_goal(10)

    rclpy.spin(action_client)


if __name__ == '__main__':
    main()
```
### 3.4 在一个程序里面运行多个节点

先创建一个容器：
```shell
ros2 run rclcpp_components component_container
```
把server client加进容器中运行
```shell
ros2 component load /ComponentManager composition composition::Server
ros2 component load /ComponentManager composition composition::Client
```
```shell
ros2 component load /ComponentManager composition composition::Talker
ros2 component load /ComponentManager composition composition::Listener
```
运行常用的组件
```shell
ros2 run composition manual_composition
```
接着运行：
```shell
ros2 run composition dlopen_composition `ros2 pkg prefix composition`/lib/libtalker_component.so `ros2 pkg prefix composition`/lib/liblistener_component.so
```
停止节点运行：
```shell
ros2 component unload /ComponentManager 1 2
```
### 3.5 launch file
#### 3.5.1 创建launch文件
系统配置包括运行哪些程序、在哪里运行、传递哪些参数，以及 ROS 特有的约定，通过为每个组件提供不同的配置，可以方便地在整个系统中重复使用组件

创建一个launch文件夹：
```shell
mkdir launch
```
创建一个文件launch/turtlesim_mimic_launch.py：
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='turtlesim',
            namespace='turtlesim1',
            executable='turtlesim_node',
            name='sim'
        ),
        Node(
            package='turtlesim',
            namespace='turtlesim2',
            executable='turtlesim_node',
            name='sim'
        ),
        Node(
            package='turtlesim',
            executable='mimic',
            name='mimic',
            remappings=[
                ('/input/pose', '/turtlesim1/turtle1/pose'),
                ('/output/cmd_vel', '/turtlesim2/turtle1/cmd_vel'),
            ]
        )
    ])
```