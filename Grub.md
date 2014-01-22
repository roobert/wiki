# Grub Configuration

## `menu.lst` Example

 * White foreground with a black background
 * 5 second timeout
 * Menu hidden until `esc` is pressed
 * Kernel `quiet` option set by default to hide kernel messages
 * Default console set to `/dev/ttyS0` in-order to hide boot messages
 * Optional 'boot messages' menu entry

## `grub-quiet.patch`

### Description

This patch makes `grub` less verbose during boot.

Installation of `grub` is not necessarily needed; it is possible to just build `grub` and copy the relevant 'stage' files into place.

### Installation

Download the attached `grub-quiet.patch` file and place in /usr/src or equivalent

Download, extract and patch `grub`:

```
wget ftp://alpha.gnu.org/gnu/grub/grub-0.97.tar.gz
tar xf grub-0.97.tar.gz
cd grub-0.97
cat ../grub-quiet.patch | patch -p1
```

Build `grub`

```
./configure
make
```

Either install `grub`, if it is not already available

```
make install
```

or if `grub` is already installed it is possible to just replace the stage files

```
cd /usr/lib/grub/i386-pc
mkdir backup
mv * backup/
for file in `cd backup; ls`; do cp -v /usr/src/grub-0.97/stage?/$file .; done
```

or if you wish to remove your current `grub` installation, remove it and install the newly compiled `grub`

```
make install
```

Regardless of which previous steps you have chosen, write the bootloader to disk

```
# Location of /boot
ROOT_MNT="/"
grub-install --root-directory=$ROOT_MNT /dev/sda
```


