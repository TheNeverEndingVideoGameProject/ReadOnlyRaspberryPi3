# ReadOnlyRaspberryPi3
Quick Guide on how to make your raspberry pi 3 read-only to save your sd card from corruption.  Intended for Raspberry Pi 3, also works for Pi 2 and zero.  Currently this guide assumes you are disabling the gui display and desktop environment and use command line only.

# Requirements
1. Raspberry Pi 3, 2 or Zero
2. Raspbian Jessie installed
3. keyboard and monitor or active ssh session
4. logged in with access to sudo

#1. Set raspi-config Automatic Login Option
> sudo raspi-config

* select *Boot Options* 
* select either *Console* or *Console Autologin* depending on your application
* select *Ok* then *Finish*
* then select *No* when asked *Would you like to reboot now?*

#2. Replace rsyslog with busybox-syslogd
> sudo apt-get install busybox-syslogd

#3. Remove anacron, dphys-swapfile, xserver-common, lightdm
> sudo apt-get remove --purge anacron logrotate dphys-swapfile xserver-common lightdm

#4. Disable builtin bootlogs and console-setup service
> sudo insserv -r bootlogs

> sudo insserv -r console-setup

#5. Edit /etc/fstab to set read-only mode for sd card partitions and add necessary tmpfs entries
> sudo nano /etc/fstab
* add ro to the options column - by changing the 2nd and 3rd line from
> /dev/mmcblk0p1  /boot           vfat    defaults         0       2
/dev/mmcblk0p2  /               ext4    defaults,noatime  0       1

  to

> /dev/mmcblk0p1  /boot           vfat    defaults,ro          0       2
/dev/mmcblk0p2  /               ext4    defaults,noatime,ro  0       1

  then add these lines to the end of the file
  
> tmpfs   /tmp            tmpfs   nosuid,nodev    0       0
tmpfs   /var/log        tmpfs   nosuid,nodev    0       0
tmpfs   /var/tmp        tmpfs   nosuid,nodev    0       0
tmpfs   /run            tmpfs   nosuid,nodev    0       0

##6. Add noswap and ro options to /boot/cmdline.txt
> sudo nano /boot/cmdline.txt

  at the end of line after rootwait fastboot add

> noswap ro
  
  so now the line looks like
  
> dwc_otg.lpm_enable=0 console=tty1 console=serial0,115200 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait fastboot noswap ro

#7. Reboot and check the if you are now read-only
> sudo reboot
  
  wait for your Pi to reboot and login again, then try writing a new file
  
> touch file

  if you are sucessful you should see
> touch: cannot touch ‘file’: Read-only file system

#8. If you need to make changes to your files you can remount to read-write mode
> sudo mount -o remount,rw /

  or if you need to change anything in /boot directory
> sudo mount -o remount,rw /boot


*This guide was created following parts of [this guide](https://hallard.me/raspberry-pi-read-only/).*
