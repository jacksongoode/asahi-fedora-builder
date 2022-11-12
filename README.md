# asahi-fedora-builder

**Fedora 37 release:**  
Several packages have recently been added to the offical Fedora repo, which include  
```asahi-fwextract asahi-scripts dracut-asahi m1n1 update-m1n1```  
So in order upgrade from **Fedora 36** --> **Fedora 37** please do the following
```
dnf remove uboot-asahi
dnf install dracut-asahi
dnf upgrade
# just to be safe as the latest kernel has selinux enabled and it hasn't been fully tested out yet   
sed -i s/^SELINUX=.*$/SELINUX=permissive/ /etc/selinux/config
dnf install dnf-plugin-system-upgrade
dnf system-upgrade download --releasever=37
dnf system-upgrade reboot
```
An offical Fedora release seems imminent. When that happens, this project will no longer be needed.  
When an offical release happens, it'll be super-easy to transition over (I'll provide instructions of how to do that).   
At this point, all but a couple packages are offical Fedora packages anyway.   
But....until that happens I'll continue to work on this project.

**known issues:**  
**lvm2:** There's currently an issue with the lvm2-monitor service that delays the system boot by a couple minutes   
If you have the `lvm2` package installed, consider disabling the `lvm2-monitor` service   
```
systemctl disable lvm2-monitor.service
```

Builds a minimal Fedora image to run on Apple M1/M2 systems.

<img src="https://user-images.githubusercontent.com/12903289/200475188-41b1faf1-9b00-4376-ad8c-b9da19ef4d3f.png" width=65%>

**fedora package install:**  
```dnf install mkosi arch-install-scripts systemd-container zip qemu-user-static```  

note: ```qemu-user-static``` is not needed if building the image on an ```aarch64``` system.   

**To install a prebuilt image:**  
Make sure to update your macOS to version 12.3 or later, then just pull up a Terminal in macOS and paste in this command:
```
curl https://leifliddy.com/fedora.sh | sh
```

**Notes:** 
1. The root password is **fedora**
2. On the first boot the ```asahi-firstboot.service``` will run and will take around ```45 seconds``` to complete.  
   Do not shutdown or reboot the system before this service has completed.  
3. The Asahi Linux-related RPM's (and Source RPM's) used in this image can be found here:  
   https://leifliddy.com/asahi-linux/36/  
   All RPM's signed are signed by a GPG key.  
   The repo config can be found here:   
   https://leifliddy.com/asahi-linux/asahi-linux.repo  
4. The Fedora kernel config used is nearly identical to the kernel config used by the Asahi Linux project:  
   \*\*only a few Fedora-specific modifications were made  
   https://github.com/AsahiLinux/PKGBUILDs/blob/main/linux-asahi/config
5. ```systemd-networkd``` is the sole network service that's installed in this image.  
   Basic config files for the ```eth0``` and ```wlp1s0f0``` interfaces are included in the image   
   ie.  
   **/etc/systemd/network/eth0.network**
   ```
   [Match]
   Name=eth0

   [Network]
   DHCP=yes
   ```
   The ```eth0``` interface is what an external usb ethernet adapter "should" be assigned to.   
6. Use ```iwd``` to setup the wifi interface (see info below)   

**Setting up wifi**  
   
The ```iwd``` service is enabled by default.  
The wireless interface name "should" be ```wlp1s0f0```  
```
wlp1s0f0: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether b0:be:83:1f:5b:c9  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
``` 

A basic ```systemd-networkd``` config for this interface is included in the image and is located at:  
**/etc/systemd/network/wlp1s0f0.network**
```
[Match]
Name=wlp1s0f0

[Network]
DHCP=yes
```

To connect to a wireless network, use the following sytanx:  
```iwctl --passphrase passphrase station device connect SSID```  
an actual example:  
```iwctl --passphrase supersecretpassword station wlp1s0f0 connect blacknet-ac```  
..and that's it. ```systemd-networkd``` should then pull in an address via ```dhcp```   
and your system should re-connect to this network upon reboot.   

Connection info for ```iwd``` connections are stored under ```/var/lib/iwd```   

For more information on ```iwd``` and ```systemd-networkd``` functionality:   
https://wiki.archlinux.org/title/Iwd   
https://wiki.archlinux.org/title/systemd-networkd

If you install a desktop environment that uses `NetworkManager` (gnome, kde, cinnamon...etc), then you'll probably want to disable (and stop) these two services:  
```systemctl disable --now iwd systemd-networkd```


**Cinnamon Desktop: Double (Hi-DPI) scaling**  
The current state of the kernel has several limitations (this is an Alpha release after all).  
It's not possible to enable Double Interface Scaling though "normal" means.  
The Cinnamon display application shows nothing (it's completely blank), so it's not possible to set the scaling there.  
I tried just about everything, but I cannot seem to make the double scaling setting survive a reboot.  
I tried creating a dconf config under ```/etc/dconf/db/local.d``` with the following contents:   
```
[org/cinnamon/desktop/interface]
scaling-factor=uint32 2
```
\*\* I tried locking the key as well...to no avail

 So for now, the only method I've found to enable double scaling for a desktop session is to create an autostart config that runs:  
 ```gsettings set org.cinnamon.desktop.interface scaling-factor 2``` upon logging in to the desktop.  

You do that through the Startup Application app or by running the following command.  
```
cat <<EOF > ~/.config/autostart/double-scaling.desktop
[Desktop Entry]
Type=Application
Exec=gsettings set org.cinnamon.desktop.interface scaling-factor 2
X-GNOME-Autostart-enabled=true
NoDisplay=false
Hidden=false
Name[C]=double-scaling
Comment[C]=enable double (Hi-DPI) scaling
X-GNOME-Autostart-Delay=0
EOF
```

\*\* if anyone has found a better solution, please let me know.  

**Wiping Linux**  
Bring up a Terminal in macOS and run the following Asahi Linux script:  
```curl -L https://alx.sh/wipe-linux | sh```  
You should definitely understand what this script does before running it.  
You can find more info here:  
https://github.com/AsahiLinux/docs/wiki/Partitioning-cheatsheet  

**Boot from USB device**  
Once Linux is installed on an M1 system, you can then boot a compatible usb drive via ```u-boot```.  
This project will create a bootable USB drive for M1 systems.  
https://github.com/leifliddy/asahi-fedora-usb  
