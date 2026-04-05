Proxmox + Ubuntu VM GPU Passthrough for Local LLMs (Zeroclaw & Ollama)

This guide walks you through setting up a Proxmox VM running Ubuntu with NVIDIA GPU passthrough, then installing Zeroclaw and Ollama to run small local LLMs.

Prerequisites
Proxmox VE installed and running.
NVIDIA GPU you want to passthrough (example: RTX 3050).
Ubuntu ISO for VM installation.
Basic knowledge of Linux and Proxmox administration.
1. Configure Proxmox for GPU Passthrough

Edit GRUB to enable IOMMU:

nano /etc/default/grub


Set:

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt vfio-pci.ids=10de:2584,10de:2291"


Update GRUB:

update-grub


Load VFIO modules on boot:

nano /etc/modules


Add:

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd


Configure VFIO PCI IDs:

nano /etc/modprobe.d/vfio.conf


Add:

options vfio-pci ids=10de:2584,10de:2291


Blacklist default NVIDIA drivers:

nano /etc/modprobe.d/blacklist.conf


Add:

blacklist nouveau
blacklist nvidia
blacklist nvidiafb
blacklist nvidia_drm


Update initramfs and reboot:

update-initramfs -u -k all
reboot

2. Create and Configure Ubuntu VM
BIOS: UEFI
VM type: Q35
Add the GPU as a PCI device:
Check All functions
Check ROM-Bar and PCI-Express

Install Ubuntu:

Boot the VM with the ISO.
Complete Ubuntu installation.
Update Ubuntu:
sudo apt update && sudo apt upgrade -y
sudo reboot


Install NVIDIA drivers:

sudo apt install -y software-properties-common
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install -y nvidia-driver-535 nvidia-utils-535
sudo reboot


Verify GPU passthrough:

nvidia-smi

3. Install Zeroclaw & Ollama
Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup update


Configure shell for Cargo:

source "$HOME/.cargo/env"

Install Git
sudo apt-get install git -y

Clone Zeroclaw
git clone https://github.com/zeroclaw-labs/zeroclaw.git
cd zeroclaw

One-Click Bootstrap
./install.sh

4. Notes
Ensure your VM always boots with UEFI; GPU passthrough often fails with legacy BIOS.
If nvidia-smi does not detect the GPU, double-check IOMMU and VFIO configuration on the Proxmox host.
After installing Zeroclaw, follow the repository instructions for configuring Ollama and running local LLMs.

This setup allows you to run small LLMs locally with GPU acceleration inside an Ubuntu VM on Proxmox.
