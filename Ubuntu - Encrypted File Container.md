Ubuntu - Encrypted File Container 
================================================

## Install

```
apt install cryptsetup
```

## Create Encrypted File Container
```
fallocate -l 1G /encrypted.img
```
OR
```
dd if=/dev/zero of=/encrypted.img bs=1 count=0 seek=1G
```

## Creating Key

You can either create a key file:
```
dd if=/dev/urandom of=/mykey.keyfile bs=1024 count=1
```

Or (RECOMMENDED) generate a passphrase on another system and provide via prompt.
To generate secure passphrase:
```
openssl rand -base64 32
```
which would give something like: Tf7hcIV77EGalcdUB6GNpL8G6f8Ylwilrnaek0qy6wg=

## Encrypt File Container

```
cryptsetup -c aes-xts-plain64 -v luksFormat /encrypted.img [/mykey.keyfile]
```
Note the [/mykey.keyfile] is if your using the key file. This will be optional in all the below steps but not mentioned in this document again.
*Make sure to type YES (uppercase!) when prompted by the command above or your image will not be encrypted.*

## Opening

Open the encrypted container:
```
cryptsetup luksOpen /encrypted.img EncryptedTest [--key-file mykey.keyfile]
```

You should see the new "EncryptedTest" device in /dev/mapper:
```
ls /dev/mapper
```

## Closing

Make sure any mounted file systems pertaining to the image are unmounted then close the encrypted container:
```
cryptsetup luksClose EncryptedTest
```

## Rotating Keys

You can up to 8 keys available for the image.

```
cryptsetup luksDump /encrypted.img
```

To add a key to an unoccupied slot (slot 1 in example):
```
cryptsetup luksAddKey --key-slot 1 -v /encrypted.img
```
Once it asks for a key, use any other available passphrase.

To remove a key from an occupied slot (slot 0 in example):
```
cryptsetup luksKillSlot /encrypted.img 0
```

## Using

Really after opening the container the usual Linux stuff applies. The below is just for a quick reference if needed and doesn't really pertain to only encrypted containers, so feel free to skip.

### Mounting
After opening and seeing in /dev/mapper:

To format:
```
mkfs.ext4 /dev/mapper/EncryptedTest
```

To mount:
```
mkdir -p /mnt/EncryptedTest
mount /dev/mapper/EncryptedTest /mnt/EncryptedTest/
```

To unmount:
```
umount /mnt/EncryptedTest
```