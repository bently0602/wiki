apt install cpu-checker

kvm-ok

apt install -y qemu qemu-kvm libvirt-daemon libvirt-clients bridge-utils virt-manager

systemctl status libvirtd

systemctl enable --now libvirtd

qemu-img create -f qcow2 -o preallocation=metadata /root/win2k.qcow2 10G
qemu-system-i386 -enable-kvm -L pc-bios -m 256 -vnc :0 -hda win2k.qcow2 -cdrom win2000DC.iso -boot d
