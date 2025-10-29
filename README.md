# ESXi8u1 Automated Installation on Baremetal
This repository provides a fully automated solution for installing the ESXi8u1 operating system on HPE physical servers using iLO. With this code, you can effortlessly deploy ESXi servers by simply completing the variables and running the main playbook on your Ansible server. Say goodbye to manual installation hassles and hello to streamlined and efficient deployment!

### 📖 Full Article on Medium
[Medium/salehmiri90](https://medium.com/@salehmiri90/production-ready-esxi-deployment-by-ansible-without-human-interaction-a32011689a49) 

###  Video Demo on Youtube

## Description
In this code, I’m using Kickstart without PXE to do the ESXi version 8u1 installation over HPE Gen10 Baremetal and do the post configuration to set NTP and etc. 
I wrote custom roles to define the main playbook, so please star this code to easily find it later.

## Technologies and Tools
ESXi installation process can be simplified by means of Kickstart Installation method This method utilizes so called Kickstart File, which describes the configuration, required setup and post installation tasks for Kickstart installation.
Kickstart File can be placed in the remote repository, accessible via NFS, HTTP, FTP, etc…, or can be included in ISO image, which is pretty convenient to store a Kickstart File.

## How it's works
In this tutorial we downloaded original ESXi8u1 ISO image then run one playbook file which has two block and 7 roles to mount iso in the Linux file system, modify it by adding Kickstart File `ks.cfg` and re-pack it to create custom UEFI bootable ESXi8u1 ISO image using `mkisofs` command.

My original ISO which I’m using to do below steps was included grub menu modifying as `ins.ks=cdrom:/KS.CFG`

## Why use this project?
⭐ Automate the VM creation and OS installation process

⭐ Save time and effort

⭐ Reduce the risk of human error

### Project Requirements
⭐ A server running the Linux operating system is required to function as the control node. (This codes had been tested on Red Hat 8.6).

⭐ Install Ansible on your control node.

⭐ Install `pyvmomi` and configure it on control node.

⭐ Install VMware SDK for integration.
````
pip install git+https://github.com/vmware/vsphere-automation-sdk-python.git
````
⭐ Grant administrative privileges to a user on the VMware environment from the Ansible control node.

## Start to Use this code
### Step 1: Transfer codes to you Ansible Server
&#9745; To clone this repository from my GitHub using the command line, you can use the following command:
````
git clone https://github.com/salehmiri90/VMware_Automation.git
````

&#9745; Use the 'mv' command to move the contents of the cloned directory 'VMware_Automation' to '/etc/ansible' as follows: 
````
mv -r VMware_Automation/Auto_Install_ESXi/* /etc/ansible/
````

&#9745; Verify that the files exist in the destination path '/etc/ansible' by first changing to the directory using the 'cd' command: 
````
cd /etc/ansible/
````
And then listing the contents with the command: 
````
ll
````
&#9745; Set a hostname for your server on your control node's hosts file located in `vi /etc/hosts`. For example:
````
192.168.1.1  ilo-esxi
````

&#9745; Verify that the DNS name 'ilo-esxi' is properly set on your control node by attempting to ping it using the command:
````
ping ilo-esxi
````
### Step 2: Defining Hosts Variables
&#9745; In the Ansible hosts inventory located in the below path, place the name of your server, 'ilo-esxi', under [ilo-esxi].
````
vi /etc/ansible/inventory/hosts
````
````yml
[ilo-esxi]
ilo-esxi
````
&#9745; In the Ansible inventory esxi host variables located in the below path, place details about esxi hosts which you want to install OS on those.
````
vi /etc/ansible/inventory/host_vars/ilo-esxi.yml
````
````yml
hosts:
  - hostName: srv33.saleh.miri.local
    esxi_ip: 1.6.29.9
    ilo_ip: 1.18.66.9

  - hostName: srv34.saleh.miri.local
    esxi_ip: 1.6.29.10
    ilo_ip: 1.18.66.10
````

### Step 3: Defining Group Variables
&#9745; Modify the esxi hosts and iLO authentication details in the below directory. In this file, update the `esxi root password`, `esxi vlan id`, `esxi netmask`, `esxi dns`, `esxi ntp`, `ilo password` and `name of iso file` parameters as I have done.
````
vi /etc/ansible/inventory/group_vars/ilo-esxi.yml
````
````yml
root_password: 'password'
global_vlan_id: 1146
global_netmask: 255.255.255.0
global_gw: 1.6.20.26
global_dns1: 1.6.20.1
global_dns2: 1.6.20.2
global_ntp1: 1.6.20.4
global_ntp2: 1.6.20.5

ilo_pass: password

src_iso_file: VMware-ESXi-8.0.1.iso
````

### Step 4: Running Playbooks 
&#9745; There is only one playbook that needs to be executed.

&#9745; Navigate to the playbook directory using the command:
````
cd /etc/ansible/playbooks/
````
Notice❗️ The physical HPE servers have to be powered off before execute below command.

&#9745; Then execute all parts with a single command using 
````
ansible-playbook ilo_iso_esxi.yaml
````

## Playbook Steps
&#9745; 1st role: copy-iso-mount

Mounting and copy cdrom items will do in this role.

````yml
mkdir /mnt/{{ item.hostName }}
mount -o loop -t iso9660 {{ isosrc }}/{{ src_iso_file }} /mnt/{{ item.hostName }}/
mkdir {{ file_path }}/{{ item.hostName }}
cp -avRf /mnt/{{ item.hostName }}/* {{ file_path }}/{{ item.hostName }}/
````
&#9745; 2nd role: vm-custome-boot

In this step, I remove boot.cfg file from root and efi/boot directories, then put custom boot file to both destinations.

````yml
rm -f /home/deploy/baremetal/{{ item.hostName }}/boot.cfg
rm -f /home/deploy/baremetal/{{ item.hostName }}/efi/boot/boot.cfg
````

````yml
copy:
          src: BOOT.CFG
          dest: /home/deploy/baremetal/{{ item.hostName }}/
          remote_src: no
          owner: root
          group: root
          mode: '0744'
````

````yml
copy:
          src: BOOT.CFG
          dest: /home/deploy/baremetal/{{ item.hostName }}/efi/boot/
          remote_src: no
          owner: root
          group: root
          mode: '0744'
````

&#9745; 3rd role: vm-ks

In this step the Kickstart content for each host will be create and after that I set some esxi post configuration in this file. For example disable IPv6, set ntp service up and run, set secondary dns ip address will do here. 

````yml
  copy:
          force: yes
          dest: /home/deploy/baremetal/{{ item.hostName }}/KS.CFG
          content: |
                  vmaccepteula
                  clearpart --firstdisk=local --overwritevmfs
                  install --firstdisk=local --overwritevmfs
                  rootpw {{ root_password }}
                  network --bootproto=static --addvmportgroup=1 --vlanid={{ global_vlan_id }} --ip={{ item.esxi_ip }} --netmask={{ global_netmask }} --gateway={{ global_gw }} --nameserver={{ global_dns1 }} --hostname={{ item.hostName }}
                  %firstboot --interpreter=busybox
                  vim-cmd hostsvc/enable_ssh
                  vim-cmd hostsvc/start_ssh
                  vim-cmd hostsvc/enable_esx_shell
                  vim-cmd hostsvc/start_esx_shell
                  esxcli system module parameters set -m tcpip6 -p ipv6=0
                  esxcli network ip set --ipv6-enabled=false
                  esxcli system ntp set --server={{ global_ntp1 }} --server={{ global_ntp2 }}
                  esxcli network ip dns server add --server={{ global_dns2 }}
                  esxcli system ntp set --enabled=true
                  esxcli system ntp start
                  #esxcli system settings advanced set -o /UserVars/HostClientCEIPEnabled -i 0
                  reboot
````

&#9745; 4th role: vm-gen-iso

Using mkisofs command to create an ISO and put on the specific path which can located by nginx webserver.

````yml
  shell: >
          mkisofs
          -o {{ iso_path }}/{{ item.hostName }}.iso
          -relaxed-filenames
          -J
          -R
          -b isolinux.bin
          -c boot.cat
          -no-emul-boot
          -boot-load-size 4
          -boot-info-table
          -eltorito-alt-boot
          -e efiboot.img
          -boot-load-size 1
          -no-emul-boot
          "{{ file_path }}"/{{ item.hostName }}/
````
&#9745; 5th role: iso-uefi

Using isohybrid command to force iso to be compatible with uefi and bios methods. 

````yml
sudo isohybrid --uefi {{ iso_path }}/{{ item.hostName }}.iso
````

&#9745; 6th role: clean-stage

Clear the stage and delete unnecessary files except created ISO files. 

````yml
sudo umount /mnt/{{ item.hostName }}
sudo rm -rf {{ file_path }}/{{ item.hostName }}
sudo rm -rf /mnt/*
````
&#9745; 7th role: ilo-provisioning

Using group_vars to authenticate to HPE iLO and use nginx webserver path to mount related ISO to the iLO media.

````yml
  hpilo_boot:
          host: "{{ item.ilo_ip }}"
          login: "{{ ilo_user }}"
          password: "{{ ilo_pass }}"
          state: "{{ ilo_state }}"
          media: cdrom
          image: http://salehmiri.com:443/{{ item.hostName }}.iso
  delegate_to: localhost
````

# ✍️ Contribution
I am confident that working together with skilled individuals like yourself can improve the functionality, efficiency, and overall quality of our projects. Therefore, I would be delighted to see any forks from this project. Please feel free to use this code and share any innovative ideas to enhance it further.

## ☎️ Contact information
### 📧 salehmiri90@gmail.com
### [Linkedin.com/in/salehmiri](https://www.linkedin.com/in/salehmiri)
