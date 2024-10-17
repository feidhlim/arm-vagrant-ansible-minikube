**Vagrant and Ansible Setup for Minikube on Ubuntu 22 ARM with Podman driver**

This project provides a Vagrant setup with an Ansible playbook for provisioning a virtual machine running Ubuntu 22.04 on ARM architecture provided through VMware-fusion. 

The goal is to install and configure Minikube to run with the Podman driver.

## **Table of Contents**

* Prerequisites  
* Vagrantfile Configuration  
* Usage  
* Common Errors and Solutions  
* Notes

## **Prerequisites**

* Vagrant installed on your system.  
* VMware Fusion installed and configured.  
* Ansible installed on your system.

**VAGRANT**  
**$ brew install vagrant**

**$ vagrant global-status**   
\# shows which provider has been used to provision the vm

**Vagrant Plugins:**  
[Vagrant VMware Utility Installation](https://developer.hashicorp.com/vagrant/docs/providers/vmware/vagrant-vmware-utility)  
**$ vagrant plugin list**  
vagrant-vbguest (0.32.0, global)  
**vagrant-vmware-desktop (3.0.4, global)**

**Vagrant Boxes:**   
[https://portal.cloud.hashicorp.com/vagrant/discover](https://portal.cloud.hashicorp.com/vagrant/discover)  
**gyptazy\_ubuntu22.04-arm64**  
**gutehall/debian12**

**VMWare-fusion 13 Pro** 

Broadcom entitlements discussion:  
[**https://community.broadcom.com/discussion/vmware-fusion-13-pro-download-links**](https://community.broadcom.com/discussion/vmware-fusion-13-pro-download-links)

Workaround for download of VMWare-fusion without entitlements:  
[**https://williamlam.com/2024/05/no-entitlements-needed-for-these-popular-vmware-downloads-on-broadcom-support-portal-bsp.html**](https://williamlam.com/2024/05/no-entitlements-needed-for-these-popular-vmware-downloads-on-broadcom-support-portal-bsp.html)

Navigation: My Downloads-\>VMware Fusion-\>VMware Fusion Pro for Personal Use  
[**https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Fusion\&displayGroup=VMware%20Fusion%2013%20Pro%20for%20Personal%20Use\&release=13.5.2\&os=\&servicePk=520445\&language=EN**](https://support.broadcom.com/group/ecx/productfiles?subFamily=VMware%20Fusion&displayGroup=VMware%20Fusion%2013%20Pro%20for%20Personal%20Use&release=13.5.2&os=&servicePk=520445&language=EN)

**ANSIBLE**  
**$ brew install ansible**  
brew install ansible failed with: \==\> Fetching pkg-config \==\> Downloading https://ghcr.io/v2/homebrew/core/pkg-config/blobs/sha256:xyz 100.0%   
Error: python@3.12: the bottle needs the Apple Command Line Tools to be installed.   
You can install them, if desired, with: **xcode-select \--install** ...   
xcode-select: note: install requested for command line developer tools  
**$ xcode-select \--install**  
**$ xcode-select \-p**  
**$ brew install ansible**

**(base) x@macOS % ansible-playbook \-i inventory.ini playbook.yml**  
fatal: \[ubuntu22.04\]: UNREACHABLE\! \=\> {"changed": false, "msg": "Failed to connect to the host via ssh: vagrant@127.0.0.1: Permission denied (publickey,password).", "unreachable": true}  
fatal: \[127.0.0.1\]: UNREACHABLE\! \=\> {"changed": false, "msg": "Failed to connect to the host via ssh: no such identity: /gyptazy\_ubuntu22.04-arm64/.vagrant/machines/default/vmware\_fusion/private\_key:   
No such file or directory\\r\\nvagrant@127.0.0.1: Permission denied (publickey,password).", "unreachable": true}  
fatal: \[ubuntu22\]: FAILED\! \=\> {"msg": "to use the 'ssh' connection type with passwords or pkcs11\_provider,  
 **you must install the sshpass program**"}  
[https://stackoverflow.com/questions/42835626/ansible-to-use-the-ssh-connection-type-with-passwords-you-must-install-the-s](https://stackoverflow.com/questions/42835626/ansible-to-use-the-ssh-connection-type-with-passwords-you-must-install-the-s)  
**$ brew install hudochenkov/sshpass/sshpass**

## **Vagrantfile Configuration**

The following is the configuration used for the Vagrantfile, which sets up the Ubuntu VM and provisions it using Ansible:

  `# Ubuntu VM Configuration`  
  `config.vm.define "ubuntu22" do |ubuntu|`  
    `ubuntu.vm.hostname = "ubuntu22"`  
    `ubuntu.vm.box = "gyptazy/ubuntu22.04-arm64"`  
    `ubuntu.vm.provider "vmware_fusion" do |vb|`  
      `vb.memory = 4096`  
      `vb.cpus = 2`  
    `end`  
  `end`

  `config.vm.provision "ansible" do |ansible|`  
    `ansible.playbook = "ansible/playbook.yml"`  
    `ansible.inventory_path = "ansible/inventory.ini"`  
  `end`

## **Usage**

To provision the VM and install Minikube with Podman, run:

`vagrant up`

If you make changes to the playbook and want to reapply the provisioning, use:

`vagrant provision`

## **Common Errors and Solutions**

### **Error: CNI config file validation failure**

`Error validating CNI config file /etc/cni/net.d/minikube.conflist: plugin bridge does not support config version "1.0.0"...`

* **Solution**: Refer to [GitHub Issue \#17754](https://github.com/kubernetes/minikube/issues/17754) for more information.

### **Error: Downgraded Packages for Podman**

`Error: Packages were downgraded and -y was used without --allow-downgrades`

* **Solution**: Modify the playbook to include `allow_downgrade: yes` and `force_apt_get: yes` for Podman installation.

### **Error: Podman driver used with root privileges**

`Error: The "podman" driver should not be used with root privileges`

* **Workaround**: Ensure that the Minikube command is executed without escalating privileges:

`- name: Start Minikube`  
  `ansible.builtin.command: minikube start --driver=podman`  
  `become: false`  
  `become_user: vagrant`  
  `environment:`  
    `MINIKUBE_HOME: /home/vagrant`

### **Unstable SSH connection**

* **Solution**: Update `/etc/ssh/sshd_config` on the server and include:

`IPQoS lowdelay throughput`

## Minikube on Ubuntu for ARM:

**Manual setup / Teething:**

macOS m1 arm64 VMWare, vagrant Ubuntu22.04 VM for arm \+ minkube

You can manually verify the setup and architecture as follows:  
**vagrant@vagrant**:**/usr/local/bin**$ arch  
aarch64  
curl \-LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64  
sudo install minikube-linux-arm64 /usr/local/bin/minikube  
minikube version  
minikube version: v1.34.0  
commit: 210b148df93a80eb872ecbeb7e35281b3c582c61

Do You Need Extra Drivers?  
Minikube requires a driver to manage virtual machines. Since you're running the Ubuntu VM in VMware, the VM might already have a hypervisor or driver, but Minikube typically uses:

KVM for virtualization on Linux.  
Alternatively, you can use the none driver to run Minikube directly on your Ubuntu VM without additional virtualization.  
To install KVM and the required dependencies, you can run: ?

sudo apt install \-y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils  
sudo usermod \-aG libvirt $(whoami)  
If you prefer to use the none driver (which runs Kubernetes directly on your VM without creating an additional virtual machine), you can do so with:  
minikube start \--driver=none  
. /etc/os-release  
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/${ID}\_${VERSION\_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list  
curl \-L "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/${ID}\_${VERSION\_ID}/Release.key" | sudo apt-key add \-

sudo apt-get install podman

dpkg \-l | grep conntrack  
**sudo apt-get install conntrack**  
conntrack \--version

**(base) x@macOS gyptazy\_ubuntu22.04-arm64 % ssh \-i**   
Host key verification failed.  
**Delete entries from /Users/x/.ssh/known\_hosts**

## ARM Minikube Podman driver

**vagrant@ubuntu22**:**\~**$ **minikube start \--driver=podman**  
**Error validating CNI config file /etc/cni/net.d/minikube.conflist: plugin bridge does not support config version "1.0.0"**  
[https://github.com/kubernetes/minikube/issues/17754](https://github.com/kubernetes/minikube/issues/17754)  
**vagrant@ubuntu22**:**\~**$ apt list \--all-versions podman  
Listing... Done  
podman/jammy-updates,jammy-security,now 3.4.4+ds1-1ubuntu1.22.04.2 amd64 \[installed\]  
podman/jammy 3.4.4+ds1-1ubuntu1 amd64  
**sudo apt install podman=3.4.4+ds1-1ubuntu1**  
**minikube delete \--all**  
podman/jammy-updates,jammy-security 3.4.4+ds1-1ubuntu1.22.04.2 arm64 \[upgradable from: 3.4.4+ds1-1ubuntu1\]  
podman/jammy,now 3.4.4+ds1-1ubuntu1 arm64 \[installed,upgradable to: 3.4.4+ds1-1ubuntu1.22.04.2\]

**Error:** **Packages were downgraded and \-y was used without \--allow-downgrades**  
   \- name: Ensure Podman is installed at a specific version  
       allow\_downgrade: yes \# \--allow-downgrades  
       force\_apt\_get: yes 

**Error: The \\"podman\\" driver should not be used with root privileges**  
vagrant/ansible provision using the podman driver together assumes root privileges.  
workaround:  
   \- name: Start Minikube  
     ansible.builtin.command: minikube start \--driver=podman  
     become: false     \# Ensures it doesn't escalate privileges to root  
     become\_user: vagrant  \# Run the command as the vagrant user  
     environment:  
       MINIKUBE\_HOME: /home/vagrant

(base) x@macOS gyptazy\_ubuntu22.04-arm64 % vagrant reload

**vagrant@ubuntu22**:**\~**$ minikube status  
Minikube type: Control Plane host: Stopped kubelet: Stopped apiserver: Stopped kubeconfig: Stopped  
**vagrant@ubuntu22**:**\~**$ minikube **start** \--driver=podman  
🏄  Done\! kubectl is now configured to use "minikube" cluster and "default" namespace by default  
Minikube type: Control Plane host: Running kubelet: Running apiserver: Running kubeconfig: Configured

(base) x@macOS gyptazy\_ubuntu22.04-arm64 % vagrant provision

**Error: Unstable ssh connection between vagrant and vmware**  
[**vagrant up/ssh fails to connect (on vmware) without ssh\_info\_public=true · Issue \#10730**](https://github.com/hashicorp/vagrant/issues/10730)  
Instead I found that updating the /etc/ssh/sshd\_config on the server and including this:  
IPQoS lowdelay throughput  
