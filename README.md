## Preface

This guide was updated in October 2025, following some restructuring within Gentoo.

Fortunately, these changes make setting up a 3D-accelerated system with X11 far easier than traditionally was.

This guide will set you up to have a Gentoo install for a dedicated ITGmania PC. It also has a few important files like the `make.conf` already set up how you will need it.

Gentoo is a great option for a dedicated ITGmania PC, especially when running on older hardware, because in these cases any little improvement in game performance is very welcome, and the high optimization of Gentoo can make a difference. Some older but still-usable hardware (like pre-Ryzen AMD APU's) also have non-standard instruction sets which can further boost performance, but will only be taken advantage of with both the kernel and game being custom built, which is easiest done in a Gentoo setup.

------------------

## Note about `emerge`

At some point, it is likely that you will be unable to install packages until you update your configuration files (you'll see someting like `Use --autounmask-write to write changes to config files `). When this happens, simply run

`sudo etc-update`

and you should get asked if you want to apply the update.

Generally you should respond `-3` (with the dash) and hit enter.  After that you will get a `mv` (file move) prompt you have to answer `y` to. Then it should tell you that you're all set.

------------------

## Install Guide

The Gentoo Handbook for AMD64 (64-bit CPU's) can be generally followed for the initial setup of the system: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/About

If you're using Ethernet for internet, and have a USB of the Gentoo installer ready to go (by using something like Rufus to write the ISO, for example), then you can skip to the 4th page of the Handbook instead: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks

If you don't have any special ideas for partitioning in mind, feel free to follow their steps for **Partitioning the disk with GPT for UEFI**.

You'll notice early on that you have to make a choice between **OpenRC** or **systemd**. This guide will have you set up **systemd**.

When you get to the step where you install the **stage file** on page 5, be sure you download the stage file titled **"desktop profile | systemd"**.

Be sure you have the correct `make.conf` from this repository copied over, and named `make.conf` (not `make.conf-amd` or whatever) and continue to follow the guide.

When you get to page 7 ("*Configuring the Linux Kernel*"), follow the step to install Linux Firmware, and then skip to "3.1.3 Installing a distribution kernel". Install `gentoo-kernel`, NOT `gentoo-kernel-bin`. After this step, emerge `gentoo-sources` as well, and then skip directly to page 8. From there, continue through the end of the guide.

I recommend setting up the GRUB bootloader since it's the least hassle to set up. Don't forget to add a user, install sudo, add your user with visudo, and disable root login before rebooting into your system. 

After rebooting, sign in either on the machine or by SSH and install the following packages.

```
sudo emerge --ask x11-base/xorg-server x11-apps/xinit x11-terms/xterm dev-vcs/git dev-build/cmake media-libs/alsa-lib media-libs/glew media-libs/mesa media-libs/libglvnd dev-libs/libusb dev-lang/nasm media-libs/libpulse x11-libs/gtk+ media-sound/alsa-utils
```

Additionally, you'll need drivers for your GPU, so be sure to emerge this as well:

- Intel: `x11-drivers/xf86-video-intel`
- AMD (modern): `x11-drivers/xf86-video-amdgpu`
- AMD (legacy): `x11-drivers/xf86-video-ati`
- NVIDIA (open): `x11-drivers/nouveau`
- NVIDIA (proprietary): `x11-drivers/nvidia-drivers`

At this point, you should additionally define your video card to portage:

```
sudo nano /etc/portage/package.use/00video
```

- Intel (modern): `*/* VIDEO_CARDS: -* intel`
- Intel (legacy): `*/* VIDEO_CARDS: -* intel i915`
- AMD (modern): `*/* VIDEO_CARDS: -* amdgpu`
- AMD (legacy): `*/* VIDEO_CARDS: -* radeon radeonsi`
- nVidia (official): `*/* VIDEO_CARDS: -* nvidia`
- nVidia (open source): `*/* VIDEO_CARDS: -* nouveau`

Paste the appropriate line, and close and save the `00video` file. 

---------

## Building ITGmania

```
cd ~
git clone https://github.com/itgmania/itgmania
cd itgmania
git submodule update --init --recursive
cmake -B build -DWITH_FFMPEG_JOBS="$(nproc)"
cmake --build build --parallel "$(nproc)"
```

You can start the game by typing `./itgmania &`.

-----------

## Low Latency Kernel optimization (optional)

A lot of performance can be gained by optimizing the kernel for low latency.

First run `eselect kernel list` to ensure the latest `-dist` kernel is selected and activated.  If not, run `eselect kernel set #` with the number of the desired kernel.

Use nano or any other editor to make the file `/etc/systemd/system/set-cpufreq.service` and save it with these contents:

```
[Unit]
Description=Set CPU governor to performance
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/bash -c 'for cpu in /sys/devices/system/cpu/cpu[0-9]*; do echo performance > $cpu/cpufreq/scaling_governor; done'

[Install]
WantedBy=multi-user.target
```

and then run `sudo systemctl enable set-cpufreq.service`.

Now make another file called `/etc/kernel/config.d/50-misc.config` and save it with these contents:

```
CONFIG_HZ_1000=y
CONFIG_HZ=1000
CONFIG_PREEMPT=y
```

Finally, rebuild the kernel with `sudo emerge --ask sys-kernel/gentoo-kernel`. You should see near the start of the kernel build that your frequency and 1000hz overrides are applied. Afterwards, ensure grub is up to date by running `grub-mkconfig -o /boot/grub/grub.cfg`.

--------

## Automatic login (optional)

There are many ways to achieve this goal. The example here is a very minimalistic one which uses features built into systemd to do the job, but you could also use a login manager (gdm, lightdm, etc) to do this.

Create a `.xinitrc` file in the user home directory containing a command to run ITGmania:

```
echo exec /home/[your username]/itgmania/itgmania
```

This way, when you run `startx`, ITGmania will directly open.

You can set up automatic login and execution of ITGmania using `systemctl`:

```
sudo systemctl edit getty@tty1
```

And then add something like:

```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin [your username] --noclear %I $TERM
```

Then run these commands to reload and enable:

```
sudo systemctl daemon-reexec
sudo systemctl restart getty@tty1
```

Next, we can use `.bash_profile` to automatically call `startx` on login:

```
nano ~/.bash_profile
```

Here, you can add the following to the end of the file: 

```
if [[ -z $DISPLAY ]] && [[ $(tty) == /dev/tty1 ]]; then
  exec startx
fi
```

------------

## Troubleshooting

If you have low FPS, ensure you have 3D acceleration supported and your GPU is properly configured.

https://wiki.gentoo.org/wiki/Xorg/Hardware_3D_acceleration_guide
