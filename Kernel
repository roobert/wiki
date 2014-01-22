# Kernel Configuration

[[PageOutline(0-4,contents,inline)]]

## References

Linux Kernel in a Nutshell: http://www.kroah.com/lkn/

## Kernel Version
Download the kernel and extract:

```
KERNEL_VERSION=2.6.29.4
cd /usr/src
wget http://www.kernel.org/pub/linux/kernel/v2.6/$KERNEL_VERSION.tar.bz2
tar xf $KERNEL_VERSION.tar.bz2
cd $KERNEL_VERSION
```

## Optional: Apply any patches

Such as the OpenVZ patch as described on the EeeKernel page

## Optional: Configure the kernel

Such as described on the EeeKernel page

```
cd /usr/src/$KERNEL_VERSION
make defconfig
```

## Kernel Installation
Building the default kernel

```
cd /usr/src/$KERNEL_VERSION
make defconfig targz-pkg
```

Installing the kernel package

```
# /usr/src/$KERNEL_VERSION/tar-install/boot/vmlinuz-2.6.29.4  - Compressed kernel (bzImage)
# /usr/src/$KERNEL_VERSION/tar-install                        - Pre-compressed contents of the kernel package
# tar ztf /usr/src/$KERNEL_VERSION/$KERNEL_VERSION.tar.gz     - List the kernel package contents
tar zxf /usr/src/$KERNEL_VERSION/$KERNEL_VERSION.tar.gz -C /
```


```
# either
cd /usr/src/$KERNEL_VERSION
make bzImage modules modules_install
cp arch/i386/boot/bzImage /boot/
cp .config /boot/config
cp System.map /boot/
# or
make targz-pkg
# or
make install
```

## Optional: Saving Space
Remove the source and distfiles to save space:

```
cd /usr/src/
# Optionally remove any patches now, too
rm /usr/src/$KERNEL_VERSION.tar.gz
# either:
cd $KERNEL_VERSION
make clean
# or
rm -rf $KERNEL_VERSION
```
