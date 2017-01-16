# 配置Gmapping
====================

## Overview
-----------------------------------------------
环境：Ubuntu 16.04，ROS：kinetic，需要以下packages：

- urg_node：hokuyo的驱动
- gmapping：gmapping的核心package
- laser\_scan\_matcher：为gmapping提供odom

## laser\_scan\_matcher
----------------------------------------------
**详细安装过程参考：**[Durant35/laser\_scan\_matcher](https://github.com/Durant35/laser_scan_matcher) 

为了方便，复制了其中Ubuntu上的配置的部分如下：

- 安装python-wstool和python-rosinstall-generator：

	```
	$ sudo apt-get install python-wstool
	$ sudo apt-get install python-rosinstall-generator
	```

- 安装PCL package：

	```
	$ cd <your-catkin-workspace-path>
	$ rosinstall_generator pcl_conversions pcl_msgs pcl_ros ‐‐rosdistro <your-ros-version> ‐‐deps ‐‐wet‐only ‐‐exclude roslisp ‐‐tar > ros_pcl.rosinstall
	$ wstool init src ros_pcl.rosinstall
	# This rosdep command installs all the missing system dependency
	# (must be described in package.xml) in all the packages in your src directory.
	$ rosdep install ‐‐from‐paths src ‐‐ignore‐src ‐‐rosdistro <your-ros-version> ‐y ‐r
	$ catkin_make
	```

- 安装csm

	```
	$ sudo apt-get install gsl-bin libgsl0-dev
	$ cd <your-catkin-workspace-path>/src
	$ git clone https://github.com/AndreaCensi/csm.git
	$ cd csm/
	$ ./install_quickstart.sh
	```

- 安装laser\_scan\_matcher

	```
	$ cd <your-catkin-workspace-path>/src
	# clone laser_scan_matcher into catkin workspace's src
	$ git clone https://github.com/Durant35/laser_scan_matcher.git
	# add csm path into your cmake's PKG_CONFIG_PATH
	$ export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:<your-catkin-workspace-path>/src/csm/sm/pkg‐config
	# make targets
	$ cd <your-catkin-workspace-path>
	$ catkin_make laser_scan_matcher
	```

- 测试[laser\_scan\_matcher](http://wiki.ros.org/laser_scan_matcher)

   先source当前的workspace，然后运行demo.launch：
	
	```
$ roslaunch laser_scan_matcher demo.launch
	```

## Gampping
---------------------------------------------

**参考链接：**

  - [slam\_gmapping](http://wiki.ros.org/laser_scan_matcher)

  - [gmapping](http://wiki.ros.org/gmapping)

  - [How to Build a Map Using Logged Data](http://wiki.ros.org/slam_gmapping/Tutorials/MappingFromLoggedData)

  - [openslam](http://openslam.org/gmapping.html)

### 具体过程如下：

- 将slam_gmapping和gmapping克隆到workspace

	```
	$ cd <your catkin workspace>/src
	$ git clone https://github.com/ros-perception/slam_gmapping.git
	```

- 上面两个package是基于openslam封装的hokuyo devel的ROS package，因此还需要原始的openslam package。因此，在workspace的src下：
	
	```
	$ git clone https://github.com/ros-perception/openslam_gmapping.git
	# 进行编译
	$ catkin_make
	```

- 测试gmapping。详见[logged file建图](http://wiki.ros.org/slam_gmapping/Tutorials/MappingFromLoggedData)。

  1. 下载一个bag。ROS上How to build a map using logged data提供的bag可能无法得到完整的地图，可以尝试克隆下面的bag。
		  
		```
		$ cd <your ros bag workspace>
		$ git clone https://github.com/RH-Song/slam-bag-files.git
		```
	
	2. 启动一个roscore:
		
		```
		$ roscore
		```
	
  3. 设置 use\_sim\_time 为true:
	
		```
		$ rosparam set use_sim_time true
		```

 4. 启动slam\_gmapping:

		```
		$ rosrun gmapping slam_gmapping scan:=base_scan
		```


 5. 启动rviz，观察建图过程。
  
     -  在workspace/src下，下载launch文件以及hokuyo驱动：
			
			```
			$ git clone https://github.com/RH-Song/gmapping-hokuyo.git
			```

	  - 到gmapping-launch目录下启动rviz：
	 	
			```
			$ rosrun rviz rviz -d gmapping.rviz
			```

 6. 开始回播bag文件，以上面的包为例子：
	
		```
		$ rosbag play --clock gmapping-1.bag
		```

 7. 图片保存。等待bag文件播放完成以后，可以将建成的图片保存下来：

		```
		$ rosrun map_server map_saver -f <map_name>
		```

  8. 最后得到的图片大致(局部)如图：
![gmapping-map1](/home/slam/Pictures/bbjietu.png)

## 使用hokuyo建图
-------------------------------------------------------------

- 添加hokuyo的驱动urg_node package，上一步已经下载了相关驱动文件，可以进行编译。
	
	```
	$ catkin_make
	```

- 连接hokuyo激光雷达

- 运行gmapping.launch，进行建图。
```
$ roslaunch gmapping-launch gmapping.launch
```