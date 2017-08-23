video at https://www.youtube.com/watch?v=UqzNlcw1U7s
refer at http://www.mattsheckells.com/research/

As part of our lab’s work on aerial manipulation, I’ve developed a ROS package for tracking the 3D pose of an object using a monocular camera.  The package takes as input a textured CAD model of the object to be tracked.  For a given image, point-based features are extracted and matched to a virtual rendering of the CAD model (using its last known pose).  A RANSAC+PnP algorithm uses these 2D-3D correspondences to find the new pose of the object.  The pose of the object is initialized by matching an input image to several images in a finite set of renderings of the CAD model.  The image with the largest number of geometrically consistent inliers is taken to be the pose of the object.  A constant velocity motion model or Extended Kalman Filter which fuses IMU measurements (if an IMU is available) can be used for more reliable tracking.

The package also has the capability to perform edge-based tracking (based on the work of Choi and Christensen in Real-time 3D model-based tracking using edge and keypoint features for robotic manipulation).

This package can also be used for image-based localization in a known map.



#MeshLocalize: Model-based object tracking in 3D for ROS

#1. Installation

##1.1 Dependencies

This has only been tested with Ubuntu 14.04 + ROS Indigo.

Install Eigen3.

Install OpenCV 2.4.11.  Source can be downloaded from https://github.com/Itseez/opencv/archive/2.4.11.zip

Install OIS (needed for OGRE). 

                 sudo apt-get install libois-dev
                 sudo apt-get install libois-1.3.0

or source can be downloaded from http://sourceforge.net/projects/wgois/files/

Install Nvidia Cg Toolkit

                 sudo apt-get install nvidia-cg-toolkit

Install Ogre 1.8.1.  You can download the source from http://sourceforge.net/projects/ogre/files/ogre/1.8/1.8.1/ogre_src_v1-8-1.tar.bz2/download

Install TooN from http://www.edwardrosten.com/cvd/toon.html

Install the object_renderer library.  It is a small wrapper around OGRE.  It is used to generate the virtual views of the object that the package uses to initialize the pose.

                 git clone https://github.com/msheckells/object_renderer
                
Build using cmake+make.

##1.2 Package Install

From your catkin source directory:

                 git clone https://github.com/jhu-asco/map_localize.git

Call catkin_make as usual

#2. Quick Start
Download a test rosbag which contains image and camera_info data from http://www.mattsheckells.com/wp-content/uploads/2015/cheezit_test.bag.zip and extract it.
                 
Launch the node:

                 roslaunch mesh_localize localize_cheezit_ogre.launch

Play the sequence:

                 rosbag play cheezit_test.bag

#3. Setup Guide
##3.1 Model Creation
Create a textured 3D model of the object you wish to track, preferably in DAE or OBJ format.  This model can be converted into an OGRE .mesh file using blender and the blender2ogre exporter which can be downloaded from https://blender2ogre.googlecode.com/files/blender2ogre-0.6.0.zip.  The path of the object must be added to ogre_cfg/resources.cfg so that the package can find the model.

##3.2 Descriptor/Pose Database Creation
Make sure the path of the object has been added to cfg/resources.cfg in the object_renderer parent folder.  Use the render_views tool in the object_renderer package to extract descriptors from images of the object rendered from known poses.  This is necessary to initialize the object tracker.  This can be done with the command

                  render_views <output_dir> <model_file> <radius> <num_samples>

This will render num_samples views of the object taken at random points lying on a sphere with the specified radius with the camera pointing at the origin.  The output will be saved to the path given as output_dir.  

##3.3 Calibration
Calibrate your camera using ROS camera calibration.

##3.4 Tracking Modes
There are three tracking modes: PNP, KLT, and EDGE.  All initialize using the descriptor/pose database.  

PNP mode extracts features from an input image and from a virtual image of the object rendered from its last known pose.  The features are matched to give a set of 2D-3D correspondences between the model and the input image.  A PnP problem is solved to give the object pose in the frame of the camera.

KLT mode first obtains a set of 2D-3D correspondences using PNP mode, then tracks the 2D keypoints using a KLT tracker.  A PnP problem is solved to give the object pose.  Correspondences are re-matched when the tracking diverges or when the re-projection error rises above a given threshold.

EDGE mode performs edge-based object tracking and is suitable for objects with little texture. A Canny edge detecttor is used on both a virtual view of the model and an input image.  Edge points are matched form the model to the input image by performing a 1D search in the gradient direction of the edge.  The distance between matched edges is minimized to estimate the objects pose.

#3. Topics
##3.1 Published
/mesh_localize/image [sensor_msgs::Image] Rectified version of the input image on which tracking is performed

/mesh_localize/camera_info [sensor_msgs::CameraInfo] Camera info corresonding to resized and cropped input image.

/mesh_localize/depth [sensor_msgs::Image] Depth map corresponding to /mesh_localize/image

/mesh_localize/estimated_pose [geometry_msgs::PoseStamped] Pose of the object in the frame of the camera

##3.2 Subscribed
/image [sensor_msgs::Image] Unrectified input image on which tracking will be performed

/camera_info [sensor_msgs::CameraInfo] Camera info of input image

#4. Parameters
TODO
