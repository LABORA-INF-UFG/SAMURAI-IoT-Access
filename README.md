# SAMURAI-IoT-Access
This repository describes steps for installing the elements that make up SAMURAI-IoT-Access (UE-non3GPP + N3IWF + Free5gc). The installation process involves 3 different machines 1st responsible for running 5GC, 2nd responsible for running N3IWF and 3rd responsible for running UE-non3GPP. The main objective is to provide communication with IoT devices and the 5G core.

#### 5GC environment
* SO: Ubuntu 20.04 (LTS) x64
* Uname -r: >= 5.4.0-122-generic
* Memory: 4 GB
* Disk: 80 GB

#### N3IWF environment
* SO: Ubuntu 20.04 (LTS) x64
* Memory: 2 GB
* Disk: 20 GB

#### UE-non3GPP environment
* SO: Ubuntu 20.04 (LTS) x64
* Memory: 2 GB
* Disk: 20 GB

## SAMURAI-IoT-Access components installation

Before starting the installation, it is necessary to edit the ```5gc-install.yml```, ```n3iwf-install.yml``` and ```ue-non3gpp-install.yml``` files, informing the respective IP addresses where each of the artifacts will be deployed in the variable declaration structure.

The installation steps for each of the 3 components are described below. 
**Attention: All installation steps described below must be performed with root privileges (SUDO SU).**


#### 1º 5GC (Free5gc)
Access via terminal the machine where the 5GC will be executed. 
Install python + git + ansible:
```
sudo apt update && apt -y install python && sudo apt -y install git && sudo apt -y install ansible
```

Clone this repository:
```
git clone https://github.com/LABORA-INF-UFG/SAMURAI-IoT-Access.git
```

Install GOLang 1.14.4:
```
cd SAMURAI-IoT-Access &&  ansible-playbook -K golang-install.yml
source ~/.bashrc
```

Run ```ifconfig``` and get the name of **internet network interface**, that provides to the host with access to the internet, like as illustrated in the figure below:
<p align="center">
    <img src="images/if_config.png"/> 
</p>

This interface that will be used by the UPF to provide internet access to the UE's registered in the 5GC.
Edit the ```5gc-install.yml``` and replace the ```<<IP-address-visible-to-n3iwf>>``` marker with the IP address of the machine where the core is being deployed, and which must be visible to n3iwf.

Run the following Ansible playbook (password for sudo is required):
```
cd SAMURAI-IoT-Access && ansible-playbook -K 5gc-install.yml -e  "internet_network_interface=<< internet network interface name>>"
```

