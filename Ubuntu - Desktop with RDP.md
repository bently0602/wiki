Ubuntu - Desktop with RDP
================================================

## Install Requirements

I recommend a LUBUNTU environment for lower spec systems (< 1GB, 1 core).

XFCE
```bash
apt-get install xubuntu-desktop
apt-get install xorgxrdp xrdp
```

OR

LUBUNTU
```
apt install -y lubuntu-desktop
apt install -y xorgxrdp xrdp
```

The following information taken from:
- https://askubuntu.com/questions/1031519/xrdp-on-ubuntu-18-04lts
- https://askubuntu.com/questions/316025/how-to-install-and-configure-wine

Then add the desktop appropriate environment start to startwm.sh.

```
startxfce4 to /etc/xrdp/startwm.sh; comment out the last two lines test and export
or
startlxde
```

Make sure 3389 is open or tunneled.
Use "xorg" option in once RDP connection is started to login.
