---
title: "Real time vehicle detection and tracking with speed estimation for drone surveillance, a summary"
date: 2023-11-23T16:29:53+01:00
draft: true
---

# Real time vehicle detection and tracking with speed estimation for drone surveillance, a summary


## Introduction 

My Bachelor's thesis was presented this september, and while it is accessible through [the university's repository](https://upcommons.upc.edu/handle/2117/394340) I thought it might be more interesting to read a short summary with a more practical perspective. The code used for the tool is fully available in [here](https://github.com/dsaizvazquez/realtimevehicletracker). 

The objective of the thesis was to desing, implement and test a vehicle speed estimator which uses exclusively inputs from a drone. The idea was to build it in such a way that any aerial vehicle with minimal modifications could be used, and the cost came in the shape of precision.


## Technical aspects

The tool is built to be used on a cloud server, and optimized for AWS. While it is easier to execute on a powerful EC2 instance, it has been also used successfully in my home computer, which has a NVIDIA 1060 and hasn't been upgraded since 2016 when I bought it, so as long as the computer has a graphics card of similar capacity it should be possible to run it locally. The reason behind it was that it needed to be executed isolated from the actual drone and it causes a bit of delay in the video, but even during field tests the results were tolerable. 

To obtain the speed it was necessary to first detect and track vehicles and afterwards transform those 2D positions into 3D coordinates. For the detection and tracking the [SORT](https://arxiv.org/abs/1602.00763) is used with a YOLOv7 neural network trained with COCO. While it would've been better to train it for the task due to time constraints it was impossible. The tool can use NN based in YOLOv5,v6 and v7 as long as the format is onnx, so it could be improved with time. If you are interested in the inner workings of the algorithm I suggest heading to Ritesh's explanation of deepSORT found [here](https://medium.com/augmented-startups/deepsort-deep-learning-applied-to-object-tracking-924f59f99104). It goes into detail of SORT and their improved version, deepSORT, which uses deep learning for object association. 

Once the positions were obtained, they had to be transformed to 3D coordinates. This step is harder than it seems, as images have no perception of depth, and the drones had no LiDAR or stereoscopic vision, so it was necessary to estimate it. Assuming the world is flat and with the attitude (roll, pitch and yaw) of the camera it is possible to estimate the position, and since there were no extra sources of information it was the only solution left. This comes with a set of shortcomings that will be explained further ahead. 

Both steps are combined and done to every frame of the video stream of the drone, that is casted to the server using RTMP. The streaming service implementation was heavily dependant on [MediaMTX](https://github.com/bluenviron/mediamtx) a linux service that accepts several video formats and is able to transform them dynamically. 

{{< figure src="/images/ImageProcessor.png" title="Image-processor" >}}

## Results, shortcomings and improvements