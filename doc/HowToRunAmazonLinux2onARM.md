# How to Run Amazon Linux 2 on ARM

## Evaluation Environment
### Use Terminal Access Point (TAP)
<!--
- IP address will be assigned automatically. The first virtual machine's IP address will be 192.168.122.76. and the second virtual machine's IP address will be 192.168.122.77.
-->
  ```
  +-----------------------------------------------------------------------------------+
  | +--------------------------------------+ +--------------------------------------+ |
  | | Virtual Machine (vm1)                | | Virtual Machine (vm2)                | |
  | | Amazon Linux 2                       | | Amazon Linux 2                       | |
  | | +------------------------+           | | +------------------------+           | |
  | | | eth0 (192.168.122.151) |           | | | eth0 (192.168.122.152) |           | |
  | | +--+---------------------+           | | +--+---------------------+           | |
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
- [Build qemu-system-aarch64 command](./HowToRunARMonX86_64.md#build-qemu-system-aarch64).
- Download Amazon Linux 2 image.
  - https://cdn.amazonlinux.com/os-images/2.0.20221103.3/kvm-arm64/

## Create seed.iso
1. Make seedconfig directory and move to that directory.
   ```sh
   mkdir seedconfig
   ```
1. Create meta-data as below. You can assign any IP address.
   ```yaml
   local-hostname: vm_hostname
   # eth0 is the default network interface enabled in the image. You can configure static network settings with an entry like the following.
   network-interfaces: |
     auto eth0
     iface eth0 inet static
     address 192.168.122.151
     network 192.168.122.0
     netmask 255.255.255.0
     broadcast 192.168.122.255
     gateway 192.168.122.1
   ```
1. Create user-data as below.
   ```yaml
   #cloud-config
   #vim:syntax=yaml
   users:
   # A user by the name `ec2-user` is created in the image by default.
     - default
   chpasswd:
     list: |
       ec2-user:plain_text_password
   # In the above line, do not add any spaces after 'ec2-user:'.
   ```
1. Create seed.iso.
   ```sh
   genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
   ```

## Run Virtual Machines
1. Save the following files on the same directory.
   ```
   -rw-r--r-- 1 root root 1.3G Dec 10 06:35 amzn2-kvm-2.0.20221103.3-arm64.xfs.gpt.qcow2
   -rw-r--r-- 1 root root 157K Dec  9 22:13 efi-virtio.rom
   -rw-r--r-- 1 root root  64M Dec  9 22:13 QEMU_EFI.img
   -rw-r--r-- 1 root root 366K Dec 10 08:44 seed.iso   
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
   -smp cpus=2 \
   -bios QEMU_EFI.img \
   -drive if=none,file=amzn2-kvm-2.0.20221103.3-arm64.xfs.gpt.qcow2,id=hd0 \
   -device virtio-blk-device,drive=hd0 \
   -nographic \
   -net nic \
   -net tap,ifname=tap0,script=no \
   -net socket,listen=localhost:1234
   ```
1. Enjoy aarch64!
   ```
   [ec2-user@al2-01 kernels]$ lscpu
   Architecture:        aarch64
   CPU op-mode(s):      32-bit, 64-bit
   Byte Order:          Little Endian
   CPU(s):              2
   On-line CPU(s) list: 0,1
   Thread(s) per core:  1
   Core(s) per socket:  2
   Socket(s):           1
   NUMA node(s):        1
   Vendor ID:           ARM
   Model:               0
   Model name:          Cortex-A57
   Stepping:            r1p0
   BogoMIPS:            125.00
   NUMA node0 CPU(s):   0,1
   Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
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
   -smp cpus=2 \
   -bios QEMU_EFI.img \
   -drive if=none,file=amzn2-kvm-2.0.20221103.3-arm64.xfs.gpt.qcow2,id=hd0 \
   -device virtio-blk-device,drive=hd0 \
   -nographic \
   -net nic,macaddr=52:54:00:12:34:57 \
   -net tap,ifname=tap1,script=no \
   -net socket,listen=localhost:1235
   ```
