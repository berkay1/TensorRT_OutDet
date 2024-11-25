# Project Review and Feedback
## General overview

I think you have forgotten something to push. Current version of this repository is far from to be ready.
If this is the case please let us know and push whatever is left on your local.

---

## Everything is under callback function.

Callback functions are special functions that are to be invoked after message received.
When `ros::spinOnce` is called, point cloud from the `ros::globalCallbackQueue` are processed with your callback function 
with the signature of 
```cpp 
void callback(const pointcloudtype& msg)
```
However you implemented everything under callback function.
According to the current way, when a message is received, model is being serialized from ONNX, cudaEngine, cudaRuntime and ExecutionContext are created and initialized from scratch, then inference is processed.
This is obviously not the ideal solution and I think you ve forgotten something to push and this is just a prototype version.

# Overall package structure

Overall structure seems appropriate but I don't understand the need for linking cmake to toplevel.
It is not required.

# CMakeLists.txt
Those lines with local paths are problematic.
``` set(CMAKE_PREFIX_PATH /var/local/home/aburai/outdet_cpp/libtorch;/opt/ros/noetic) ```
``` set(CMAKE_MODULE_PATH "/var/local/home/aburai/test_catkin/src/outdet/cmake" ${CMAKE_MODULE_PATH}) ```
I implemented a new CMakeLists that has the right paths based on container and compiles your code successfully.
Take it as reference or use it as you wish.

# Your requirements.

Based on your env.txt, versions you used are cuda 11.7, libtorch 1.13.0+cu117, tensorrt-10.5.0.
However AGX Orin is a special device and libraries are specifically delivered for those devices by Nvidia.
It doesn't support every version of CUDA and TensorRT.
Current CUDA version we use in our AGX's is CUDA 11.4 TensorRT 8.5.10 based on Drive OS 6.0.6.
Based on my observation many of your functions are supported by the TensorRT and CUDA versions we use except one line
```cpp
    conf_succes->setMemoryPoolLimit(MemoryPoolType::kTACTIC_SHARED_MEMORY, 1024*1024*1024);
```
# Summary
General overview of your code should look like
```
    your_model

    your_callback_function(point_cloud_type):
        message_received.
        your_model(point_cloud_type)
        do_your_further_processing related to received message.
        do_nothing_else in this function, it is only for callbacks.
    
    int main():
        build_your_model(ONNX or TensorRT)
        initialize_ros(topic_name, your_callback_function e.g.)
        ros::spin()
```