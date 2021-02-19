# How to Run ARM on x86_64
## Evaluation Environment
### Use Terminal Access Point (TAP)
- IP address will be assigned automatically. The first node IP address will be 192.168.122.76. and the second node IP address will be 192.168.122.77.
  ```
  +-----------------------------------------------------------------------------------+
  | +--------------------------------------+ +--------------------------------------+ |
  | | Virtual Machine                      | | Virtual Machine                      | |
  | | Ubuntu 20.04 LTS                     | | Ubuntu 20.04                         | |
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
