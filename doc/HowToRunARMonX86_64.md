# How to Run ARM on x86_64

## Index
- [Evaluation Environment](#evaluation-environment)
- [Prerequisite](#prerequisite)
- [Build qemu-system-aarch64](#build-qemu-system-aarch64)
- [Customize Image File](#customize-image-file)
- [Get initrd and Kernel File](#get-initrd-and-kernel-file)
- [Run the VM](#run-the-vm)
- [Link](#link)

## Evaluation Environment
### Use Terminal Access Point (TAP)
- IP address will be assigned automatically. The first node IP address will be 192.168.122.76. and the second node IP address will be 192.168.122.77.
  ```
  +-----------------------------------------------------------------------------------+
  | +--------------------------------------+ +--------------------------------------+ |
  | | Virtual Machine                      | | Virtual Machine                      | |
  | | Ubuntu 20.04.1 LTS                   | | Ubuntu 20.04.1 LTS                   | |
  | | +-----------------------+            | | +-----------------------+            | |
  | | | eth0 (192.168.122.76) |            | | | eth0 (192.168.122.77) |            | |
  | | +--+--------------------+            | | +--+--------------------+            | |
  | +----|---------------------------------+ +----|---------------------------------+ |
  |      |                                        |                                   |
  |      |        +-------------------------------+                                   |
  |      |        |                                                                   |
  |   +--+---+ +--+---+                                                               |
  |   | tap0 | | tap1 |                                                               |
  |   +--+---+ +--+---+                                                               |
  |      |        |                                                                   |
  |   +--+--------+------------+                                                      |
  |   | virbr0 (192.168.122.1) |                                                      |
  |   +------------------------+                                                      |
  |                                                                                   |
  | CentOS Linux release 7.9.2009 (Core)                                              |
  +-----------------------------------------------------------------------------------+
  ```
## Prerequisite
- Install Ubuntu 20.04 on the other machine to get the initrd and Kernel file.

## Build qemu-system-aarch64
1. Install [many packages](https://github.com/EXPRESSCLUSTER/QEMU/blob/master/HowToRunPOWER9onX86_64.md#build-qemu-system-ppc64).
1. Download the qemu binary file and expand it.
   ```sh
   # cd /root
   # mkdir work
   # cd /root/work
   # curl -O  https://download.qemu.org/qemu-5.2.0.tar.xz
   # tar xf qemu-5.2.0.tar.xz
   ```
1. Move to the qemu directory and build qemu binary.
   ```sh
   # cd qemu-5.2.0
   # ./configure --target-list=aarch64-softmmu
   # make
   ```
## Customize Image File
1. Download cloud image file.
   - https://cloud-images.ubuntu.com/releases/focal/release/
1. Change the password for root.
   ```sh
   # virt-customize -a ubuntu-20.04-server-cloudimg-arm64.img  --root-password password:<your password>
   ```
## Get initrd and Kernel File
1. Copy ubuntu-20.04-server-cloudimg-arm64.img to Ubuntu machine.
1. Login Ubuntu machine.
1. Run the following commands to mount the image file.
   ```sh
   $ sudo modprobe nbd max_part=63
   $ sudo qemu-nbd -c /dev/nbd0 ubuntu-20.04-server-cloudimg-arm64.img
   $ sudo mount /dev/nbd0p1 mnt
   ```
1. Copy the follwing files.
   ```sh
   $ sudo cp /mnt/boot/initrd.img-5.4.0-64-generic .
   $ sudo cp /mnt/boot/vmlinuz-5.4.0-64-generic .
   ```

## Run the VM
1. Copy the following files on the same directory.
   - efi-virtio.rom (qemu-5.2.0/pc-bios)
   - slof.bin (qemu-5.2.0/pc-bios)
   - initrd.img-5.4.0-64-generic
   - vmlinuz-5.4.0-64-generic
   - ubuntu-20.04-server-cloudimg-arm64.img
   ```
1. Run the following command.
   ```sh
   # qemu-system-aarch64 -cpu cortex-a72 -machine virt  -m 2048 \
   -kernel vmlinuz-5.4.0-64-generic \
   -append 'root=/dev/vda1 rw rootwait mem=1024M console=ttyS0 
   console=ttyAMA0,38400n8 init=/usr/lib/cloud-init/uncloud-init ds=nocloud' \
   -drive id=drive0,if=none,file=ubuntu-20.04-server-cloudimg-arm64.img \
   -initrd initrd.img-5.4.0-64-generic \
   -device virtio-blk-pci,id=scsi0,drive=drive0 -netdev user,id=user0 \
   -nodefaults -nographic -serial stdio -smp cpus=2 -net nic \
   -net tap,ifname=tap0,script=no -net socket,listen=localhost:1234
   ```
1. Login the virtual machine.
   ```sh
   ubuntu login: root
   Password: <enter root user password>
   ```
1. Enjoy!
   ```sh
   # arch
   aarch64
   ```
## Link
- https://www.mztn.org/dragon/arm64_01.html