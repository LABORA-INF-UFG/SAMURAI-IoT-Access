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
cd SAMURAI-IoT-Access &&  ansible-playbook -K golang-1.14-install.yml
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

Wait a few seconds for Ansible to finish configuring the 5GC. The result of the execution should look something like the one shown in the figure below.
<p align="center">
    <img src="images/5gc_ansible_result.png"/> 
</p>

#### Initializing the 5GC functions
After installation we must initialize the 5GC. The entire installation was performed following the standard for applications written in GOLang. Go to `` /root/go/src/free5gc `` and run the following command to init 5GC functions ``./run.sh ``. After execution, the expected result is something similar to what is shown in the following figure.
<p align="center">
    <img src="images/5gc_start.png"/> 
</p>

**Important Note:** In the image above, the __verbosity level__ of the UPF LOG was defined as ``TRACE``. In a standard installation, the __verbosity level__ is probably defined as ``INFO``, so the output will probably not be identical to the image above, which does not mean an 5GC initialization failure. Pay attention to any messages labeled as ``ERROR`` in the log output, they will be printed in red color and then they will indicate some failure in the installation process.

#### Initializing the 5GC API Server
After initializing the microservices that control the data and control planes, it is also necessary to initialize the API that controls access to the 5GC database in MongoDB. This service will be used later for the registration of UEs in the 5GC.  Open another terminal and access the  `` /root/go/src/free5gc/webconsole `` directory. Then type the following command  `` go run server.go ``. . After execution, the expected result is something similar to what is shown in the following figure.
<p align="center">
    <img src="images/5gc_api_start.png"/> 
</p>

#### 2º N3IWF (Free5gc)
Access via terminal the machine where the N3IWF will be executed. 
Install python + git + ansible:
```
sudo apt update && apt -y install python && sudo apt -y install git && sudo apt -y install ansible
```

Clone this repository:
```
git clone git clone https://github.com/LABORA-INF-UFG/SAMURAI-IoT-Access.git
```

Install GOLang 1.14.4:
```
cd SAMURAI-IoT-Access &&  ansible-playbook -K golang-1.14-install.yml
source ~/.bashrc
```

Edit the ```n3iwf-install.yml``` and replace the following markers with their respective IP addresses:
* ```<<IP-address-for-IKE-communication-visible-to-UEnon3GPP>>```: This parameter must be replaced by the IP address of the machine where the N3IWF was installed, it must be visible to UEnon3GPP and will be used for communication and subsequent establishment of communication tunnels for the data and control planes. 
* ```<<5G-core-ip-address-for-connection-to-AMF>>```: This parameter must be replaced by the IP address of the machine where free5GC was installed. It will be used to establish communication between N3IWF and the AMF for subsequent validation of UE-non3GPP access credentials and construction of communication tunnels for the data and control planes. 
* ```<<N3IWF-IP-address-for-communication-with-UPF>>```: This parameter must be replaced by the N3IWF IP address, which must be visible by the UPF (free5GC). It will be used for bi-directional communication between N3IWF and UP when building the GTP tunnel for user data plane communication traffic. 

Run the following Ansible playbook to install N3IWF (password for sudo is required):
```
cd SAMURAI-IoT-Access && ansible-playbook -K n3iwf-install.yml
```

##### Start N3IWF
After installing the N3IWF and with the 5GC properly initialized (see instructions [here](https://github.com/LABORA-INF-UFG/SAMURAI-IoT-Access#initializing-the-5gc-functions)), access the  `` /root/go/src/free5gc/NFs/n3iwf `` directory. Then type the following command  `` go run cmd/main.go `` . After execution, the expected result is something similar to what is shown in the following figure.
<p align="center">
    <img src="images/n3iwf_start.png"/> 
</p>

#### 3º UE-non3GPP
Access via terminal the machine where the UE-non3GPP will be executed. 
Install python + git + ansible:
```
sudo apt update && apt -y install python && sudo apt -y install git && sudo apt -y install ansible
```

Clone this repository:
```
git clone https://github.com/LABORA-INF-UFG/SAMURAI-IoT-Access.git
```

Install GOLang 1.21.3:
```
cd SAMURAI-IoT-Access &&  ansible-playbook -K golang-1.21-install.yml
source ~/.bashrc
```

Edit the ```ue-non3gpp-install.yml``` and replace the following markers with their respective IP addresses:
* ```<<IP-address-for-IKE-communication-with-n3iwf>>```:  This parameter must be replaced by the N3IWF IP address, visible to UE-non3GPP, to establish UDP communication for initial communication to establish communication tunnels for the data and control planes.
* ```<<IP-address-of-UEnon3GPP-visible-to-n3iwf>>```:  This parameter must be replaced by the UEnon3GPP IP address. The address must be visible to N3IWF and will be used for bidirectional communication during the tunnel construction process for data and control plane communication traffic.


Run the following Ansible playbook to install UE-non3GPP (password for sudo is required):
```
cd SAMURAI-IoT-Access && ansible-playbook -K ue-non3gpp-install.yml
```
Wait a few seconds for Ansible to finish configuring the UE-non3GPP. The result of the execution should look something like the one shown in the figure below.
<p align="center">
    <img src="images/uenon3gpp_ansible_result.png"/> 
</p>

##### Register UE-non3GPP into 5GC
After installing UE-non3GPP and with the 5GC API properly initialized (see instructions [here](https://github.com/LABORA-INF-UFG/SAMURAI-IoT-Access#initializing-the-5gc-api-server)), access the ``/root/go/src/UE-non3GPP/assets`` dir. Then open the ``include_ue.sh`` file in a text editor (VI or Nano) and replace ``<<replace-this-for-5GCoreAPI-Ip-Address>>`` with the IP address of the machine where the 5GC is running.
<p align="center">
    <img src="images/include_ue_sh.png"/> 
</p>

Attention, if the machine where the UE-non3GPP is being configured does not have access to the machine where the 5GC is, the ``include_ue.sh`` file can be moved to another machine to be executed. The purpose of the ``include_ue.sh`` is to register the UE in the 5GC's MongoDB database, avoiding the need for registration through the Free5GC Web User Interface. May need to assign permission to run ``include_ue.sh`` file (``chmod 777 -R include_ue.sh``). After executing ``include_ue.sh``, a log message will be presented in the [terminal where the 5GC API is being executed](https://github.com/LABORA-INF-UFG/SAMURAI-IoT-Access#initializing-the-5gc-api-server). The expected result is something similar to the one shown in the figure below (unconsidered the error message).
<p align="center">
    <img src="images/api_log_include_ue.png"/> 
</p>


