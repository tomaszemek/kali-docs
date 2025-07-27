
Last updated: 27-July-2025

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

### Kali version

There is packaged version of Virtualbox in Kali repository, relatively recent and it is preferred to use that one as it is well integrated to the rest of the distribution.

It can be installed as such:

```
$ sudo apt update
$ sudo apt install linux-headers-amd64
$ sudo apt install virtualbox
$ sudo apt install virtualbox-ext-pack
$ sudo usermod -a -G vboxusers $USER
```

In order to apply the group membership, logout/login shall work, but sometimes it won't and reboot is necessary.

Notes: 

- With this version, I'm not getting stack traces in dmesg, as I used to get otherwise when using Oracle version
- With this version of Virtualbox, I was having sometimes situation when the kernel modules were not loaded on boot and I wasn't able to run machines. Had to reboot in order to get it working again. Haven't found the root cause yet.
- With this version of Virtualbox, there are weird messages shown during update-initramfs. Haven't found the root cause yet.
- With this version of Virtualbox, I wasn't able to convert boot partition to LUKS1 as the device was still in use after unmounting. Haven't found the root cause yet.


### Oracle version

Alternatively, if you really want to stick to the latest greatest from Oracle directly, you might hit a conflict on libvpx7 and other dependencies, which are not available in Kali repository.
Those dependencies are however available in Debian stable repository, so you can add that one:

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

## Firefox

Kali comes with Firefox-ESR by default, which is bit outdated.

Switch to recent Firefox:

```
$ (optional, run gpg at least once, if on fresh installation) gpg
$ wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null
$ gpg -n -q --import --import-options import-show /etc/apt/keyrings/packages.mozilla.org.asc | awk '/pub/{getline; gsub(/^ +| +$/,""); if($0 == "35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3") print "\nThe key fingerprint matches ("$0").\n"; else print "\nVerification failed: the fingerprint ("$0") does not match the expected one.\n"}'
$ echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" | sudo tee -a /etc/apt/sources.list.d/mozilla.list > /dev/null
$ echo '\nPackage: *\nPin: origin packages.mozilla.org\nPin-Priority: 1000\n' | sudo tee /etc/apt/preferences.d/mozilla
$ sudo apt-get update && sudo apt-get install firefox
$ sudo apt purge firefox-esr
```

## Node.js

Via NVM.

This example install version 24:

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash \. "$HOME/.nvm/nvm.sh"
$ nvm install 24
```

## Code:

Possibly already installed as code-oss.
Launcher might be missing but the program might be already installed:

```
$ code   
┏━(Message from Kali developers)
┃ code is not the binary you may be expecting.
┃ You are looking for \"code-oss\"
┃ Starting code-oss for you...
┗━
```
