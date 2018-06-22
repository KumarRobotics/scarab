# Scarab setup instructions for Ubuntu 16.04 and ROS Kinetic

Install Ubunutu 16.04. Make admin user `scarab_admin` and hostname `scarab40` (use scarab id for hostname)

Update, Upgrade
```
sudo apt-get update
sudo apt-get upgrade
```

Enable graphics support for kernels 4.13.0-41-generic and below
```
sudo update-pciids
sudo edit /etc/default/grub
```

Change line GRUB_CMDLINE_LINUX_DEFAULT="quiet splash" to say GRUB_CMDLINE_LINUX_DEFAULT="quiet splash i915.alpha_support=1"  in /etc/default/grub
```
sudo update-grub
sudo reboot
```

Install wifi drivers - if not supported by default (Note, this has to be installed again if the kernel is updated)
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
https://github.com/KumarRobotics/scarab/tree/devel_cleanup/config



Copy network interfaces and wpa_supplicant file
```
cd ~/Downloads
wget https://raw.githubusercontent.com/KumarRobotics/scarab/devel_cleanup/config/wpa_supplicant.conf
wget https://raw.githubusercontent.com/KumarRobotics/scarab/devel_cleanup/config/interfaces
```
Change the ip in interfaces file to reflect scarab id (example scarab44 will be 192.168.131.44, replace all ips)
```
sudo cp interfaces /etc/network/
sudo cp wpa_supplicant.conf /etc/wpa_supplicant/
```

Copy udev rules for setting USB persissions
```
wget https://raw.githubusercontent.com/KumarRobotics/scarab/devel_cleanup/config/99-scarab.rules
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

Install ROS Kinetic (ros-kinetic-desktop-full)
http://wiki.ros.org/kinetic/Installation

Other ROS packages
```
sudo apt-get install ros-kinetic-urg-node ros-kinetic-joy
```

Scarab dependencies to install
```
sudo apt-get install libarmadillo-dev libncurses5-dev libnl-3-dev libnl-route-3-dev libnl-genl-3-dev libnl-nf-3-dev libcgal-dev libcgal-qt5-dev
```
Install Primesense camera ROS packages
```
sudo apt-get install ros-kinetic-rgbd-launch ros-kinetic-openni2-camera ros-kinetic-openni2-launch
```
Install gmapping package
```
sudo apt-get install ros-kinetic-gmapping
```
Set up servo motor
```
sudo apt-get install ros-kinetic-dynamixel-motor
sudo usermod -a -G dialout $USER
sudo chmod 777 /dev/ttyUSB0
On scarab, make sure mode switch on servo controller is all the way towards the end the USB connects to.
```

Install other packages
```
sudo apt-get install openssh-server vim emacs tmux tmuxinator python-catkin-tools git
```

Setup catkin workspace
```
cd ~/
mkdir ws_scarab
cd ws_scarab
mkdir src
catkin init
catkin config -DCMAKE_BUILD_TYPE=Release
cd src
git clone https://github.com/KumarRobotics/scarab.git
cd scarab
git checkout -t origin/devel_cleanup
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
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
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
