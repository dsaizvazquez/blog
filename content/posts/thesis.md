---
title: "Real time vehicle detection and tracking with speed estimation for drone surveillance, how to use"
date: 2023-11-20T10:52:23+01:00
draft: false
---

# Real time vehicle detection and tracking with speed estimation for drone surveillance, how to use


## Introduction 

My Bachelor's thesis was presented this september, and while it is accessible through [the university's repository](https://upcommons.upc.edu/handle/2117/394340) I thought it might be more interesting to write a short how to use for a more practical perspective. The code used for the tool is fully available in [here](https://github.com/dsaizvazquez/realtimevehicletracker). 

The objective of the thesis was to desing, implement and test a vehicle speed estimator which uses exclusively inputs from a drone. The objective was to make it adaptable to any airborne machine with minimal alterations, and the downside was a decrease in accuracy.

![Test Image](images/image19.gif)

## Installation

To install is necessary to compile OpenCV by hand with CUDA and CUDnn dependencies. Regardless of the system used, head [here]({{< ref "/content/posts/Installing_OpenCV_On_AWS.md" >}}) for a tutorial. 

Assuming OpenCV is installed, it should be as easy as executing the install script from the root directory. The line should be: ``` bash scripts/install.sh ```.

Once installed, the binary file should be ```build/bin/main```.

The binary by itself is not enough to run the software. It needs to have a running instance of [MediaMTX](https://github.com/bluenviron/mediamtx) and a mysql server. mediamtx can be downloaded from Github and mysql it depends on the linux based OS. 

It is also necessary to have a coco file and a NN model. Both can be downloaded from the Git Large File Storage.

## Usage

To use the main binary a configuration file has to be used. This file has to be based in ```config.yaml``` file, which is an example of which properties are variable. 

As a quick test, it is only necessary to modify the SQL database in the ```config.yaml``` file and run ```build/bin/main config.yaml```. The speed obtained is going to be wrong as the properties set for the projection are incorrect, but it should be possible to see the detection of objects and the video processing. For more information on the parameters modifiable head to [Configuration parameters]({{< ref "thesis.md#config" >}})

## Drone usage and simulation

For the system to be used the attitude parameters have to be sent to the server. If the video is static and the parameters are known, they can be introduced manually via the config.yaml file: 

```
#initial projection Parameters
focalLength: 0.5
aspectRatio: 0.75
offsetX: 0
offsetY: 0
sensorWidth: 1

height: 5
roll: 0
pitch: -45
yaw: -90
```

The camera parameters such as aspectRatio and sensor width are fixed and can be found in the camera's datasheet. 

If the intention is to use it in pair with a drone, I recommend importing the tracking library and using the ```Message``` union. This package can be sent directly as a string once it is filled via the UDP port and it will be absorbed by the server. 

In case of wanting to do manual tests it is possible to use the ```dronePos.yaml``` in pair with the ```build/bin/udpConnSimulator``` binary to send a package to the server.


## Configuration parameters {#config}

#### UDP configuration

The configuration of the UDP port for the telemetry, the IP can be changed depending on the necessities of the usage.

```
#UDP config 
port: 8998
IP: 0.0.0.0

```

#### TCP configuration

The configuration of the TCP server output. It was used as a showcase of the system and it contains information of the vehicles detected and their speed. It can be utilised independently of video to conduct speed analyses without visual aids.

```
#TCP config
tcpPort: 12520
updateRate: 5 # one every 15 frames

```

#### Stream Input

The input configuration. The url is set to work with MediaMTX, but it is possible to use directly without the server. I heavily recommend using the media server. 

You can also reduce the load by changing the number of frames it skips, dividing the video frame rate, and setting the error delay - how long it waits to retry if an input video fails. This is helpful when the system tends to fail as it will reconnect to the video once it recovers. If the video source is stable, a larger error delay may be better.

```
#Input config
input: rtsp://localhost:8554/in
skippedFrames: 1
errorDelay: 1 #s

```
#### Stream Output

The output configuration. It can be switched between a video stream and a video. The output line is quite long but it is to allow a manual configuration. 

```

# Output config
outputType: 's' #types are s: stream v: video
fps: 15
output: "appsrc ! videoconvert ! x264enc \
speed-preset=ultrafast bitrate=1800  key-int-max=40 \
! rtspclientsink location=rtspt://localhost:8554/out"
# output: "outcpp.avi"

```

#### Neural network configuration

The neural network used in the SORT algorithm can be set using the configuration. These examples have been tested and are based on the NN in the models folder. 

```

# NN config
cocoFile: "models/coco.names"

# # ---------------YOLOV7 TINY----------------------------------
# YOLOv: 7
# nnetFile: "models/yolov7-tiny.onnx"
# InputHeight: 640
# INPUT_WIDTH: 640

# ---------------YOLOV7----------------------------------
YOLOv: 7
nnetFile: "models/yolov7.onnx"
InputHeight: 640
InputWidth: 640


```


#### NN Thresholds

The score, similarity and confidence threshold affect the inner workings of the detection algorithm. Can be optimized for use.

```

# Detection Config
ScoreThreshold: 0.3
NmsThreshold: 0.3
ConfidenceThreshold: 0.3

```


#### SORT Config

It is possible to configure the SORT algorithm  and change the covariance of the initial noise. This will affect how it tracks the vehicles and has to be optimized. Furthermore the iou Threshold and the maximum age affect to how long it will keep tracking. They affect the missed detections (maxAge) and when it is considered that the detections are too far appart (iouThreshold).

```
# Tracking config
processNoiseCov: 0.01
measurementNoiseCov: 0.1
errorCovPost: 1

iouThreshold: 0.3
maxAge: 2

```

#### SQL config

The SQL database is used for storing all the data points measured. It has to be configured with an SQL password and schema regardless of the data is going to be used or not. Might eventually make it not necessary for the use of the tool.

```
#SQL config

sqlHostName: "tcp://127.0.0.1:3306"
sqlUserName: "root"
sqlPassword: "XXXXXXX"
sqlSchema:   "YYYYYY"
```

