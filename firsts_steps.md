# proxmox-project

NoteBook
ssh-keygen -t rsa

ssh-copy-id user@ip_proxmox

Guide How to start Proxmox

Before I do anything on Proxmox, I do this first...

https://technotim.live/posts/first-11-things-proxmox/


IOMMU (PCI Passthrough)
You’ll first want to be sure that Vt-d / IOMMU is enabled in your BIOS before continuing.

GRUB
If you’re using GRUB, use the following commands:

nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"


VFIO modules

Edit /etc/modules

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

run

update-initramfs -u -k all

then reboot

reboot


NVIDIA

If you’re planning on using an NVIDIA card, I’ve found this helps prevent some apps like GPUz from crashing on the VM.

echo "options kvm ignore_msrs=1 report_ignored_msrs=0" > /etc/modprobe.d/kvm.conf


PVE Post Install
Added 2024-04-28•Proxmox Node
Description

This script provides options for managing Proxmox VE repositories, including disabling the Enterprise Repo, adding or correcting PVE sources, enabling the No-Subscription Repo, adding the test Repo, disabling the subscription nag, updating Proxmox VE, and rebooting the system.

bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"


