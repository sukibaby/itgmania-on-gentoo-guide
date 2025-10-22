This guide was updated in October 2025, following some restructuring within Gentoo.

Fortunately, these changes are very beneficial for users looking to set up a system with the basics needed to run 3D-accelerated X11.

This guide will set you up to have a Gentoo install for a dedicated ITGmania PC. It also has a few important files like the `make.conf` already set up how you will need it.

Gentoo is a great option for a dedicated ITGmania PC, especially when running on older hardware, because in these cases any little improvement in game performance is very welcome, and the high optimization of Gentoo can really make a difference.

I have this running very smoothly on an AMD A8-7600B with 8GB of PC3-10600 with its internal graphics at 1080p :)

------------------

The Gentoo Handbook for AMD64 (64-bit CPU's) can be generally followed for the initial setup of the system: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/About

If you're using Ethernet for internet, and have a USB of the Gentoo installer ready to go (by using something like Rufus to write the ISO, for example), then you can skip to the 4th page of the Handbook instead: https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks

If you don't have any special ideas for partitioning in mind, feel free to follow their steps for **Partitioning the disk with GPT for UEFI**.

You'll notice early on that you have to make a choice between **OpenRC** or **systemd**. This guide will have you set up **systemd**.

When you get to the step where you install the **stage file** on page 5, be sure you download the stage file titled **"desktop profile | systemd"**.

Be sure you have the correct `make.conf` from this repository copied over, and named `make.conf` (not `make.conf-amd` or whatever) and continue to follow the guide.

When you get to page 7 ("*Configuring the Linux Kernel*"), follow the step to install Linux Firmware, and then skip to "3.1.3 Installing a distribution kernel". Install `gentoo-kernel`, NOT `gentoo-kernel-bin`. After this step, skip directly to page 8, and continue through the end of the guide.

I recommend setting up the GRUB bootloader since it's the least hassle to set up. Don't forget to add a user, install sudo, add your user with visudo, and disable root login before rebooting into your system. 

After rebooting, sign in either on the machine or by SSH and install the following packages.

```
sudo emerge --ask x11-base/xorg-server x11-apps/xinit x11-terms/xterm dev-vcs/git dev-build/cmake media-libs/alsa-lib media-libs/glew media-libs/mesa media-libs/libglvnd dev-libs/libusb dev-lang/nasm media-libs/libpulse x11-libs/gtk+ media-sound/alsa-utils
```

Additionally, you'll need drivers for your GPU, so be sure to emerge this as well:

- Intel: `x11-drivers/xf86-video-intel`
- AMD: `x11-drivers/xf86-video-amdgpu`
- NVIDIA (open): `x11-drivers/nouveau`
- NVIDIA (proprietary): `x11-drivers/nvidia-drivers`

You can then clone ITGmania and build it.

```
git clone https://github.com/itgmania/itgmania
cd itgmania
git submodule update --init --recursive
cmake -B build -DWITH_FFMPEG_JOBS="$(nproc)"
cmake --build build --parallel "$(nproc)"
```

At this point you have ITGmania built and ready to use! :-)

---

## Automatic login (optional)

Once this is done, you can create a `.xinitrc` file in the user home directory containing a command to run ITGmania:

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

