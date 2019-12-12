# Packer VirtualBox OVF Generator for ACM Labs
The files in this directory specify the steps needed to create a
virtual machine with Bro, MongoDB, RITA, Wireshark, and the ACM labs.

# Packer
[Packer](https://www.packer.io/) is a tool for provisioning virtual machines. Packer can be installed by following the instructions at https://www.packer.io/intro/getting-started/install.html#precompiled-binaries.

## Building the Ubuntu 16.04 Lab VM
Once Packer is installed, simply run `packer build ubuntu-16.json`. The resulting vm will be located in `./output-ubuntu-16`.
The username and password for the generated vm are `ubuntu`.
The ACM labs can be found on the desktop and do not require internet access.
The virtual machine requires 4 GB of RAM and 2 CPU cores. These requirements are driven by Wireshark.