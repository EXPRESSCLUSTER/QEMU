# How to Run ARM on Ubuntu (x86_64)
- This article describe how to run ARM virtual machine on Ubuntu Server running on x86_64.

## Index
- [Notes](#notes)
- [Configuration](#configuration)
- [Install Packages](#install-packages)
- [Run Virtual Machines](#run-virtual-machines)

## Notes
- This configuration uses TAP for guest OS network. You need TAP for each virtual machine.
- MAC address will be assigned automatically. The default one is  52:54:00:12:34:56. If you want to assign another one you need set MAC address (e.g., 52:54:00:12:34:57) manually.

## Configuration
```
+-------------------------------------------------------+
| Host OS                                               |
| Ubuntu Server 22.04.2 LTS                             |
| QEMU 6.2.0                                            |
|                                                       |
| +------------------------+ +------------------------+ |
| | Guest OS               | | Guest OS               | |
| | Amazon Linux 2 (ARM)   | | Amazon Linux 2 (ARM)   | |
| | MAC: 52:54:00:12:34:56 | | MAC: 52:54:00:12:34:57 | |
| | IP: 192.168.122.11     | | IP: 192.168.122.12     | |
| +--+---------------------+ +--+---------------------+ |
|    |                          |                       |
| +--+---+                   +--+---+                   |
| | tap0 |                   | tap1 |                   |
| +--+---+                   +--+---+                   |
|    |                          |                       |
| +--+--------------------------+---+                   |
| | virbr0 (192.168.122.1)          |                   |
| +--+------------------------------+                   |
|    |                                                  |
| +--+---+                                              |
+-| eth0 |----------------------------------------------+
  +--+---+
     |
     :
```

## Install Packages
1. Install the following packages.
   ```sh
   sudo apt install -y qemu-system-arm libvirt-daemon libvirt-daemon-system genisoimage
   ```

## Run Virtual Machines
### Amazon Linux 2
#### First Node
1. Run the following commands with sudo user.
   ```sh
   sudo ip tuntap add tap0 mode tap
   ```
   ```sh
   sudo ip link set tap0 promisc on
   ```
   ```sh
   sudo ip link set dev tap0 master virbr0
   ```
   ```sh
   sudo ip link set dev tap0 up
   ```
1. Create an account (e.g., user) and move to the home directory.
   ```
   $ pwd
   /home/user
   ```
1. Create a directory.
   ```sh
   mkdir -p qemu/aarch64/amzn2-11/seedconfig
   ```
1. Create seed.iso on the seedconfig directory.
   - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-2-virtual-machine.html#amazon-linux-2-virtual-machine-prepare
   - https://github.com/EXPRESSCLUSTER/QEMU/blob/master/doc/HowToRunAmazonLinux2onARM.md#create-seediso
1. Download qcow2 file, rename it (e.g., amzn2-11.qcow2) and save it on the amzn2-11 directory.
   - https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/amazon-linux-2-virtual-machine.html#amazon-linux-2-virtual-machine-download
1. Copy QEMU_EFI.fd from /usr/share/qemu-efi-aarch64 to the amzn2-11 directory.
   ```sh
   cp /usr/share/qemu-efi-aarch64/QEMU_EFI.fd /home/user/qemu/amzn2-11/
   ```
   ```sh
   chown user:user /home/user/qemu/aarch64/amzn2-11/QEMU_EFI.fd
   ```
1. Create an additional disk (e.g., md1.qcow2).
   ```sh
   qemu-img create -f qcow2 md1.qcow2 10G
   ```
1. Save the above files on the same directory as below.
   ```
   +--- amzn2-11.qcow2
   +--- md1.qcow2
   +--- QEMU_EFI.fd
   +--- seedconfig
        +--- meta-data
        +--- seed.iso
        +--- user-data   
   ```   
1. Create run.sh as below.
   ```sh
   qemu-system-aarch64 \
   -M virt \
   -cpu cortex-a72 \
   -m 2G \
   -smp cpus=2 \
   -nographic \
   -cdrom ./seedconfig/seed.iso \
   -bios QEMU_EFI.fd \
   -drive if=none,file=amzn2-11.qcow2,id=hd0 \
   -device virtio-blk-device,drive=hd0 \
   -drive if=none,file=md1.qcow2,id=hd1 \
   -device virtio-blk-device,drive=hd1 \   
   -net nic \
   -net tap,ifname=tap0,script=no
   ```
1. Run VM!
   ```sh
   ./run.sh
   ```

#### Second Node
1. Run the following commands with sudo user.
   ```sh
   sudo ip tuntap add tap1 mode tap
   ```
   ```sh
   sudo ip link set tap1 promisc on
   ```
   ```sh
   sudo ip link set dev tap1 master virbr0
   ```
   ```sh
   sudo ip link set dev tap1 up
   ```   
1. Create seed.iso, an additional disk and copy QEMU_EFI.fd as you do those for the first node.
1. Create run.sh as below. You must set the different MAC address as below.
   ```sh
   qemu-system-aarch64 \
   -M virt \
   -cpu cortex-a72 \
   -m 2G \
   -smp cpus=2 \
   -nographic \
   -cdrom ./seedconfig/seed.iso \
   -bios QEMU_EFI.fd \
   -drive if=none,file=amzn2-12.qcow2,id=hd0 \
   -device virtio-blk-device,drive=hd0 \
   -drive if=none,file=md1.qcow2,id=hd1 \
   -device virtio-blk-device,drive=hd1 \
   -net nic,macaddr=52:54:00:12:34:57 \
   -net tap,ifname=tap1,script=no
   ```
1. Run VM!
   ```sh
   ./run.sh
   ```