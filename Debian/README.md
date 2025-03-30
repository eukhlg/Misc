<h2 id="bkmrk-gnome-customization">Gnome customization</h2>
<ol id="bkmrk-install-extension-ma">
<li class="null">Install Extension Manager</li>
<li class="null">Install <strong>Blur my Shell, Dash to Dock, User Themes</strong> (if not installed)</li>
<li class="null">Install <a href="https://flatpak.org/setup/Debian"><strong>Flatpak</strong></a><br></li>
<li class="null">Restart</li>
<li class="null">Download and install <strong>GTK3 themes</strong> from <a href="https://github.com/lassekongo83/adw-gtk3">https://github.com/lassekongo83/adw-gtk3</a><br></li>
<li class="null">Install <strong>GDM Settings</strong> app from FlatHub and copy user's settings to GDM</li>
<li class="null">Adjust Touchpad speed <a href="https://ubuntuhandbook.org/index.php/2023/05/adjust-touchpad-scrolling-ubuntu/">https://ubuntuhandbook.org/index.php/2023/05/adjust-touchpad-scrolling-ubuntu/</a></li>
</ol>
<h3 id="bkmrk-fractional-scaling-a">Fractional Scaling and HiDPI</h3>
<ol id="bkmrk-enable-qt-apps-auto-">
<li class="null">Enable QT apps auto scaling via setting environment variables<br><br>
<pre><code class="language-">sudo nano /etc/security/pam_env.conf
QT_AUTO_SCREEN_SCALE_FACTOR=1
QT_ENABLE_HIGHDPI_SCALING=1</code></pre>
</li>
<li class="null">Enable Wayland HiDPI support for all users<br><br>
<pre><code class="language-"># This is if GDM settings app is present are 
sudo touch /etc/dconf/db/gdm.d/00-hidpi
sudo vi 00-hidpi
[org/gnome/mutter]
experimental-features=['scale-monitor-framebuffer']
# In Fedora this trick has helped
sudo -u gdm dbus-launch gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"</code></pre>
</li>
<li class="null">Enable HiDPI for X11<br><br>
<pre><code class="language-">gsettings set org.gnome.settings-daemon.plugins.xsettings overrides "[{'Gdk/WindowScalingFactor', &lt;2&gt;}]"
vi ~/.config/autostart/xrandr-settings.desktop

[Desktop Entry]
Type=Application
Version=1.0
Name=custom xrandr settings

# Replace with your own xrandr command:
Exec=xrandr --output eDP --scale 1.25x1.25
</code></pre>
</li>
<li class="null">Flatpak<br><br>
<pre><code class="language-">STEAM_FORCE_DESKTOPUI_SCALING=1.5
GDK_SCALE=1.5
ELM_SCALE=1.5
QT_SCALE_FACTOR=2</code></pre>
</li>
</ol>
<h2 id="bkmrk-drivers-installation">Drivers installation</h2>
<h3 id="bkmrk-nvidia-driver">NVIDIA Driver (debian repository)</h3>
<p id="bkmrk-add-%22contrib%22%2C-%22non-">Add "contrib", "non-free" and "non-free-firmware" components to /etc/apt/sources.list, for example:</p>
<pre id="bkmrk-%23-debian-bookworm-de"><code class="language-"># Debian Bookworm
deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware</code></pre>
<p id="bkmrk-update-the-list-of-a">Update the list of available packages, then we can install the nvidia-driver package, plus the necessary firmware:</p>
<pre id="bkmrk-%23-apt-update-%23-apt-i"><code class="language-"># apt update
# apt install nvidia-driver firmware-misc-nonfree</code></pre>
<p id="bkmrk-dkms-will-build-the-">DKMS will build the nvidia module for your system, via the nvidia-kernel-dkms package.</p>
<p id="bkmrk-%C2%A0"><br></p>
<p id="bkmrk-note-about-secureboo" class="callout info">Note about Secureboot : if you have SecureBoot enabled, you need to sign the resulting modules. Detailed instructions are available here.&nbsp;</p>
<h3 id="bkmrk-nvidia-driver-%28offic">NVIDIA Driver (Offical Download)<br></h3>
<p id="bkmrk-download-driver-from">Download driver from here <a href="https://download.nvidia.com/XFree86/Linux-x86_64/">https://download.nvidia.com/XFree86/Linux-x86_64/</a></p>
<p id="bkmrk-the-latest-stable-ve">The latest stable version can be found here <a href="https://www.nvidia.com/Download/driverResults.aspx/216728/en-us/">https://www.nvidia.com/Download/driverResults.aspx/216728/en-us/</a></p>
<pre id="bkmrk-apt-autoremove-%24%28dpk"><code class="language-">apt autoremove $(dpkg -l nvidia-drivers* | grep ii |awk '{print $2}')
apt install linux-headers-$(uname -r) gcc make acpid dkms libglvnd-core-dev libglvnd0 libglvnd-dev dracut
touch /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" &gt;&gt; /etc/modprobe.d/blacklist.conf
GRUB_CMDLINE_LINUX_DEFAULT="quiet rd.driver.blacklist=nouveau"
update-grub
dracut -q /boot/initrd.img-$(uname -r) $(uname -r) --force
systemctl set-default multi-user.target
# Rebbot and run installer in console
# Do not create X11 config this causes issues on GDM boot
systemctl set-default graphical.target

# After reboot
apt install xwayland libxcb1 libnvidia-egl-wayland1</code></pre>
<h4 id="bkmrk-prime-ru">Prime-run</h4>
<ol id="bkmrk-make-sure-the-nvidia">
<li class="null">Make sure the NVIDIA drivers are installed and reboot if you just installed them.</li>
<li class="null">Open a terminal and type: nano prime-run</li>
<li class="null">Copy and paste this script into nano, then save and quit nano:<br>
<pre><code class="language-">#!/bin/bash
__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia "$@"</code></pre>
<p><br></p>
</li>
<li class="null">In terminal type:<br>
<pre><code class="language-">$ sudo mv prime-run /bin
$ cd /bin
$ sudo chmod 755 prime-run</code></pre>
</li>
<li class="null">Test it by typing <code>prime-run glxinfo | grep "OpenGL renderer"</code></li>
</ol>
<h2 id="bkmrk-issues">Issues</h2>
<h3 id="bkmrk-amdgpu-issue-on-dmes">AMDGPU issue on DMESG</h3>
<pre id="bkmrk-%5B-2.123808%5D-amdgpu-0"><code class="language-">[    2.123808] amdgpu 0000:66:00.0: amdgpu: Fetched VBIOS from VFCT
[    2.123809] amdgpu: ATOM BIOS: 113-PHXGENERIC-001
[    2.123814] [drm] VCN(0) encode/decode are enabled in VM mode
[    2.123815] amdgpu 0000:66:00.0: [drm:jpeg_v4_0_early_init [amdgpu]] JPEG decode is enabled in VM mode
[    2.123967] amdgpu 0000:66:00.0: firmware: failed to load amdgpu/gc_11_0_1_mes_2.bin (-2)
[    2.124013] firmware_class: See https://wiki.debian.org/Firmware for information about missing firmware
[    2.124045] amdgpu 0000:66:00.0: firmware: failed to load amdgpu/gc_11_0_1_mes_2.bin (-2)
[    2.124073] amdgpu 0000:66:00.0: Direct firmware load for amdgpu/gc_11_0_1_mes_2.bin failed with error -2
[    2.124074] [drm] try to fall back to amdgpu/gc_11_0_1_mes.bin
[    2.124115] amdgpu 0000:66:00.0: firmware: direct-loading firmware amdgpu/gc_11_0_1_mes.bin
[    2.124148] amdgpu 0000:66:00.0: firmware: direct-loading firmware amdgpu/gc_11_0_1_mes1.bin</code></pre>
<p id="bkmrk-solution-is-here-htt">Solution is here <a href="https://unix.stackexchange.com/questions/765130/ive-got-a-new-lenovo-yoga-pro-7-14aph8-and-after-grub-bootloader-screen-im-see">https://unix.stackexchange.com/questions/765130/ive-got-a-new-lenovo-yoga-pro-7-14aph8-and-after-grub-bootloader-screen-im-see</a></p>
<pre id="bkmrk-cd-%2Ftmp-git-clone-ht"><code class="language-">cd /tmp
git clone https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/
cd linux-firmware
sudo cp -a --no-preserve=ownership amdgpu /lib/firmware
sudo update-initramfs -u #or sudo update-initramfs -k all -u -v</code></pre>
<h3 id="bkmrk-wayland-not-availabl">Wayland not available in GDM after installing Nvidia drivers in Debian 12</h3>
<h4 id="bkmrk-option-1-%28problemati">Option 1</h4>
<pre id="bkmrk-nano-%2Fusr%2Flib%2Fudev%2Fr"><code class="language-">nano /usr/lib/udev/rules.d/61-gdm.rules

#just commented the last two lines here:
LABEL="gdm_prefer_xorg"
#RUN+="/usr/lib/gdm-runtime-config set daemon PreferredDisplayServer xorg"
GOTO="gdm_end"
LABEL="gdm_disable_wayland"
#RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"
GOTO="gdm_end" </code></pre>
<h4 id="bkmrk-option-2">Option 2</h4>
<p id="bkmrk-on-gnome-desktops%2C-a" class="line874">On GNOME desktops, although a proper version of the NVIDIA driver is used, the greeter <span id="bkmrk--6" class="anchor"></span>(GDM3) may still not offer the option to start a Wayland session, either because kernel <span id="bkmrk--7" class="anchor"></span>modesetting is not enabled, or because the hibernate/suspend/resume helper scripts have <span id="bkmrk--8" class="anchor"></span>not been installed on the system. <span id="bkmrk--9" class="anchor"></span><span id="bkmrk--10" class="anchor"></span></p>
<p id="bkmrk-to-enable-kernel-mod" class="line874">To enable kernel mode-setting with the NVIDIA driver: <span id="bkmrk--11" class="anchor"></span><span id="bkmrk--12" class="anchor"></span></p>
<p id="bkmrk-" class="line867"><span id="bkmrk--13" class="anchor"></span><span id="bkmrk--14" class="anchor"></span><span id="bkmrk--15" class="anchor"></span></p>
<pre id="bkmrk-%23-echo-%27grub_cmdline"><code class="language-">touch /etc/default/grub.d/nvidia-modeset.cfg
echo 'GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX nvidia-drm.modeset=1"' &gt; /etc/default/grub.d/nvidia-modeset.cfg
update-grub</code></pre>
<p id="bkmrk-to-install-the-hiber" class="line874">To install the hibernate/suspend/resume helper scripts: <span id="bkmrk--16" class="anchor"></span><span id="bkmrk--17" class="anchor"></span></p>
<ol id="bkmrk-navigate-to-https%3A%2F%2F" type="1">
<li>
<p class="line862">Navigate to <a class="https" href="https://download.nvidia.com/XFree86/">https://download.nvidia.com/XFree86/</a> and click on your appropriate architecture (for example - Linux-x86_64 for 64-bit). <span id="bkmrk--18" class="anchor"></span></p>
</li>
<li>
<p class="line862">Then choose your NVIDIA-driver version. Can be found using the <tt class="backtick">nvidia-smi</tt> command. <span id="bkmrk--19" class="anchor"></span></p>
</li>
<li>
<p class="line862">Download the <tt class="backtick">.run</tt> file and execute it using bash, with the <tt class="backtick">--extract-only</tt> flag. This will extract the files we require into a directory with the same name as the <tt class="backtick">.run</tt> file. <span id="bkmrk--20" class="anchor"></span><span id="bkmrk--21" class="anchor"></span></p>
<pre># bash NVIDIA-Linux-x86_64-525.147.05.run --extract-only</pre>
<span id="bkmrk--22" class="anchor"></span></li>
<li>
<p class="line862">Run the following commands to install the extracted scripts: <span id="bkmrk--23" class="anchor"></span><span id="bkmrk--24" class="anchor"></span><span id="bkmrk--25" class="anchor"></span><span id="bkmrk--26" class="anchor"></span><span id="bkmrk--27" class="anchor"></span><span id="bkmrk--28" class="anchor"></span><span id="bkmrk--29" class="anchor"></span><span id="bkmrk--30" class="anchor"></span><span id="bkmrk--31" class="anchor"></span><span id="bkmrk--32" class="anchor"></span></p>
<pre>TMPL_PATH=/path/to/the/extracted/directory/systemd
sudo install --mode 644 "${TMPL_PATH}/system/nvidia-suspend.service" /etc/systemd/system
sudo install --mode 644 "${TMPL_PATH}/system/nvidia-hibernate.service" /etc/systemd/system
sudo install --mode 644 "${TMPL_PATH}/system/nvidia-resume.service" /etc/systemd/system
sudo install "${TMPL_PATH}/system-sleep/nvidia" /lib/systemd/system-sleep
sudo install "${TMPL_PATH}/nvidia-sleep.sh" /usr/bin
sudo systemctl enable nvidia-suspend.service
sudo systemctl enable nvidia-hibernate.service
sudo systemctl enable nvidia-resume.service</pre>
</li>
</ol>
<p id="bkmrk-in-addition%2C-you-wil" class="line862">In addition, you will need to verify whether the <tt class="backtick">PreserveVideoMemoryAllocations</tt> NVIDIA module parameter is turned on. Without the parameter being enabled, the udev rules in <tt class="backtick">/usr/lib/udev/rules.d/61-gdm.rules</tt> will force a fallback to X11. To check the value of <tt class="backtick">PreserveVideoMemoryAllocations</tt>: <span id="bkmrk--33" class="anchor"></span><span id="bkmrk--34" class="anchor"></span></p>
<p id="bkmrk--0" class="line867"><span id="bkmrk--35" class="anchor"></span><span id="bkmrk--36" class="anchor"></span><span id="bkmrk--37" class="anchor"></span></p>
<pre id="bkmrk-%24-cat-%2Fproc%2Fdriver%2Fn">$ cat /proc/driver/nvidia/params | grep PreserveVideoMemoryAllocations
PreserveVideoMemoryAllocations: 1</pre>
<p id="bkmrk--1"><span id="bkmrk--38" class="anchor"></span><span id="bkmrk--39" class="anchor"></span></p>
<p id="bkmrk-if-this-parameter-is" class="line862">If this parameter is set to zero, you should be able to override it by adding a configuration into <tt class="backtick">modprobe.d</tt> (assuming the file doesn't already exist): <span id="bkmrk--40" class="anchor"></span><span id="bkmrk--41" class="anchor"></span></p>
<p id="bkmrk--2" class="line867"><span id="bkmrk--42" class="anchor"></span><span id="bkmrk--43" class="anchor"></span></p>
<pre id="bkmrk-%23-echo-%27options-nvid"># echo 'options nvidia NVreg_PreserveVideoMemoryAllocations=1' &gt; /etc/modprobe.d/nvidia-power-management.conf</pre>
<p id="bkmrk-if-after-reboot-it-w">If after reboot it was still not working, force-enable Wayland, by overriding some udev rules<br></p>
<p id="bkmrk-looks-like-this-is-a" class="callout warning">Looks like this is almost like option 1, not sure this is good solution</p>
<h2 id="bkmrk--3" class="experiment-name" style="color: var(--primary-color); display: inline-block; font-size: 0.8125rem; font-weight: 500; line-height: 1.5; margin: 0px; padding: 0px; font-family: Roboto, Cantarell, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; background-color: rgba(255, 255, 255, 0.04); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;" title="Experiment enabled"></h2>
<pre id="bkmrk-ln--s-%2Fdev%2Fnull-%2Fetc"><code class="language-">ln -s /dev/null /etc/udev/rules.d/61-gdm.rules</code></pre>
<h2 id="bkmrk-software">Software</h2>
<ol id="bkmrk-remove-debian%27s-mozi">
<li class="null">Remove Debian's Mozila ESR</li>
<li class="null">Remove Libre Office<br><br>
<pre><code class="language-">sudo apt remove libreoffice-common libreoffice-core libreoffice-gnome libreoffice-gtk3 libreoffice-help-common libreoffice-help-en-us libreoffice-style-colibre libreoffice-style-elementary</code></pre>
</li>
<li class="null">Remove Games</li>
<li class="null">Install ones from FlatHub<br></li>
</ol>
<h3 id="bkmrk-chromium">Chromium Wayland Support<br></h3>
<p id="bkmrk-to-enable-wayland-su">To enable Wayland support change the following option within <code>chrome://flags/</code></p>
<h2 id="bkmrk-preferred-ozone-plat" class="experiment-name" style="color: var(--primary-color); display: inline-block; font-size: 0.8125rem; font-weight: 500; line-height: 1.5; margin: 0px; padding: 0px; font-family: Roboto, Cantarell, Arial, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; background-color: rgba(255, 255, 255, 0.04); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;" title="Experiment enabled">Preferred Ozone platform = Wayland</h2>
<h3 id="bkmrk-multimedia-codecs">Multimedia codecs</h3>
<p id="bkmrk-install-codecs-pack-">Install codecs pack and VLC<br></p>
<pre id="bkmrk-sudo-apt-install-lib"><code class="language-">sudo apt install libavcodec-extra vlc</code></pre>
<h2 id="bkmrk--4"></h2>
<h3 id="bkmrk-vmware-player-instal">VMWare Player installation</h3>
<pre id="bkmrk-apt-install-gcc-apt-"><code class="language-">apt install gcc
apt-get install gcc
apt-get install gcc
apt-get install make
apt install dwarves
apt-get update
ln -sf /usr/lib/modules/$(uname -r)/vmlinux.xz /boot/
cp /sys/kernel/btf/vmlinux /usr/lib/modules/`uname -r`/build/
sudo vmware-modconfig --console --install-all</code></pre>
<p id="bkmrk--5"></p>
<h3 id="bkmrk-references">References</h3>
<ol id="bkmrk-https%3A%2F%2Fwww.youtube.">
<li class="null"><a href="https://www.youtube.com/watch?v=dzyfSSPGOfM">https://www.youtube.com/watch?v=dzyfSSPGOfM</a><br></li>
<li class="null"><a href="https://www.youtube.com/watch?v=K72XJHurdUY&amp;t=33s">(3) Debian 12 - The First 12 Things You Should Do After Installation! - YouTube</a></li>
<li class="null"><a href="https://wiki.debian.org/NvidiaGraphicsDrivers?action=show&amp;redirect=NVIDIA#Debian_12_.22Bookworm.22">https://wiki.debian.org/NvidiaGraphicsDrivers?action=show&amp;redirect=NVIDIA#Debian_12_.22Bookworm.22</a></li>
<li class="null"><a href="https://www.reddit.com/r/debian/comments/149jx0y/wayland_not_available_in_gdm_after_installing/">https://www.reddit.com/r/debian/comments/149jx0y/wayland_not_available_in_gdm_after_installing/</a></li>
<li class="null"><a href="https://download.nvidia.com/XFree86/Linux-x86_64/525.147.05/README/">https://download.nvidia.com/XFree86/Linux-x86_64/525.147.05/README/</a></li>
<li class="null"><a href="https://forums.developer.nvidia.com/t/prime-and-prime-synchronization/44423/280">https://forums.developer.nvidia.com/t/prime-and-prime-synchronization/44423/280</a></li>
<li class="null"><a href="https://www.youtube.com/watch?v=aPi8NfDyDMU">https://www.youtube.com/watch?v=aPi8NfDyDMU</a></li>
<li class="null"><a href="https://www.reddit.com/r/flatpak/comments/y9jmqj/the_general_flatpak_qt_and_gtk_theming_guide/">https://www.reddit.com/r/flatpak/comments/y9jmqj/the_general_flatpak_qt_and_gtk_theming_guide/</a></li>
<li class="null"><a href="https://bugreports.qt.io/browse/QTBUG-53022">https://bugreports.qt.io/browse/QTBUG-53022</a></li>
</ol>
