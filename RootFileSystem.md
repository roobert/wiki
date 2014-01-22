# Root File System

[[PageOutline(0-4,contents,inline)]]

## References
Filesystem Hierarchy Standard: http://www.pathname.com/fhs/pub/fhs-2.3.html#BOOTSTATICFILESOFTHEBOOTLOADER

See also: DiskManagement

## Creating a Root File System from the LFS LiveCD

### Overview

The reason for using the LFS LiveCD for creating a root file system is as follows:

 * The LiveCD comes with a stable set of packages that will always compile to produce the same root file system.
 * A fairly minimal set of software is included
 * The JHALFS script provides a way to automate builds
 * Modular design allows for easy extension and customization

### Step 1. Download, Burn and Boot the LiveCD

Mirrors: http://www.linuxfromscratch.org/livecd/download.html


```
wget http://kerrek.linuxfromscratch.org/pub/lfs-livecd/lfslivecd-x86-6.3-r2160.iso
cdrecord -scanbus
cdrecord dev=?,?,? lfslivecd-x86-6.3-r2160.iso
```

Whilst the CD is booting the following questions will be asked:

 * Timezone
 * Clock localtime or GMT?
 * Locale

TODO: Link to livecd-customization project.

### Step 2. Build Root File System

Mount a temporary disk and start jhalfs build

```
mkdir /mnt/build_dir
mount /dev/sdc3 /mnt/build_dir
chown jhalfs:jhalfs -R /mnt/build_dir
su jhalfs
cd ~/jhalfs-<version>
make
# dont change any settings, exit and save
# type 'yes' when prompted.
```

### Step 3. Archiving the Root File System for Later Use

Compress the root file system and archive onto another disk

```
cd /mnt/build_dir
tar zcpf /mnt/other_disk/jhalfs-x86-6.3-r2160-rootfs.tgz *
```

Write archive to cdrom

```
...
```

### Step 4. Install the Root File System

Extract root file system to new root file system partition. E.g. where <target> = `/mnt/disk0`

```
tar xpzf jhalfs-x86-6.3-r2160-rootfs.tgz -C <target>
```

### Step 5. System Configuration

For the system to be in a usable state upon first boot, it is advisable (but not always necessary) to configure  all or some of the following:

 * root password
 * fstab
 * hostname
 * grub menu.lst

#### Set `root` Password

Set a root password

```
passwd
# or
echo "root:newpassword" | chpasswd
```

#### Configure `/etc/fstab`

Change the first entry in `/etc/fstab` file for mount-point `/`. Type should be the file system type used when formatting the partition. File system should be the file system device used when formatting the partition. See DiskManagement for further information.

```
# EeePC example. ext2 and noatime are used because /dev/sdc1 is solid state.
/dev/sda1 / ext2 noatime,defaults 1 1
```

#### Set the Host Name

Configure the hostname

```
echo "HOSTNAME=hostname" > /etc/sysconfig/network
```

### Step 6. Non-Essential System Configuration

Remove cruft left over from jhalfs/rootfs build system

```
rm -rf jhalfs/ sources/ tools/
```

skeleton and root bash configuration

```
cat >> /etc/skel/.bashrc << - EOF

for DIR in `ls -d /opt/*/*`; do
        CHK_IF_BIN_DIR=`basename $DIR`
        if [ "$CHK_IF_BIN_DIR" ## 'sbin' ] || [ "$CHK_IF_BIN_DIR" 'bin' ]; then
                PATH="$PATH:$DIR"
        fi
done

PS1="\h\\$ "

export PS1 PATH
unset HISTFILE
EOF
ln -s /etc/skel/.bashrc /etc/skel/.bash_profile
cp -d /etc/skel/.bash* /root/
```

link `/bin/find` for compatibility with pkgsrc

```
ln -s /bin/find /usr/bin/find
```

Enable extra 6 VTs in `/etc/inittab`

```
7:2345:respawn:/sbin/agetty tty7 9600
8:2345:respawn:/sbin/agetty tty8 9600
9:2345:respawn:/sbin/agetty tty9 9600
10:2345:respawn:/sbin/agetty tty10 9600
11:2345:respawn:/sbin/agetty tty11 9600
12:2345:respawn:/sbin/agetty tty12 9600
```

Configure resolv.conf

```
echo "nameserver 4.2.2.2" > /etc/resolv.conf
```

Remove annoying colours, brackets, and unecessary messages boot messages during `init`, modify `/etc/rc.d/init.d/functions`:

```
...
#INFO="\\033[1;36m"           # Information is light cyan
INFO="\\033[0;39m"           # Information is standard console grey
...
#       ${ECHO} -n -e "${CURS_UP}${SET_COL}${BRACKET}[${SUCCESS}  OK  ${BRACKET}]"
#       ${ECHO} -e "${NORMAL}"
...
#        ${ECHO} -n -e "${CURS_UP}${SET_COL}${BRACKET}[${FAILURE} FAIL ${BRACKET}]"
        ${ECHO} -n -e "${CURS_UP}${SET_COL}${BRACKET} ${FAILURE} FAIL ${BRACKET} "
...
#        ${ECHO} -n -e "${CURS_UP}${SET_COL}${BRACKET}[${WARNING} WARN ${BRACKET}]"
        ${ECHO} -n -e "${CURS_UP}${SET_COL}${BRACKET} ${WARNING} WARN ${BRACKET} "
...
```

Install net-tools, this package is not in pkgsrc and not included in the lfs-livecd generated root filesystem. It includes `ifconfig`, `arp`, `netstat`, `hostname` and other tools for controlling the network subsystem for the Linux kernel.

```
cd ~/src
wget http://www.tazenda.demon.co.uk/phil/net-tools/net-tools-1.60.tar.bz2
tar jxf net-tools-1.60.tar.bz2
cd net-tools-1.60
sed -i 's/\# BASEDIR # \/mnt/BASEDIR \/opt\/net-tools/g' Makefile
cd lib/
wget "http://rw.dioptre.org/trac/rd/attachment/wiki/RootFileSystem/inet_sr.c.patch?format=raw" -O inet_sr.c.patch
patch -p0 < inet_sr.c.patch
cd ../
wget "http://rw.dioptre.org/trac/rd/attachment/wiki/RootFileSystem/hostname.c.patch?format=raw" -O hostname.c.patch
patch -p0 < hostname.c.patch
wget "http://rw.dioptre.org/trac/rd/attachment/wiki/RootFileSystem/config.h?format=raw" -O config.h
touch config.h
make
make install

```

Which

```
wget http://ftp.gnu.org/gnu/which/which-2.19.tar.gz
tar zxf which-2.19.tar.gz
cd which-2.19
./configure --prefix=/opt/which
make install
```

dhcpcd

```
wget http://www.phystech.com/ftp/dhcpcd-1.3.22-pl4.tar.gz
tar xf dhcpcd-1.3.22-pl4.tar.gz
cd dhcpcd-1.3.22-pl4
./configure --prefix=/opt/dhcpcd --mandir=/opt/dhcpcd/man
make install
```

whois

```
wget http://www.phystech.com/ftp/whois-1.5.tar.gz
tar xf whois-1.5.tar.gz
cd whois-1.5
make
mkdir -p /opt/whois/{man,bin}
install -c -m 0555 whois /opt/whois/bin
install -c -m 0444 whois.1 /opt/whois/man
```

traceroute, although `mtr` is also nice

```
wget http://surfnet.dl.sourceforge.net/sourceforge/traceroute/traceroute-2.0.11.tar.gz
tar xf traceroute-2.0.11.tar.gz
cd traceroute-2.0.11
sed -i 's/prefix # \/usr\/local/prefix \/opt\/traceroute/' Make.rules
make install
```

Install `ssmtp` as sendmail replacement, edit `/home/pkgadmin/pkgsrc-$PKGSRC_VERSION/etc/ssmtp/ssmtp.conf`

```
Root=user@gmail.com
Hostname=user@gmail.com
AuthUser=user@gmail.com
AuthPass=pass
FromLineOverride=YES
mailhub=smtp.gmail.com:587
UseSTARTTLS=YES

```

```
ln -s /home/pkgadmin/pkgsrc-$PKGSRC_VERSION/inst/sbin/ssmtp /usr/sbin/sendmail
```

Install `GNU mailutils`

```
wget http://ftp.gnu.org/gnu/mailutils/mailutils-2.0.tar.gz
tar zxf mailutils-2.0.tar.gz
cd mailutils-2.0
./configure --prefix=/opt/mailutils
make install

```

`mdadm` for RAID management

```
wget http://www.kernel.org/pub/linux/utils/raid/mdadm/mdadm-2.6.9.tar.gz
sed -e 's/DESTDIR # /DESTDIR \/opt\/mdadm/' -i Makefile
tar zxf mdadm-2.6.9.tar.gz
cd mdadm-2.6.9
make
make install

```


```
ln -s ../init.d/mdadm /etc/rc.d/rc0.d/K76mdadm
ln -s ../init.d/mdadm /etc/rc.d/rc5.d/S26mdadm
ln -s ../init.d/mdadm /etc/rc.d/rc6.d/K76mdadm
```


```
#!/bin/sh

. /etc/sysconfig/rc
. ${rc_functions}

case "${1}" in
        start)
                boot_mesg "Starting mdadm monitoring..."
                /opt/mdadm/sbin/mdadm --monitor --mail user@gmail.com --delay 1800 /dev/md0 --daemonise --pid-file=/var/run/mdadm-md0.pid
                /opt/mdadm/sbin/mdadm --monitor --mail user@gmail.com --delay 1800 /dev/md0 --daemonise --pid-file=/var/run/mdadm-md1.pid

                ;;

        stop)
                boot_mesg "Stopping mdadm monitoring..."
                killproc -p /var/run/mdadm-md0.pid /opt/mdadm/sbin/mdadm TERM
                killproc -p /var/run/mdadm-md1.pid /opt/mdadm/sbin/mdadm TERM
                ;;

        restart)
                ${0} stop
                sleep 1
                ${0} start
                ;;

        *)
                echo "Usage: ${0} {start|stop|restart}"
                exit 1
                ;;
esac

```



`smartmontools` SMART harddisk monitoring software

```
wget http://kent.dl.sourceforge.net/sourceforge/smartmontools/smartmontools-5.38.tar.gz
tar zxvf smartmontools-5.38.tar.gz
cd smartmontools-5.38
./configure --prefix=/opt/smartmontools
make install
```


```
#!/bin/sh

. /etc/sysconfig/rc
. ${rc_functions}

case "${1}" in
        start)
                boot_mesg "Starting smartd..."
                /opt/smartmontools/sbin/smartd -p /var/log/smartd.pid
                ;;

        stop)
                boot_mesg "Stopping smartd..."
                killproc -p /var/log/smartd.pid /opt/smartmontools/sbin/smartd TERM
                ;;

        restart)
                ${0} stop
                sleep 1
                ${0} start
                ;;

        *)
                echo "Usage: ${0} {start|stop|restart}"
                exit 1
                ;;
esac

```


```
ln -s ../init.d/smartd /etc/rc.d/rc0.d/K75smartd
ln -s ../init.d/smartd /etc/rc.d/rc5.d/S27smartd
ln -s ../init.d/smartd /etc/rc.d/rc6.d/K75smartd
```

`/opt/smartmontools/etc/smartd.conf`

```
/dev/sda -d sat -a -s (S/../.././4|L/../../1/1|C/../.././6) -m user@gmail.com
/dev/sdb -d sat -a -s (S/../.././5|L/../../1/1|C/../.././7) -m user@gmail.com

```

dcron

```
wget http://www.ibiblio.org/pub/Linux/system/daemons/cron/dcron-2.3.3.tar.gz
# find dcron-2.3.3.patch
tar zxvf dcron-2.3.3.tar.gz
cd dcron-2.3.3
cat ../dcron-2.3.3.patch | patch -p1
# modify makefile and swap usr for opt/dcron
make
make install
mkdir /var/spool/cron/crontabs
```

dcron init script

```
#!/bin/sh

. /etc/sysconfig/rc
. ${rc_functions}

case "${1}" in
        start)
                boot_mesg "Starting..."
                loadproc -p /var/run/crond.pid /opt/dcron/sbin/crond -l8 >> /var/log/cron.log 2>&1
                pidof crond > /var/run/crond.pid

                ;;

        stop)
                boot_mesg "Stopping..."
                killproc -p /var/run/crond.pid /opt/dcron/sbin/crond TERM
                ;;

        restart)
                ${0} stop
                sleep 1
                ${0} start
                ;;

        *)
                echo "Usage: ${0} {start|stop|restart}"
                exit 1
                ;;
esac

```


```
ln -s ../init.d/dcron /etc/rc.d/rc3.d/S22dcron
ln -s ../init.d/dcron /etc/rc.d/rc0.d/K77dcron
ln -s ../init.d/dcron /etc/rc.d/rc6.d/K77dcron
```

See the [wiki:OpenVZ OpenVZ page] for [wiki:OpenVZ OpenVZ] software and configuration

----

## Configure and Install a Kernel

See the ConfigKernel page

## Configure and Install a Boot Loader

See the ConfigGrub page

## Making the Root File System Read Only

Aims:

 * Any files that are modified should be first moved to a read/write partition and linked to from their original location. This makes it easy to track which files changes have been made to. There are, however, exceptions such as `/etc/fstab` which must be edited in-place since its needed before the read/write partition has been mounted and all the symlinked targets are in place.

 * Any directorys that are written to should be moved and linked to, these include:
  * `/var`
  * `/tmp`
  * `/usr/src`
  * `/opt`
  * `/lib/modules`
  * `/boot`
  * `/home`

 Alternatively, `/tmp` can be a mounted ramdisk and `/var` can be extracted upon each boot to a ramdisk for each reboot. Note that the var tarball can create symlinks for directorys such as `/var/log` if its necessary to keep certain directorys functionality.

First, mount a writable partition for /home and other writable files to be stored on

```
# This assumes that a livecd has been booted, otherwise ignore RO_DISK and set RO_MNT as /
RO_DISK="/dev/hda1"
RW_DISK="/dev/hdb2"

RO_MNT="/mnt/hdb2"
RW_MNT="/mnt/hda1"

mount $RO_DISK $RO_MNT
mount $RW_DISK $RW_MNT
```

Re-link files that should be written to and/or modified

 NOTE: add note on 'risky system' where new var is extracted on each boot


```
# Incase rootfs isn't located at the base of the disk
RO_ROOTFS="$RO_MNT/"
RW_ROOTFS="$RW_MNT/"

ln -s $RW_ROOTFS/opt		$RO_ROOTFS/opt

mv $RO_ROOTFS/var		$RW_ROOTFS/
ln -s $RW_ROOTFS/var		$RO_ROOTFS/var


mv $RO_ROOTFS/home		$RW_ROOTFS/
ln -s $RW_ROOTFS/home		$RO_ROOTFS/home

mkdir $RW_ROOTFS/usr
mv $RO_ROOTFS/usr/src		$RW_ROOTFS/usr/
ln -s $RW_ROOTFS/usr/src	$RO_ROOTFS/usr/src

# Moving the kernel modules and kernel is optional, it makes sense so the kernel can
# be updated yet kept separate from the root file system
mkdir $RW_ROOTFS/lib
ln -s $RW_ROOTFS/lib/modules	$RO_ROOTFS/lib/modules

mv $RO_ROOTFS/boot		$RW_ROOTFS/
ln -s $RW_ROOTFS/boot		$RO_ROOTFS/boot

# mkdir $RW_ROOTFS/etc
# mv $RO_ROOTFS/etc/fstab		$RW_ROOTFS/etc/
# ln -s $RW_ROOTFS/etc/fstab		$RO_ROOTFS/etc/fstab
```


Notes:

```


set password, hostname
 * /etc/group
 * /etc/passwd
 * /etc/shadow
 * /etc/fstab
 * /etc/mtab
 * /etc/hosts
 * /etc/resolv.conf
 * /etc/sysconfig/network
 
modify /etc/rc.d/init.d/mountfs and remove lines that remount root file system read-write (first three lines of 'start')

# /dev is already mounted from tmpfs so should be unaffected

# fix /etc/mtab so remounting remaining filesystems works

# fastboot and forcefsck? /etc/rc.d/init.d/mountfs, remove -f, why is -f necessary?
# -f is used so they fail silently if files dont exist, use test -f && instead.


# Re-order 'recording existing mounts in mtab' and 'mount remaining file systems'
# then comment out > /etc/mtab 

# remove shm mounting from /etc/fstab - we dont need it?

# remove STACK LEVEL? warnings.. from default kernel

#add AMD network driver

```

