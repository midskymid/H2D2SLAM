<font size=6> **定位+建图+轨迹规划模块** </font>

本文档参考了视频教程[从零制作自主空中机器人](https://www.bilibili.com/video/BV1WZ4y167me?p=1),github链接为https://github.com/ZJU-FAST-Lab/Fast-Drone-250。 vins-fusion gpu参考了https://github.com/pjrambo/VINS-Fusion-gpu。

<font color="#dd0000">安全事项</font>

* 本模块硬件部分参考高飞老师课程内容设计并搭建，下面是与本模块使用和设置相关的视频课程：

- [第四章：飞控设置与试飞](#第四章飞控设置与试飞)
- [第五章：机载电脑与传感器的安装](#第五章机载电脑与传感器的安装)
- [第八章：常用实验与调试软件的安装与使用](#第八章常用实验与调试软件的安装与使用)
- [第九章：Ego-Planner代码框架与参数介绍](#第九章ego-planner代码框架与参数介绍)
- [第十章：VINS的参数设置与外参标定](#第十章vins的参数设置与外参标定)
- [第十一章：Ego-Planner的实验](#第十一章ego-planner的实验)
- [Q&A 常见问题及解答](#qa-常见问题及解答)


## 第四章：飞控设置与试飞

* 请烧录本git项目下的`/firmware/px4_fmu-v5_default.px4`固件，这个固件是官方1.11.0版本固件编译而来，如有需要可以自行编译。实测1.13版本固件存在BUG，不建议使用，更老的固件版本未经测试。

* 在飞控的sd卡的根目录下创建`/etc/extras.txt`，写入

  ```
  mavlink stream -d /dev/ttyACM0 -s ATTITUDE_QUATERNION -r 200
  mavlink stream -d /dev/ttyACM0 -s HIGHRES_IMU -r 200
  ```
  
  以提高imu发布频率，但可能非该品牌飞控不起作用，请参考shfiles/slam.sh文件中使用命令提高IMU频率。
  
* 修改机架类型为 `Generic 250 Racer`，代指250mm轴距机型。如果是其他尺寸的机架，请根据实际轴距选择机架类型

* 下面部分跟无人机相关，跟该模块无关：
* 修改`dshot_config`为dshot600
* 上面部分跟无人机相关，跟该模块无关：

* 修改`CBRK_SUPPLY_CHK`为894281 *执行这步跳过了电源检查，因此左侧栏的电池设置部分就算是红的也没关系*

* 修改`CBRK_USB_CHK`为197848

* 修改`CBRK_IO_SAFETY`为22027

* 修改`SER_TEL1_BAUD`为921600

* 修改`SYS_USE_IO`为0（搜索不到则不用管）


* 下面部分跟无人机相关，跟该模块无关：
* 上电前请先用万用表通断档检测电源正负焊点是否短接，强烈建议第一次上电前先接一个[短路保护器](https://item.taobao.com/item.htm?spm=a230r.1.14.6.72b83b20uNbZk7&id=656973651729&ns=1&abbucket=19#detail)

* <font color="#dd0000">检测电机转向前确保没有安装螺旋桨！！！！</font>

* 修改电机转向：进入mavlink控制台

  ```
  dshot reverse -m 1
  dshot save -m 1
  ```

  修改`1`为需要反向的电机序号
  
* <font color="#dd0000">第一次试飞请务必找有自稳模式下飞行经验的飞手协助，只飞过大疆无人机的飞手99%无法飞好！</font>

* 身边没有有经验的飞手怎么办？详见Q&A

* 上面部分跟无人机相关，跟该模块无关：

## 第五章：机载电脑与传感器的安装

* 碳板已经预留了拆壳NUC的安装空位。如果想拆壳安装NUC，需要额外购买USB网卡，或者拆下自带的网卡天线找地方固定住，并且由于碳纤维板导电，请务必用尼龙柱把NUC支起来，相关资料请自行查阅。（本模块选择全部使用3D打印件）
* 机载电脑使用4S航模电池直接供电，正常情况下没有问题。但理论上最好接一个稳压模块，否则会出现机载电脑关机的情况。（其实可以电池上接一个BB响，电池电压低于15V后就不要继续使用了，理论上2300mAh的4S电池能够使用30分钟以上。）


## 第八章：常用实验与调试软件的安装与使用

* VScode：`sudo dpkg -i ***.deb`
* Terminator：`sudo apt install terminator`
* Plotjuggler：
  * `sudo apt install ros-noetic-plotjuggler`
  * `sudo apt install ros-noetic-plotjuggler-ros`
  * `rosrun plotjuggler plotjuggler`
* Net-tools：
  * `sudo apt install net-tools`
  * `ifconfig`
* ssh：
  * `sudo apt install openssh-server`
  * 在笔记本上：`ping 192.168.**.**`
  * `sudo gedit /etc/hosts`
  * 加上一行：`192.168.**.** fast-drone`
  * `ping fast-drone`
  * `ssh fast-drone@fast-drone`(`ssh 用户名@别名`)

## 第九章：Ego-Planner代码框架与参数介绍
* `src/planner/plan_manage/launch/single_run_in_exp.launch`下的：
  * `map_size`：当你的地图大小较大时需要修改，注意目标点不要超过map_size/2
  * `fx/fy/cx/cy`：修改为你的深度相机的实际内参（下一课有讲怎么看）
  * `max_vel/max_acc`：修改以调整最大速度、加速度。
  * `flight_type`：1代表rviz选点模式，2代表waypoints跟踪模式
* `src/planner/plan_manage/launch/advanced_param_exp.xml`下的：
  * `resolution`：代表栅格地图格点的分辨率，单位为米。越小则地图越精细，但越占内存。最小不要低于0.1
  * `obstacles_inflation`：代表障碍物膨胀大小，单位为米。建议至少设置为搭载机器半径的1.5倍以上，但不要超过`resolution`的4倍。如果机器轴距较大，请相应改大`resolution`
  
## 第十章：VINS的参数设置与外参标定
* 检查飞控mavros连接正常
  * `ls /dev/tty*`，确认飞控的串口连接正常。一般是`/dev/ttyACM0`
  * `sudo chmod 777 /dev/ttyACM0`，为串口附加权限
  * `roslaunch mavros px4.launch`
  * `rostopic hz /mavros/imu/data_raw`，确认飞控传输的imu频率在200hz左右
* 检查realsense驱动正常
  * `roslaunch realsense2_camera rs_camera.launch`
  * 进入远程桌面，`rqt_image_view`
  * 查看`/camera/infra1/image_rect_raw`,`/camera/infra2/image_rect_raw`,`/camera/depth/image_rect_raw`话题正常
* VINS参数设置
  * 进入`realflight_modules/VINS_Fusion/config/`
  
  * 驱动realsense后，`rostopic echo /camera/infra1/camera_info`，把其中的K矩阵中的fx,fy,cx,cy填入`left.yaml`和`right.yaml`
  
  * 在home目录创建`vins_output`文件夹(如果你的用户名不是fast-drone，需要修改config内的vins_out_path为你实际创建的文件夹的绝对路径)
  
  * 修改`fast-drone-250.yaml`的`body_T_cam0`和`body_T_cam1`的`data`矩阵的第四列为你的无人机上的相机相对于飞控的实际外参，单位为米，顺序为x/y/z，第四项是1，不用改
  
* VINS外参精确自标定
  * `sh shfiles/rspx4.sh`
  * `rostopic echo /vins_fusion/imu_propagate`
  * 拿起飞机沿着场地<font color="#dd0000">尽量缓慢</font>地行走，场地内光照变化不要太大，灯光不要太暗，<font color="#dd0000">不要使用会频闪的光源</font>，尽量多放些杂物来增加VINS用于匹配的特征点
  * 把`vins_output/extrinsic_parameter.txt`里的内容替换到`fast-drone-250.yaml`的`body_T_cam0`和`body_T_cam1`
  * 重复上述操作直到走几圈后VINS的里程计数据偏差收敛到满意值（一般在0.3米内）
* 建图模块验证
  * `sh shfiles/rspx4.sh`
  * `roslaunch ego_planner single_run_in_exp.launch`
  * 进入远程桌面 `roslaunch ego_planner rviz.launch`

## 模块的实验
* 启动Realsense，pixhawk，SLAM：
  * 打开一个终端窗口
  * `cd ~/H2D2SLAM`
  * `source ./devel/setup.sh`
  * `sh shfiles/slam.sh`
  * 等待上面的终端窗口出现slam输出数据时，再在H2D2SLAM文件夹路径下打开一个终端窗口
  * `rostopic echo /vins_fusion/imu_propagate`
  * 拿起模块进行缓慢的小范围晃动，放回原地后确认positions没有太大误差
  * 进入远程桌面, 再在H2D2SLAM文件夹路径下打开一个终端窗口
  * `source ./devel/setup.sh`
  * `sh shfiles/envplan.sh`，
  * 在弹出窗口中按下G键加鼠标左键点选目标点即可显示规划的轨迹

## Q&A 常见问题及解答
	
	Q: 能不能用D435i自带的imu运行vins?
	A: 不行，因为435的imu噪声很大
	
	Q: 提供的v1.11.0固件有什么改动吗？必须使用这个固件吗？
	A: 没有任何改动，是直接从px4官方下载的。目前仅在该版本上测试通过了本套代码，且在v1.13上测试失败，表现为VINS会经常崩溃。其他飞控/其他版本固件没有测试，有需要的同学可以自行测试。
	
	Q: 运行vins后报红字错误？
	A: 大概率是你改config后格式错误，照着报错去修改对应的config
	
	Q: VINS飘怎么办？
	A: 1. 检查环境中是否有强反光物体（瓷砖、玻璃等）
	   2. 尽量缓慢地移动模块，场景内尽量不要有运动物体
	   3. 尽量准确地测量初始外参
	   4. 不要在运行vins的时候在远程桌面上运行rviz（会占用大量CPU资源），实在想开建议去配一下ROS多机，然后在笔记本上开（其实可以，在向日葵上运行是没问题的，因为使用的是vins-fusion gpu）
	   5. 检查环境中是否有频闪光源（肉眼无法看出，在realsense的单目画面中检查）
	

