# Shinobi - Harware Acceration &

### Nvidia Cuda, Object and Lisence Plate Guide

NOTE: This is using nvida driver 430.50 make sure your video card is supported. If not then your going to have to figure it out beyond this guide

## Table of Contents
* [Dependencies_needed](#Dependencies_needed)
* [Install_and_setup_Shinobi](#Install_and_setup_Shinobi)


## Dependencies_needed
<br>

##### Nvidia Graphics Drivers and Cuda
<br>

Add graphics-drivers ppa
```
$ sudo add-apt-repository ppa:graphics-drivers/ppa
```
<br>

Add key
```
$ sudo apt-key adv --fetch-keys  http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
```

<br>

Add repos
```
$ sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
```
```
$ sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
```
<br>

Update and upgrade system
```
$ sudo apt update
```
```
$ sudo apt upgrade
```
<br>
Install CUDA 10.1 will install nvidia drives as well

```
$ sudo apt install cuda-10-1
```
<br>

Install libcudnn7 depends

```
$ sudo apt install libcudnn7 libcudnn7-dev
```
<br>

Edit CUDA profile
```
$ sudo nano ~/.profile
```
<br>
Add the following lines to your ~/.profile file for CUDA 10.1

```
# set PATH for cuda 10.1 installation
if [ -d "/usr/local/cuda-10.1/bin/" ]; then
    export PATH=/usr/local/cuda-10.1/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
fi
```
<br>

Reboot the computer
```
$ sudo reboot
```
<br>

Check install of the above is working
<br>

NVIDIA Cuda Compiler Check
```
$ nvcc --version
```
<br>

Check libcudnn version
```
$ /sbin/ldconfig -N -v $(sed 's/:/ /' <<< $LD_LIBRARY_PATH) 2>/dev/null | grep libcudnn
```

<br>
Check NVIDIA driver with

```
$ nvidia-smi
```
<br>
<br>


#### FFMPEG install and dependencies
<br>

```
$ sudo apt-get install libopus-dev libmp3lame-dev libfdk-aac-dev libvpx-dev libx264-dev yasm libass-dev libtheora-dev libvorbis-dev libavcodec-dev libavformat-dev libzvbi-dev zvbi mercurial cmake
```

<br>

NOTE: Nvidia Video Codec SDK - use sdk 9.0 as it supports video drivers

You will need to create an account and navigate NVIDIA's website to get these files

Download and copy Video_Codec_SDK_9.0.20/Samples/common.mk to home user dir
<br>

Copy file to /usr/local/include/ (use ftp or wget, your choice getting it on the server if your ssh into thebox)
```
$ sudo cp ./Video_Codec_SDK_9.0.20/Samples/common.mk /usr/local/include/
```
<br>

Install nv-codec-headers - use nv-codec-headers-n9.0.18.2

found here - https://github.com/FFmpeg/nv-codec-headers/releases

Copy /nv-codec-headers-n9.0.18.2/ to home dir for building (use ftp or wget, your choice getting it on the server if your ssh into thebox)

Enter into the Directory, make, and install
```
$ cd nv-codec-headers-n9.0.18.2
```
```
$ sudo make
```
```
$ sudo make install
```
<br>
Exit back to home dir

```
$ cd
```
<br>
Installing FFMPEG

```
$ git clone git://source.ffmpeg.org/ffmpeg.git
```
```
$ cd ffmpeg/
```
<br>

Configure to install with depends
```
$ ./configure \
--enable-shared \
--enable-cuda \
--enable-cuvid \
--enable-nvenc \
--enable-nonfree \
--enable-libnpp \
--enable-nvenc \
--enable-libx264 \
--enable-gpl \
--enable-avresample \
--extra-cflags=-I/usr/local/cuda-10.1/include \
--extra-ldflags=-L/usr/local/cuda-10.1/lib64
```
```
$ sudo make
```
```
$ sudo make install
```
<br>
Check NVENC support in FFmpeg: This should be broken we will fix it next

```
$ ffmpeg -codecs | grep nv
```
<br>

<br>
Install this dependency after ffmpg install to fix breaking issue (NOT BEFORE!)

```
$ sudo apt install libavdevice-dev

```
<br>

After install Check NVENC support in FFmpeg:
```
$ ffmpeg -codecs | grep nv
```
<br>

Back to home dir

```
cd
```
<br>


#### Install OpenCV with cuda support
<br>
Install dependencies

```
$ sudo apt-get install libxvidcore-dev libatlas-base-dev gfortran build-essential pkg-config unzip qtbase5-dev python-dev python3 python3-dev python-numpy python3-numpy libhdf5-dev libgtk-3-dev libdc1394-22 libdc1394-22-dev libjpeg-dev libtiff5-dev libtesseract-dev libswscale-dev libxine2-dev libgstreamer-plugins-base1.0-0 libgstreamer-plugins-base1.0-dev libpng16-16 libpng-dev libv4l-dev libv4l-0 libtbb-dev libopencore-amrnb-dev libopencore-amrwb-dev v4l-utils libleptonica-dev libpango1.0-dev libgif-dev gcc-6 g++-6 libgdal-dev liblapacke-dev
```
<br>
Link videodev.h to fix find issue

```
$ sudo ln -s /usr/include/libv4l1-videodev.h   /usr/include/linux/videodev.h
```
<br>

Download OpenCV
```
$ sudo git clone https://github.com/opencv/opencv.git
```
<br>
Move into directory

```
$ cd opencv
```
<br>
Select openCV Version to install

```
$ sudo git checkout 3.4.8
```
<br>
Move back to home dir

```
$ cd
```

Download OpenCV Modules

```
$ sudo git clone https://github.com/opencv/opencv_contrib.git
```
<br>
Move into directory

```
$ cd opencv_contrib
```
<br>
Select openCV Version to install

```
sudo git checkout 3.4.8
```
<br>
Move back to home dir

```
$ cd
```
<br>

Make a build dir if non exzists
```
$ cd opencv
```
```
$ sudo mkdir build
```
<br>

Enter OpenCV Build Directory

```
$ cd build
```

NOTE!!!! Using OpenCV with CUDA 10.+ requires -DWITH_NVCUVID=OFF removes dependency on the NVIDIA CODEC SDK. cmake support for the NVIDIA CODEC SDK has issue as it is now a transitive dependency. CUDA still works

```
$ sudo cmake -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_NVCUVID=OFF -D FORCE_VTK=ON -D WITH_XINE=ON -D WITH_CUDA=ON -D WITH_OPENGL=ON -D WITH_TBB=ON -D WITH_OPENCL=ON -D CMAKE_BUILD_TYPE=RELEASE -D CUDA_NVCC_FLAGS="-D_FORCE_INLINES --expt-relaxed-constexpr" -D WITH_GDAL=ON -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules/ -D ENABLE_FAST_MATH=1 -D CUDA_FAST_MATH=1 -D WITH_CUBLAS=1 -D CXXFLAGS="-std=c++11" -DCMAKE_CXX_COMPILER=g++-6 -DCMAKE_C_COMPILER=gcc-6 ..
```
<br>

OpenCV Build or use sudo make VERBOSE=1 for info if you have a build failure.

NOTE: This may take a long... long time. may look like it is stuck at 99% just wait. And by wait it could take hours

```
$ sudo make -j "$(nproc)"
```
<br>
Now we can install it

```
sudo make install
```
<br>
<br>

#### Lisence plate detection openalpr
<br>

remove leptonica
```
$ sudo apt-get remove libleptonica-dev
```
make sure other dependencies are removed too
```
$ sudo apt-get autoclean
```
```
$ sudo apt-get autoremove --purge
```
Install liblog4cplus-dev, liblog4cplus-1.1-9 and build-essential

```
$ sudo apt-get install liblog4cplus-1.1-9 liblog4cplus-dev build-essential
```
<br>

Download leptonica 1.74.1 from its GitHub Releases page

https://github.com/DanBloomberg/leptonica/releases/tag/1.74.1

Copy /leptonica-1.74.1/ to home dir for building (use ftp or wget, your choice getting it on the server if your ssh into thebox)

Enter into the Directory

```
$ cd leptonica-1.74.1
```
<br>
configure, make, install

```
$ autoreconf -ivf
```
```
$ ./configure
```
```
$ sudo make
```
```
$ sudo make install
```
<br>
Back to home directory

```
$ cd
```

Download tesseract-3.0.5 from its GitHub Releases page

https://github.com/tesseract-ocr/tesseract/releases/tag/3.05.00

Copy /tesseract-3.05.00/ to home dir for building (use ftp or wget, your choice getting it on the server if your ssh into thebox)

Enter into the Directory

```
$ cd tesseract-3.05.00
```
<br>

configure, make, install

```
$ ./autogen.sh
```
```
$ ./configure --enable-debug
```
```
$ LDFLAGS="-L/usr/local/lib" CFLAGS="-I/usr/local/include" make
```
```
$ sudo make install
```
```
$ sudo make install-langs
```
```
$ sudo ldconfig
```
<br>

Check if i works
```
$ tesseract --version
```
You should see 3.05.00


<br>
Back to home directory

```
$ cd
```

<br>

Install libcurl3 There is a confirmed bug in Ubuntu 18.04 saying that can’t have both libcurl3 and cmake 3.10 on the same machine.

Use unofficial PPA built by Eugene Brazgin that helps with solving this problem.

Add this unsupported PPA:
```
$ sudo add-apt-repository ppa:xapienz/curl34
```
```
$ sudo apt-get update
```

Update libcurl4:
```
$ sudo apt-get install libcurl4 libcurl4-openssl-dev
```

Clone the latest OpenALPR code from GitHub:
```
$ git clone https://github.com/openalpr/openalpr.git
```
Setup the build directory:
```
$ cd openalpr/src
```
```
$ mkdir build
```
```
$ cd build
```
Setup the compile environment:
```
$ cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr -DCMAKE_INSTALL_SYSCONFDIR:PATH=/etc –DCOMPILE_GPU=1 -D WITH_GPU_DETECTOR=ON ..
```
Compile the library:
```
$ make
```
Install the binaries/libraries to your local system (prefix is /usr)
```
$ sudo make install
```
Test the library:
```
$ wget http://plates.openalpr.com/h786poj.jpg -O lp.jpg
```
```
$ alpr lp.jpg
```
If everything is fine, you should get a list of plates detected


Back to home directory

```
$ cd
```



## Install_and_setup_Shinobi

Enter Commands
```
$ sudo su
```
```
$ bash <(curl -s https://gitlab.com/Shinobi-Systems/Shinobi-Installer/raw/master/shinobi-install.sh)
```

Make sure, when asked, to install a database of your choice. you need it for object detector

<br>
It should now be running - exit su
<br>

```
$ exit
```

<br>
using the msg login to the browser as directed
<br>

#### Change default video Storage ( OPTIONAL )
Don't do this if you dont understand HardDrives in linux

If you would like to change where the videos are stored to another Hard Drive or Medium do this:

Click the Configuration Tab

edit the JSON and add Add "videosDir": "/mnt/YourMountedHardDriveHere/ShinobiVideos" making sure to change the mount path of YourMountedHardDriveHere to your drive mount location and make sure to add the directory /ShinobiVideos to it with mkdir ShinobiVideos



Example:
```
{
   "port": 8080,
   "passwordType": "sha256",
   "detectorMergePamRegionTriggers": true,
   "videosDir": "/mnt/YourMountedHardDriveHere/ShinobiVideos",
   "addStorage": [
      {
         "name": "second",
         "path": "__DIR__/videos2"
      }
   ],
   "db": {
      "host": "127.0.0.1",
      "user": "majesticflame",
      "password": "",
      "database": "ccio",
      "port": 3306
   },
   "mail": {
      "service": "gmail",
      "auth": {
         "user": "your_email@gmail.com",
         "pass": "your_password_or_app_specific_password"
      }
   },
   "cron": {
      "key": "change_this_to_something_very_random__just_anything_other_than_this"
   },
   "pluginKeys": {}
}
```

Save the JSON - you will be asked to restart camera. Do not do this in web. Do this in Command line interface

Stop Shinobi
```
$ sudo su
```

```
$ pm2 stop camera
```

Start Shinobi and Check permissions for the drive to see that the "os user" can read and write.
```
$ pm2 flush && pm2 start camera && pm2 logs
```
If you get a msg like - { Error: ENOENT: no such file or directory, mkdir '/mnt/YourMountedHardDriveHere/ShinobiVideos/' then you need to fix your mount or user permissions for the mount

You will now have to edit the conf.json in the shinobi home folder to fix it before bringing the camera back up
see - https://www.reddit.com/r/ShinobiCCTV/comments/bwf6oh/use_specific_drive_for_writing_footage/


<b> no error ? </b> then all should be running well

hold down
```
$ ctrl+c
```
to exit

exit su
```
$ exit
```

#### Create a user

Click +Add Accounts

Fill in the fields as you want and save

#### object detection - Yolo and openalpr
<br>

<b>Yolo - install and run the plugin</b>

```
$ cd /home/Shinobi/plugins/yolo
```

```
$ sudo su
```
```
$ sudo sh INSTALL.sh
```
```
$ pm2 start shinobi-yolo.js
```

add this to automatically boot at Start
```
$ pm2 startup
```
```
$ pm2 save
```
Now we need to edit the JSON for the plugin

```
$ nano /home/Shinobi/plugins/yolo/conf.json
```
Change the contents to this :
```
{
  "plug":"Yolo",
  "hostPort":8082,
  "key":"Yolo123123",
  "mode":"host",
  "type":"detector"
}
```
press ctrl+x and press y to Save

<br>

<b>openalpr - install and run the plugin</b>

Copy the config file to be edited
```
$ cd /home/Shinobi
```
```
$ cp plugins/openalpr/conf.sample.json plugins/openalpr/conf.json

```

Edit it the new file

```
$ nano plugins/openalpr/conf.json
```
Change the contents to this :
```
{
  "plug":"OpenALPR",
  "hostPort":8083,
  "key":"SomeOpenALPRkeySoPeopleDontMessWithYourShinobi",
  "mode":"host",
  "type":"detector"
}
```
press ctrl+x and press y to Save

Start the plugin:


```
$ pm2 start /home/Shinobi/plugins/openalpr/shinobi-openalpr.js
```
add this to automatically boot at Start
```
$ pm2 startup
```
```
$ pm2 save
```


<br>

<b>Add the plugins to the main configuration JSON</b>


edit the main shinobi JSON
```
$ nano /home/Shinobi/conf.json
```
at the bottom you should see "pluginKeys": {}

Change just the plugins section to look like this at the end of the file
```

   "plugins": [
      {
         "id": "Yolo",
         "https": false,
         "host": "localhost",
         "port": 8082,
         "key": "Yolo123123",
         "mode": "host",
         "type": "detector"
      },
      {
         "id": "OpenALPR",
         "https": false,
         "host": "localhost",
         "port": 8083,
         "key": "SomeOpenALPRkeySoPeopleDontMessWithYourShinobi",
         "mode": "host",
         "type": "detector"
      }
   ]
}
```
press ctrl+x and press y to Save

<br>

exit su
```
$ exit
```


<b>Reboot System and make sure it all works</b>

```
$ sudo Reboot
```

See documentation for setting up these systems in the web control

https://shinobi.video/articles/2019-10-26-how-to-setup-yolo-object-detection-and-use-different-weights
