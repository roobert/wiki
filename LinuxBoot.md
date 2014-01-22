# Linux Boot

 * A Clean Linux Boot, Init, Console and Logging Configuration.
 * all logs should be silent
 * init should be silent
 * bootloader should be silent
 * logs should be properly organised
 * logs should rotate
 * init/dmesg log should be merged and timestamps removed (only for last boot)
 * previous init logs should be archived seperately to dmesg because dmesg doesnt 'end' on boot
 * 'previous' init and dmesg logs should be moved on next boot so they are complete, rather than still in use...

## Aims
 * silence grub but maintain functionality
 * silence kernel by patching source
 * dmesg logs
 * log init using http://sources.gentoo.org/viewcvs.py/gentoo-x86/app-admin/showconsole/showconsole-1.08.ebuild?view=markup
 * modify agetty to not print new line ftp://ftp.kernel.org/pub/linux/utils/util-linux/util-linux-2.12r.tar.gz
 * hide all logging with console= and 'quiet' boot option.
 * http://tips4all.in/linux/init.html

### Silencing `grub`

See ConfigGrub for how to patch `grub` to silence it and the relative `menu.lst` boot options to keep the kernel and operating system quiet.

### Silencing the Linux Kernel
When the kernel is booted it prints 'Uncompressing Linux... Ok, booting the kernel.'.

Edit `/usr/src/linux-2.6.22/arch/i386/boot/compressed/misc.c`, scroll to the bottom of the file until you see the following

```
        putstr("Uncompressing Linux... ");
        gunzip();
        putstr("Ok, booting the kernel.\n");
```

Modify to look like the following

```
        /* putstr("Uncompressing Linux... "); */
        gunzip();
        /* putstr("Ok, booting the kernel.\n"); */
```

### `agetty` Modification
`agetty` prints a new line before displaying the `/etc/issue` file and eventually running `login`.

ftp://ftp.kernel.org/pub/linux/utils/util-linux/

`agetty` is part of util-linux, download and extract

```
wget ftp://ftp.kernel.org/pub/linux/utils/util-linux/util-linux-2.12r.tar.gz
tar xf util-linux-2.12r.tar.gz
cd util-linux-2.12r
./configure
```


```
cd login-utils
```

edit `agetty.c`, search for `\r\n`, look for the following line

```
    (void) write(1, "\r\n", 2);                 /* start a new line */
```

Modify it to read the following, then write the file and quit

```
    (void) write(1, "\r", 2);                 /* start a new line */
```

Make a backup of the currently installed `/sbin/agetty` and install.

```
mv /sbin/agetty /sbin/agetty.old
make agetty
make agetty install
```

### logging
As noted http://sources.gentoo.org/viewcvs.py/gentoo-x86/app-admin/showconsole/showconsole-1.08.ebuild?view=markup

Official sysvinit distribution site: ftp://ftp.cistron.nl/pub/people/miquels/sysvinit/

There are two ways to get the `blogd` daemon; either download `showconsole-1.08.tar.bz2` from gentoo distfiles or download and convert the `sysvinit-2.86-129.src.rpm` from opensuse.

Download sysvinit rpm from suse:

```
wget http://mirrors.kernel.org/opensuse/distribution/SL-OSS-factory/inst-source/suse/src/sysvinit-2.86-129.src.rpm
```

Convert and extract (making sure `rpm2cpio.pl` is installed before-hand)

```
mkdir sysvinit
rpm2cpio.pl sysvinit-2.86-129.src.rpm > sysvinit/sysvinit-2.89-129.src.cpio
cd sysvinit/
cpio -d -i < sysvinit-2.89-129.src.cpio
```

Download gentoo distfile

```
wget http://distfiles.gentoo.org/distfiles/showconsole-1.08.tar.bz2
```

Find a mirror of gentoo portage such as http://kambing.ui.edu/gentoo-portage/app-admin/showconsole/files/

Download the patch `1.07-no-TIOCGDEV.patch`
Download the patch `showconsole-1.08-quiet.patch`
Download the shell script `bootlogger.sh`

After aquiring `showconsole-1.08.tar.bz2`, extract it, patch it and build it

```
tar xf showconsole-1.08.tar.bz2
cd showconsole-1.08
cat ../1.07-no-TIOCGDEV.patch | patch -p1
cat ../showconsole-1.08-quiet.patch | patch -p1
make
make install
blogd -q # start without outputting any information
killall -IO blogd
killall -QUIT blogd
```

Download `bootlogger.sh` and add a line to /etc/rc.d/init.d/functions

```
test -r /etc/rc.d/init.d/bootlogger.sh && . /etc/rc.d/init.d/bootlogger.sh
```
 * is it possible to run bootlogd/blogd with console=/dev/ttyS0
 * is it possible to run bootlogd/blogd straight after proc is mounted rather than later on? this way a minimum amount of messages are lost.
 * backup old logs
 * log shutdown as well
 * dont display any messages to consoles
