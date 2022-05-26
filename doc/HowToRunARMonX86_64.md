# How to Run ARM on x86_64

## Index
- [Evaluation Environment](#evaluation-environment)
- [Prerequisite](#prerequisite)
- [Build qemu-system-aarch64](#build-qemu-system-aarch64)
- [Customize Cloud Image File](#customize-cloud-image-file)
- [Run Virtual Machines](#run-virtual-machines)
- [Link](#link)

## Evaluation Environment
### Use Terminal Access Point (TAP)
- IP address will be assigned automatically. The first virtual machine's IP address will be 192.168.122.76. and the second virtual machine's IP address will be 192.168.122.77.
  ```
  +-----------------------------------------------------------------------------------+
  | +--------------------------------------+ +--------------------------------------+ |
  | | Virtual Machine (vm1)                | | Virtual Machine (vm2)                | |
  | | CentOS Linux release 7.8.2003        | | CentOS Linux release 7.8.2003        | |
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
<!--
  ```
  +-----------------------------------------------------------------------------------+
  | +--------------------------------------+ +--------------------------------------+ |
  | | Virtual Machine                      | | Virtual Machine                      | |
  | | CentOS 7.6                           | | CentOS 7.6                           | |
  | |  or                                  | |  or                                  | |
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
-->  
## Prerequisite
- Install some OS (e.g. CentOS) on a x86_64 virtual machine (called vm0) to change root password.

## Build qemu-system-aarch64
1. Install [many packages](https://github.com/EXPRESSCLUSTER/QEMU/blob/master/doc/HowToRunPOWER9onX86_64.md#build-qemu-system-ppc64).
1. Download the qemu binary file and expand it.
   ```sh
   cd /root
   mkdir work
   cd /root/work
   curl -O  https://download.qemu.org/qemu-5.2.0.tar.xz
   tar xf qemu-5.2.0.tar.xz
   ```
1. Move to the qemu directory and build qemu binary.
   ```sh
   cd qemu-5.2.0
   ./configure --target-list=aarch64-softmmu
   make
   ```
## Customize Cloud Image File
1. Download a cloud image file.
   - https://cloud.centos.org/centos/7/images/
     - I have used CentOS-7-aarch64-GenericCloud-2003.qcow2.
1. Start vm0 and copy CentOS-7-aarch64-GenericCloud-2003.qcow2 to vm0.
1. Run the following command on vm0.
   ```sh
   yum install -y libguestfs-tools libvirt
   ```
1. Change the password for root.
   ```sh
   export LIBGUESTFS_BACKEND=direct
   virt-customize -a CentOS-7-aarch64-GenericCloud-2003.qcow2 --root-password password:<your passowrd>
   ```

## Run Virtual Machines
1. Create a directory (e.g. /vm1/centos7-aarch64-01) to save some files.
1. Copy the custoized CentOS-7-aarch64-GenericCloud-2003.qcow2 to /vm1/centos7-aarch64-01.
1. Copy efi-virtio.rom (in qemu-5.2.0/pc-bios) to /vm1/centos7-aarch64-01.
1. Download QEMU_EFI.img.gz from the following web site.
   - http://snapshots.linaro.org/components/kernel/leg-virt-tianocore-edk2-upstream/latest/QEMU-AARCH64/RELEASE_GCC5/QEMU_EFI.img.gz
1. Check if the following files are on the same directory.
   ```sh
   ll -h
   total 945M
   -rw-r--r-- 1 root root 881M Apr  1 17:04 CentOS-7-aarch64-GenericCloud-2003.qcow2
   -rw-r--r-- 1 root root 157K Apr  1 17:00 efi-virtio.rom
   -rw-r--r-- 1 root root  64M Apr  1 16:59 QEMU_EFI.img
   ```
1. Run the following commands to add a network interface for the second virtual machine (vm1).
   ```sh
   ip tuntap add tap0 mode tap
   ip tuntap show tap0
   ifconfig tap0 0.0.0.0 promisc up
   brctl addif virbr0 tap0
   ```
1. Run the following command to start the first virtual machine (vm1).
   ```sh
   qemu-system-aarch64 \
   -M virt \
   -cpu cortex-a57 \
   -m 2G \
   -smp cpus=4 \
   -bios QEMU_EFI.img \
   -drive if=none,file=CentOS-7-aarch64-GenericCloud-2003.qcow2,id=hd0 \
   -device virtio-blk-device,drive=hd0 \
   -nographic \
   -net nic \
   -net tap,ifname=tap0,script=no \
   -net socket,listen=localhost:1234
   ```
1. Enjoy aarch64!
   ```
   [root@localhost ~]# lscpu
   Architecture:          aarch64
   Byte Order:            Little Endian
   CPU(s):                4
   On-line CPU(s) list:   0-3
   Thread(s) per core:    1
   Core(s) per socket:    4
   Socket(s):             1
   NUMA node(s):          1
   Model:                 0
   BogoMIPS:              125.00
   NUMA node0 CPU(s):     0-3
   Flags:                 fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
   [root@localhost ~]# arch
   aarch64
   ```
1. Run the following commands to add a network interface for the second virtual machine (vm2).
   ```sh
   ip tuntap add tap1 mode tap
   ip tuntap show tap1
   ifconfig tap1 0.0.0.0 promisc up
   brctl addif virbr0 tap1
   ```
1. Run the second virtual machine (vm2).
   ```sh
   qemu-system-aarch64 \
   -M virt \
   -cpu cortex-a57 \
   -m 2G \
   -smp cpus=4 \
   -bios QEMU_EFI.img \
   -drive if=none,file=CentOS-7-aarch64-GenericCloud-2003.qcow2,id=hd0 \
   -device virtio-blk-device,drive=hd0 \
   -nographic \
   -net nic,macaddr=52:54:00:12:34:57 \
   -net tap,ifname=tap1,script=no \
   -net socket,listen=localhost:1235
   ```

<!--
## Ubuntu
### Customize Ubuntu Image File
1. Download a cloud image file.
   - https://cloud-images.ubuntu.com/releases/focal/release/
1. Change the password for root.
   ```sh
   virt-customize -a ubuntu-20.04-server-cloudimg-arm64.img --root-password password:<your password>
   ```

### Get initrd and Kernel File
1. Copy ubuntu-20.04-server-cloudimg-arm64.img to Ubuntu machine.
1. Login Ubuntu machine.
1. Run the following commands to mount the image file.
   ```sh
   sudo modprobe nbd max_part=63
   sudo qemu-nbd -c /dev/nbd0 ubuntu-20.04-server-cloudimg-arm64.img
   sudo mount /dev/nbd0p1 mnt
   ```
1. Copy the follwing files.
   ```sh
   sudo cp /mnt/boot/initrd.img-5.4.0-64-generic .
   sudo cp /mnt/boot/vmlinuz-5.4.0-64-generic .
   ```

### Run the VM
1. Copy the following files on the same directory.
   - efi-virtio.rom (qemu-5.2.0/pc-bios)
   - slof.bin (qemu-5.2.0/pc-bios)
   - initrd.img-5.4.0-64-generic
   - vmlinuz-5.4.0-64-generic
   - ubuntu-20.04-server-cloudimg-arm64.img
1. Run the following command.
   ```sh
   qemu-system-aarch64 -cpu cortex-a72 -machine virt  -m 2048 \
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
   arch
   aarch64
   ```
-->
## Link
- https://ideal-reality.com/computer/linux/qemu-alpine-virt/
- https://www.mztn.org/dragon/arm64_01.html
- https://astr0baby.wordpress.com/2019/01/13/enabling-kvm-in-aarch64-debian-9-6-for-accelerated-virtualization-of-centos-7-6-aarch64/
- https://www.os-museum.com/qemutapnet/qemutapnet.htm