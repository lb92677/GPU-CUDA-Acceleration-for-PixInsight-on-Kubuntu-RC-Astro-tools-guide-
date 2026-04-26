# GPU CUDA Acceleration for PixInsight on Kubuntu RC Astro tools guide
This guide enables GPU (CUDA) acceleration with Linux (Kubuntu and other Ubuntu versions).  Kubuntu is a Linux version which has a Windows like environment and is what PixInsight recommends. It is updated to support Kubuntu 26.04 LTS.  It also supports Kubuntu 24.04 LTS.



**Install Kubuntu**

get a USB stick with 64 gb or greater

First install Kubuntu .

Download rufus

https://rufus.ie/downloads/

download the Kubuntu 26.04 LTS version (the current stable version with long term support). If you want to install the former 24.04 LTS version you can download that instead. For clarity I will only refer to the 26.04 LTS version. 
 

go to kubuntu.org


select the 26.04 LTS version

insert the USB stick in your computer.

in rufus select your USB stick as your device and the kubuntu iso as your iso. Change the partition scheme from MBR to GPT. Rufus will say this is a hybrid/iso version and DD imaging will be enforced. that is OK.

Rufus will copy the kubuntu iso file to the usb stick

reboot and press the key which will get you into your bios

go into your computer bios and set your computer to boot from usb. put it first in the boot order

then reboot .

insert the Usb stick you created from Rufus

Kubuntu will boot up.

You will see two choices.. try Kubuntu or install Kubuntu

I selected to install Kbuntu with normal installation. Do not select install third-party software for graphics. You will install this later.

I selected Kubuntu to install alongside Windows 11. I then dragged the divider to allocate the amount of drive space I wanted for Kubuntu. This will install a boot option menu (grub menu) when you boot your computer. When you reboot choose the Kbuntu option at the top.
You can still use Windows by selecting the windows option
If you boot into windows instead of the boot option menu the boot order may have changed in your bios and you will boot into Windows on startup. To change that, go into your bios and change the boot order for Kubuntu first and windows second.




**Once Kubuntu is installed and you have booted into it, open the Kubuntu Konsole (terminal)**


**Update and upgrade your machine**

sudo apt update

sudo apt upgrade


**Install the GCC compiler**

sudo apt install build-essential



**Install the Nvidia driver**

 

The following command will pick the nvidia driver that is best suited for your system. In my case it selected the Nvidia 595 open driver

sudo ubuntu-drivers install


sudo reboot



check to see if it installed

nvidia-smi

In my case it said driver version 595.58.03 and CUDA 13.2
CUDA 13.2 is not actually installed. Nvidia-smi refers to the highest version CUDA driver that the Nvidia driver 595 can support.



**Install CUDA 12.8 toolkit**

 

cd ~/Downloads

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-ubuntu2404.pin

sudo mv cuda-ubuntu2404.pin /etc/apt/preferences.d/cuda-repository-pin-600

wget https://developer.download.nvidia.com/compute/cuda/12.8.1/local_installers/cuda-repo-ubuntu2404-12-8-local_12.8.1-570.124.06-1_amd64.deb


sudo dpkg -i cuda-repo-ubuntu2404-12-8-local_12.8.1-570.124.06-1_amd64.deb

sudo cp /var/cuda-repo-ubuntu2404-12-8-local/cuda-*-keyring.gpg /usr/share/keyrings/

sudo apt-get update

sudo apt-get -y install cuda-toolkit-12-8


the cuda files install into /usr/local/cuda-12.8 (bin files install into /usr/local/cuda-12.8/bin and library files install into /usr/local/cuda-12.8/lib64) Path commands which follow in this guide do not need to specify what version of cuda you have installed, Kubuntu handles that for you.

next edit the environment .bashrc file

sudo nano ~/.bashrc

then go to the bottom of the file and copy the following and paste it to the file


export PATH="/usr/local/cuda/bin:$PATH"

export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"


press ctrl O and press enter ( to write the file) and ctrl X and press enter to exit the editor

refresh environment

source ~/.bashrc


then add these environment commands


echo "/usr/local/cuda/lib64" | sudo tee -a /etc/ld.so.conf

sudo ldconfig


 


**Install cuDNN 8.9.7 libraries**


(note: even though these libraries were released for Ubuntu 22.04 they are compatible with Ubuntu 26.04)


in your browser go to

https://developer.nvidia.com/cudnn


join the website

In order to download cuDNN, ensure you need to be registered for the NVIDIA Developer Program.


You will need to install google authenticator on your phone for this (iphone users can get it from the app store)

https://apps.apple.com/us/app/google-authenticator/id388497605

then join the NVIDIA developer program

the Nvidia developer site will have you scan a QR code to add it to the authenticator

https://developer.nvidia.com/login

and use the authenticator app to login for the security challenge ( you will enter a 6 digit number)


agree to terms

click on the following URL which will bring you to the cuDNN archives

https://developer.nvidia.com/rdp/cudnn-archive

you will see a list, select "Download cuDNN v8.97 (December 5th, 2023) for CUDA 12.x"

it will drop down to the different versions

select Local Installer for Ubuntu22.04 x86_64 (Deb)

you will download this file:

cudnn-local-repo-ubuntu2204-8.9.7.29_1.0-1_amd64.deb

right click on properties and rename it to cudnn.deb

then move the file into an install directory and unpack it

mkdir cudnn_install

mv cudnn.deb cudnn_install

cd cudnn_install

ar -x cudnn.deb

this will unzip new files, one of which is data.tar.xz

unzip that file

tar -xvf data.tar.xz

new folders will extract . Change directory to the var/cudnn-local-repo-ubuntu2204-8.9.7.29/ folder and install the libcudnn8.9.7.29 files

cd var/cudnn-local-repo-ubuntu2204-8.9.7.29/

sudo dpkg -i libcudnn8_8.9.7.29-1+cuda12.2_amd64.deb

sudo dpkg -i libcudnn8-dev_8.9.7.29-1+cuda12.2_amd64.deb

sudo dpkg -i libcudnn8-samples_8.9.7.29-1+cuda12.2_amd64.deb

the library files install to the /usr/lib/x86_64-linux-gnu/ folder



**Install Tensorflow 2.15.0 libraries**

you need version 2.15.0 which works with CUDA 12.8. I got this info from this table: https://www.tensorflow.org/install/source#gpu
(although the table offically says CUDA 12.2 works with tensorflow library 2.15.0, 12.8 works for me and in another guide for WSL, 12.5 works). It also mentions clang comipler.. this is not needed. We are installing prebuilt libraries.. the gcc compiler works fine.

Now that I have explained this click the following URL which will download the tensorflow libraries version 2.15.0

https://storage.googleapis.com/tensorflow/libtensorflow/libtensorflow-gpu-linux-x86_64-2.15.0.tar.gz

hit enter and it will download

now install the Tensorflow libraries and create a dynamic link (pathway) in order for them to be used by PixInsight

cd ~/Downloads

sudo tar -C /usr/local -xzf libtensorflow-gpu-linux-x86_64-2.15.0.tar.gz

sudo ldconfig /usr/local/lib

the tensorflow libraries install into /usr/local/lib folder



**Install PixInsight for Linux**

go to https://pixinsight.com/downloads/index.html

select software distribution

select the linux version

Download it

PI-linux-x64-1.9.3-20250402-c.tar.xz is the latest version as of this post.

to install:

tar -xf PI-linux-x64-1.9.3-20250402-c.tar.xz ( or whatever the latest version is)

sudo ./installer

**Configure PixInsight**

PixInsight installs its own Tensorflow libraries into the the /opt/PixInsight/bin/lib folder.

Remove these libraries from the /opt/Pixinsight/bin/lib folder. Pixinsight will then use Tensorflow 2.15.0 libraries you just installed in the /usr/local/lib folder. These libraries work with CUDA 12.8.

sudo rm /opt/PixInsight/bin/lib/libtensorflow*

now edit the .bashrc file again and add another environment variable

sudo nano ~/.bashrc

scroll to bottom and paste

export TF_FORCE_GPU_ALLOW_GROWTH="true"

then ctrl O enter to write and ctrl X to exit.

you are done !!

You can now go into PixInsight and install the RC Astro tools or Starnet++ to test Cuda acceleration


**Installation of future versions of PixInsight**

Upon reinstalling PixInsight installs its own Tensorflow libraries. Remove them from the /opt/PixInsight/bin/lib folder . This ensures the 2.15.0 Tensorflow libraries located in the /usr/local/lib folder continue to be used.

do not restart PixInsight until you enter the following:

sudo rm /opt/PixInsight/bin/lib/libtensorflow*

if you mistakenly restart PixInsight, just reboot and it should work. you can reboot by entering the following:

sudo reboot

after doing this you can restart PixInsight and keep GPU acceleration

 

 

**Solution if an instability occurs and nvidia-smi does not see your nvidia driver**

Sometimes an instability might occur. Cuda acceleration will not work and running nvidia-smi will not show your driver. In that case you need to remove and reinstall the Nvidia driver. Do the following:

sudo apt-get purge -y 'nvidia*'

sudo apt autoremove -y

sudo apt autoclean

sudo apt update

sudo ubuntu-drivers install

sudo reboot

nvidia-smi

It should say a driver version  and CUDA version. Cuda acceleration should be restored.




**Special situation: Installing Kubuntu (KDE plasma desktop) if you already installed Ubuntu**

(I like the standard version)

sudo apt install kde-standard

During the long installation a screen will come up asking you to choose gdm3 or sddm. Select sddm.

after Kubuntu installed, I found an onscreen keyboard came up on boot. I had to select the keyboard up symbol to get past it to the login screen.

I disabled it by doing the following:

sudo nano /etc/sddm.conf

now copy and paste the following

InputMethod=

ctrl O and enter to write the file and ctrl X to exit

If all goes well StarXterminator will speed up. On my computer StarXterminator took 40 seconds without GPU acceleration and 13.9 seconds with GPU acceleration. PixInsight is also faster under Kubuntu vs Windows 11



**Special situation if you use an NTFS drive or partition for your data**

Since many people install Kubuntu on a computer which first has windows installed, you may be using an NTFS drive or partition for your data since both Windows and Kubuntu can see it. If that is the case you may be using a slow ntfs driver called ntfs-3g. Kubuntu defaults to this. The new fast driver is ntfs3.

Eventually newer versions of Kubuntu will use this. Here is how to default to this faster driver.


open terminal

check to see how your NTFS drive is mounted

mount | grep ntfs.

if you see "type fuseblk", you are using ntfs-3g, if you see "type ntfs3" you are fine

find your drives UUID and mount point

sudo lsblk -f

you will see something like
NAME FSTYPE LABEL UUID MOUNTPOINT
nvme0n1
├─nvme0n1p1 vfat 7E2A-0E1C /boot/efi
├─nvme0n1p2 ext4 12a3b456-7890-4def-1234-56789abcdef0 /
└─nvme0n1p3 ntfs Data 1234ABCD5678EFGH /mnt/ext


in this example, the last line shows the ntfs drive info, the UUID is 1234ABCD5678EFGH and the mount point is /mnt/ext

now edit the fstab file

sudo nano /etc/fstab
scroll down to the line that has the ntfs partition

it might look like
UUID=1234ABCD5678EFGH /mnt/ext ntfs-3g defaults,uid=1000,gid=1000 0 0

or
/dev/sdb1 /media/username/Data ntfs-3g defaults 0 0

replace that line with
UUID=1234ABCD5678EFGH /mnt/ext ntfs3 defaults,noatime,uid=1000,gid=1000 0 0
(replace the 1234ABCD5678EFGH with your UUID you got above) and ext with what your mountpoint is called. You always have the /mnt so it will be /mnt/yourmountpointname.


ctrl 0, ctrl X to write and exit

now test
unmount and remount the drive
sudo umount /mnt/yourmountpointname ( in my case it is sudo umount /mnt/ext )

sudo mount -a

then check
mount | grep ntfs

you should see

/dev/nvme0n1p3 on /media/username/Data type ntfs3 (rw,nosuid,nodev,relatime,uid=1000,gid=1000,windows_names)

this will have huge effect. This cut WPPP processing to about 1/2 the time using an NTFS drive

