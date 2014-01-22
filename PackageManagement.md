# Package Management Using NetBSD `pkgsrc`

[[PageOutline(0-4,contents,inline)]]

## Description

## Installation

### Step 1. The `pkgadmin` User

Create a `pkgadmin` user

```
useradd -m pkgadmin
```

Add the `bin` and `sbin` paths to the environment in order to use the pkgsrc tools, for example, in `~pkgadmin/.bashrc`:

```
PKGSRC_VERSION="2009Q1"
PATH="$PATH:/home/pkgadmin/pkgsrc-$PKGSRC_VERSION/inst/sbin"
PATH="$PATH:/home/pkgadmin/pkgsrc-$PKGSRC_VERSION/inst/bin"
export PATH PKGSRC_VERSION
```

`su` and create directories for `pkgsrc` installation

```
su - pkgadmin
mkdir -p ~/pkgsrc-$PKGSRC_VERSION/{distfiles,packages,pkgsrc}
```

### Step 2. The `pkgsrc` Tree

Depending on the configuration its possible to either download and extract `pkgsrc` or if, for example, using virtualization, then its possible to bind the hardware nodes `pkgsrc` tree into the VEs root file system, as described at OpenVZ#SharingpkgsrcBetweenHostandVEs. Note that the `vz.sh` script on the aforementioned page handles mounting of the tree during VE start.

#### Either: Normal Configuration

```
wget ftp://ftp.netbsd.org/pub/pkgsrc/pkgsrc-$PKGSRC_VERSION/pkgsrc-$PKGSRC_VERSION.tar.gz
mkdir pkgsrc-$PKGSRC_VERSION
# this directory is only needed for mounted pkgsrc tree
test -d pkgsrc-$PKGSRC_VERSION/pkgsrc && rm -rf pkgsrc-$PKGSRC_VERSION/pkgsrc
tar zxf pkgsrc-$PKGSRC_VERSION.tar.gz -C pkgsrc-$PKGSRC_VERSION
```

This configuration prefers nothing in pkgsrc/ be written to. This also enables pkgsrc/ to be shared between multiple systems such as virtual instances, which saves a lot of space.

```
# this breaks the ability to bootstrap - fixme
chmod -R 550 pkgsrc-$PKGSRC_VERSION/pkgsrc
```

#### Or: Mounting a `pkgsrc` tree

Restart the VE using the `vz.sh` script on this page: OpenVZ#SharingpkgsrcBetweenHostandVEs

`pkgsrc` should now be mounted, check with `df`.

Change to the pkgadmin user and continue

```
su - pkgadmin
```

### Step 3. Bootstrapping `pkgsrc`

This installs the necessary tools and configuration to use the `pkgsrc` tree.

Bootstrap `pkgsrc`

```
export PKGADMIN_BASE="/home/pkgadmin/pkgsrc-$PKGSRC_VERSION"
$PKGADMIN_BASE/pkgsrc/bootstrap/bootstrap --workdir=$PKGADMIN_BASE/work \
                                           --prefix=$PKGADMIN_BASE/inst \
                                           --pkgdbdir=$PKGADMIN_BASE/db \
                                           --sysconfdir=$PKGADMIN_BASE/etc \
                                           --varbase=$PKGADMIN_BASE/var \
                                           --unprivileged

```

### Step 4. Post-Bootstrap Configuration

Add the following to `~pkgadmin/pkgsrc-$PKGSRC_VERSION/etc/mk.conf` just before ".endif                  # end pkgsrc settings":

```
DISTDIR=                /home/pkgadmin/pkgsrc-$PKGSRC_VERSION/distfiles
PACKAGES=               /home/pkgadmin/pkgsrc-$PKGSRC_VERSION/packages
WRKOBJDIR=              /home/pkgadmin/pkgsrc-$PKGSRC_VERSION/work
```

And to make package dependencies also be built as packages, add:

```
DEPENDS_TARGET=         package
```

Also, this could be necessary in-order to properly build X11

```
X11_TYPE=modular
```
### Step 5. Using Installed Packages as a User

For any user which wishes to use installed packages, add the `inst/bin` and `inst/sbin` paths to your environment, for example, in `~/.bashrc`. And preferably add to `/etc/skel/.*`

```
PKGSRC_VERSION="2009Q1"
PATH="$PATH:/home/pkgadmin/pkgsrc-$PKGSRC_VERSION/inst/sbin"
PATH="$PATH:/home/pkgadmin/pkgsrc-$PKGSRC_VERSION/inst/bin"
export PATH PKGSRC_VERSION
```

## Installing `pkgsrc` Packages

Example:

```
su - pkgadmin
cd pkgsrc-$PKGSRC_VERSION/pkgsrc/*/wget
bmake package install clean-depends clean
```
