# How to Run POWER9 on x86_64
## Index
- [Evaluation Environment](#evaluation-environment)
- [Download and Customize the CentOS Image](#download-and-customize-the-centos-image)
- [Build qemu-system-ppc64](#build-qemu-system-ppc64)
- [Run the VM](#run-the-vm)

## Evaluation Environment
```
+------------------------------------------+
| +--------------------------------------+ |
| | Virtual Machine                      | |
| | CentOS Linux release 8.2.2004 (Core) | |
| | +--------------------+               | |
| | | eth0 (192.168.2.x) |               | |
| | +--+-----------------+               | |
| +----|---------------------------------+ |
|      |                                   |
|   +--+-----------------+                 |
|   | tap0               |                 |
|   +--+-----------------+                 |
|      |                                   |
|   +--+-----------------+                 |
|   | br0 (192.1686.2.1) |                 |
|   +--------------------+                 |
|                                          |
| CentOS Linux release 7.8.2003 (Core)     |
+------------------------------------------+
```
## Download and Customize the CentOS Image
1. Install CentOS 8 on the other machine.
1. Install the following packages.
   ```sh
   # dnf install -y libguestfs-tools libvirt
   ```   
1. Downlowd CentOS cloud image.
   - https://cloud.centos.org/centos/8/ppc64le/images/
1. Run the following command to change the root password.
   ```sh
   # export LIBGUESTFS_BACKEND=direct
   virt-customize -a CentOS-8-GenericCloud-8.2.2004-20200611.2.ppc64le.qcow2  --root-password password:<your password>
   ```
## Build qemu-system-ppc64
1. Install KVM packages folloing the below steps.
   - https://github.com/EXPRESSCLUSTER/KVM/blob/master/KVMandWindowsServerSetup.md#kvm-setup.
1. Install the following packages.
   - Development tools
     ```sh
     # yum groupinstall "Development tools"
     ```
   - Python3
     ```sh
     # yum install python3
     ```
   - glib2-devel
     ```sh
     # yum install glib2-devel
     ```
   - pixman-devel
     ```sh
     # yum install pixman-devel
     ```
   - meson and ninja
     ```sh
     # pip3 --proxy <your proxy server> install meson ninja --user
     ```
     - Add bin directory of meson and ninja. 
       ```sh
       # vi /root/.bash_profile
        :
       PATH=$PATH:$HOME/bin:/root/.local/bin     
        :
       # source .bash_profile
       ```
1. Download the qemu binary file and expand it.
   ```sh
   # cd /root
   # mkdir work
   # cd /root/work
   # curl -O  https://download.qemu.org/qemu-5.0.0.tar.xz
   # tar xf qemu-5.0.0.tar.xz
   ```
1. Move to the qemu directory and build qemu binary.
   ```sh
   # cd qemu-5.0.0
   # ./configure --target-list=ppc64-softmmu
   ```
## Run the VM
1. Open ~/.bash_profile and add **/root/.local/bin** to PATH.
   ```sh
   PATH=$PATH:$HOME/bin:/root/.local/bin
   ```   
1. Enable PATH.
   ```sh
   # source ~/.bash_profile
   ```
1. Create the followin file.
   ```sh
   # vi /etc/qemu/bridge.conf
   allow virbr0
   ```
1. Make a directory and copy the following files.
   ```sh
   # mkdir centos8
   # cd centos8
   # cp <some directory>/CentOS-8-GenericCloud-8.2.2004-20200611.2.ppc64le.qcow2 .
   # cp /root/work/qemu-5.0.0/pc-bios/efi-virtio.rom .
   # cp /root/work/qemu-5.0.0/pc-bios/slof.bin .
   ```
1. Run the follwoing command.
   ```sh
   # qemu-system-ppc64 -machine pseries -cpu power9 -m 4096 -device virtio-blk-pci,id=scsi0,drive=drive0 -drive id=drive0,if=none,file=CentOS-8-GenericCloud-8.2.2004-20200611.2.ppc64le.qcow2  -nodefaults -nographic -serial stdio -smp cpus=2 -net nic -net tap,ifname=tap0 -net socket,listen=localhost:1234
   ```
   - **CAUTION**: DO NOT type Ctrl + C on the console. If you type it, VM shut down.
     - E.g. If you want to ping, you should add -c option.
       ```sh
       # ping -c 4 192.168.122.1
       ```
1. Login root user with your password.
   ```
    :
   [   87.400702] crypto_register_alg 'aes' = 0
   [   90.063969] crypto_register_alg 'cbc(aes)' = 0
   [   92.475134] crypto_register_alg 'ctr(aes)' = 0
   [  101.927418] crypto_register_alg 'xts(aes)' = 0
   [  176.516008] IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
   
   CentOS Linux 8 (Core)
   Kernel 4.18.0-193.6.3.el8_2.ppc64le on an ppc64le
   
   Activate the web console with: systemctl enable --now cockpit.socket
   
   localhost login:
   ```
1. Congraturations!
   ```sh
   # lscpu
   Architecture:        ppc64le
   Byte Order:          Little Endian
   CPU(s):              2
   On-line CPU(s) list: 0,1
   Thread(s) per core:  1
   Core(s) per socket:  1
   Socket(s):           2
   NUMA node(s):        1
   Model:               2.0 (pvr 004e 1200)
   Model name:          POWER9 (architected), altivec supported
   Hypervisor vendor:   KVM
   Virtualization type: para
   L1d cache:           32K
   L1i cache:           32K
   NUMA node0 CPU(s):   0,1
   ```
1. If you run the additional VM, run the following command.
   ```sh
   # qemu-system-ppc64 -machine pseries -cpu power9 -m 4096 -device virtio-blk-pci,id=scsi0,drive=drive0 -drive id=drive0,if=none,file=CentOS-8-GenericCloud-8.2.2004-20200611.2.ppc64le.qcow2  -nodefaults -nographic -serial stdio -smp cpus=2 -net nic,macaddr=52:54:00:12:34:57 -net socket,connect=localhost:1234
   ```
