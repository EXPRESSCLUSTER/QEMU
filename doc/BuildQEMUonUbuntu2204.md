# Build qemu-system Command on Ubuntu 22.04

## Index
- [How to Build the Command](#how-to-build-the-command)
- [Run Virtual Machine (POWER10)](#run-virtual-machines-power10)
- [Reference](#reference)

## How to Build the Command
1. Install the following packages.
   ```sh
   sudo apt-get -y install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build make gcc libcap-ng-dev libattr1-dev
   ```
   - See also: https://developer.ibm.com/tutorials/run-a-full-system-lop-env-from-ms-windows/
1. In addition, install the following packages, too.
   ```sh
   sudo apt install python3-pip
   ```
   ```sh
   sudo apt install flex
   ```
1. Clone the QEMU repository.
   ```sh
   git clone https://github.com/qemu/qemu.git
   ```
   - If you have a proxy server, you should run the following command.
     ```sh
     git config --global http.proxy <your proxy server>
     ```
     ```sh
     git config --global https.proxy <your proxy server>
     ```
1. Move to qemu directory.
1. Run configure.
   ```sh
   ./configure --target-list=ppc64-softmmu --enable-virtfs
   ```
1. Build the qemu-system-ppc64 command.
   ```sh
   make -j
   ```
1. You can find qemu-system-ppc64 command in the build directory.
   ```
   $ ./qemu-system-ppc64 --version
   QEMU emulator version 8.0.91 (v8.1.0-rc1-46-gccb86f079a)
   Copyright (c) 2003-2023 Fabrice Bellard and the QEMU Project developers
   ```

## Run Virtual Machines (POWER10)
### AlmaLinux OS 9.2
### First Node
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
1. Save the above files on the same directory as below.
   ```
   +--- alma9-101.qcow2
   +--- seedconfig
        +--- meta-data
        +--- seed.iso
        +--- user-data   
   ```   
1. Create run.sh as below.
   ```sh
   /home/user/work/GitHub/qemu/build/qemu-system-ppc64 \
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
   - Please change the first line dependes on your environment.
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
    Last login: Mon Jul 31 12:07:56 on hvc0
    [almalinux@alma9-101 ~]$ cat /proc/cpuinfo
    processor       : 0
    cpu             : POWER10 (architected), altivec supported
    clock           : 1000.000000MHz
    revision        : 2.0 (pvr 0080 1200)

    processor       : 1
    cpu             : POWER10 (architected), altivec supported
    clock           : 1000.000000MHz
    revision        : 2.0 (pvr 0080 1200)

    timebase        : 512000000
    platform        : pSeries
    model           : IBM pSeries (emulated by qemu)
    machine         : CHRP IBM pSeries (emulated by qemu)
    MMU             : Radix
    ```

## Reference
- https://developer.ibm.com/tutorials/run-a-full-system-lop-env-from-ms-windows/