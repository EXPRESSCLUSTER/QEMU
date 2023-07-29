# How to Run POWER on Ubuntu (x86_64)
- This article describe how to run POWER virtual machine on Ubuntu Server running on x86_64.

## Index
- [Notes](#notes)
- [Configuration](#configuration)
- [Install Packages](#install-packages)
- [Run Virtual Machines (POWER9)](#run-virtual-machines-power9)
- [Run Virtual Machines (POWER10)](#run-virtual-machines-power10)
  - We have got OS panic :-(

## Notes
- This configuration uses TAP for guest OS network. You need TAP for each virtual machine.
- MAC address will be assigned automatically. The default one is  52:54:00:12:34:56. If you want to assign another one you need set MAC address (e.g., 52:54:00:12:34:57) manually.

## Configuration
```
+---------------------------------------------------------------+
| Host OS                                                       |
| Ubuntu Server 22.04.2 LTS                                     |
| QEMU 6.2.0                                                    |
|                                                               |
| +----------------------------+ +----------------------------+ |
| | Guest OS                   | | Guest OS                   | |
| | AlmaLinux OS 9.2 (ppc64le) | | AlmaLinux 9.2 OS (ppc64le) | |
| | MAC: 52:54:00:12:34:56     | | MAC: 52:54:00:12:34:57     | |
| | IP: 192.168.122.101        | | IP: 192.168.122.102        | |
| +--+-------------------------+ +--+-------------------------+ |
|    |                              |                           |
| +--+---+                       +--+---+                       |
| | tap0 |                       | tap1 |                       |
| +--+---+                       +--+---+                       |
|    |                              |                           |
| +--+------------------------------+---+                       |
| | virbr0 (192.168.122.1)              |                       |
| +--+----------------------------------+                       |
|    |                                                          |
| +--+---+                                                      |
+-| eth0 |------------------------------------------------------+
  +--+---+
     |
     :
```

## Install Packages
1. Install the following packages.
   ```sh
   sudo apt install -y qemu-system-ppc libvirt-daemon libvirt-daemon-system genisoimage
   ```

## Run Virtual Machines (POWER9)
### AlmaLinux OS 9.2
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
   mkdir -p qemu/ppc64le/alma9-101/seedconfig
   ```
1. Generate seed.iso on the seedconfig directory.
   1. meta-data
      ```
      network-interfaces: |
        auto eth0
        iface eth0 inet static
        address 192.168.122.101
        network 192.168.122.0
        netmask 255.255.255.0
        broadcast 192.168.122.255
        gateway 192.168.122.1
      ```
   1. user-data
      ```
      ssh_pwauth: yes
      password: cluster-0
      chpasswd:
        expire: false
      ```
   1. Create seed.iso.
      ```
      genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
      ```
      - See also: https://wiki.almalinux.org/cloud/Generic-cloud-on-local.html#cloud-init
1. Download qcow2 file, rename it (e.g., alma9-101.qcow2) and save it on the alma9-101 directory.
   - https://wiki.almalinux.org/cloud/Generic-cloud.html
1. Create an additional disk (e.g., md1.qcow2).
   ```sh
   qemu-img create -f qcow2 md1.qcow2 10G
   ```
1. Save the above files on the same directory as below.
   ```
   +--- alma9-101.qcow2
   +--- md1.qcow2
   +--- seedconfig
        +--- meta-data
        +--- seed.iso
        +--- user-data   
   ```   
1. Create run.sh as below.
   ```sh
   qemu-system-ppc64le \
   -M pseries \
   -cpu power9 \
   -m 2G \
   -smp cpus=2 \
   -nographic \
   -cdrom ./seedconfig/seed.iso \
   -device virtio-blk-pci,id=scsi0,drive=drive0 \
   -drive id=drive0,if=none,file=alma9-101.qcow2 \
   -nodefaults \
   -serial stdio \
   -net nic \
   -net tap,ifname=tap0,script=no
   ```
1. Run VM!
   ```sh
   ./run.sh
   ```
1. Enjoy!
   ```
   AlmaLinux 9.2 (Turquoise Kodkod)
   Kernel 5.14.0-284.11.1.el9_2.ppc64le on an ppc64le
   
   Activate the web console with: systemctl enable --now cockpit.socket
   
   alma9-101 login: almalinux
   Password:
   Last login: Fri Jul 28 22:35:31 from 192.168.122.1
   [almalinux@alma9-101 ~]$ arch
   ppc64le
   [almalinux@alma9-101 ~]$ cat /proc/cpuinfo
   processor       : 0
   cpu             : POWER9 (architected), altivec supported
   clock           : 1000.000000MHz
   revision        : 2.0 (pvr 004e 1200)
   
   processor       : 1
   cpu             : POWER9 (architected), altivec supported
   clock           : 1000.000000MHz
   revision        : 2.0 (pvr 004e 1200)
   
   timebase        : 512000000
   platform        : pSeries
   model           : IBM pSeries (emulated by qemu)
   machine         : CHRP IBM pSeries (emulated by qemu)
   MMU             : Radix
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
   qemu-system-ppc64le \
   -M pseries \
   -cpu power9 \
   -m 2G \
   -smp cpus=2 \
   -nographic \
   -cdrom ./seedconfig/seed.iso \
   -device virtio-blk-pci,id=scsi0,drive=drive0 \
   -drive id=drive0,if=none,file=alma9-101.qcow2 \
   -nodefaults \
   -serial stdio \
   -net nic,macaddr=52:54:00:12:34:57 \
   -net tap,ifname=tap1,script=no
   ```
1. Run VM!
   ```sh
   ./run.sh
   ```

## Run Virtual Machines (POWER10)
1. Create run.sh as below.
   ```sh
   qemu-system-ppc64le \
   -M pseries \
   -cpu power10 \
   -m 2G \
   -smp cpus=2 \
   -nographic \
   -cdrom ./seedconfig/seed.iso \
   -device virtio-blk-pci,id=scsi0,drive=drive0 \
   -drive id=drive0,if=none,file=alma9-101.qcow2 \
   -nodefaults \
   -serial stdio \
   -net nic \
   -net tap,ifname=tap0,script=no
   ```
1. Run VM!
   ```sh
   ./run.sh
   ```
1. We have got OS panic...
   ```
   [    5.461792] Run /init as init process
   [    5.822257] systemd[1]: illegal instruction (4) at 7fff86928fd8 nip 7fff86928fd8 lr 7fff868c8188 code 1 in libc.so.6[7fff86840000+240000]
   [    5.825248] systemd[1]: code: 4bffff78 48000014 60000000 60000000 60000000 60000000 7c852378 10259006
   [    5.825931] systemd[1]: code: 10449006 10679006 10869006 7ca53214 <10e80e42> 11081642 11281e42 11482642
   [    5.842785] Core dump to |/bin/false pipe failed
   [    5.844513] Kernel panic - not syncing: Attempted to kill init! exitcode=0x00000004
   [    5.845414] CPU: 1 PID: 1 Comm: systemd Not tainted 5.14.0-284.11.1.el9_2.ppc64le #1
   [    5.846554] Call Trace:
   [    5.846876] [c00000000354f9f0] [c000000000844d40] dump_stack_lvl+0x74/0xa8 (unreliable)
   [    5.849413] [c00000000354fa30] [c000000000149294] panic+0x160/0x3ec
   [    5.849871] [c00000000354fad0] [c000000000152df4] do_exit+0x554/0x560
   [    5.850298] [c00000000354fb70] [c000000000152fac] do_group_exit+0x4c/0xd0
   [    5.850748] [c00000000354fbb0] [c000000000168edc] get_signal+0xc8c/0xcc0
   [    5.851177] [c00000000354fca0] [c0000000000205dc] do_signal+0x7c/0x320
   [    5.851608] [c00000000354fd40] [c000000000021710] do_notify_resume+0xb0/0x140
   [    5.852070] [c00000000354fd70] [c00000000002f308] interrupt_exit_user_prepare_main+0x198/0x270
   [    5.852595] [c00000000354fde0] [c00000000002f96c] interrupt_exit_user_prepare+0x5c/0xc0
   [    5.853122] [c00000000354fe10] [c00000000000c774] interrupt_return_srr_user+0x8/0x138
   [    5.853734] --- interrupt: 700 at 0x7fff86928fd8  
   ```