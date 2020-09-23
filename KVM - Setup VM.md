apt install cpu-checker

kvm-ok

apt install -y qemu qemu-kvm libvirt-daemon libvirt-clients bridge-utils virt-manager

systemctl status libvirtd

systemctl enable --now libvirtd

lsmod | grep -i kvm

virt-install --virt-type qemu --name=winB-vm --os-variant=win2k --vcpu=1 --ram=256 --graphics spice,listen=* --cdrom=EN_WIN2000_PRO_SP4.ISO --network network=default --disk demoB.img,size=6

spice://XXXXXXXXXXXXXXX:5900
