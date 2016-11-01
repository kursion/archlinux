# archlinux
Here is how I installed ArchLinux and more...

# Download the ISO
- On VirtualBox: insert the media
- Else you can use an external hard drive (USB, Flash, CDs...), burn the ISO image and boot on the device.



# INSTALLATION

## Tips
To scroll in an old cool terminal use `SHIFT+PageUp` and `PageDown`

## Optionnal: Keyboard for Switzerland
`$ loadkeys fr_CH-latin1` 

## Partition (/, /boot and /home) 
Lets say that we have a hard drive of 20 GB and that we want to create the following partitions:
- / (root): 5 GB for the system
- /boot   : 200 MB for the boot partition
- /home   : ~15 GB for your documents, medias, projects, etc.

Notice: you can choose between `MBR` and `GPT` [Choosing between MRB and GPT](https://wiki.archlinux.org/index.php/partitioning#Choosing_between_GPT_and_MBR). I have choose `GPT` since it's newer and can handle bigger partitions. For the boot partition, you will only need 200 MB if you are not using UEFI. In this example, we are not going to have UEFI and this is why we won't need more space for this partition. Feel free to increase the boot partition space size if needed.


1. List the hard drives: `fdisk -l`
2. Create the root, /boot and home partitions:
```sh
$ fdisk /dev/sda

# CREATING THE BOOT PRIMARY PARTITION 
Command (m for help): n    # To create a new partition
Partition type:
  p primary (0 primary, ...)
  e extended (container for logical ...)
Select (default p): p
Partition number: (1-4, default 1): 1
First sector (2048-..., default 2048): 2048    # Take the default one
Last sector ...: +200M
Created a new partition 1 of type 'Linux' and size 200 MiB.
Command (m for help): a
The bootable flag on partition 1 is enabled now.
Command (m for help): t
Partition number 1 selected


# CREATING THE EXTENDED PARTITION for / and /home
Command (m for help): n    # To create a new partition
Partition type:
  p primary (0 primary, ...)
  e extended (container for logical ...)
Select (default p): 2
Partition number: (1-4, default 1): 2
First sector (411648-..., default 411648):     # Take the default one
Last sector ...: +19.8G    # Remember that we used 200MB for the boot partition
Created a new partition 2 of type 'Extended' and of size 19.8 GiB


# CREATING THE LOGICAL PARTITION for / and /home
Command (m for help): n
All space for primary partitions is in use    # If your hard drive is full
Adding logical partition 5
First sector (413696-..., default 413696):    # Take the default one
Last sector ...: +5G
Created a new partition 5 of type 'Linux' and of size 5 GiB.
Command (m for help): n
All space for primary partitions is in use    # If your hard drive is full
Adding logical partition 6
First sector (10901504-..., default 10901504):    # Take the default one
Last sector ...: +14.8G  # Or take the default one to get the remaining space
Created a new partition 6 of type 'Linux' and of size 14.8 GiB.

# WRITING YOUR CHANGE
Command (m for help): w
The partition table has been altered
Calling ioctl() to re-read partition table
Syncing disks

$ 
```

## Formatting your partitions

- Your boot partition should be in FAT32 format and located in `/dev/sda1`. Thus you should run:
```sh
$ mkfs.vfat /dev/sda1
```
- Your `/` (root) partition can be located at `/dev/sda5` and your `/home` partition can be located at `/dev/sda6`. You can format your partitions with whatever filesystem(s) you want. We choose `Ext4` for the purpose of this example, thus running:  
```sh
$ mkfs.ext4 /dev/sda5
$ mkfs.ext4 /dev/sda6
```

# Mounting your points (to install stuff) !
```sh
# MOUNTING THE ROOT /
$ mkdir -p /mnt
$ mount /dev/sda5 /mnt

# MOUNTING THE BOOT /BOOT
$ mkdir -p /mnt/boot
$ mount /dev/sda1 /mnt/boot/

# MOUNTING THE HOME /HOME
$ mkdir -p /mnt/home
$ mount /dev/sda6 /mnt/home/
```

## Installing the basic ArchLinux
`$ pacstrap /mnt base base-devel vim linux`

## Generates Filesystem Table 
`genfstab -U /mnt >> /mnt/etc/fstab`

## Basic configuration of your system
1. Chroot to your newly created system:

        arch-chroot /mnt
        
2. Set the timezone: 
        
        ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime
        
3. Optionnal: If your hardware clock is set in `UTC`: 
        
        $ hwclock --systohc
        
4. Uncomment `en_US.UTF-8`, `UTF-8` and other needed localizations in 
   `/etc/locale.gen`, and generate them with: 
   
        $ locale-gen
        
5. Set the `LANG` variable in `/etc/local.conf` accordingly, for example:

        $ vi /etc/locale.conf
        LANG=en_US.UTF-8

6. If you set the keyboard layout, make the changes persistent in `/etc/vconsole.conf`:

        $ vi /etc/vconsole.conf
        KEYMAP=fr_CH-latin1

7. Install the boot system: 
   - `pacman -S grub`
   - `grub-install /dev/sda`
   - `grub-mkconfig -o /boot/grub/grub.cfg`
   
         Generating grub configuration file...
         Found linux image: /boot/vmlinuz-linux
         Found initrd image(s) in /boot: initramfs-linux.img
         Found fallback initrd image(s) in /boot: initramfs-linux-fallback.img
         done

8. Finally, set the admin password: 
         
         $ passwd
         
9. Enable DHCPCD to have internet
       
         $ systemctl start dhcpcd
         
10. Exit the chroot and umount the disks:
        
        $ exit
        $ umount -R /mnt
        $ restart
        
Enjoy installing your tools :)

# Virtualbox

```sh
$ pacman -S mesa-libgl virtualbox virtualbox-guest-utils 
[...]    # Choose 'virtualbox-host-modules-arch' for linux kernel
         # which should be number 2
Choose a number (default is 1): 2
[...]
$ systemctl enable vboxservice.service
$ VBoxClient --clipboard --draganddrop --seamless --display --checkhostversion
(OR $ VBoxClient-all) TODO: put it in .xprofile to see
$ reboot
```

*more information [Archlinux - Virtualbox](https://wiki.archlinux.org/index.php/VirtualBox)

# Xorg
The graphical server can be install like that:

```sh
$ pacman -S xorg-server xorg-server-utils xorg-xinit xterm
[...]    # Choose the 'xf86-input-evdev' which should 
         # be the number 1
Choose a number (default is 1): 1
```

# Install the video driver 
You need to install the corresponding driver for your graphic card. You can check what to install in the presented table on [Archlinux Xorg](https://wiki.archlinux.org/index.php/xorg).
For VirtualBox, you should install `$ pacman -S xf86-video-vesa`

# I3

I'm in love with [i3 Windows Manager](https://i3wm.org/). But of course you can choose a different one. Here how to install it:

1. `$ pacman -S i3`
2. `$ echo "exec i3" > ~/.xinitrc`

# Set Keyboard layout in Xorg
`$ setxkbmap ch fr`

# Usefull tools
`$ pacman -S wget git`

# ZSH
Installing ZSH with the great Oh-my-ZSH enhancement suite.

```sh
$ pacman -S zsh
$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

NOTE: don't forget to relog yourself to have the new shell

# Configure Xterm (colors, fonts)
Edit the following file `$ vim ~/.Xresources` and paste the following

```xterm
xterm*faceName: DejaVu Sans Mono Book
xterm*faceSize: 11
xterm*vt100*geometry: 80x60
xterm*saveLines: 16384
xterm*loginShell: true
xterm*charClass: 33:48,35:48,37:48,43:48,45-47:48,64:48,95:48,126:48
xterm*termName: xterm-color
xterm*eightBitInput: false

!BLK Cursor
#define _color0        #000d18
#define _color8        #000d18
!RED Tag
#define _color1        #e89393
#define _color9        #e89393
!GRN SpecialKey
#define _color2        #9ece9e
#define _color10       #9ece9e
!YEL Keyword
#define _color3        #f0dfaf
#define _color11       #f0dfaf
!BLU Number
#define _color4        #8cd0d3
#define _color12       #8cd0d3
!MAG Precondit
#define _color5        #c0bed1
#define _color13       #c0bed1
!CYN Float
#define _color6        #dfaf8f
#define _color14       #dfaf8f
!WHT Search
#define _color7        #efefef
#define _color15       #efefef
!FMT Include, StatusLine, ErrorMsg
#define _colorBD       #ffcfaf
#define _colorUL       #ccdc90
#define _colorIT       #80d4aa
!TXT Normal, Normal, Cursor
#define _foreground    #dcdccc
#define _background    #1f1f1f
#define _cursorColor   #8faf9f
URxvt*color0         : _color0
URxvt*color1         : _color1
URxvt*color2         : _color2
URxvt*color3         : _color3
URxvt*color4         : _color4
URxvt*color5         : _color5
URxvt*color6         : _color6
URxvt*color7         : _color7
URxvt*color8         : _color8
URxvt*color9         : _color9
URxvt*color10        : _color10
URxvt*color11        : _color11
URxvt*color12        : _color12
URxvt*color13        : _color13
URxvt*color14        : _color14
URxvt*color15        : _color15
URxvt*colorBD        : _colorBD
URxvt*colorIT        : _colorIT
URxvt*colorUL        : _colorUL
URxvt*foreground     : _foreground
URxvt*background     : _background
URxvt*cursorColor    : _cursorColor
XTerm*color0         : _color0
XTerm*color1         : _color1
XTerm*color2         : _color2
XTerm*color3         : _color3
XTerm*color4         : _color4
XTerm*color5         : _color5
XTerm*color6         : _color6
XTerm*color7         : _color7
XTerm*color8         : _color8
XTerm*color9         : _color9
XTerm*color10        : _color10
XTerm*color11        : _color11
XTerm*color12        : _color12
XTerm*color13        : _color13
XTerm*color14        : _color14
XTerm*color15        : _color15
XTerm*colorBD        : _colorBD
XTerm*colorIT        : _colorIT
XTerm*colorUL        : _colorUL
XTerm*foreground     : _foreground
XTerm*background     : _background
XTerm*cursorColor    : _cursorColor
```

Don't forget to add the following command into your `~/.xinitrc` in order to: 

    $ echo "xrdb -merge ~/.Xresources" >> .xinitrc

And logout !

*NOTE: you sould probably install the `ttf-dejavu` font*

# Install a font

1. Copy paste the `.ttf` file into /usr/share/fonts/TTF (eg: `sudo cp ~/DejaVu Sans Mono for Powerline.tff /usr/share/fonts/TTF`).
2. `$ sudo fc-cache`
3. Optionnal: to use it, edit your .Xresources by updating the `xterm*faceName` line to:
`xterm*faceName: DejaVu Sans Mono for Powerline`

# Recommended tools and comments
- On surface Pro 4: I was facing an issue related to the resolution. Apparently, the resolution was very big causing the fonts and windows to be very small. To fixes this, I went into the Configuration of the VM and choosed a factor of 200% for the Display. Then I installed `arandr` and choosed a smaller resolution.
- Easily access your applications using DMenu: `$ pacman -S dmenu`
- Chromium can't be run with the root user.
- Install Clipit for system clipboard: `$ sudo pacman -S clipit` 
