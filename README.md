# QEMU

## Ubuntu Server 22.04.2
- You don't need build the commands by yourself.
- [ARM on x86_64](doc/RunARMonUbuntu2204.md)
- [POWER on x86_64](doc/RunPOWERonUbuntu2204.md)
  - We couldn't run a virtual machine on POWER10 with QEMU 6.2.0. So, we have built qemu-system-ppc64 command.
    - [Build QEMU on Ubuntu 22.04](doc/BuildQEMUonUbuntu2204.md)

## CentOS 7.9
- You must build qemu-system commands by yourself.
- [ARM on x86_64](doc/HowToRunARMonX86_64.md)
  - [Amazon Linux 2 on ARM](doc/HowToRunAmazonLinux2onARM.md)
- [POWER9 on x86_64](doc/HowToRunPOWER9onX86_64.md)

## Link
- How to create virtual machines with each architecture on QEMU (written in Japanese)
  - https://hackmd.io/@bata24/ryWzOHEMw