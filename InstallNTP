# How to Install NTP

Create `/etc/rc.d/init.d/ntpdate`

```
#!/bin/sh

. /etc/sysconfig/rc
. ${rc_functions}

case "${1}" in
        start)
                boot_mesg "Setting time with ntpdate..."
                /opt/ntp/bin/ntpdate -s -b ntp.demon.co.uk 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 0.uk.pool.ntp.org 1.uk.pool.ntp.org 2.uk.pool.ntp.org
                ;;
        *)
                echo "Usage: ${0} {start}"
                exit 1
                ;;
esac

```

Create `/etc/rc.d/init.d/ntpd`

```
#!/bin/sh

. /etc/sysconfig/rc
. ${rc_functions}

case "${1}" in
        start)
                boot_mesg "Starting ntpd..."
                /opt/ntp/bin/ntpd
                ;;

        stop)
                boot_mesg "Stopping ntpd..."
                killproc -p /var/run/ntpd.pid /opt/ntp/bin/ntpd TERM
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

Create `/etc/rc.d/init.d/hwclock`

```
#!/bin/sh

. /etc/sysconfig/rc
. ${rc_functions}

case "${1}" in
        stop)
                boot_mesg "Setting hardware clock from system time..."
                hwclock --systohc
                ;;
        *)
                echo "Usage: ${0} {stop}"
                exit 1
                ;;
esac

```

Create `/etc/ntp.conf`

```
# servers, quickest first
server ntp.demon.co.uk iburst
server 0.uk.pool.ntp.org iburst
server 1.uk.pool.ntp.org iburst
server 2.uk.pool.ntp.org iburst
server 0.europe.pool.ntp.org iburst
server 1.europe.pool.ntp.org iburst
server 2.europe.pool.ntp.org iburst

# drift file
driftfile /etc/ntp.drift

# pid file (used by init scripts)
pidfile /var/run/ntpd.pid

# log file
logfile /var/log/ntp.log

```

Make executable `init` scripts

```
chmod +x /etc/rc.d/init.d/{ntpdate,ntpd,hwclock}
```

Compile and install `ntp`

```
wget http://www.eecis.udel.edu/~ntp/ntp_spool/ntp4/ntp-4.2.4p7.tar.gz
tar zxvf ntp-4.2.4p7.tar.gz
cd ntp-4.2.4p7
./configure --prefix=/opt/ntp
make install

```

Force set date on boot 

```
ln -s ../init.d/ntpdate /etc/rc.d/rc3.d/S24ntpdate
```

Start ntpd after ntpdate, stop it on boot/reboot

```
ln -s ../init.d/ntpd /etc/rc.d/rc3.d/S25ntpd
ln -s ../init.d/ntpd /etc/rc.d/rc0.d/K35ntpd
ln -s ../init.d/ntpd /etc/rc.d/rc6.d/K35ntpd
```

Sync the system clock to the hardware clock on shutdown/reboot (run before or after ntpd shutdown?)

```
ln -s ../init.d/hwclock /etc/rc.d/rc0.d/K40hwclock
ln -s ../init.d/hwclock /etc/rc.d/rc6.d/K40hwclock
```



