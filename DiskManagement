# Disk Management

[[PageOutline(0-4,contents,inline)]]

## Disk Settings

 * hdparm ...

## Disk Testing

 * hdparm -t ...

## Disk Monitoring

 * mdadm --monitor
 * smartmontools

```
wget http://kent.dl.sourceforge.net/sourceforge/smartmontools/smartmontools-5.38.tar.gz
tar zxvf smartmontools-5.38.tar.gz
cd smartmontools-5.38
./configure --prefix=/opt/smartmontools
make install
```

 * remote monitoring
 * http://aisalen.wordpress.com/2007/07/09/raid-arrays-in-linux/

## Software RAID

### References

 * http://unthought.net/Software-RAID.HOWTO/Software-RAID.HOWTO.html
 * http://linux-raid.osdl.org/index.php/Main_Page
 * `mdadm(8)` manual page

### Description

 * Configure the kernel
 * Partition the disks 
 * Create the RAID1 array

 Note: The following messages may appear on shutdown:
  md: stopping all md devices[[br]]
  md: md0 still in use.
 These messages can safely be ignored since at this point the root filesystem has been mounted read-only and the buffers have been flushed.

### Kernel Configuration

This is an example of options needed to enable disk, disk controller and RAID1 support.
 * Device Drivers -> SCSI device support -> SCSI device support
 * Device Drivers -> SCSI device support -> SCSI disk support
 * Device Drivers -> SCSI device support -> SCSI low-level drivers -> Adaptec AIC7xxx Fast -> U160 support (New Driver)
 * Device Drivers -> Multi-device support (RAID and LVM) -> RAID-1 (mirroring) mode

`.config` options:

```
CONFIG_SCSI=y
CONFIG_BLK_DEV_SD=y
CONFIG_SCSI_AIC7XXX=y
CONFIG_SCSI_SPI_ATTRS=y
CONFIG_MD_RAID1=y
```

### Partition the Disks

It is possible to either partition the disks and RAID the resulting partitions into an array, e.g:

```
# /dev/md0 consists of:
# /dev/sda1
# /dev/sdb1
# /dev/sdc1
```

Or with newer kernels its possible to RAID the disk devices, then partition the resulting RAID device, e.g:


```
# /dev/md0 consists of:
# /dev/sda
# /dev/sdb
# /dev/sdc

# Partitions on RAID device:
# /dev/md/d0p1
# /dev/md/d0p2
```

This example uses the older method to keep things simple.

Make sure `mdadm` is installed and that the `md` driver is loaded or compiled into the kernel

```
# Check userspace tools exist
mdadm --version

# If /proc/mdstat exists, the md driver is working
cat /proc/mdstat

# Otherwise try and load the module
modprobe md
```

|| Mount point || RAID Device || Array Devices                   || Partition Size ||
|| /           || /dev/md0    || /dev/sda1, /dev/sdb1, /dev/sdc1 || 8gb            ||
|| swap        || /dev/md1    || /dev/sda2, /dev/sdb2, /dev/sdc2 || 512mb          ||

 * To partition the disks either `cfdisk` or `fdisk` can be used.
 * Create two partitions on each disk, one will be used as the root partition and the other will be swap.
 * Each partition that is part of the same array must be of equal size.
 * On each disk, mark the root partition as 'bootable'
 * On each disk change both partition types to 'fd' which is the 'Linux raid autodetect' type necessary for the array to be automatically re-created at boot.

```
cfdisk /dev/sda
cfdisk /dev/sdb
cfdisk /dev/sdc
```



 * persistant superblock for disk autodetection on boot: http://unthought.net/Software-RAID.HOWTO/Software-RAID.HOWTO-5.html#ss5.9, http://unthought.net/Software-RAID.HOWTO/Software-RAID.HOWTO.html#toc7.2

create `md` arrays:

```
# --auto means create /dev/md* devices if they don't exist.
mdadm --create --verbose /dev/md0 --auto=md --level=raid1 --raid-devices=2 /dev/sda1 /dev/sdb1 --spare-devices=1 /dev/sdc1
mdadm --create --verbose /dev/md1 --auto=md --level=raid1 --raid-devices=2 /dev/sda2 /dev/sdb2 --spare-devices=1 /dev/sdc2
```

Later, its possible to assemble an already created array from a list of devices.

```
mdadm --assemble /dev/md0 /dev/sda1 /dev/sdb1 /dev/sdc1 --auto=md
```

Now it is possible to format and use both /dev/md0 and /dev/md1 as if they were normal partitions.

## Disk Encryption

`dm-crypt` is used with the cryptsetup-luks frontend.
 
 * Create kernel
 * Create initrd
 * Create usb key or cdrom with kernel, initrd, cryptsetup and any gpg key
 * Write random data to partition, format disk
 * Create encrypted partition
 * Remove bootloader from drive

Note: if using the LFS LiveCD it doesn't come with most crypto modules. Download, extract, add crypto modules then install new modules. These will be lost after next reboot.

```
export KERNEL_VERSION=`uname -r`
wget http://www.eu.kernel.org/pub/linux/kernel/v2.6/linux-$KERNEL_VERSION.tar.gz
tar xf linux-$KERNEL_VERSION.tar.gz
cd linux-$KERNEL_VERSION
cp /usr/src/linux-$KERNEL_VERSION/.config .config
make oldconfig
make menuconfig
make modules
make modules_install
modprobe dm_crypt
modprobe md_mod
modprobe aes
modprobe sha512
modprobe sha256
modprobe sha1
```

Download binary or source from: http://code.google.com/p/cryptsetup/

```
wget http://cryptsetup.googlecode.com/files/cryptsetup-1.0.6.tar.bz2
tar jxf cryptsetup-1.0.6.tar.bz2
cd cryptsetup-1.0.6
./configure
make
make install
```

Write random data to disk [Todo: explain why] this will take some time.

```
shred -v /dev/md0 
```

Enable options in kernel for different ciphers.
 * Loop device + cryptoloop?
 * Device Drivers -> Multi-device support (RAID and LVM) -> Device mapper support
 * Device Drivers -> Multi-device support (RAID and LVM) -> Crypt target support
 * Cryptographic options -> SHA384 and SHA512 digest algorithms   
 * Cryptographic options -> Blowfish cipher algorithm    
 * Cryptographic options -> Twofish cipher algorithm
 * Cryptographic options -> Twofish cipher algorithms (i586)         
 * Cryptographic options -> AES cipher algorithms
 * Cryptographic options -> AES cipher algorithms (i586)
 * Cryptographic options -> Serpent cipher algorithm

### Format the Partitions

[TODO: Explain cipher+hash choice]


```
# -y = confirm password
#cryptsetup -y luksFormat /dev/md0
cryptsetup -y --cipher serpent-cbc-essiv:sha256 --key-size 256 luksFormat /dev/md0
cryptsetup -y luksFormat /dev/md1
cryptsetup luksOpen /dev/md0 rootfs
cryptsetup luksOpen /dev/md1 swapfs
cryptsetup status /dev/mapper/rootfs
cryptsetup status /dev/mapper/swapfs
# Warning! Do not use journaled file systems!
mkfs.ext2 /dev/mapper/rootfs
mkswap /dev/mapper/swapfs
```

### Creating Boot Media

#### Environment Setup

 * `bzImage` and optionally `.config` renamed to 'config' must be in `${BOOTCD_WORKDIR}/`

make a directory to create cd in

```
mkdir /mnt/disk0/bootcd
export BOOTCD_WORKDIR="/mnt/disk0/bootcd"
```


```
mkdir ${BOOTCD_WORKDIR}/src
mkdir ${BOOTCD_WORKDIR}/rootfs

```

#### Tool Installation


```
cd ${BOOTCD_WORKDIR}/src
wget http://cryptsetup.googlecode.com/files/cryptsetup-1.0.6.tar.bz2
tar jxf cryptsetup-1.0.6.tar.bz2
cd cryptsetup-1.0.6
./configure
make
make install
```


```
cd ${BOOTCD_WORKDIR}/src
# make sure system date is correct
date --set="Sat Mar 15 13:30:23 GMT 2008"
wget ftp://ftp.berlios.de/pub/cdrecord/cdrtools-2.01.tar.gz
tar zxf cdrtools-2.01.tar.gz
cd cdrtools-2.01

```

Edit DEFAULTS/Defaults.linux and change DEFINSUSR and DEFINSGRP from 'bin' to 'root'


```
make
make install

```

Optional: Install gpg for gpg-encrypted key file

```
cd ${BOOTCD_WORKDIR}/src
wget ftp://ftp.gnupg.org/gcrypt/gnupg/gnupg-1.4.9.tar.bz2
tar jxf gnupg-1.4.9.tar.bz2
cd gnupg-1.4.9
./configure
make
make install

```

Optional: shartools contains uuencode, which is used to make sure all charicters in key file are base64

```
cd ${BOOTCD_WORKDIR}/src
wget ftp://ftp.gnu.org/gnu/sharutils/REL-4.7/sharutils-4.7.tar.bz2
tar jxf sharutils-4.7.tar.bz2
cd sharutils-4.7
./configure
make
make install

```

Create `${BOOTCD_WORKDIR}/linuxrc` script  

```
#!/bin/sh

export PATH=/bin:/sbin

echo "Starting initrd's /linuxrc..."

# check KERNEL_ARGS and see if this break point has been specified
# $1 = breakpoint number (bp#) 
check_breakpoint() {
        for kernel_arg in ${KERNEL_ARGS}; do
                case ${kernel_arg} in
                        $1 )
                        /bin/sh
                        ;;
                 esac
        done
}

# Kernel command line parameters (depends on /proc being mounted)
mount -t proc proc /proc
KERNEL_ARGS=`cat /proc/cmdline`
CONTROL_DEV_MAJOR=`awk '/misc$/ {print $1}' /proc/devices`
CONTROL_DEV_MINOR=`awk '/device-mapper$/ {print $1}' /proc/misc`

# if debug is a kernel option, set -x
for kernel_arg in ${KERNEL_ARGS}; do
        case ${kernel_arg} in
                debug )
                set -x
                ;;
        esac
done

check_breakpoint bp0

# create device mapper control device
test -e /dev/mapper || mkdir /dev/mapper
test -e /dev/mapper/control && rm -f /dev/mapper/control
mknod /dev/mapper/control c ${CONTROL_DEV_MAJOR} ${CONTROL_DEV_MINOR}

check_breakpoint bp1

# Even though the md devices have been assembled the devices wont exist, restarting them will create them
# these will fail but enable the devices to be stopped
mknod /dev/md0 b 9 0
#mknod /dev/md1 b 9 1

ROOTDEV=/dev/md0
#SWAPDEV=/dev/md1

check_breakpoint bp2

sleep 1

if [ ! -e "$ROOTDEV" ]; then
        echo "Root device '$ROOTDEV' doesnt exist!"
        /bin/sh
fi

# Mount real root and change to it
false
while [ "$?" != "0" ]; do
        if [ -e "/rootfs.key" ]; then
                cryptsetup --key-file /rootfs.key luksOpen ${ROOTDEV} rootfs
        fi

        if [ -e "/rootfs.key.gpg" ]; then
                ORIGINAL_STTY_SETTINGS=`stty -g </dev/console`
                echo "Enter passphrase for /rootfs.key.gpg and press return to continue"
                stty -echo </dev/console
                read GPG_PASSWORD </dev/console
                stty ${ORIGINAL_STTY_SETTINGS} </dev/console
                echo "${GPG_PASSWORD}" | gpg --passphrase-fd 0 --no-tty --quiet --decrypt /rootfs.key.gpg 2> /dev/null | cryptsetup luksOpen ${ROOTDEV} rootfs -d -
        fi

        if [ ! -e "/dev/mapper/rootfs" ]; then
                cryptsetup -y luksOpen ${ROOTDEV} rootfs
        fi
done

check_breakpoint bp3

# Re-create swap partition on each boot with new, randomly generated key
#cryptsetup -d /dev/urandom create swapfs ${SWAPDEV}
#mkswap /dev/mapper/swapfs

if [ -e "/dev/mapper/rootfs" ]; then
        ROOTFS_MAJOR=`ls -l /dev/mapper/rootfs | awk '{ print $5 }' | tr -d '[:punct:]'`
        ROOTFS_MINOR=`ls -l /dev/mapper/rootfs | awk '{ print $6 }'`
        echo "/dev/mapper/rootfs has MAJOR: ${ROOTFS_MAJOR} and MINOR: ${ROOTFS_MINOR}"
fi

# Record swapfs and rootfs MAJORs and MINORs for use by init script
#if [ -e "/dev/mapper/swapfs" ]; then
#        SWAPFS_MAJOR=`ls -l /dev/mapper/swapfs | awk '{ print $5 }' | tr -d '[:punct:]'`
#        SWAPFS_MINOR=`ls -l /dev/mapper/swapfs | awk '{ print $6 }'`
#        echo "/dev/mapper/swapfs has MAJOR: ${SWAPFS_MAJOR} and MINOR: ${SWAPFS_MINOR}"
#fi

check_breakpoint bp4

# Mount unencrypted root file system - fsck options taken from /etc/rc.d/init.d/checkfs 
e2fsck -p -C -T /dev/mapper/rootfs
mount -t ext2 /dev/mapper/rootfs /rootfs

check_breakpoint bp5

/bin/umount /proc

# Pivot the root file system into place
cd /rootfs
test -d initrd || mkdir initrd
pivot_root . initrd

check_breakpoint bp6

# exec rootfs's chroot (not cd chroot!)
exec /usr/sbin/chroot . /bin/sh <<- EOF >/dev/console 2>&1
/usr/bin/test -f /dev/ram0 && /sbin/blockdev --flushbufs /dev/ram0
umount /initrd
rm -rf /initrd

if [ ! -e /lib/udev/devices/mapper/rootfs ]; then
     if [ -d /lib/udev/devices/ ]; then
          # system does use udev!
          mkdir -p /lib/udev/devices/mapper
          mknod rootfs c ${ROOTFS_MAJOR} ${ROOTFS_MINOR}
     fi
fi
exec /sbin/init
EOF

# creating /lib/udev/devices/mapper/rootfs c 253 0 in 'real' operating system

```

Create `${BOOTCD_WORKDIR/mkinitrd` script which creates 15mb `${BOOTCD_WORKDIR}/initrd` file

```
#!/bin/bash

export LINUXRC_SCRIPT="${BOOTCD_WORKDIR}/linuxrc"
# used if not found in $PATH
export CRYPTSETUP_BIN="{BOOTCD_WORKDIR}/cryptsetup"
export INITRD="${BOOTCD_WORKDIR}/initrd"
export INITRD_MNT="${BOOTCD_WORKDIR}/initrd_mnt"
export LOOP_DEV="/dev/loop2"

touch ${INITRD}

# Size of initrd, in this case, 15mb.
dd if=/dev/zero of=${INITRD} bs=1024k count=15

losetup ${LOOP_DEV} ${INITRD}
mke2fs ${LOOP_DEV}

mkdir ${INITRD_MNT}
mount ${LOOP_DEV} ${INITRD_MNT}

test -d ${INITRD_MNT} || mkdir -p ${INITRD_MNT}
mkdir ${INITRD_MNT}/{etc,dev,lib,bin,sbin,proc,rootfs,usr,usr/lib}
# fstab should be blank
touch ${INITRD_MNT}/etc/fstab

export BIN_LIST="cp sh stty ls cd df mkdir touch mv ln grep du cat awk mdadm mount umount mkdir chroot cryptsetup sleep mknod sed rm pivot_root blockdev init udevsettle udevtrigger udevd udevcontrol mkswap gpg"

for bin in ${BIN_LIST}; do

        # For easy debugging
        echo "----- ${bin} -----"

        # Binary with full path
        BIN_WPATH=`which ${bin}`

        # Test to see if binary exists
        if [ -f "${BIN_WPATH}" ]; then

                echo "path to ${bin} is \"${BIN_WPATH}\""

                # This has to be forced since the same libs are used by multiple binaries (!)
                for lib in `ldd $BIN_WPATH|awk '{ print $3 }'|grep -v \(0x`; do
                        cp -v ${lib} ${INITRD_MNT}/$lib
                done

                cp -v `which ${bin}` ${INITRD_MNT}/bin
        else
                echo "${bin} does not exist"
        fi
done

# Copy special libraries
cp -v /lib/ld-linux.so.2 ${INITRD_MNT}/lib

# Copy minimal set of device files
cp -a /dev/{console,null,tty,sd*,hd*} ${INITRD_MNT}/dev/

test -x `which cryptsetup` && CRYPTSETUP_BIN=`which cryptsetup`
test -x ${CRYPTSETUP_BIN} || { echo "FATAL: cryptsetup binary: \"${CRYPTSETUP}\" not found"; }
cp ${CRYPTSETUP_BIN} ${INITRD_MNT}/sbin

# necessary for cryptsetup
ln -s /bin/udevsettle ${INITRD_MNT}/sbin/udevsettle
ln -s /bin/udevcontrol ${INITRD_MNT}/sbin/udevcontrol
ln -s /bin/udevd ${INITRD_MNT}/sbin/udevd
ln -s /bin/udevtrigger ${INITRD_MNT}/sbin/udevtrigger
cp -Rv /etc/udev ${INITRD_MNT}/etc

# copy key file and gpg encrypted keyfile if either exists
test -f /root/rootfs.key && cp -v /root/rootfs.key ${INITRD_MNT}/
test -f /root/rootfs.key.gpg && cp -v /root/rootfs.key.gpg ${INITRD_MNT}/

test -f ${LINUXRC_SCRIPT} || { echo "FATAL: LINUXRC script: \"${LINUXRC_SCRIPT}\" not found"; }
cp ${LINUXRC_SCRIPT} ${INITRD_MNT}/

chmod +x ${INITRD_MNT}/linuxrc

umount ${INITRD_MNT}
rm -rf ${INITRD_MNT}
losetup -d ${LOOP_DEV}
```

Optionally generate a `/root/rootfs.key.gpg`. Write it to the livecd ramdisks `/root/` to avoid the key being recovered later. There should be a better way to do this that doesn't involve writing the unencrypted key to disk.

```
# make sure gpg and uuencode are in the PATH 
export PATH=$PATH:/usr/local/bin
head -c 45 /dev/random | uuencode -m - | head -n 2 | tail -n 1 > /root/rootfs.key
cryptsetup luksAddKey /dev/md0 /root/rootfs.key
cat /root/rootfs.key | gpg --symmetric -a > /root/rootfs.key.gpg
rm /root/rootfs.key
```

Run the script

```
sh mkinitrd
```

#### Creating a Bootable CD

Create iso root directory

```
mkdir ${BOOTCD_WORKDIR}/rootfs
```


```
cd ${BOOTCD_WORKDIR}/src
wget http://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-3.82.tar.gz
tar zxf syslinux-3.82.tar.gz
```


```
mkdir ${BOOTCD_WORKDIR}/rootfs/isolinux
cp syslinux-3.82/core/isolinux.bin ${BOOTCD_WORKDIR}/rootfs/isolinux/
cp syslinux-3.82/com32/menu/menu.c32 ${BOOTCD_WORKDIR}/rootfs/isolinux/
```


```
cp ${BOOTCD_WORKDIR}/initrd ${BOOTCD_WORKDIR}/rootfs/isolinux/
cp ${BOOTCD_WORKDIR}/bzImage ${BOOTCD_WORKDIR}/rootfs/isolinux/
cp ${BOOTCD_WORKDIR}/config ${BOOTCD_WORKDIR}/rootfs/isolinux/
```


```
cat >> ${BOOTCD_WORKDIR}/rootfs/isolinux/isolinux.cfg << EOF
DEFAULT Linux
PROMPT 1
TIMEOUT 50
LABEL Linux
KERNEL bzImage
# Without root=/dev/ram0 and (possibly) init=linuxrc, init will fail because /sbin/init won't get PID of 1!
# ide=nodma is for compact flash based systems
APPEND initrd=initrd init=linuxrc root=/dev/ram0 rw ide=nodma
EOF

```


```
/opt/schily/bin/mkisofs -o ${BOOTCD_WORKDIR}/boot.iso \
        -b isolinux/isolinux.bin \
        -no-emul-boot \
        -boot-load-size 4 \
        -boot-info-table \
	${BOOTCD_WORKDIR}/rootfs
```

Record cd

```
/opt/schily/bin/cdrecord dev=4,0,0 boot.iso
```

#### Creating a Bootable USB Stick

```
export USB_PARTITION="/dev/sdd1"
export USB_DEVICE="/dev/sdd"
export USB_MNT="/mnt/usb"
export KERNEL_ARCH="i386"
export KERNEL_SRC_DIR="/mnt/disk0/linux-2.6.22-ovz-raid1-serpent"
```

The first thing to do is to wipe its MBR, partition table and beginning of an existing filesystem.


```
dd if=/dev/zero of=${USB_DEVICE} bs=1024k count=5 conv=notrunc
```

Use fdisk(or cfdisk) to create a primary partition and set the boot flag, it should be big enough to hold the kernel, initramfs archive, keyfiles, etc..(if in doubt create a single partition of the whole drive).


```
cfdisk ${USB_DEVICE}
```

Create a partition with no space reserved for the `root` user

```
mke2fs -m0 ${USB_PARTITION}
```

Get mbr.bin from the `syslinux` package and install to usb device

```
wget http://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-3.62.tar.gz
tar xf syslinux-3.62.tar.gz
cat syslinux-3.62/mbr/mbr.bin > ${USB_DEVICE}
```

Mount the usb device and copy

```
test -d ${USB_PARTITION} || mkdir ${USB_MNT}
mount ${USB_PARTITION} ${USB_MNT}
```



```
cp -v ${KERNEL_SRC_DIR}/arch/${KERNEL_ARCH}/boot/bzImage ${USB_MNT}
cp -v ${KERNEL_SRC_DIR}/System.map ${USB_MNT}
cp -v ${KERNEL_SRC_DIR}/.config ${USB_MNT}/config
```


```
# copy this file to get a simple menu
cp syslinux-*/com32/menu/menu.c32 ${USB_MNT}
```

NOTE: "ide=nodma" option is for compact flash based systems.

```
extlinux --install ${USB_MNT}
cat >> ${USB_MNT}/extlinux.conf << EOF
DEFAULT Linux
TIMEOUT 0
PROMPT 0

LABEL Linux
        KERNEL bzImage
        APPEND initrd=initrd init=linuxrc root=/dev/ram0 rw ide=nodma
EOF
```


```
sync
umount ${USB_MNT}
```

## Changing the password ...

To add an additional password, so you can unlock your partition with a choice of different passwords (you can do this with the encrypted partition mounted, if you wish):

```
# cryptsetup luksAddKey /dev/sdc1
Enter any LUKS passphrase: (enter an existing password for this partition)
key slot 0 unlocked.
Enter new passphrase for key slot: (enter the extra password)
#
```

To delete an existing password (but don't delete the last one, your data will be lost forever, you will be warned if you try this), you need to know which slot the password is in. The first password goes in slot 0, any additional passwords go in slot 1, 2 etc. You can do this with the encrypted partition mounted, if you wish. So to delete the very first password you used, use:

```
cryptsetup luksDelKey /dev/sdc1 0
```

To change the password, first add an additional password then delete the original password.



# TODO?

 * Move to using modern initramfs which replaces initrd - no real benefit, complicates matters using busybox
 * Add gpg key to bootcd if its preferable to rely on physical security of the boot media than memory. (http://www.gentoo-wiki.com/SECURITY_System_Encryption_DM-Crypt_with_LUKS#Security_levels)
 * Discuss hash/crypto choice and other LUKS security features



