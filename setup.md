
Last updated: 25-July-2025

# System

## Swapfile

If you haven't set separate swap partition, you can still add swapfile residing on the root parition (which might provide encryption for it).

This example creates swapfile of 16 GB on the root partition:

```
$ sudo dd if=/dev/zero of=/swapfile-16G bs=1024 count=16M
$ sudo chmod 600 /swapfile-16G
$ sudo mkswap /swapfile-16G

(add the swapfile to fstab)
(example:)
(/swapfile-16G none swap sw 0 0)
$ sudo nano /etc/fstab

(apply the change)
$ sudo systemctl daemon-reload

(enable the swapfile:)
$ sudo swapon -a

(check it is used:)
$ sudo swapon
$ free
```

# Services

Kali is designed to be silent by default. 

That means that many services are not enabled out of the box.

## Bluetooth

```
$ sudo systemctl status bluetooth.service
$ sudo systemctl enable bluetooth.service
$ sudo systemctl start bluetooth.service
```

## KDE Connect

```
$ ls -al /usr/share/dbus-1/services/org.kde.kdeconnect.service*
$ sudo cp /usr/share/dbus-1/services/org.kde.kdeconnect.service.original /usr/share/dbus-1/services/org.kde.kdeconnect.service
```

# Additional software

Kali is derived from Debian testing, meaning:

- there are no security updates provided by default
- the Debian codename for the current stable release is bookworm
- the Debian codename for the upcoming stable release is trixie (there is no such release out there at the moment but that's the name for it once it gets released)

Third party software providers sometimes open a Debian format repository referred by the Debian stable version. Try using the most recent one (bookworm).

Some third party software depends on packages that are not available in Kali repositories.

In that case you can try adding Debian stable (bookworm) repository, if that contains the missing packages.

Also, the stable release has security updates repository, which can be also added to Kali.

## VirtualBox

There is packaged version of Virtualbox in Kali repository and it is preferred.

If you rather want to download latest greatest from Oracle directly, you might hit a conflict on libvpx7, which is not available in Kali repository.
It is available in Debian stable repository, so you can add that one.

```
(/etc/apt/sources.list.d/bookworm.list)
deb https://deb.debian.org/debian bookworm main contrib non-free non-free-firmware
deb https://deb.debian.org/debian bookworm-updates main contrib non-free non-free-firmware
deb https://deb.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```

And then Virtualbox shall install:

```
(/etc/apt/sources.list.d/virtualbox.list)
deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-virtualbox-2016.gpg] https://download.virtualbox.org/virtualbox/debian bookworm contrib
```

```
$ (optional, run gpg at least once, if on fresh installation) sudo gpg
$ wget -O- https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo gpg --yes --output /usr/share/keyrings/oracle-virtualbox-2016.gpg --dearmor
$ sudo apt update
$ sudo apt install virtualbox-7.1
$ sudo usermod -a -G vboxusers $USER
```

In order to apply the group membership, logout/login shall work, but sometimes it won't and reboot is necessary.

## Docker

Straightforward:

```
$ echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian bookworm stable" | sudo tee /etc/apt/sources.list.d/docker.list
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli containerd.io
$ sudo usermod -a -G docker $USER
```

In order to apply the group membership, logout/login shall work, but sometimes it won't and reboot is necessary.

