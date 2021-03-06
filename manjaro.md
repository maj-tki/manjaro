# Manjaro

Arch turns out to be a bit of a nightmare due to a `rstan` dependency on `V8`.  
`rstan` needs the `V8` lib but `V8` doesn't currently compile without manual intervention.

For a related discussion see https://discourse.mc-stan.org/t/dangerous-design-in-rstan-2-21/16483

Going with minimal with xfce: `manjaro-xfce-20.0.3-minimal-200606-linux56.iso`.
Also tried gnome `manjaro-gnome-20.0.3-minimal-200606-linux56.iso` but did not like.

Install updates to bring you up to date.

Make sure to `sudo pacman -Syu base-devel` to finish off.

Remember, `dmesg | grep blah` `dmesg | tail -20` are your friends.

```
sudo dmesg --level=err,warn
# timestamps
dmesg -T
```
Also, `modprobe` for adding/removing modules from kernel.

## Software

### Management 

`pacman` https://wiki.manjaro.org/index.php?title=Pacman_Tips

Note `checkupdates` provides a safe way to check for upgrades to installed packages without running a system update at the same time.

Never install a package without updating the system first.

| command                        | desc                                       |
|--------------------------------|--------------------------------------------|
|pacman -Sy                      | download fresh copy of master package db   |
|pacman -Syu pkg                 |	Install (and update package list)         |
|pacman -S pkg                   |	Install only                              |
|pacman -R pkg                   |	Uninstall pkg                             |
|pacman -Rsu pkg                 |	Uninstall pkg and unneeded dep            |
|pacman -Ss keywords	           | Search                                     |
|pacman -Syu	                   | Upgrade everything                         |
|pacman -Qdt                     | list orphan pkgs not used by anything else |
|sudo pacman -Rs $(pacman -Qdtq) | remove all the orphans                     |
|sudo pacman -Sc                 | clear out cache                            |
|pactree -U package              | dependencies                               |
|pactree -r package              | dep                                        |
|pacman -Qe	                     | List explictly-installed packages          |
|pacman -Ql pkg	                 | What files does this package have?         |
|pacman -Qii pkg	               | List information on package                |
|pacman -Qo file	               |  Who owns this file?                       |
|pacman -Qs query	               | Search installed packages for keywords     |
|pacman -Q --info pkg            | get info on installed package              |


Also

```
# update to closest mirror and update system
sudo pacman-mirrors --geoip  && sudo pacman -Syyu

# for local files
sudo pacman -U /var/cache/pacman/pkg/smplayer-19.5.0-1-x86_64.pkg.tar.xz
```

Urgh, install additional package manger `yay` to get to Arch User Repository.  
Commands pretty much as per `pacman`

```
sudo pacman -Syu yay
```

### R

Libs:
```
yay -Syu v8

```

```
sudo pacman -Syu gcc-fortran
# use faster openblas - need to install prior to install r
sudo pacman -Syu openblas
sudo pacman -Syu r
yay -Syu rstudio-desktop
# select 1 rstudio-desktop
```

Now `rstudio` will probably not show windows due to a `qt` issue.  
Solution at bottom of thread here: https://forum.manjaro.org/t/rstudio-gui-broken-after-update/120635/8

```
sudo vi /usr/lib/qt/libexec/qt.conf
# add and then save and try rstudio again:
[Paths]
Prefix = /usr/lib/qt
Data = /usr/share/qt
Translations = /usr/share/qt/translations
```


### Python

Don't bother with `spyder` etc; more hassle than it is worth.


### Email

See around 8:15 mins for config (under gnome) at https://www.youtube.com/watch?v=yCEK2hNP7bg  
It looks like EWS is dead so not sure that access via DavMail etc is going to work regardless of app.  
Install `hiri` works with full calendar integration; no idea how.

### Zoom

Note comments on mic for zoom https://aur.archlinux.org/packages/zoom/


### Neovim

```
sudo pacman -Syu neovim
```

### Winmerge alternative

Use `meld`

### Keepassxc

```
sudo pacman -Syu keepassxc
```

## Video drivers

The standard guide is here:
https://wiki.manjaro.org/index.php?title=Configure_NVIDIA_(non-free)_settings_and_load_them_on_Startup

also relevant https://wiki.archlinux.org/index.php/NVIDIA_Optimus

Note the summary here:
https://forum.manjaro.org/t/howto-set-up-prime-with-nvidia-proprietary-driver/40225

If you can deal with it then the simplest approach is to turn off the internal gpu via the uefi and to power the output ports by the gpu.  

Specifically, note the 'in short' text here:  
http://www.daknetworks.com/blog/453-dell-precision-7720-graphics

The NVIDIA guide:  
http://us.download.nvidia.com/XFree86/Linux-x86/370.28/README/index.html

Useful commands

```
# install screen configurator
sudo pacman -Syu xorg-xrandr
# List configs
xrandr
# driver
mhwd -li
# system desc
inxi -Fxxxza
inxi -G
hwinfo --display --monitor

xrandr --listproviders 
xrandr --prop
pacman -Qs | grep -Ei 'prime|nvidia|optimus|bbsw|vesa|xf86-video'
ls -laR /etc/X11 ; cat /etc/X11/xorg.conf.d/*.conf
ls -la /etc/modprobe.d ; cat /etc/modprobe.d/*.conf
ls -la /etc/modules-load.d ; cat /etc/modules-load.d/*.conf
ls -la /usr/share/X11/xorg.conf.d ; grep -v /usr/share/X11/xorg.conf.d/*.conf
```

Default res on external 3840 x 2160 16:9. Pick something more sensible e.g. 1920  x 1080.

## Video capture 

First, a simple tool to test things:

```
sudo pacman -Syu guvcview

# alt using vlc, should open web cam and audio capture:
vlc v4l2:// :input-slave=alsa:// :v4l-vdev="/dev/video0"
```

```
v4l2-ctl --list-devices

# to get detail on the actual cam - what you can modify:
v4l2-ctl -d /dev/video0 --list-ctrls
                     brightness 0x00980900 (int)    : min=-64 max=64 step=1 default=0 value=0
                       contrast 0x00980901 (int)    : min=0 max=95 step=1 default=0 value=0
                     saturation 0x00980902 (int)    : min=0 max=100 step=1 default=64 value=64
                            hue 0x00980903 (int)    : min=-2000 max=2000 step=1 default=0 value=0
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
                          gamma 0x00980910 (int)    : min=100 max=300 step=1 default=100 value=100
                           gain 0x00980913 (int)    : min=1 max=8 step=1 default=1 value=1
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=2 value=2
      white_balance_temperature 0x0098091a (int)    : min=2800 max=6500 step=1 default=4600 value=4600 flags=inactive
                      sharpness 0x0098091b (int)    : min=1 max=7 step=1 default=2 value=2
         backlight_compensation 0x0098091c (int)    : min=0 max=3 step=1 default=3 value=3
                  exposure_auto 0x009a0901 (menu)   : min=0 max=3 default=3 value=3
              exposure_absolute 0x009a0902 (int)    : min=10 max=626 step=1 default=156 value=156 flags=inactive
```


Kernel status modules.  
The `uvcvideo` is a kernel driver module meant to support any usb video class compliant device.

```
lsmod | grep uvcvideo
```


ffmpeg seems like a bit of a nightmare as a bundle of utilities but probably worth having some familiarity with.

http://www.ffmpeg.org/

and

https://wiki.archlinux.org/index.php/FFmpeg#Recording_webcam

some translation of options for ffmpeg http://4youngpadawans.com/stream-camera-video-and-audio-with-ffmpeg/


## Audio

Dated but may be useful https://download.nvidia.com/XFree86/gpu-hdmi-audio-document/  

Also, earlier zoom link https://aur.archlinux.org/packages/zoom/

```
# show cards and devices
pactl list cards & pacmd list-sinks
# launch pulseaudio mixer 
alsamixer
# f6 to select card (intel is the Realtek ALC289) set mic as reqd then 
sudo alsactl store
# Note that alsamixer PCM = Pulse Code Modulation. This is where all sound is 
# sent to (I think). Review (selecting the appropriate card num):
amixer --card 0

# list all mics
arecord -l
# card 0: PCH [HDA Intel PCH], device 0: ALC289 Analog [ALC289 Analog]
# Subdevices: 0/1
# Subdevice #0: subdevice #0

# arecord and aplay for command line record and play
```

Aside - sound recording via audacity `sudo pacman -Syu audacity`


## Network access e.g. pawsey

SSH
Tools for converting ppk from win box to ssh key.

```
sudo pacman -Syu putty
# convert priv - you will need to enter the keynim for the original .ppk file
puttygen id_dsa.ppk -O private-openssh -o id_dsa
puttygen id_dsa.ppk -O public-openssh -o id_dsa.pub
# agent running?
eval "$(ssh-agent -s)"
# you will need to use the passphrase again
ssh-add ~/.ssh/id_dsa
# then just
ssh -p 22 usernamem@199.19.19.1
```

or do it from scratch: https://support.pawsey.org.au/documentation/display/US/Logging+in+with+SSH


