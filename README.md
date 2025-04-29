# Gentoo-based ITGmania PC
This is a guide anyone can follow along to, you just get the Gentoo installer going on your machine and basically copy paste these commands in until ITGmania appears on the screen. You don't really even need Linux knowledge here.

Following this guide will give you the absolute bare minimum needed to run ITGmania, but lets you choose what to do with the system after that; whereas [dins' image](https://docs.google.com/document/d/1_lO2ddaYogve08u7CsjC6OojXy36ZfGgo7VCRVkLJhU/edit?tab=t.0#heading=h.f4jo4mmoacz4) is a designed to be a fast & easy way to get a working setup without any user input, but does not allow customization of the system.

Gentoo is a great choice for a dedicated ITGmania PC, because the game really benefits significantly from the OS level features being built to your specific PC, especially if your PC or graphics card is old or is low on memory. From the moment you push the power button, until you are in the game, is extremely fast, under 10 seconds, even on a 15 year old office PC (assuming SSD/non-totally-ancient graphics card).

A common concern with Gentoo is that compiling stuff takes a long time. True, you can expect to wait a while for GNOME to compile. Especially if this is an older PC, like mine, that will take maybe even 12 hours.  But once it's done, you never have to do it again.

![Screenshot_from_2025-04-28_03-45-21](https://github.com/user-attachments/assets/61414c7b-d5e7-465c-8c25-4dc58b8f0ae6)
<sup><sub>I can actually get 180Hz 1080p with this setup :D</sub></sup>

This guide will set you up to automatically sign in and launch the game, as one would want for an arcade setup (or behavior like [dins' image](https://docs.google.com/document/d/1_lO2ddaYogve08u7CsjC6OojXy36ZfGgo7VCRVkLJhU/edit?tab=t.0#heading=h.f4jo4mmoacz4)). Following this guide will also install a minimal version of GNOME. You will also have SSH/SFTP login to this PC.


### Table of Contents
<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Getting started](#getting-started)
- [Preparing the drive](#preparing-the-drive)
- [After rebooting](#after-rebooting)
- [(OPTIONAL) Set up autologin](#optional-set-up-autologin)
- [Installing ITGmania](#installing-itgmania)
- [Automatically login and start ITGmania](#automatically-login-and-start-itgmania)
- [How to update the game in the future](#how-to-update-the-game-in-the-future)
- [(OPTIONAL) Install a web browser](#optional-install-a-web-browser)
- [(OPTIONAL) Give the normal user permission to shut down via command line](#optional-give-the-normal-user-permission-to-shut-down-via-command-line)
- [(OPTIONAL) Enable desktop icons](#optional-enable-desktop-icons)
- [(OPTIONAL) Install or Update Simply Love Zmod](#optional-install-or-update-simply-love-zmod)

<!-- TOC end -->

## Getting started
Before getting started, it will make your life MUCH easier if the ITGmania PC is hooked up to Ethernet. NetworkManager is optional; you don't need it if you won't be using WiFi. I'll mention when you need to install NetworkManager if you want to install that.

Get the Gentoo installer (**"Minimal Installation CD"**) from https://www.gentoo.org/downloads/. If you're on Windows and you want to install from a USB stick, use this tool to write the installer ISO to a USB stick https://rufus.ie/en/.

If you're going to install to a machine that doesn't have internet, or is WiFi only, download "Stage 3 (desktop profile | systemd)" at this point, you'll need to get it on the machine you're setting up one way or another.

I've provided all the config files you need to change in the files section above. You can reference them or use them directly. The `itgmania.desktop` files are optional.

### In the installer, make sure internet is working
Once you have the installer running, you'll be on a text prompt where it'll tell you to set your password and start SSH if you want. Set the password with `passwd root`. This password is just for while you're in the installer. Note Linux doesn't show feedback as you're typing a password. Just type and hit enter.

If you'd like to continue this from another computer,  enable SSH by typing `/etc/init.d/sshd start` and then note the IP address of your computer by typing `ip addr`.  Look for a line like this:
```
inet 10.0.0.61/24 metric 1024 brd 10.0.0.255 scope global dynamic eno1
```
that would mean you ssh into root@10.0.0.61. On Windows 11 you can open the Terminal and then type `ssh root@10.0.0.61`. It'll ask you if you trust the connection (say yes) and then you'll be asked for the password you just set, and then  you're in.

You can do it on the computer you're setting up too, but you can more easily copy paste commands in  over ssh.

At this point,  type `ping -c 3 www.google.com`. If you see an output like this, it works. 
```
# ping -c 3 www.google.com
PING www.google.com (2607:f8b0:400a:806::2004) 56 data bytes
64 bytes from sea09s28-in-x04.1e100.net (2607:f8b0:400a:806::2004): icmp_seq=1 ttl=56 time=18.4 ms
64 bytes from sea09s28-in-x04.1e100.net (2607:f8b0:400a:806::2004): icmp_seq=2 ttl=56 time=16.3 ms
64 bytes from sea09s28-in-x04.1e100.net (2607:f8b0:400a:806::2004): icmp_seq=3 ttl=56 time=11.8 ms

--- www.google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2163ms
rtt min/avg/max/mdev = 11.831/15.527/18.448/2.756 ms
```
If it didn't work, go here: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Networking#Automatic_network_configuration

### Preparing the drive
Basically anything made after 2010 uses UEFI, so we'll assume you're installing to a machine with UEFI.
Type `fdisk -l` and find the drive you want to install to. My computer only has a single NVME drive, so I get this:
```
# fdisk -l
Disk /dev/nvme0n1: 238.47 GiB, 256060514304 bytes, 500118192 sectors
```
If you have a SATA HDD or SSD, it will probably be `/dev/sda` instead of `/dev/nvme0n1`. From here on, I'll be using `/dev/nvme0n1` in the instructions shown.

Prepare the drive by running fdisk.
```
fdisk /dev/nvme0n1
```
Make a new partition layout by pressing `g`
```
Command (m for help): g
Created a new GPT disklabel (GUID: 3E56EE74-0571-462B-A992-9872E3855D75).
```
Press `w` to save that layout to disk and then reboot.
Once you're rebooted, run fdisk again. Now make your partitions. 

First partition (EFI)
```
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-1953525134, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1953525134, default 1953523711): +1G
 
Created a new partition 1 of type 'Linux filesystem' and of size 1 GiB.
Command (m for help): t

Selected partition 1
Partition type or alias (type L to list all): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.
```

Second partition (swap)
```
Command (m for help): n
Partition number (2-128, default 2): 2
First sector (2099200-1953525134, default 2099200): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2099200-1953525134, default 1953523711): +4G
 
Created a new partition 1 of type 'Linux filesystem' and of size 4 GiB.
Command (m for help): t

Partition number (1,2, default 2): 2
Partition type or alias (type L to list all): 19
 
Changed type of partition 'Linux filesystem' to 'Linux swap'.
```

For the main partition (root), press `n` and then leave all the default values (just press enter a few times) to make a partition to use the rest of the remaining space.
```
Command (m for help): n

Partition number (3-128, default 3): 3
First sector (10487808-1953525134, default 10487808):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (10487808-1953525134, default 1953523711):

Created a new partition 3 of type 'Linux filesystem' and of size 926.5 GiB.
```
Now write that to the disk by pressing `w`.
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
We now need to put a filesystem on each partition.
```
mkfs.vfat /dev/nvme0n1p1
mkfs.xfs /dev/nvme0n1p3
mkswap /dev/nvme0n1p2
```
And now activate the swap partition.
```
swapon /dev/nvme0n1p2
```
Make the EFI boot folder.
```
mkdir --parents /mnt/gentoo/efi
```
Now we need to download the Gentoo stage file. Go back to https://www.gentoo.org/downloads/. If you're connected by ssh, then all you need to do is copy the link to "**Stage 3 (desktop profile | systemd)**".
Then download it by typing in `wget` and then paste in the URL.
```
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/___NOT_A_REAL_URL______example.tar.xz
```
Once it's downloaded, run this command to extract it.
```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
```
At this point you need to get your make.conf set up.
```
nano /etc/portage/make.conf
```
Open the **make.conf** in the file listing above and paste it in. To save a file in nano, hold `Ctrl `and press `X`. It will ask you if you want to save. Press `y` and then press enter.

Copy the network DNS info to the new system.
```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```
Log in to the newly set up system.
```
arch-chroot /mnt/gentoo
```
Mount the EFI partition.
```
mount /dev/nvme0n1p1 /efi
```
Update the package manager, and then re-sync it just to be safe.
```
emerge-webrsync
emerge --sync
```
(OPTIONAL) pick a server in your country. 
```
emerge --ask --verbose --oneshot app-portage/mirrorselect
mirrorselect -i -o >> /etc/portage/make.conf
```
Set your system's profile. Run `eselect profile list` and then look for the one that ends in `desktop/gnome/systemd`. For me it's profile 26:
```
[26]  default/linux/amd64/23.0/desktop/gnome/systemd (stable) *
```
Set the profile:
```
eselect profile set 26
```
Make sure all possible optimizations are enabled:
```
emerge --ask --oneshot app-portage/cpuid2cpuflags
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```
At this point, if you skipped over it earlier, make sure you have the right VIDEO_CARDS value set in make.conf. The make.conf I provided is pre-set for AMD graphics.

Now update everything on the system.
```
emerge --verbose --update --deep --changed-use @world
```
Set your timezone. The timezone options are here:
```
ls -l /usr/share/zoneinfo
```
It's organized by region and then general location. So for Brussels (Belgium) you would type:
```
ln -sf ../usr/share/zoneinfo/Europe/Brussels /etc/localtime
```
or for NYC USA:
```
ln -sf ../usr/share/zoneinfo/America/New_York /etc/localtime
```
Set what locales you would like to generate. You need to uncomment at least `en_US.UTF-8 UTF-8`. Also recommend building Japanese.
```
nano /etc/locale.gen
```
Then generate the locales.
```
locale-gen
```
And finally select the locale.
```
# eselect locale list
Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   en_US
  [4]   en_US.iso88591
  [5]   en_US.utf8 *
  [6]   ja_JP
  [7]   ja_JP.eucjp
  [8]   ja_JP.utf8
  [9]   POSIX
  [ ]   (free form)
# eselect locale set 5
```
Reload the environment with the new locale.
```
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```
Install firmware. This allows stuff like WiFi or GPU drivers to work. It'll take a little while.
```
emerge --ask sys-kernel/linux-firmware
```
Install the kernel. This will take a pretty long time.
```
emerge --ask sys-kernel/gentoo-kernel
```
Once that's done you need to clean up.
```
emerge --depclean
```
Get the kernel sources too.  This does not take a long time.
```
emerge --ask sys-kernel/gentoo-sources
```
We now need to tell Linux where the drives are so it can boot up correctly.
Type `blkid` and you'll get something like this:
```
# blkid
/dev/nvme0n1p3: UUID="4c9bdb18-96b2-446d-9206-704b36ac0702" BLOCK_SIZE="512" TYPE="xfs" PARTLABEL="Linux filesystem" PARTUUID="18819cb6-d312-423c-b1dd-e571b4d88d79"
/dev/nvme0n1p1: UUID="BD49-9A1A" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="Linux filesystem" PARTUUID="937c65f5-abe9-485e-b6d4-b59d1f47e021"
/dev/nvme0n1p2: UUID="e170401a-b7bb-443f-aeed-0a4689f3fce3" TYPE="swap" PARTLABEL="Linux swap" PARTUUID="3b6045e1-258f-4c7f-87ac-95b327915284"
```
The long number next to UUID= is the one you need. So a correct fstab looks like this.
**Don't forget to remove the `"`'s!**
```
UUID=4c9bdb18-96b2-446d-9206-704b36ac0702  /  xfs    defaults      0  0
UUID=BD49-9A1A                             /efi  vfat   defaults      0  2
UUID=e170401a-b7bb-443f-aeed-0a4689f3fce3  swap           swap   defaults      0  0
```
Make your fstab based on what the blkid output was. Then write it to the fstab file.
```
nano /etc/fstab
```
Download dhcpcd. We probably won't use it but it's good to have.
```
emerge --ask net-misc/dhcpcd
```
Set up DHCP within systemd.
```
nano /etc/systemd/network/50-dhcp.network
```
Copy paste in the following, and save the file.
```
[Match]
Name=*

[Network]
DHCP=yes
```

Decide on a hostname.  I call my machine `in-the-groove` for example.
```
echo in-the-groove > /etc/hostname
```
Add your hostname to the hosts file.
```
nano /etc/hosts
```
Here you just add your hostname to the end of the 127.0.0.1 and ::1 lines like so:
```
127.0.0.1       localhost in-the-groove
::1             localhost in-the-groove
```
Set the root user's password (you only need this if you wish to do system maintenance in the future after everything's installed):
```
passwd
```
Install filesystem tools.
```
emerge --ask sys-fs/xfsprogs sys-fs/e2fsprogs sys-fs/dosfstools sys-block/io-scheduler-udev-rules
```
Download the GRUB bootloader.
```
emerge --ask sys-boot/grub
```
Install the GRUB bootloader. Make sure it finishes with no errors reported. If there are errors, check the troubleshooting steps at https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Bootloader
```
# grub-install --efi-directory=/efi

Installing for x86_64-efi platform.
Installation finished. No error reported.
```

Configure GRUB. As long as it says it found at least one linux image, you're good
```
# grub-mkconfig -o /boot/grub/grub.cfg

Generating grub.cfg ...
Found linux image: /boot/vmlinuz-6.6.21-gentoo
Found initrd image: /boot/initramfs-genkernel-amd64-6.6.21-gentoo
done
```
Add a user account. For example this would make a user account with the name max300.
```
# useradd -m -G users,wheel,usb -s /bin/bash max300
# passwd max300
Password: (Enter the password for max300)
Re-enter password: (Re-enter the password to verify)
```
Now it is the moment of truth. We will reboot and everything should work.
```
(chroot) livecd # exit
# cd
# reboot
```
Remove the USB stick once it reboots. It should boot quickly into the new Gentoo installation. If you're not sure if it's started,  hit enter a few times and see if it prompts you for a login username.

### After rebooting

At this point you should be back in your Gentoo install!  Nice work.

Log in as root.

Run these commands to get systemd set up. You may have to answer a few questions.
```
systemd-machine-id-setup
systemd-firstboot --prompt
systemctl preset-all --preset-mode=enable-only
```

Enable the networkd service so you can get online.
```
systemctl enable --now systemd-networkd.service
```
Start the SSH server.
```
systemctl start sshd.service
```
(OPTIONAL) Set the SSH server to start up automatically.
```
systemctl enable sshd.service
```
(OPTIONAL but recommended) Enable NTP. Note NTP can't possibly affect your in-game clock.
```
systemctl enable systemd-timesyncd.service
```
Install a minimal version of libsndfile before installing GNOME.
```
USE="minimal" emerge --ask --oneshot libsndfile
```
Once that's done, install GNOME. This is the biggest wait of the whole setup. At least as long of a wait as `gentoo-kernel` was.
But before you do that, decide if you're happy to let ITGmania have full control over the sound device. If so, continue. If you want to be able to control it with the system settings as well, add pulseaudio to your USE flags and emerge `media-audio/pulseaudio`.
```
emerge --ask gnome-base/gnome-light
```
Make sure GTK+, ALSA and pipewire are installed. ALSA sound driver is best for ITGmania if you don't need it to share the device with any other program.  *ALSA is the best on sync, so I recommend sticking with that.*
```
emerge --ask x11-libs/gtk+ media-libs/alsa-lib media-sound/alsa-utils media-video/pipewire
```
(OPTIONAL) Install NetworkManager if you want.
```
emerge --ask net-misc/networkmanager
systemctl enable --now NetworkManager
```
Enable the login manager.
```
systemctl enable gdm.service
```
Update the environment.
```
env-update && source /etc/profile
```
Add your user to the plugdev group.
```
gpasswd -a max300 plugdev pipewire
```
Make sure nothing needs to be rebuilt at this stage.
```
emerge --ask --verbose --update --deep --changed-use @world
```
Install everything needed for ITGmania.
```
emerge --ask media-libs/glew media-libs/libglvnd dev-libs/libusb dev-lang/nasm dev-vcs/git dev-util/cmake 
```
(OPTIONAL) If you wanted to use PulseAudio for the system, additionally run this
```
systemctl --global enable pulseaudio.service pulseaudio.socket
```
Get a text editor just in case.
```
emerge --ask app-editors/gnome-text-editor
```

### (OPTIONAL) Set up autologin
Open up the login manager configuration file.
```
nano /etc/gdm/custom.conf
```
Add these lines to the `daemon` section, under the `#WaylandEnable=false` line:
```
AutomaticLoginEnable=True
AutomaticLogin=(YOUR USERNAME HERE)
```

### Installing ITGmania

We're almost there!!!
Unlike earlier, you don't want to be in the root account for this.
Connect with SSH as the normal user, or open up a terminal on the ITG PC itself.

Clone ITGmania from the GitHub page.
```
git clone https://github.com/itgmania/itgmania.git
```
Enter the itgmania folder.
```
cd itgmania
```
Build the game.
```
git submodule update --init --recursive
cmake -B build
cmake --build build
```
Everything should have finished successfully without any error messages. Try to run the game (you do have to do this step from the ITG PC itself).
```
./itgmania
```
The game should start up. Congratulations on making it this far.

There is one step left to do though. We need to switch from Wayland to Xorg. You may have noticed the keyboard doesn't work, this will fix it. Close out of ITGmania and log out using the menu in the upper right corner (with the power button). At the login screen, click your username. Click the gear that appears in the bottom right corner. Select "**GNOME on Xorg**". Log back in. This will make GNOME on Xorg the default going forward, so you only have to do this once.

### Automatically login and start ITGmania
Make a folder called autostart in your user directory.
```
mkdir ~/.config/autostart
```
Now make a file to autostart itgmania:
```
nano ~/.config/autostart/itgmania.desktop
```
And then paste the following in:
```
[Desktop Entry]
Type=Application
Exec=/home/[YOUR-USERNAME-HERE]/itgmania/itgmania
Name=ITGmania
X-GNOME-Autostart-enabled=true
```
and then save.

Now if you reboot the PC, it should skip the login screen and open up ITGmania immediately.

### How to update the game in the future
If you want to build a new release, follow the **"Installing ITGmania"** section again. 
To prevent overwriting the old game, you can rename the folder with `mv` like so:
```
mv ~/itgmania ~/itgmania-1.0.2
```
Then when you follow the steps in **"Installing ITGmania"**, it will put it in a folder called `itgmania`.
All your user data is stored in a separate hidden directory (`~/.itgmania`), so moving or even deleting the folder the program itself exists in poses no risk to your data.

### (OPTIONAL) Install a web browser

Brave is really easy to install since it's distributed as a binary. Otherwise compiling a browser takes forever, so I just use Brave.

First you have to add this keyword before it'll work:
```
nano /etc/portage/package.accept_keywords/libpthread-stubs
```
Paste in `dev-libs/libpthread-stubs **` and save the file.
Then install it:
```
emerge --ask libpthread-stubs app-eselect/eselect-repository
eselect repository add brave-overlay git https://gitlab.com/jason.oliveira/brave-overlay.git
emerge --sync
emerge --ask www-client/brave-bin::brave-overlay
```

### (OPTIONAL) Give the normal user permission to shut down via command line
You can shut down thru GNOME as it is, but maybe users want to be able to shut down from the command line without being required to log in as root, without the risks of giving the normal user full root level control.

Install sudo:
```
emerge sudo
```
Run visudo:
```
visudo
```
Scroll to a few lines above the very end of the file and add this: `your_username_here ALL=(ALL) NOPASSWD: /sbin/shutdown -h now`
Then press Ctrl-x to save. Now the normal user can run `sudo shutdown -h now` to shut down the system.

### (OPTIONAL) Enable desktop icons

If you like to have desktop icons, you need to install the Desktop Icons NG plugin.

First install gnome browser integration support.
```
emerge -av gnome-extra/gnome-browser-connector
```
Then install the browser extension at https://extensions.gnome.org/. Once the browser extension is installed, use that same page to search for "desktop icons" and look for **Desktop Icons NG (DING)**.

You can now add a desktop icon for ITGmania by opening any text editor (you have GNOME Text Editor for example) and paste the following in, and then save it to the Desktop as `itgmania.desktop`:
```
[Desktop Entry]
Name=ITGmania
GenericName=Rhythm and dance game
TryExec=/home/[YOUR-USERNAME-HERE]/itgmania/itgmania
Exec=/home/[YOUR-USERNAME-HERE]/itgmania/itgmania
Terminal=false
Icon=/home/[YOUR-USERNAME-HERE]/itgmania/Data/logo.svg
Type=Application
Categories=Game;ArcadeGame
Comment=A cross-platform rhythm video game.
```
Set it as executable:
```
chmod +x ~/Desktop/itgmania.desktop
```

You should then see the ITGmania icon on the desktop. Right click the icon and select the option to say you trust it. Then it will work when you open it.

### (OPTIONAL) Install or Update Simply Love Zmod

Simply Love Zmod is located at https://github.com/zarzob/Simply-Love-SM5

Enter the ITGmania themes folder.
```
cd ~/.itgmania/Themes
```
Run the git command to get the latest version of Simply Love Zmod.
```
git clone https://github.com/zarzob/Simply-Love-SM5.git
```
If you previously had Zmod installed this way, the above commands will update it.
