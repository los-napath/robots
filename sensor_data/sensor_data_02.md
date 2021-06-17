# Sensor data (2)

[README](../README.md)

---

In this lecture, we add some programs to `sensors.py`, and challenge simple image processing.

## Excercise

Make a python file inside of the `oit_pbl_ros_samples` package.

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import rospy
import tf
from sensor_msgs.msg import LaserScan, Image
from nav_msgs.msg import Odometry

import cv2  # add
from cv_bridge import CvBridge  # add

class Sensors(object):
    def __init__(self):
        self.laser = SensorMessageGetter("/base_scan", LaserScan)
        self.odom = SensorMessageGetter("/odom", Odometry)
        self.img = SensorMessageGetter("/image", Image)
        # add
        self.cv_bridge = CvBridge()
        self.image_pub = rospy.Publisher("/image_mod", Image, queue_size=1)
....

    def process_img(self, msg):
        if msg:
            rospy.loginfo("Recv sensor data. type = %s", type(msg))
            # check reference
            # http://docs.ros.org/en/api/sensor_msgs/html/msg/Image.html
            rospy.loginfo("msg.width = %d, msg.height = %d",
                          msg.width, msg.height)
            # add
            try:
                cv_image = self.cv_bridge.imgmsg_to_cv2(msg, "bgr8")
                hsv = cv2.cvtColor(cv_image, cv2.COLOR_BGR2HSV)
                blue = cv2.inRange(hsv, (100, 200, 200), (140, 255, 255))
                send = self.cv_bridge.cv2_to_imgmsg(blue, "mono8")
                self.image_pub.publish(send)
            except Exception as e:
                rospy.logerr("%s:%s", rospy.get_name(), str(e))
...                
    def process(self):
        rate = rospy.Rate(20)
        tm = rospy.Time.now()
        while (rospy.Time.now().to_sec() - tm.to_sec()) < 100: # change 10 -> 100

```

## Run

At first, launch the simulator.

```shell
$ roslaunch oit_stage_ros navigation.launch
```

Drag blue block to the front of the robot.

![2021-06-19_154803.png](./2021-06-19_154803.png)

After a while run the `sensors.py`.

- You can see the information about received sensor data.

```shell
$ rosrun oit_pbl_ros_samples sensors.py
[INFO] [1624081858.601793, 16.200000]: /sensors:Started
[INFO] [1624081858.706859, 16.300000]: Recv sensor data. type = <class 'sensor_msgs.msg._LaserScan.LaserScan'>
[INFO] [1624081858.710373, 16.300000]: len(msg.ranges) = 720
[INFO] [1624081858.714078, 16.300000]: msg.ranges[0] = 1.412500
```

Open another terminal and type the following command.

```shell
$ rqt_image_view
```

Image viewer window will come up. Select `image` topic from drop down box and you can see the image grabed by the virtual camera.

![2021-06-19_154626.png](./2021-06-19_154626.png)

Select `image_mod`, you can see the image processing result, which shows extracted blue colord region.

![2021-06-19_154610.png](./2021-06-19_154610.png)

## Question (1)

- Try to extract yellow and green objects like as blue block.

## ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+)Checkpoint(sensor data)

- It's OK, you can finish the question 1.

---

[README](../README.md)