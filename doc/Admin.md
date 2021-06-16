# Scarab setup instructions for Ubuntu 20.04 and ROS Noetic

Install Ubunutu 20.04. Make admin user `scarab_admin` and hostname `scarab#`, replacing # with the scarab's ID.

Update, Upgrade
```
sudo apt update
sudo apt upgrade
```

If WiFi does not work automatically, you must install the diver manually (note that this has to be repeated every time the kernel is updated).
```
cd ~/Downloads
wget https://github.com/lwfinger/rtlwifi_new/archive/extended.zip
unzip extended.zip
cd rtlwifi_new-extended
make
sudo make install
sudo modprobe rtl8822be
```

Scarab system config files are located in
https://github.com/KumarRobotics/scarab/tree/devel_noetic/config


Copy network interfaces and wpa_supplicant file
```
cd ~/Downloads
wget https://raw.githubusercontent.com/KumarRobotics/scarab/devel_noetic/config/wpa_supplicant.conf
wget https://raw.githubusercontent.com/KumarRobotics/scarab/devel_noetic/config/interfaces
```
Change the ip in interfaces file to reflect scarab id (example scarab44 will be 192.168.131.44, replace all ips)
```
sudo cp interfaces /etc/network/
sudo cp wpa_supplicant.conf /etc/wpa_supplicant/
```

Copy udev rules for setting USB persissions
```
wget https://raw.githubusercontent.com/KumarRobotics/scarab/devel_kinetic/config/99-scarab.rules
sudo cp 99-scarab.rules /etc/udev/rules.d
sudo udevadm control --reload-rules && udevadm trigger
```
Disable automatic updates (because they mess up the wifi)
```
Open the Software Updater settings
Go to the Updates tab
Change the drop down menus to:
Automatically check for updates: Weekly
When there are security updates: Display immediately
When there are other updates: Display weekly
```

Install ROS Noetic (ros-noetic-desktop-full)
http://wiki.ros.org/noetic/Installation

Other ROS packages
```
sudo apt install ros-noetic-urg-node ros-noetic-joy
```

Scarab dependencies to install
```
sudo apt install libarmadillo-dev libncurses5-dev libnl-3-dev libnl-route-3-dev libnl-genl-3-dev libnl-nf-3-dev libcgal-dev libcgal-qt5-dev
```
Install Primesense camera ROS packages
```
sudo apt install ros-noetic-rgbd-launch ros-noetic-openni2-camera ros-noetic-openni2-launch
```
Install gmapping package
```
sudo apt install ros-noetic-gmapping
```

Install other packages
```
sudo apt install openssh-server vim emacs tmux tmuxinator python3-osrf-pycommon python3-catkin-tools git
```

Set up servo motor (TODO: change to dynamixel_sdk)
```
sudo apt install ros-noetic-dynamixel-motor
sudo usermod -a -G dialout $USER
sudo chmod 777 /dev/ttyUSB0
```
On scarab, make sure mode switch on servo controller is all the way towards the end the USB connects to.

Setup catkin workspace
```
cd ~
mkdir -p ws_scarab/src
cd ~/ws_scarab/src
catkin init
catkin config -DCMAKE_BUILD_TYPE=Release
cd src
git clone https://github.com/KumarRobotics/scarab.git
cd scarab
git checkout -t origin/devel_noetic
catkin build
```

Change the AGENT to reflect the current scarab id
```
echo "export AGENT=scarab40" >> ~/.bashrc
echo "export ROS_HOSTNAME=$AGENT" >> ~/.bashrc
source ~/.bashrc
source ~/ws_scarab/devel/setup.bash

Turn on the H-BRDG and ROBO CLAW PWR switches

roslaunch scarab scarab.launch
```

Install scarab drivers to system. catkin-tools provides profiles for devel/install space
http://catkin-tools.readthedocs.io/en/latest/verbs/catkin_profile.html
```
cd ~/ws_scarab
catkin profile add install
catkin profile set install
catkin config --install -DCMAKE_BUILD_TYPE=Release 
catkin clean
catkin build
```
This will create 'install' folder in the workspace. Copy it to system location.

```
sudo cp -r install /opt/ros/scarab/
```

Making account for other users
```
sudo adduser scarab
sudo usermod -a -G dialout scarab
```
set password to 'mrsl'

ssh to 'scarab' user
```
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
echo "source /opt/ros/scarab/setup.bash" >> ~/.bashrc
echo "export AGENT=scarab40" >> ~/.bashrc
echo "export ROS_IP=$(ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p')" >> ~/.bashrc
```

Set ROS_MASTER_URI in .bashrc: The following bash commands sets the ROS_MASTER_URI to the IP of the ssh_client. Users don't have to explicitely set this.

```
echo "export MASTER_IP="${SSH_CLIENT%% *}"
if [[ !  -z  $MASTER_IP  ]];then
   export ROS_MASTER_URI=http://${MASTER_IP}:11311
fi " >> ~/.bashrc
```

Test the launch files
