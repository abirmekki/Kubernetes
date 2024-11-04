# Kubernetes
##  VMs configuration :
*	2 Go ou plus de RAM par machine.
*	2 CPU ou plus pour les machines de plan de contrôle.
*	Connectivité réseau complète entre toutes les machines du cluster.
*	Nom d’hôte unique, adresse MAC.
*	 Master :
  
![image](https://github.com/user-attachments/assets/cdb1b72a-eadc-40a4-b6ec-f0c7e0e03c5b)

![image](https://github.com/user-attachments/assets/2e183ed4-7034-4ffd-a03f-cceaf1660f56)

![image](https://github.com/user-attachments/assets/fc9f29e0-cc4c-47b6-9b83-6ff2e654ceb3)

*	 Worker :

  ![image](https://github.com/user-attachments/assets/45680708-485b-44e5-ad31-04362ed7bc43)

![image](https://github.com/user-attachments/assets/4c4d6853-e4d7-44b3-98b2-cd927d8bdb23)

![image](https://github.com/user-attachments/assets/afd98a09-8d7e-4e4f-936a-85a8824413c2)

## Install K8S with Kubeadm :
# 1.	Installing Kubeadm :
Installing a container runtime (on master and worker) :
* Enable IPv4 packet forwarding
*  Verify 
# Containerd :  follow the instructions on getting started with containerd 

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

At first you must install docker engine on Ubuntu :
*	 uninstall all conflicting packages
*	Set up Docker's apt repository.
*	update
*	Install the Docker packages :  install the latest version : 
sudo apt-get install docker-ce docker-ce-cli containerd.io
*	Verify that the Docker Engine installation is successful
*	Systemctl status containerd
# cgroup drivers :
There are two cgroup drivers available :
*	cgroupfs : the default for container runtime
*	systemd
kublet and container runtime must use the same driver, they have to both match
if you use systemd init system you must use systemd driver
we set both of them to systemd : to verify the init system    $ps -p 1
Configuring the systemd cgroup driver : add the config to /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
