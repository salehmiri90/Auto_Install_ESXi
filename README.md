# ESXi8u1 Automated Installation on Baremetal

This repository provides a fully automated solution for installing the ESXi8u1 operating system on HPE physical servers using iLO. With this code, you can effortlessly deploy ESXi servers by simply completing the variables and running the main playbook on your Ansible server. Say goodbye to manual installation hassles and hello to streamlined and efficient deployment!

In this code, I’m using Kickstart without PXE to do the ESXi version 8u1 installation over HPE Gen10 Baremetal and do the post configuration to set NTP and etc. 

I wrote custom roles to define the main playbook, so please star this code to easily find it later.

ESXi installation process can be simplified by means of Kickstart Installation method This method utilizes so called Kickstart File, which describes the configuration, required setup and post installation tasks for Kickstart installation.
Kickstart File can be placed in the remote repository, accessible via NFS, HTTP, FTP, etc…, or can be included in ISO image, which is pretty convenient to store a Kickstart File.

In this tutorial we downloaded original ESXi8u1 ISO image then run one playbook file which has two block and 7 roles to mount iso in the Linux file system, modify it by adding Kickstart File (ks.cfg) and re-pack it to create custom UEFI bootable ESXi8u1 ISO image using mkisofs command.

Notice: The OS disks on bare metal server must exist on the first bays on your physical servers.

My original ISO which I’m using to do below steps was included grub menu modifying as “ins.ks=cdrom:/KS.CFG”

## Playbook Steps
1st role: copy-iso-mount

Mounting and copy cdrom items will do in this role.

2nd role: vm-custome-boot

In this step, I remove boot.cfg file from root and efi/boot directories, then put custom boot file to both destinations.

3rd role: vm-ks

In this step the Kickstart content for each host will be create and after that I set some esxi post configuration in this file. For example disable IPv6, set ntp service up and run, set secondary dns ip address will do here. 

4th role: vm-gen-iso

Using mkisofs command to create an ISO and put on the specific path which can located by nginx webserver.

5th role: iso-uefi

Using isohybrid command to force iso to be compatible with uefi and bios methods. 

6th role: clean-stage

Clear the stage and delete unnecessary files except created ISO files. 

7th role: ilo-provisioning

Using group_vars to authenticate to HPE iLO and use nginx webserver path to mount related ISO to the iLO media.

## Requirements

Before using this automation code, make sure you have the following:

- An Ansible server with Ansible installed. If you don't have Ansible installed, refer to the official [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/index.html).

- Access to HPE servers with iLO functionality. Ensure that you have the necessary credentials and network connectivity to interact with the iLO interface.

- Familiarity with ESXi and the specific configuration requirements for your environment.

## Customization

Feel free to customize the playbook and variables according to your specific needs. You can modify network settings, storage configurations, and other parameters to align with your infrastructure requirements.

## Contributing

Contributions are welcome! If you have any improvements or suggestions, please feel free to submit a pull request. Ensure that your changes align with the existing coding style and include relevant documentation.

## License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and distribute this automation code as per the terms of the license.

---
