# Gentoo install guide

![gentoo_logo](images/200px-gentoo-logo-dark.svg.png)

- [Gentoo install guide](#gentoo-install-guide)
  - [Introduction](#introduction)
  - [Installation concerns](#installation-concerns)
  - [Start live-cd environment](#start-live-cd-environment)
    - [Prepare Hard disk](#prepare-hard-disk)
    - [Prepare encrypted container](#prepare-encrypted-container)
    - [Create LVM volumes](#create-lvm-volumes)
    - [Create filesystems](#create-filesystems)
    - [Mount the new filesystems](#mount-the-new-filesystems)
  - [Installing the Gentoo base system](#installing-the-gentoo-base-system)
    - [Install the stage3 tarball](#install-the-stage3-tarball)
    - [Configuring compile options](#configuring-compile-options)
    - [Selecting mirrors](#selecting-mirrors)
    - [Configuring the main Gentoo repository](#configuring-the-main-gentoo-repository)
    - [Copy DNS info](#copy-dns-info)
    - [Mounting the necessary filesystems](#mounting-the-necessary-filesystems)
    - [Entering the new environment](#entering-the-new-environment)
    - [Configuring Portage](#configuring-portage)
    - [Updating the Portage tree](#updating-the-portage-tree)
    - [Choosing the right profile](#choosing-the-right-profile)
    - [Configuring the USE variable](#configuring-the-use-variable)
    - [Configuring the CPU Flags](#configuring-the-cpu-flags)
    - [Re-compile and update @world](#re-compile-and-update-world)
  - [Configuring the base system](#configuring-the-base-system)
    - [Allow licenses for packages](#allow-licenses-for-packages)
    - [Timezone](#timezone)
    - [Configure locales](#configure-locales)
    - [Configuring Linux kernel](#configuring-linux-kernel)
      - [Installing external firmware](#installing-external-firmware)
      - [Installing the sources](#installing-the-sources)
        - [Manual set up](#manual-set-up)
        - [Semi-automatic set up using genkernel](#semi-automatic-set-up-using-genkernel)
      - [Configuring the modules](#configuring-the-modules)
    - [LVM Configuration](#lvm-configuration)
    - [Fstab](#fstab)
    - [Configuring crypttab](#configuring-crypttab)
    - [Configuring mtab](#configuring-mtab)
    - [Systemd boot (bootloader)](#systemd-boot-bootloader)
    - [Add bootloader entries](#add-bootloader-entries)
    - [Efibootmgr](#efibootmgr)
  - [Rebooting into our new Gentoo systemd](#rebooting-into-our-new-gentoo-systemd)
    - [Enable lvm2](#enable-lvm2)
    - [Change the root password](#change-the-root-password)
    - [And reboot to your new Gentoo system](#and-reboot-to-your-new-gentoo-system)
  - [Post-installation](#post-installation)
    - [Setting the Hostname](#setting-the-hostname)
    - [Configuring Network](#configuring-network)
    - [Using systemd-networkd](#using-systemd-networkd)
    - [Using NetworkManager](#using-networkmanager)
    - [Setting locales](#setting-locales)
    - [Setting time and date](#setting-time-and-date)
    - [File indexing](#file-indexing)
    - [Filesystem tools](#filesystem-tools)
    - [Adding a user for daily use](#adding-a-user-for-daily-use)
    - [Exec as root with sudo](#exec-as-root-with-sudo)
    - [Removing installation tarballs](#removing-installation-tarballs)
    - [Power consumption optimization with Powertop](#power-consumption-optimization-with-powertop)
    - [GPU](#gpu)
    - [Input devices](#input-devices)
    - [X server](#x-server)
    - [A bunch of useful stuff](#a-bunch-of-useful-stuff)
    - [Portage nice value](#portage-nice-value)
    - [Setting portage branches](#setting-portage-branches)
    - [Masked and unmasked packages](#masked-and-unmasked-packages)
    - [Overlays](#overlays)
      - [Custom overlay](#custom-overlay)
    - [Virtualization (How to use Qemu \& kvm)](#virtualization-how-to-use-qemu--kvm)
      - [Kernel options](#kernel-options)
    - [Speed up the system with prelink](#speed-up-the-system-with-prelink)
  - [Last notes](#last-notes)

## Introduction

Gentoo is a Linux distribution that, unlike binary distros like Arch, Debian, and many others, the software is compiled locally according to the user preferences and optimizations.

The name Gentoo comes from the penguin species, which are known to be the fastest in the world.

I've been using Gentoo since 2002 (20+ years already :astonished: ), and what I like most about Gentoo is:

* It's fun
* The possibility of getting everything under control
* Deep customization
One learns **a lot** about GNU/Linux from using it.

Disclaimer: Gentoo is the perfect distribution to use the quote from an ancient adage that says: **With great power comes great responsibility**. Remember that although Gentoo gives you a lot of flexibility and customization, it also requires time, dedication, and patience.

If you are that kind of person, speak *emerge* and enter :sunrise:

| :warning: Disclaimer                                                                                                                              |
| :------------------------------------------------------------------------------------------------------------------------------------------------ |
| This guide is intended to be a learning experience, and I will try to explain all steps to give you an understanding of what we're doing and why. |

## Installation concerns

> Stop before further reading. It's important to know that this guide only contemplates an installation with UEFI, crypt/luks disk, lvm partitioning, and systemd init system. Please remember that if you want a different setup, don't take this guide step-by-step but as a general guideline.

## Start live-cd environment

The first thing we need to install our Gentoo is a live-cd environment with UEFI vars enabled.

~~I like [systemrescuecd](http://www.system-rescue-cd.org) because is Gentoo-based and has UEFI vars enabled. You can download it here [bootable usb](https://www.system-rescue-cd.org/Sysresccd-manual-en_How_to_install_SystemRescueCd_on_an_USB-stick).~~

As [systemrescuecd](http://www.system-rescue-cd.org) is now based on Arch, we will use the Gentoo LiveGUI USB Image. You can find it in the [Gentoo downloads section](https://www.gentoo.org/downloads/). Scroll down to `Advanced choices and other architectures` and download the `Boot media` called ` LiveGUI USB Image`.

To make sure that we boot on UEFI mode, by running:

```sh
efivar -l
```

If the command listed the UEFI variables, we're good to go :checkered_flag:

### Prepare Hard disk

I suggest you use whatever you're familiar with, I will show you the process using `gdisk`, but others like `cgdisk` or `parted` will do the job.

| :information_source: Info point                                             |
| :-------------------------------------------------------------------------- |
| Once you run the first `gdisk` command, you will enter the `gdisk` console. |

Go ahead and set the following environment variables:
```sh
# bash/zsh
export GENTOO_DISK="the disk you want to install gentoo on"
export GENTOO_ROOT="/mnt/gentoo"
export GENTOO_ROOT_PART="$GENTOO_DISK"p2 # Change p2 to the partition you're using as your rootfs
export GENTOO_BOOT_PART="$GENTOO_DISK"p1

# fish
set -x GENTOO_DISK "the disk you want to install gentoo on"
set -x GENTOO_ROOT "/mnt/gentoo"
set -x GENTOO_ROOT_PART "$GENTOO_DISK"p2 # Change p2 to the partition you're using as your rootfs
set -x GENTOO_BOOT_PART "$GENTOO_DISK"p1
```

Enter `gdisk`:
```sh
gdisk "$GENTOO_DISK"
```

Create a new GUID partition table and destroy everything on the disk:

```shell
o (Create a new empty GUID partition table (GPT))
Proceed? Y
```

Now that we've got the disk wiped, we'll create some new partitions. The standard partition schema would be a partition for EFI, a partition for SWAP, and one or more for the system. I'm not going to create a partition for SWAP because I have enough RAM to handle the system's load, but feel free to make one for your needs.

```shell
n (Add a new partition)
Partition number 1
First sector 2048 (default)
Last sector +512M
Hex code EF00

n (Add a new partition)
Partition number 2
First sector 1050624 (default)
Last sector (press Enter to use remaining disk)
Hex code 8E00
```

Now, apply the changes to the disk and quit:

```shell
w
Y
```

Partitions should look something like this:

```sh
gdisk -l $GENTOO_DISK
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1050623   512.0 MiB   EF00  EFI
   2         1050624       500118158   238.0 GiB   8300  LVM
```

### Prepare encrypted container

At this point, we have the disk partitioned. Next, we want to create an encrypted container that will hold the LVM volumes.

To get the best performance of working with encrypted containers, we should use an encryption algorithm supported by our CPU. We have to be wise in picking up because we won't be able to change it afterward. Luckily for us, `cryptosetup` is here :ok_hand:

Run:

```shell
cryptsetup benchmark
```

You should see something like this:

```shell
# Tests are approximate using memory only (no storage IO).
PBKDF2-sha1      3355443 iterations per second for 256-bit key
PBKDF2-sha256    5447148 iterations per second for 256-bit key
PBKDF2-sha512    2068197 iterations per second for 256-bit key
PBKDF2-ripemd160 1182160 iterations per second for 256-bit key
PBKDF2-whirlpool  784862 iterations per second for 256-bit key
argon2i      10 iterations, 1048576 memory, 4 parallel threads (CPUs) for 256-bit key (requested 2000 ms time)
argon2id     10 iterations, 1048576 memory, 4 parallel threads (CPUs) for 256-bit key (requested 2000 ms time)
#     Algorithm |       Key |      Encryption |      Decryption
        aes-cbc        128b      1876.9 MiB/s      7495.4 MiB/s
    serpent-cbc        128b       113.2 MiB/s       816.2 MiB/s
    twofish-cbc        128b       283.5 MiB/s       516.3 MiB/s
        aes-cbc        256b      1431.1 MiB/s      5949.0 MiB/s
    serpent-cbc        256b       114.3 MiB/s       816.3 MiB/s
    twofish-cbc        256b       284.0 MiB/s       516.4 MiB/s
        aes-xts        256b      6022.2 MiB/s      6000.8 MiB/s
    serpent-xts        256b       767.7 MiB/s       728.6 MiB/s
    twofish-xts        256b       475.3 MiB/s       481.8 MiB/s
        aes-xts        512b      5261.2 MiB/s      5233.4 MiB/s
    serpent-xts        512b       776.3 MiB/s       729.4 MiB/s
    twofish-xts        512b       477.6 MiB/s       480.9 MiB/s
```

Choose the option with better overall performance. For example, in the output above would be `aes-xts` and the key size `PBKDF2-sha256`.

When you have yours chosen, then proceed by running:

```shell
cryptsetup -v --cipher aes-xts-plain64 --key-size 256 -y luksFormat $GENTOO_ROOT_PART
```

The command will ask you to confirm with a `YES`. After a second, you should see something like `Command successful`.

Then we need to open (decrypt) the container to start creating the volumes inside.

When we open the container, we need to specify a label name to identify the container once opened. As you can see, in the command below, I've used **cryptcontainer**, be creative :stuck_out_tongue_winking_eye:

```shell
cryptsetup open --type luks $GENTOO_ROOT_PART cryptcontainer
```

And... we're done with the encryption. Now, volume time! :sound:

### Create LVM volumes

| :information_source: Info point                                                                                                                                              |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| For those of you that are not familiar with this acronym, LVM stands for **Logical Volume Management**. You can read more about it [here](https://wiki.gentoo.org/wiki/LVM). |

The good thing about logical volumes is that you can modify them at any point. In this example, I'm going to create two logical volumes, one for the `root` partition and one for the `home`, but feel free to create as many as you want.

Also, I'll use the name `vg0` to identify this volume group. This is a trivial name, so again, be creative :D

```shell
pvcreate /dev/mapper/cryptcontainer
vgcreate vg0 /dev/mapper/cryptcontainer
```

If running the previous commands, you see some warnings like:

```shell
WARNING: Failed to connect to lvmetad. Falling back to device scanning.
```

It's fine as long as you see a `Physical volume "/dev/mapper/cryptcontainer" successfully created.` following.

At this point, let's check that what we have is what we want:

```shell
vgdisplay
```

You should see something like:

```shell
  --- Volume group ---
  VG Name               vg0
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               63.48 GiB
  PE Size               4.00 MiB
  Total PE              16251
  Alloc PE / Size       0 / 0
  Free  PE / Size       16251 / 63.48 GiB
  VG UUID               uQEI07-lgcP-hQ6j-jTR8-BhsO-AsTA-IlLMwC
```

If what you see makes sense, let's create the volumes:

```shell
lvcreate --size 50G vg0 --name root
lvcreate --extent 100%FREE vg0 --name home
```

To double-check what we've done, run:

```shell
lvdisplay
```

You should see something like:

```shell
  --- Logical volume ---
  LV Path                /dev/vg0/root
  LV Name                root
  VG Name                vg0
  LV UUID                ggiZPZ-ygBQ-0fxR-oiXN-qLjC-KDOm-phY1ip
  LV Write Access        read/write
  LV Creation host, time livecd, 2022-08-26 09:35:21 +0000
  LV Status              available
  # open                 0
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/vg0/home
  LV Name                home
  VG Name                vg0
  LV UUID                PZwAiM-kbw5-Cq1y-63xY-nApf-VEms-t31rJI
  LV Write Access        read/write
  LV Creation host, time livecd, 2022-08-26 09:35:32 +0000
  LV Status              available
  # open                 0
  LV Size                1.77 TiB
  Current LE             464003
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
```

If you do, well done :muscle: Let's keep rolling!

### Create filesystems

The good thing about Linux is that we have a lot of filesystems that we can use, but this is also the wrong thing :sweat_smile: "what to choose" and "when" will most probably come to your mind at some point.

You can read as much as you want [here](https://wiki.gentoo.org/wiki/Filesystem), but we'll keep it easy. We will use `ext4`, the default filesystem for many Linux distributions.

Let's create our three main filesystems volumes, one FAT32 for UEFI (:fearful: yes, I know, but this is how UEFI works), and then our main `ext4` filesystems. If you've created more volumes, remember to make a filesystem for all of them:

```shell
mkfs.vfat -F32 $GENTOO_BOOT_PART
mkfs.ext4 /dev/mapper/vg0-root
mkfs.ext4 /dev/mapper/vg0-home
```

### Mount the new filesystems

With our logical volumes created and our filesystems ready, let's mount our partitions to start building our Gentoo system:

```shell
mkdir -p $GENTOO_ROOT
mount /dev/mapper/vg0-root $GENTOO_ROOT
mkdir -p $GENTOO_ROOT/boot
mount $GENTOO_BOOT_PART $GENTOO_ROOT/boot
mkdir $GENTOO_ROOT/home
mount /dev/mapper/vg0-home $GENTOO_ROOT/home
```

## Installing the Gentoo base system

Before installing Gentoo, ensure the date and time are set correctly. A misconfigured clock may lead to strange results in the future, and you don't want this :)

To check our current system date just run:

```shell
date
```

Select the timezone:

```shell
tzselect
```

Then use NTP to set sync the time and date:

```shell
ntpdate pool.ntp.org
```

### Install the stage3 tarball

To avoid installing Linux from scratch, the awesome Gentoo developers provide a Stage 3 build, mainly a **base-binary-semi-working-non-bootable-environment** (no joke :satisfied: ) created to save us tons of time.

We're going to grab that **base-binary-semi-working-non-bootable-environment**, untar it into our Gentoo directory structure. This will create all the necessary binaries and files to start compiling our Gentoo system.

We first download the tarball:

```shell
curl -o $GENTOO_ROOT/stage3-amd64-systemd.tar.xz -L https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20230307T201702Z/stage3-amd64-systemd-20230307T201702Z.tar.xz
```

And we unpack it in our root directory that is mounted in `/mnt/gentoo` directory:

```shell
cd $GENTOO_ROOT
tar xvf stage3-*.tar.xz --xattrs
```

At this point, we have all files to start setting up our new Gentoo environment.

Now is when our CPU starts panicking :worried:

### Configuring compile options

I would like to introduce you to `Portage`, Gentoo's cornerstone.

For those who don't know (yet), Portage is Gentoo's auto-build system and package management tool. It will grab the source code of anything we want to install (defined in the `ebuild` file), compile it (based on CPU architecture Flags and which features are available for every package with what are known as "use flags"), and install it in our system. There's also the option to download pre-compiled packages, but I like to make my CPU work hard :D

This might be intimidating for some of you, but no worries, we will take it step by step.

And the firsts are the CPU flags. The CPU flags tell the compiler the options natively supported for our CPU. This translates to our CPU, and not any other will compile the system binaries.

The easiest way is to go to the Gentoo wiki [here](https://wiki.gentoo.org/wiki/Safe_CFLAGS) and check the best `COMMON_FLAGS` to use for your CPU, but if you're interested, you can also do it manually by:

Running:

```shell
gcc -c -Q -march=native --help=target | awk '/^  -march=/ {print $2}'
```

You should see one word, in my case, was `tigerlake`. So basically, this is the best CPU flag for my CPU type.

Before setting it, it's worth double checking the value with what is defined in the Gentoo wiki, just in case :)

Now, edit this file:

```shell
nano -w $GENTOO_ROOT/etc/portage/make.conf
```

And set the `-march=tigerlake` (or the type you got) at the beginning of `COMMON_FLAGS` to something like:

```shell
COMMON_FLAGS="-march=tigerlake -O2 -pipe"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
```

Now we're going to set up `MAKEOPTS`. `MAKEOPTS` describes the number of parallel jobs used by Portage.

There's always been some discussion on what to set here, but I'm going to follow what the Gentoo wiki recommends [here](https://wiki.gentoo.org/wiki/MAKEOPTS), which is: `less than or equal to the minimum of the size of RAM/2GB or CPU thread count`.

We will use the number of CPU threads by running:

```shell
lscpu | awk '/^CPU\(s\):/ {print $2}'
```

Additionally, to keep the system responsive when compiling, the build system supports limiting the maximum load for compilation with the parameter `--load-average`.

With all that we've said in this section, let's edit the `make.conf` file again:

```shell
nano $GENTOO_ROOT/etc/portage/make.conf
```

And this time, we set `MAKEOPTS`:

```shell
MAKEOPTS="--jobs 8 --load-average 9"
```

### Selecting mirrors

Gentoo uses the closes mirror to sync the packages index, so setting the best mirrors for your location is essential. Luckily the tool `mirrorselect` is going to do the hard work for us :D

:warning: Make sure that you have Internet access from your live-cd:

```shell
mirrorselect -D -s4 -o >> $GENTOO_ROOT/etc/portage/make.conf
```

You should now have an entry for `GENTOO_MIRRORS` in `/mnt/gentoo/etc/portage/make.conf`.

### Configuring the main Gentoo repository

Copy the Gentoo repository configuration file from Portage package:

```shell
mkdir -p $GENTOO_ROOT/etc/portage/repos.conf
cp $GENTOO_ROOT/usr/share/portage/config/repos.conf $GENTOO_ROOT/etc/portage/repos.conf/gentoo.conf
```

### Copy DNS info

Copy the DNS information from your working live-cd environment into the new system to make sure that we will be able to resolve domain names once we switch to it:

```shell
cp --dereference /etc/resolv.conf $GENTOO_ROOT/etc/
```

### Mounting the necessary filesystems

In addition to the LVM filesystem we created in our local disk, other pseudo-filesystems are created during system boot that is needed to chroot into our new environment:

:warning: If you are setting an environment without `SystemD`, then you can skip the `--make-rslave` lines

```shell
mount --types proc /proc $GENTOO_ROOT/proc
mount --rbind /sys $GENTOO_ROOT/sys
mount --make-rslave $GENTOO_ROOT/sys
mount --rbind /dev $GENTOO_ROOT/dev
mount --make-rslave $GENTOO_ROOT/dev
mount --bind /run $GENTOO_ROOT/run
mount --make-slave $GENTOO_ROOT/run
```

### Entering the new environment

Chroot into the new environment:

```shell
chroot $GENTOO_ROOT /bin/bash
```

```shell
source /etc/profile
export PS1="(chroot) $PS1"
```

Great! We are inside our Gentoo system :tada: Unfortunately, it still needs a little bit more time baking :cake:

Please take a deep breath, and let's keep rolling.

### Configuring Portage

Portage does the heavy lifting of repository and package management. Everything from dependency resolution, building source code, and installing the software on our system is done by Portage, using some of the tools that it provides, such as `emerge`.

First, we need to synchronize the remote repositories with the local Portage tree to know what packages are available to be installed.

### Updating the Portage tree

Then update the snapshot with the latest version of the repository:

```shell
emerge --sync
```

### Choosing the right profile

A Portage profile specifies default values for global and per-package USE flags, specifies default values for most variables found in `/etc/portage/make.conf`, and defines a set of system packages. The Gentoo developers maintain the profiles as part of the Portage tree.

List the available profiles:

```shell
eselect profile list
```

From the output list, we must select our best fitting option. Because the system we're building is based on `SystemD`, the best profile we can choose is the `systemd`. Of course, we can choose any other that has `systemd` in it, like `default/linux/amd64/17.0/desktop/gnome/systemd`, but we want a minimal environment to be configured.

```shell
[..]
[17]  default/linux/amd64/17.1/systemd (stable) *
[..]
```

This should be the default profile selected, as we can tell for the `*` at the end. But if not, make sure you choose it with `eselect`:

```shell
eselect profile set 17 # or the number that you have on your list
```

### Configuring the USE variable

As we said, [USE flags](https://wiki.gentoo.org/wiki/USE_flag) are a core feature of Gentoo. Therefore, a good understanding of how to deal with them is needed to have a customized and healthy Gentoo system.

The USE variables can be defined `system-wide` or `per package` domain. You can read more info here:

- System-wide in [/etc/portage/make.conf](https://wiki.gentoo.org/wiki//etc/portage/make.conf#USE)
- Per package in [/etc/portage/package.use](https://wiki.gentoo.org/wiki//etc/portage/package.use)

We can see the list of USE flags that are set up in our system by running:

```shell
emerge --info | grep ^USE
```

Don't get scared by the list. You will most probably increase this once we finish with this section.

We can find a description of all USE flags in `less /var/db/repos/gentoo/profiles/use.desc`.


Additionally, the utility named `quse` from `portage-utils` package can tell us which package uses what USE flags.

For example, if you want to know which packages use the `systemd` flag we simply need to run:

```shell
quse systemd
```

And you will have the list of packages affected by `systemd`.

| :hand: Reminder                                                                                                                                                                                                                                                                                                                                                                                           |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| At this point is normal if you feel overwhelmed. Setting and choosing the proper USE flags before installing anything in our system will save us time later. It's crucial to remember that USE flags we use can always be changed, and if we choose something that we don't want to use anymore or we made a mistake, we can always change them and re-compile the affected packages. So, relax :relaxed: |

Said that we're going to use a tool called `ufed` that will help use-setting the USE flags that we want in any domain.

```shell
emerge app-portage/eix app-portage/ufed
```

And then, run it to select the USE flags with a beautiful user interface :smile: This step will take some time, so let's grab a :tea: or :coffee: and let's do it!

```shell
ufed
```

| :warning: Required USE flags                                                                                                         |
| :----------------------------------------------------------------------------------------------------------------------------------- |
| To make sure that our setup will work as expected, we need at least these USE flags set `cryptsetup`, `lvm`, and `lvm2create-initrd` |


Once we're done, we can see how many system-wide USE flags we've defined by looking at the `/etc/portage/make.conf` file:

```shell
nano -w /etc/portage/make.conf
```

| :warning: Don't use this as an example                               |
| :------------------------------------------------------------------- |
| These are the USE flags for my test system, don't use them for yours |

```shell
USE="10bit 12bit 256-color 7zip a-like-o aac aacs aalib acpi aio
     bash-completion boost branding caps contrib cpu cpudetection cpuid
     cpuinfo cpuload cryptsetup dbus efi64 gd gzip hardened hddtemp highlight
     highlighting imagemagick initramfs int64 ipv4 jemalloc jpeg json
     kmod lm-sensors lvm lvm2create-initrd lz4 lzip lzma lzo lzo2 nano
     nano-syntax networkmanager nss numa opencl opengl openssl python ssh
     sslv2 sslv3 startup-notification tcmalloc threaded threads thunderbolt
     truetype udisks uefi unzip upower usb x264 x265 xfs xv zeroconf
     zsh-completion -cups"
```

### Configuring the CPU Flags

USE flags that are available for our system. We could use the advantages of some CPU instruction sets using what are called CPU flags. You can read more on the [Gentoo wiki page](https://wiki.gentoo.org/wiki/CPU_FLAGS_X86).

To know what optimizations we can use for our CPU, we're going to use a tool called `cpuid2cpuflags`:

```shell
emerge --ask app-portage/cpuid2cpuflags
```

And then run it to get those values:

```shell
cpuid2cpuflags
```

You should see an output like:

```shell
CPU_FLAGS_X86: aes avx avx2 avx512f avx512dq avx512cd avx512bw avx512vl avx512vbmi f16c fma3 mmx mmxext pclmul popcnt rdrand sha sse sse2 sse3 sse4_1 sse4_2 ssse3
```

We have to add this into our `/etc/portage/make.conf`, but instead of colon (`:`), we will use this format. CPU flags could be added to `/etc/portage/package.use/` *instead* (with another format), but we will use our beloved :heart: `make.conf` :

```shell
nano /etc/portage/make.conf
```

And we add our CPU_FLAGS_X86 somewhere in the file.

```shell
CPU_FLAGS_X86="aes avx avx2 avx512f avx512dq avx512cd avx512bw avx512vl avx512vbmi f16c fma3 mmx mmxext pclmul popcnt rdrand sha sse sse2 sse3 sse4_1 sse4_2 ssse3"
```

### Re-compile and update @world

After setting up the USE and CPU flags, we're ready to re-compile and update all packages that we have installed in our base system before moving forward:

```shell
emerge --ask --verbose --update --deep --newuse @world
```

## Configuring the base system

In addition to Portage, some other options should be configured before we can take our Gentoo out of the oven :pizza:

### Allow licenses for packages

Each package in our Gentoo system defines what kind of license it uses. Setting what licenses we accept in our system is crucial to avoid problems installing `free` or `binary redistributable` packages. Also, to get installed automatically without asking us every time they get installed or updated.

You can read more about Gentoo licenses [here](https://wiki.gentoo.org/wiki/License_groups#Existing_license_groups).

Licenses acceptance is set in our `/etc/portage/make.conf` in the variable `ACCEPT_LICENSE`, like:

```shell
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
```

### Timezone

Set our time zone. We can use the tool `tzselect` to interactively select our country, and it will tell us what the value of `/etc/timezone` file should be.

```shell
tzselect
```

Take the last line from the output and add it to the timezone file like:

```shell
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

### Configure locales

Locales are a set of information that most programs use to determine the country and language of your system. Most users would use one or a few locales instead of all. This could save us compiling time, to avoid compiling all languages available for each package, and space on the filesystem by not installing man pages that we can't understand :)

Edit the `/etc/locale.gen` and uncomment the locale you are interested. Make sure you prioritize `UTF-8` locales because at least one is needed for the build system to work correctly.

```shell
nano -w /etc/locale.gen
```

Now, execute `locale-gen` to generate all locales specified in the previous file, and write them to the locale-archive `/usr/lib/locale/locale-archive`.

```shell
locale-gen
```

The more locales you select, the slower this process will be.

To check if the locales you've selected are active, run the following command and see if they are on the list. No worries if there're some locales you didn't choose, that's part of the system.

```shell
locale -a
```

Once the locale is available, let's ensure that it is the default one used in our system. Again we use `eselect` for this, now with the locale module:

```shell
eselect locale list
```

You should see something like:

```shell
Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   POSIX
  [4]   en_US.utf8
  [5]   C.UTF8 *
  [ ]   (free form)
```

Pick what locale you would activate and then run the following command with the number that you want to select:

```shell
eselect locale set 4
```

Now reload the environment:

```shell
env-update && source /etc/profile && export PS1="(chroot) $PS1"
```

### Configuring Linux kernel

While Portage is the core of Gentoo Linux system, the Linux kernel is the operating system's core. Therefore, this last offers an interface for programs to access the hardware via the modules, aka drivers.

The first step is installing the kernel source code. Gentoo's recommendation is, of course, the kernel package called `sys-kernel/gentoo-sources`. It is maintained by the Gentoo team and patched to fix security vulnerabilities and functionality problems and improve compatibility with rare system architectures. But, as always, there're many other options available.

To get a complete list of kernel sources with short descriptions can be found by searching with emerge:

```shell
emerge --search sources
```

#### Installing external firmware

Linux kernel is an open-source project, but some hardware manufacturers don't like the idea of open-source the code of its firmware (or modules), releasing a compiled binary version of them. These binaries can't be included in the Linux kernel repository, and they are packed in a package called `sys-kernel/linux-firmware`.

So, before compiling the kernel, it is essential to know that some devices require additional firmware to make the system run properly. Second, these are more common than we think, from network adapters, wireless cards, graphic cards, and CPUs.

For the most common firmware requirements, we will install the `sys-kernel/linux-firmware`:

```shell
emerge --ask sys-kernel/linux-firmware
```

Additionally, check the [Microocode page](https://wiki.gentoo.org/wiki/Microcode) in the Gentoo wiki to see if you need any additional firmware to make your CPU work properly. For example, intel CPUs are usually required to install the Intel microcode package `sys-firmware/intel-microcode`.

#### Installing the sources

We're not going to get "too crazy" and install Zen kernel (my favorite). Instead, we will install the default Gentoo kernel:

```shell
emerge --ask sys-kernel/gentoo-sources
```

Once installed, we have to select what kernel we want our system to use by using our magic wand `eselect` :zap:

```shell
eselect kernel list
```

You will probably see only one kernel on the list:

```shell
Available kernel symlink targets:
  [1]   linux-5.15.59-gentoo
```

So we go with that:

```shell
eselect kernel set 1
```

What this actually does is create a symlink `/usr/src/linux` to point to `/usr/src/linux-5.15.59-gentoo`

From this point, only two things are missing: configure and compile it. There are two ways to do that:

1. Manual configuration and automated build
2. Semi-automated by using `genkernel`
3. Fully automated via `distribution kernels` (not covered in this guide. Follow the official documentation [here](https://wiki.gentoo.org/wiki/Project:Distribution_Kernel))

##### Manual set up

The manual process it's intimidating for beginners because you can break your system if you miss specific options. In addition, the process involves a lot of reading and digging.

As much as I would love to spend time explaining every single option of the kernel, doing so will be time-consuming and at least double the size of this guide :bangbang:

So, if you feel brave, I will give you some hints:

1. Install `sys-apps/pciutils`. This package contains a tool called `lspci` that you will use to identify what PCI hardware you have in your system. You will have to enable all drivers on the kernel.

    ```shell
    emerge --ask sys-apps/pciutils
    ```

2. Configure the kernel by running `make menuconfig` inside `/usr/src/linux`:

    ```shell
    cd /usr/src/linux
    make menuconfig
    ```

3. After the required options are activated either as a module or part of the kernel binary, compile the kernel and the modules, and install them by:

    ```shell
    make && make modules_install
    make install
    ```

4. Install `sys-kernel/dracut`. This package will help you create an `initrmfs` (Init Ram Filesystem) with the required tools to make the kernel bootable:

    ```shell
    emerge --ask sys-kernel/dracut
    ```

5. Run `dracut` to generate an `initramfs` for our kernel version:

    ```shell
    dracut --kver=5.15.59-gentoo
    ```

##### Semi-automatic set up using genkernel

For those who prefer a more pleasant initial experience, I'll explain how to use `genkernel`.

What `genkernel` really does is configure a generic kernel that works with most of the hardware. Like the LiveCD that we're using does.

1. Install `genkernel`:

    ```shell
    emerge --ask sys-kernel/genkernel
    ```

2. Genkernel needs the `/boot` entry in the `/etc/fstab` file. So go to the [fstab](#fstab) section now and continue with point 2 when you're done.

3. Edit `/etc/genkernel.conf`:

    ```shell
    nano -w /etc/genkernel.conf
    ```

    Ensure that `LVM` and `LUKS` are set to `yes`; otherwise, the system will not boot. Leave the rest of the options as they are:

    ```shell
    # Add LVM support
    LVM="yes"

    # Add LUKS support
    LUKS="yes"
    ```

4. Once `genkernel` is configured, then run to generate the kernel binary:

    ```shell
    genkernel all
    ```

#### Configuring the modules

If we need to auto-load a kernel module each time to system boots, we should specify it in `/etc/conf.d/modules` file.

You can list your available modules with:

```shell
find /lib/modules/<kernel version>/ -type f -iname '*.o' -or -iname '*.ko' | less
```

### LVM Configuration

Install lvm2 tools if it is not yet installed:

```shell
emerge --ask sys-fs/lvm2
```

Then edit the package configurations:

```shell
nano -w /etc/lvm/lvm.conf
```

```shell
use_lvmetad = 1
issue_discards = 1
volume_list = ["vg0"] # Our VG volume name, check with vgdisplay
```

### Fstab

Before editing `fstab` we need to know which UUID are using our devices inside and outside `lvm` and `luks` volumes:

```shell
blkid /dev/mapper/vg0-root | awk '{print $2}' | sed 's/"//g'
UUID="576e229c-cf68-4010-8d85-ff8149158416"
blkid /dev/mapper/vg0-home | awk '{print $2}' | sed 's/"//g'
UUID="95fa5807-ea57-4cf5-b717-74f4aba190e2"
```

Then edit `/etc/fstab`:

```shell
nano -w /etc/fstab
```

```shell
/dev/sda1                                       /boot   vfat    noatime                                         1 2
UUID="576e229c-cf68-4010-8d85-ff8149158416"     /       ext4    discard,noatime,commit=600,errors=remount-ro    0 1
UUID="95fa5807-ea57-4cf5-b717-74f4aba190e2"     /home   ext4    discard,noatime,commit=600                      0 0
tmpfs                                           /var/tmp tmpfs  nodev,nosuid                    0 0
tmpfs                                           /tmp    tmpfs   nodev,nosuid                    0 0
```

### Configuring crypttab

Warning!!! As we don't have encrypted partitions other than root, which the `systemd` must mount before the whole system start, we don't need to set it up there, so our `crypttab` must be empty.

### Configuring mtab

In the past, some utilities wrote information (like mount options) into `/etc/mtab`; thus, it was supposed to be a regular file. Nowadays all software is supposed to avoid this problem. Still, before switching the file to become a symbolic link to /proc/self/mounts.

To create the symlink, run:

```shell
ln -sf /proc/self/mounts /etc/mtab
```

### Systemd boot (bootloader)

Systemd-boot is a simple UEFI boot manager which executes configured EFI images. The default entry is selected by an on-screen menu.

It is simple to configure, but can only start EFI executables, such as the Linux kernel EFISTUB, UEFI Shell, GRUB, and go on.

Before installing the EFI binaries we need to check if EFI variables are accessible:

```shell
emerge --ask sys-libs/efivar
efivar -l
```

Verify that you have mounted /boot:

```shell
mount | grep boot
```

Once got it mounted then install the systemd-boot binaries:

```shell
bootctl --path=/boot install
```

This command will copy the systemd-boot binary to your EFI System Partition ($esp/EFI/systemd/systemd-bootx64.efi and $esp/EFI/Boot/BOOTX64.EFI - both of which are identical - on x64 systems) and add systemd-boot itself as the default EFI application (default boot entry) loaded by the EFI Boot Manager.

Every time there's a new version of the systemd you should copy the new binaries to that System Partition by running:

```shell
bootctl --path=/boot update
```

### Add bootloader entries

Add one entry into bootloader with this options:

```shell
nano -w /boot/loader/entries/gentoo.conf
```

```shell
title    Gentoo Linux
efi      /kernel-genkernel-x86_64-4.4.6-gentoo
options  initrd=/initramfs-genkernel-x86_64-4.4.6-gentoo crypt_root=/dev/sda2 root=/dev/mapper/vg0-root root_trim=yes init=/usr/lib/systemd/systemd ro dolvm
```

Edit default loader:

```shell
nano -w /boot/loader/loader.conf
```

```shell
default gentoo
timeout 3
```

### Efibootmgr

Efibootmgr is not a bootloader itself it's a tool that interacts with the EFI firmware of the system, which itself is acting as a boot loader. With the efibootmgr application, boot entries can be created, reshuffled and updated.

First we need to install the package:

```shell
emerge --ask sys-boot/efibootmgr
```

To list the current boot entries:

```shell
efibootmgr -v
```

```shell
BootCurrent: 0003
Timeout: 1 seconds
BootOrder: 0003
Boot0000* Linux Boot Manager  HD(1,GPT,3eb8effe-8e1d-4670-987c-9b49b5f605b2,0x800,0x1ff801)/File(\EFI\systemd\systemd-bootx64.efi)
Boot0001* gentoo  HD(1,GPT,02f231b8-8f9a-471c-b3a9-dc7edb1bd70e,0x800,0xee000)/File(\EFI\gentoo\grubx64.efi)
Boot0003* Gentoo Linux  PciRoot(0x0)/Pci(0x1f,0x2)/Sata(2,32768,0)/HD(1,GPT,73f682fe-e07b-4870-be82-d85077f8aaa2,0x800,0x100000)/File(\EFI\systemd\systemd-bootx64.efi)
```

I'm only Gentoo in my system so I don't really need anything but the Gentoo entry so I just delete everything with:

```shell
efibootmgr -b <entry_id> -B
```

Once everything is delete we can add our new systemd-boot loader entry:

```shell
efibootmgr -c -d /dev/sda -p 2 -L "Gentoo" -l "\efi\boot\bootx64.efi"
```

Perfect, we're almost ready to reboot.

## Rebooting into our new Gentoo systemd

### Enable lvm2

```shell
systemctl enable lvm2-lvmetad.service
```

### Change the root password

While in chroot we need to change the root password of our new system just before rebooting it.

```shell
passwd
```

### And reboot to your new Gentoo system

Don't worry you don't need to cross your fingers, everything should goes fine :smirk:

```shell
exit # To exit from chroot
sync # To sync filesystems
reboot
```

## Post-installation

### Setting the Hostname

When booted using systemd, a tool called hostnamectl exists for editing /etc/hostname and /etc/machine-info. So we don't need to edit the file manually simlpy run:

```shell
hostnamectl set-hostname <hostname>
```

### Configuring Network

Let's plug our new Gentoo system to the world!! :earth_africa:

We can choose between two options, to use systemd as network manager or standalone one, I prefer networkmanager but here are the two options:

### Using systemd-networkd

systemd-networkd is useful for simple configuration of wired network interfaces. As it's disabled by default we need to configure it by creating a \*.network file under /etc/systemd/network.

Here is an example for a simple ethernet DHCP configuration:

```shell
nano -w /etc/systemd/network/50-dhcp.network
```

```shell
[Match]
Name=en*
[Network]
DHCP=yes
```

And then tell systemd to manage and start that service:

```shell
systemctl enable systemd-networkd.service
systemctl start systemd-networkd.service
```

### Using NetworkManager

Often NetworkManager is used to configure network settings. I personally use nmtui because it's easy and has a ncurses client. Just install it:

```shell
emerge --ask networkmanager
```

And now simply run the following command and follow a guided configuration process through nmtui:

```shell
nmtui
```

### Setting locales

Yes, you're right, we set the locales before but once booted with systemd, the tool localectl is used to set locale and console or X11 keymaps. So, let's set it again to be sure that everything goes fine.

To change the system locale, run the following command:

```shell
localectl set-locale LANG=en_US.utf8
```

Change the virtual console keymap:

```shell
localectl set-keymap es
```

And finally, to set the X11 layout:

```shell
localectl set-x11-keymap es
```

### Setting time and date

Time and date can be set using the timedatectl utility. That will also allow users to set up synchronization without needing to rely on net-misc/ntp or other providers than systemd's own implementation.

To set the local time of the system clock directly:

```shell
timedatectl set-time "yyyy-MM-dd hh:mm:ss"
```

To set time zone:

```shell
timedatectl list-timezones
timedatectl set-timezone Europe/Madrid
```

Set systemd-timesyncd as a simple SNTP daemon. Systemd-timesyncd that only implements a client side, focusing only on querying time from one remote server. It should be more than appropriate for most installations.

```shell
timedatectl set-ntp true
```

To check the status of the daemon:

```shell
timedatectl status
```

When starting, systemd-timesyncd will read the configuration file from /etc/systemd/timesyncd.conf. To add time servers or change the provided ones, uncomment the relevant line and list their host name or IP separated by a space. I'm using [the NTP pool project](http://www.pool.ntp.org) for the main servers and the default Gentoo as fallback:

```shell
nano -w /etc/systemd/timesyncd.conf
```

```shell
[Time]
NTP=0.europe.pool.ntp.org 1.europe.pool.ntp.org 2.europe.pool.ntp.org 3.europe.pool.ntp.org
FallbackNTP=0.gentoo.pool.ntp.org 1.gentoo.pool.ntp.org 2.gentoo.pool.ntp.org 3.gentoo.pool.ntp.org
```

### File indexing

In order to index the file system to provide faster file location capabilities we will install sys-apps/mlocate:

```shell
emerge --ask sys-apps/mlocate
```

To keep databse update we need to run `updatedb` often.

### Filesystem tools

Additional to the tools for managing ext2, ext3, or ext4 filesystems (sys-fs/e2fsprogs) which are already installed as a part of the *@system* set I like to install other filesystem utilities like VFAT, XFS, NTFS and so on:

```shell
emerge --ask sys-fs/xfsprogs sys-fs/exfat-utils sys-fs/dosfstools sys-fs/ntfs3g
```

### Adding a user for daily use

Until now we have done everything as root but working as root on a Unix/Linux system is dangerous and should be avoided as much as possible. Therefore it is strongly recommended to add a user for day-to-day use.

The groups the user is member of define what activities the user can perform. The following table lists a number of important groups:

|  Group  |                                 Description                                 |
| :-----: | :-------------------------------------------------------------------------: |
|  audio  |                    Be able to access the audio devices.                     |
|  games  |                           Be able to play games.                            |
| portage |               Be able to access portage restricted resources.               |
|   usb   |                       Be able to access USB devices.                        |
|  video  | Be able to access video capturing hardware and doing hardware acceleration. |
|  wheel  |                             Be able to use su.                              |

Once you selected which groups would you like to add simply run:

```shell
useradd -m -G users,wheel,audio,video,audio,usb -s /bin/bash <username>
```

To set the user password run:

```shell
passwd <username>
```

### Exec as root with sudo

Sudo is a way that a regular user could run commands as root user. It's very useful to avoid using root password each time we need to run a command which needs root perms. To install run:

```shell
emerge --ask app-admin/sudo
```

Then edit config file with command:

```shell
visudo
```

There's lot of examples inside the config file so we should not have any problem to set it up.

### Removing installation tarballs

With the Gentoo installation finished and the system rebooted, if everything has gone well, we can now remove the downloaded stage3 tarball from the hard disk. Remember that they were downloaded to the / directory.

```shell
rm /stage3-*.tar.bz2*
```

### Power consumption optimization with Powertop

```shell
emerge --ask sys-power/powertop
```

The first step we need to do is to calibrate it, it's quite long process but the most important is to keep our system until the whole process is finished.

```shell
powertop --calibrate
```

To apply automatically optimal settings at boot we need to create a new Systemd unit:

```shell
nano -w /etc/systemd/system/powertop.service
```

```shell
[Unit]
Description=Powertop tunings

[Service]
Type=oneshot
ExecStart=/usr/sbin/powertop --auto-tune

[Install]
WantedBy=multi-user.target
```

Enable at each boot:

```shell
systemctl enable powertop.service
```

### GPU

Prior to install xorg we will configure our video card, mine is Intel Generation 6 so we need to add this line into make.conf:

```shell
nano -w /etc/portage/make.conf
```

```shell
VIDEO_CARDS="intel i965"
```

Before installing xorg we need to configure it again by adding custom graphic card information:

```shell
nano -w /etc/X11/xorg.conf.d/20-intel.conf
```

```shell
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
  Option      "AccelMethod"  "sna"
  Option      "DRI"          "3"
  Option      "Backlight"    "intel_backlight"
EndSection
```

### Input devices

We need to do the same with the input devices. I'm using a laptop so check if you need to add joystick, mouse, keyboard, ... :wink:

```shell
nano -w /etc/portage/make.conf
```

```shell
INPUT_DEVICES="evdev synaptics
```

### X server

At the time to write this guide wayland is available but I'ld like to use bspwm which only support xorg so I'll continue with xorg server installation and configuration:

```shell
emerge --ask xorg-server
```

### A bunch of useful stuff

```shell
emerge --ask app-admin/ccze app-arch/unp app-editors/vim app-eselect/eselect-awk app-misc/screen app-shells/gentoo-zsh-completions app-shells/gentoo-zsh-completions app-vim/colorschemes app-vim/eselect-syntax app-vim/genutils app-vim/ntp-syntax media-gfx/feh sys-process/htop x11-terms/rxvt-unicode
```

### Portage nice value

We're running portage with modified scheduling priority, to not impact whole system performance during compilation.

```shell
echo 'PORTAGE_NICENESS="15"' >> /etc/portage/make.conf
```

### Setting portage branches

Branch defines if portage use stable or testing packages. Every package in the portage tree has it stable and testing version. The ACCEPT_KEYWORDS variable defines what software branch to use on the system. It defaults to the stable software branch for the system's architecture, for instance amd64.

I recommend to stick with the stable branch. However, if stability is not that much important for you and/or want to help out Gentoo by submitting bug reports to [https://bugs.gentoo.org](https://bugs.gentoo.org), then the testing is your way.

There are two ways to approach the testing branch for packages:

1. System wide setting: to make or system set to testing branch.

>```shell
>nano -w /etc/portage/make.conf
>```
>```shell
>ACCEPT_KEYWORDS="~amd64"
>```

2. Per package setting: we can set testing branch only for particular packages (This is and example, set whatever you want).

>```shell
>nano -w /etc/portage/package.accept_keywords
>```
>```shell
>sys-kernel/gentoo-sources
>sys-power/powertop
>app-admin/pass
>```

### Masked and unmasked packages

Masking a package the way where Gentoo Developers block a package version from being auto installed. The reason why that package is masked is mentioned in the package.mask file (situated in /usr/portage/profiles/ by default). But if we still wants to use this package, then add the desired version (usually this will be the exact same line from the package.mask file in the profile) to the /etc/portage/package.unmask file (or in a file in that directory if it is a directory).

```shell
nano -w /etc/portage/package.unmask
```

```shell
>=app-emulation/docker-compose-1.5.2
>=app-emulation/docker-swarm-1.1.3
>=app-emulation/docker-1.7.1
```

It is also possible to ask Portage not to take a certain package or a specific version of a package into account. To do so, mask the package by adding an appropriate line to the /etc/portage/package.mask location (either in that file or in a file in this directory).

```shell
nano -w /etc/portage/package.mask
```

```shell
=sys-kernel/gentoo-sources-4.5.0
```

### Overlays

Overlays contain additional packages for your Gentoo system while the main repository contains all the software packages maintained by Gentoo developers, additional package trees are usually hosted by repositories. Users can add such additional repositories to the tree that are "laid over" the main tree - hence the name, overlays.

```shell
emerge --ask app-portage/layman
```

To list all available overlays simply run:

```shell
layman -L
```

To install an overlay run:

```shell
layman -a <overlay_name>
```

To keep your installed overlays up to date, run:

```shell
layman -S
```

#### Custom overlay

If we want to maintain a custom set of ebuilds we just only need to create a local overlay doing these few steps:

```shell
mkdir -p /usr/local/portage/{metadata,profiles}
echo '<overlay_name>' > /usr/local/portage/profiles/repo_name
echo 'masters = gentoo' > /usr/local/portage/metadata/layout.conf
chown -R portage:portage /usr/local/portage
```

Next, tell portage about the overlay:

```shell
mkdir -p /etc/portage/repos.conf
```

```shell
nano -w /etc/portage/repos.conf/local.conf
```

```shell
[<overlay_name>]
location = /usr/local/portage
auto-sync = no
```

### Virtualization (How to use Qemu & kvm)
Virtualization is widely use nowadays. Everybody wants to test some environments, apps, or simply have a virtualized machine with other operating systems. In order to do that we should use any virtualization platform available for desktop use such as virtualbox, kvm/qemu, xen or vmware. Among all these options I choose kvm/qemu because its performance and simplicity. So here we are!

First we need to check if our hardware support virtualization:

```shell
grep --color -E "vmx|svm" /proc/cpuinfo
```

We could also check if kvm device is available in our */dev* directory:

```shell
ls /dev/kvm
```

If both are available we can continue installing kvm, otherwise you shouldn't use virtualization on your machine.

#### Kernel options

Before continue building kvm with should check if we have some kernel options enabled as module or build-in.

```shell
[*] Virtualization  --->
    <*>   Kernel-based Virtual Machine (KVM) support
    <*>   KVM for Intel processors support <-- if we use Intel cpu, otherwise disable it
    <*>   KVM for AMD processors support <-- if we use AMD cpu, otherwise disable it
    <*>   Host kernel accelerator for virtio net
Device Drivers  --->
   [*] Network device support  --->
      [*]   Network core driver support
      <*>   Universal TUN/TAP device driver support
   [*] Networking support  --->
      Networking options  --->
         <*> The IPv6 protocol
         <*> 802.1d Ethernet Bridging
   File systems  --->
      <*> The Extended 4 (ext4) filesystem
      [*]   Ext4 Security Labels
```

Once we have kernel compiled and running with new options we can continue by installing package:

```shell
emerge --askv app-emulation/qemu
```

In order to run kvm as normal user and not as root we should add our user account to the *kvm* group:

```shell
gpasswd -a <username> kvm
```

Finally, to be sure that libvirt daemon is running all the time we should enable with systemd:

```shell
systemctl enable libvirtd.service
```

### Speed up the system with prelink

What is Prelink and how can it help me? I'm sure that most of you are asking this question right now well, most applications we have installed in our system use shared libraries. Every time a program call this libraries they need to be loaded into memory. As more libraries program needs as more time it takes to resolve all symbol references. So prelink simply "maps" this symbol references and makes applications run faster. Of course this is a summary of what it does, but it's enough for us.

The only thing we need to do is to prelink binaries every time we upgrade or install any new program o library. But don't worry, portage will automatically prelink our system for us each time we install a package if we have prelink installed in our system. That's great!!

So we will simply install prelink:

```shell
emerge --askv prelink
```

Then configure the package by running env-update and then editing config file:

```shell
env-update
```

Unfortunately we can't prelink files that were compiled by old versions of binutils. Be default prelink define a number of libraries and directories which as blacklisted to avoid prelinking them. We can add or remove directories in prelink config files:

```shell
nano /etc/prelink.conf
nano /etc/prelink.conf.d/*
```

Finally we will prelink our system with:

```shell
prelink -amR
```

Which is:

- **a**: prelink all binary files.
- **m**: conserve virtual memory space. If we have a lot of binaries to prelink it takes a lot of space during the process, this parameter will ensure that we not run out of memory.
- **R**: randomize the address ordering to enhance security against buffer overflows.

## Last notes

Although there's a lot of work to do, I stop this guide at that point which I think that is far from base system installation now you'd walk you path little padawan :smile:

Hope you enjoyed!
