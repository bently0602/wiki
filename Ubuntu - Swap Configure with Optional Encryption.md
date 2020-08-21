Using swap as a file not as a device with optional encryption instructions
================================================

## Credits

swapon complains about holes
- https://www.lucas-nussbaum.net/blog/?p=331

fallocate and dd
- https://askubuntu.com/questions/506910/creating-a-large-size-file-in-less-time

ubuntu problems with cryptdisk and swap
- https://askubuntu.com/questions/596813/can-ubuntu-14-04-use-encrypted-swap-without-resorting-to-unofficial-hacks

ubuntu encrypted disk does not show in /dev/mapper
- https://askubuntu.com/questions/810697/encrypted-swap-partition-does-not-show-up-in-dev-mapper

cryptsetup faq
- https://gitlab.com/cryptsetup/cryptsetup/wikis/FrequentlyAskedQuestions

calculate dd blocksize
- https://stackoverflow.com/questions/6161823/dd-how-to-calculate-optimal-blocksize

cryptsetup cipher modes
- https://security.stackexchange.com/questions/5158/for-luks-the-most-preferable-and-safest-cipher
- https://unix.stackexchange.com/questions/354787/list-available-methods-of-encryption-for-luks

## Remove Existing Swap

My install of Ubuntu 18 came setup by default with swap directed to a file.

### Check for Existing Swap

Check the results of:
```bash
cat /proc/swaps
swapon --summary
```

### Remove Existing Swap

If swap exists, turn off:
```
swapoff -a
```

And remove corresponding line from fstab like:
```
/swap.img       none    swap    sw      0       0
```

Verify swap off with:
```
swapon --summary
```

## Setup Swap as a File

### Create New Swap File

Change 4G to the size of your swap (usually mirrors your RAM size).

```
fallocate -l 4G /swapfile.img
```
OR (if using an unsupporting filesystem format, use dev/zero for faster if you can)
```
dd if=/dev/urandom of=swapfile.img count=32768 bs=128K
```

Then:

```
chmod 600 /swapfile.img
mkswap /swapfile.img
```

```
swapon /swapfile.img
```

Verify swap on with:
```
swapon --summary
```

Add new swap space to /etc/fstab:
```
/swapfile.img none swap sw 0 0
```

## Setup Encrypted Swap as File

### Prereqs:

Remove existing swap. Then install cryptsetup package if needed.

```
apt install cryptsetup
```

### Create New Swap File

Change 4G to the size of your swap (usually mirrors your RAM size).

```
fallocate -l 4G /cryptswap.img
```
OR (if using an unsupporting filesystem format, use dev/zero for faster if you can)
```
dd if=/dev/urandom of=swapfile.img count=32768 bs=128K
```

Then:

```
chmod 600 /cryptswap.img
mkswap /cryptswap.img
```

Add new swap line to /etc/crypttab:
```
cryptswap /cryptswap.img /dev/urandom swap,cipher=aes-xts-plain64,size=256
```

Restart cryptdisks service:
```
service cryptdisks reload
```

Add new swap space to /etc/fstab:
```
/dev/mapper/cryptswap none swap sw 0 0
```

Turn swap back on:
```
swapon -a
```

Restart:
```
shutdown -r now
```

Verify swap on with:
```
swapon --summary
```
and make sure /proc/swaps points to a dm interface:
```
cat /proc/swaps
```
output should should a file name like "/dev/dm-0".