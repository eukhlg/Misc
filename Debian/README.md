Gnome customization
===========

1.  Install Extension Manager
2.  Install **Blur my Shell, Dash to Dock, User Themes** (if not installed)
3.  Install [**Flatpak**](https://flatpak.org/setup/Debian)  
    
4.  Restart
5.  Download and install **GTK3 themes** from [https://github.com/lassekongo83/adw-gtk3](https://github.com/lassekongo83/adw-gtk3)  
    
6.  Install **GDM Settings** app from FlatHub and copy user's settings to GDM
7.  Adjust Touchpad speed [https://ubuntuhandbook.org/index.php/2023/05/adjust-touchpad-scrolling-ubuntu/](https://ubuntuhandbook.org/index.php/2023/05/adjust-touchpad-scrolling-ubuntu/)

### Fractional Scaling and HiDPI

1.  Enable QT apps auto scaling via setting environment variables

```
sudo nano /etc/security/pam_env.conf
QT_AUTO_SCREEN_SCALE_FACTOR=1
QT_ENABLE_HIGHDPI_SCALING=1
```

    
2.  Enable Wayland HiDPI support for all users  
      
```    
# This is if GDM settings app is present are 
sudo touch /etc/dconf/db/gdm.d/00-hidpi
sudo vi 00-hidpi
[org/gnome/mutter]
experimental-features=['scale-monitor-framebuffer']
# In Fedora this trick has helped
sudo -u gdm dbus-launch gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```
    
3.  Enable HiDPI for X11  

```
gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', <2>}]"
vi ~/.config/autostart/xrandr-settings.desktop

[Desktop Entry]
Type=Application
Version=1.0
Name=custom xrandr settings

# Replace with your own xrandr command:
Exec=xrandr --output eDP --scale 1.25x1.25
```
        
    
4.  Flatpak  
      
```    
STEAM_FORCE_DESKTOPUI_SCALING=1.5
GDK_SCALE=1.5
ELM_SCALE=1.5
QT_SCALE_FACTOR=2
```

Drivers installation
--------------------

### NVIDIA Driver (debian repository)

1. Add "contrib", "non-free" and "non-free-firmware" components to /etc/apt/sources.list, for example:

```
# Debian Bookworm
deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
```

2. Update the list of available packages, then we can install the nvidia-driver package, plus the necessary firmware:

```
apt update
apt install nvidia-driver firmware-misc-nonfree
```

DKMS will build the nvidia module for your system, via the nvidia-kernel-dkms package.

> [!TIP]
> Note about Secureboot : if you have SecureBoot enabled, you need to sign the resulting modules. Detailed instructions are available here.

### NVIDIA Driver (Offical Download)  

1. Download driver from here [https://download.nvidia.com/XFree86/Linux-x86_64/](https://download.nvidia.com/XFree86/Linux-x86_64/)
The latest stable version can be found here [https://www.nvidia.com/Download/driverResults.aspx/216728/en-us/](https://www.nvidia.com/Download/driverResults.aspx/216728/en-us/)

```
apt autoremove $(dpkg -l nvidia-drivers* | grep ii |awk '{print $2}')
apt install linux-headers-$(uname -r) gcc make acpid dkms libglvnd-core-dev libglvnd0 libglvnd-dev dracut
touch /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
GRUB_CMDLINE_LINUX_DEFAULT="quiet rd.driver.blacklist=nouveau"
update-grub
dracut -q /boot/initrd.img-$(uname -r) $(uname -r) --force
systemctl set-default multi-user.target

# Rebbot and run installer in console
# Do not create X11 config this causes issues on GDM boot
systemctl set-default graphical.target

# After reboot
apt install xwayland libxcb1 libnvidia-egl-wayland1
```

#### Prime-run

1.  Make sure the NVIDIA drivers are installed and reboot if you just installed them.
2.  Open a terminal and type: nano prime-run
3.  Copy and paste this script into nano, then save and quit nano:  

```
#!/bin/bash
__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia "$@"
```
      
    
4.  In terminal type:
   
```
sudo mv prime-run /bin
cd /bin
sudo chmod 755 prime-run
```
    
5.  Test it by typing `prime-run glxinfo | grep "OpenGL renderer"`

Issues
------

### AMDGPU issue on DMESG

```
[    2.123808] amdgpu 0000:66:00.0: amdgpu: Fetched VBIOS from VFCT
[    2.123809] amdgpu: ATOM BIOS: 113-PHXGENERIC-001
[    2.123814] [drm] VCN(0) encode/decode are enabled in VM mode
[    2.123815] amdgpu 0000:66:00.0: [drm:jpeg_v4_0_early_init [amdgpu]] JPEG decode is enabled in VM mode
[    2.123967] amdgpu 0000:66:00.0: firmware: failed to load amdgpu/gc_11_0_1_mes_2.bin (-2)
[    2.124013] firmware_class: See https://wiki.debian.org/Firmware for information about missing firmware
[    2.124045] amdgpu 0000:66:00.0: firmware: failed to load amdgpu/gc_11_0_1_mes_2.bin (-2)
[    2.124073] amdgpu 0000:66:00.0: Direct firmware load for amdgpu/gc_11_0_1_mes_2.bin failed with error -2
[    2.124074] [drm] try to fall back to amdgpu/gc_11_0_1_mes.bin
[    2.124115] amdgpu 0000:66:00.0: firmware: direct-loading firmware amdgpu/gc_11_0_1_mes.bin
[    2.124148] amdgpu 0000:66:00.0: firmware: direct-loading firmware amdgpu/gc_11_0_1_mes1.bin
```

Solution is here [https://unix.stackexchange.com/questions/765130/ive-got-a-new-lenovo-yoga-pro-7-14aph8-and-after-grub-bootloader-screen-im-see](https://unix.stackexchange.com/questions/765130/ive-got-a-new-lenovo-yoga-pro-7-14aph8-and-after-grub-bootloader-screen-im-see)

```
cd /tmp
git clone https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/
cd linux-firmware
sudo cp -a --no-preserve=ownership amdgpu /lib/firmware
sudo update-initramfs -u #or sudo update-initramfs -k all -u -v
```

### Wayland not available in GDM after installing Nvidia drivers in Debian 12

#### Option 1

```
nano /usr/lib/udev/rules.d/61-gdm.rules

#just commented the last two lines here:
LABEL="gdm_prefer_xorg"
#RUN+="/usr/lib/gdm-runtime-config set daemon PreferredDisplayServer xorg"
GOTO="gdm_end"
LABEL="gdm_disable_wayland"
#RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"
GOTO="gdm_end"
```

#### Option 2

On GNOME desktops, although a proper version of the NVIDIA driver is used, the greeter (GDM3) may still not offer the option to start a Wayland session, either because kernel modesetting is not enabled, or because the hibernate/suspend/resume helper scripts have not been installed on the system.

To enable kernel mode-setting with the NVIDIA driver:

```
touch /etc/default/grub.d/nvidia-modeset.cfg
echo 'GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX nvidia-drm.modeset=1"' > /etc/default/grub.d/nvidia-modeset.cfg
update-grub
```

To install the hibernate/suspend/resume helper scripts:

1.  Navigate to [https://download.nvidia.com/XFree86/](https://download.nvidia.com/XFree86/) and click on your appropriate architecture (for example - Linux-x86_64 for 64-bit).
    
2.  Then choose your NVIDIA-driver version. Can be found using the nvidia-smi command.
    
3.  Download the .run file and execute it using bash, with the --extract-only flag. This will extract the files we require into a directory with the same name as the .run file.
    
```bash
NVIDIA-Linux-x86_64-525.147.05.run --extract-only
```
    
4.  Run the following commands to install the extracted scripts:

```
TMPL_PATH=/path/to/the/extracted/directory/systemd
sudo install --mode 644 "${TMPL_PATH}/system/nvidia-suspend.service" /etc/systemd/system
sudo install --mode 644 "${TMPL_PATH}/system/nvidia-hibernate.service" /etc/systemd/system
sudo install --mode 644 "${TMPL_PATH}/system/nvidia-resume.service" /etc/systemd/system
sudo install "${TMPL_PATH}/system-sleep/nvidia" /lib/systemd/system-sleep
sudo install "${TMPL_PATH}/nvidia-sleep.sh" /usr/bin
sudo systemctl enable nvidia-suspend.service
sudo systemctl enable nvidia-hibernate.service
sudo systemctl enable nvidia-resume.service
```

> [!IMPORTANT]
> In addition, you will need to verify whether the PreserveVideoMemoryAllocations NVIDIA module parameter is turned on. Without the parameter being > enabled, the udev rules in /usr/lib/udev/rules.d/61-gdm.rules will force a fallback to X11. To check the value of PreserveVideoMemoryAllocations:
> ```
> cat /proc/driver/nvidia/params | grep PreserveVideoMemoryAllocations
> PreserveVideoMemoryAllocations: 1
> ```
>
> If this parameter is set to zero, you should be able to override it by adding a configuration into modprobe.d (assuming the file doesn't already > exist):
> 
> ```
> echo 'options nvidia NVreg_PreserveVideoMemoryAllocations=1' > /etc/modprobe.d/nvidia-power-management.conf
> ```
> 
> If after reboot it was still not working, force-enable Wayland, by overriding some udev rules  
> 
> Looks like this is almost like option 1, not sure this is good solution
> 
> ```
> ln -s /dev/null /etc/udev/rules.d/61-gdm.rules
> ```

Software
--------

1.  Remove Debian's Mozila ESR
2.  Remove Libre Office  
      
```
sudo apt remove libreoffice-common libreoffice-core libreoffice-gnome libreoffice-gtk3 libreoffice-help-common libreoffice-help-en-us libreoffice-style-colibre libreoffice-style-elementary
```
    
3.  Remove Games
4.  Install ones from FlatHub  
    
### Chromium Wayland Support  

To enable Wayland support change the following option within `chrome://flags/` to `Preferred Ozone platform = Wayland`

### Multimedia codecs

Install codecs pack and VLC  

```
sudo apt install libavcodec-extra vlc
```

### VMWare Player installation

    apt install gcc
    apt-get install gcc
    apt-get install gcc
    apt-get install make
    apt install dwarves
    apt-get update
    ln -sf /usr/lib/modules/$(uname -r)/vmlinux.xz /boot/
    cp /sys/kernel/btf/vmlinux /usr/lib/modules/`uname -r`/build/
    sudo vmware-modconfig --console --install-all

References

1.  [https://www.youtube.com/watch?v=dzyfSSPGOfM](https://www.youtube.com/watch?v=dzyfSSPGOfM)  
    
2.  [(3) Debian 12 - The First 12 Things You Should Do After Installation! - YouTube](https://www.youtube.com/watch?v=K72XJHurdUY&t=33s)
3.  [https://wiki.debian.org/NvidiaGraphicsDrivers?action=show&redirect=NVIDIA#Debian\_12\_.22Bookworm.22](https://wiki.debian.org/NvidiaGraphicsDrivers?action=show&redirect=NVIDIA#Debian_12_.22Bookworm.22)
4.  [https://www.reddit.com/r/debian/comments/149jx0y/wayland\_not\_available\_in\_gdm\_after\_installing/](https://www.reddit.com/r/debian/comments/149jx0y/wayland_not_available_in_gdm_after_installing/)
5.  [https://download.nvidia.com/XFree86/Linux-x86_64/525.147.05/README/](https://download.nvidia.com/XFree86/Linux-x86_64/525.147.05/README/)
6.  [https://forums.developer.nvidia.com/t/prime-and-prime-synchronization/44423/280](https://forums.developer.nvidia.com/t/prime-and-prime-synchronization/44423/280)
7.  [https://www.youtube.com/watch?v=aPi8NfDyDMU](https://www.youtube.com/watch?v=aPi8NfDyDMU)
8.  [https://www.reddit.com/r/flatpak/comments/y9jmqj/the\_general\_flatpak\_qt\_and\_gtk\_theming_guide/](https://www.reddit.com/r/flatpak/comments/y9jmqj/the_general_flatpak_qt_and_gtk_theming_guide/)
9.  [https://bugreports.qt.io/browse/QTBUG-53022](https://bugreports.qt.io/browse/QTBUG-53022)
