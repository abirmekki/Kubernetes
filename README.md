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
we set both of them to systemd : to verify the init system  ->  $ps -p 1
* Configuring the systemd cgroup driver : add the config to /etc/containerd/config.toml
```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

```bash
# restart containerd service
sudo systemctl restart containerd 
```

## Installing Kubeadm, kublet and kubectl (Master & Worker)
-	Update the apt package index and install packages needed to use the Kubernetes apt repository:
  ```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip #that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

-	Download the public signing key for the Kubernetes package repositories.
```bash
# If the directory `/etc/apt/keyrings` does not exist, it should be #created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
-	Add the appropriate Kubernetes apt repository
```bash
# This overwrites any existing configuration in #/etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
-	Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
-	(Optional) Enable the kubelet service before running kubeadm:
```bash
sudo systemctl enable --now kubelet
```
## Creating a cluster with kubeadm In master node 
 ### 1.	Initializing your control-plane :
 ```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.5.13.166
```
### 2.	set up kubectl for user :
 ```bash
$mkdir -p $HOME/.kube
$sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 3.	Using Weave Net :
 ```bash
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.29/net.yaml
```
![image](https://github.com/user-attachments/assets/7a4eca5e-fa4a-44b6-a3f8-23b63fa8d227)

### 4.	Configure internal cluster network
 ```bash
kubectl edit ds weave-net -n kube-system
```
![image](https://github.com/user-attachments/assets/ca1ef5c5-4ff0-4eda-9619-c0ceae26a06e)

If swap is not supported by default on Kubernetes, disable the swap in the machine with :

 ```bash
swapoff -a
```
### 5.Join Worker Node :

 ```bash
kubeadm join 10.5.13.166:6443 --token tc03nq.nwh5wkojg319vpaa \
	--discovery-token-ca-cert-hash sha256:869b91d31611b0f64ae08d402bad1a5be636b8cfd5b9
```
### 6.Verify the worker node join the cluster : 
![image](https://github.com/user-attachments/assets/d26e5a6f-534b-4f37-b04e-13f26bed2206)

### 7. Verify every thing is working :
![image](https://github.com/user-attachments/assets/d34f3da5-6fb0-4e27-aebb-2e2a0b1a0261)

### See pods running in witch node : 
![image](https://github.com/user-attachments/assets/fc21952e-8efc-463a-88a1-b25bde547faf)

###  See pod running in specific node :
![image](https://github.com/user-attachments/assets/d90acb4f-c507-4245-b8ef-f9a919e60e90)

### Info of the cluster : 
![image](https://github.com/user-attachments/assets/d6082580-89ba-44db-89fb-f1179105ad97)

### Describe a node : ( the same for a pod) 
![image](https://github.com/user-attachments/assets/bce0b06a-60f1-47f3-8314-ffb02bb6f1f4)

##  Manage kubernetes cluster with k9s (real view of the cluster)
### 1.	Download of k9s :

 ```bash
apt-get install snapd
snap install k9s
```

### 2. Lancer k9s :
 ```bash
k9s
```
![image](https://github.com/user-attachments/assets/a936b4a2-25ee-4957-a517-2f9d53f24b78)

### 3.	Search inside k9s :

![image](https://github.com/user-attachments/assets/6a34bf20-a559-4064-9b1e-b2180acd998f)
![image](https://github.com/user-attachments/assets/4bec17ab-8eb2-4fdb-a632-b73084f594bf)

## Configure Master node to lunch pods

 ```bash
kubectl taint nodes --all node-role.kubernetes.io/k8smaster-
```
![image](https://github.com/user-attachments/assets/32dc5a84-b191-4947-bbad-9942b2d86354)

##  Deploy an nginx-pod :
### 1.	Create yaml file : 
 ```bash
touch nginx-pod.yaml
gedit nginx-pod.yaml
```
![image](https://github.com/user-attachments/assets/7f5501f6-6eec-4e91-ba22-436238ee6c32)

### 2.	Deploy the pod :
![image](https://github.com/user-attachments/assets/76c3ec18-5ad7-416d-a7cd-715515b38fae)

### 3.	Verify deployment :
![image](https://github.com/user-attachments/assets/8316b4cd-b712-4090-bf1e-167be25a763d)

### 4.	Access nginx :
![image](https://github.com/user-attachments/assets/24606a29-97c5-451b-a99e-10abcbb16551)

![image](https://github.com/user-attachments/assets/3d0ed2eb-0d2f-458f-8fbb-8bab0fa4864d)

## Create a namespace "Development" on your k8s cluster
### 1.	Create namespace 'development' :

 ```bash
kubectl create namespace development
kubectl get namespaces
```
![image](https://github.com/user-attachments/assets/a6427366-a216-49b1-96f4-07c2a8b5f7c0)
