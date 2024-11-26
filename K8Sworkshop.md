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
### 1.	Installing Kubeadm :
Installing a container runtime (on master and worker) :
* Enable IPv4 packet forwarding
```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
# Apply sysctl params without reboot
sudo sysctl --system
```
*  Verify
 ```bash
# Verify that net.ipv4.ip_forward is set to 1
sysctl net.ipv4.ip_forward
  ```
![image](https://github.com/user-attachments/assets/db5d13b0-a07e-4502-a4ad-2f98230da186)

# Containerd : Instructions getting started with containerd 

https://github.com/containerd/containerd/blob/main/docs/getting-started.md

At first you must install docker engine on Ubuntu :
*	 uninstall all conflicting packages
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

*	Set up Docker's apt repository.
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

```
*	update
```bash
sudo apt-get update
```
*	Install the Docker packages :  install the latest version : 
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```
*	Verify that the Docker Engine installation is successful
```bash
docker --version
```
## cgroup drivers :
There are two cgroup drivers available :
*	cgroupfs : the default for container runtime
*	systemd
kublet and container runtime must use the same driver, they have to both match
if you use systemd init system you must use systemd driver
we set both of them to systemd : to verify the init system  ->   ``` ps -p1  ```
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
## Creating a cluster with kubeadm In Master node :
If swap is not supported by default on Kubernetes, disable the swap in the machine(master &  worker) with :

 ```bash
swapoff -a
```
 ### 1.	Initializing your control-plane :
 ```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.5.13.166
```
### 2.	set up kubectl for user :
 ```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 3.	Using Weave Net :
 ```bash
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.29/net.yaml
```
![image](https://github.com/user-attachments/assets/1686070b-1a8d-40d3-a9d7-afed1dade60d)

### 4.	Configure internal cluster network
 ```bash
kubectl edit ds weave-net -n kube-system
```
![image](https://github.com/user-attachments/assets/ca1ef5c5-4ff0-4eda-9619-c0ceae26a06e)


### 5.Join Worker Node :

 ```bash
kubeadm join 10.5.13.166:6443 --token tc03nq.nwh5wkojg319vpaa \
	--discovery-token-ca-cert-hash sha256:869b91d31611b0f64ae08d402bad1a5be636b8cfd5b9
```
### 6.Verify the worker node join the cluster : 
![image](https://github.com/user-attachments/assets/6ff5da31-884f-441a-92b5-5c824ebd9ac6)

### 7. Verify every thing is working :
![image](https://github.com/user-attachments/assets/d34f3da5-6fb0-4e27-aebb-2e2a0b1a0261)

### See pods running in witch node : 
![image](https://github.com/user-attachments/assets/3e68b85b-1eb1-42c9-b52e-c19ffa4b1206)

###  See pods running in specific node (master) :
![image](https://github.com/user-attachments/assets/8b1574a2-e0d1-4d24-91ab-e6302c6960bb)

### Info of the cluster : 
![image](https://github.com/user-attachments/assets/bfaaea1f-c34a-407d-8fc0-7405de9150ac)

### Describe a node : ( the same for a pod) 
![image](https://github.com/user-attachments/assets/7865e47d-5772-40f5-8279-858949725b6d)

##  Manage kubernetes cluster with k9s (real view of the cluster)
### Installation of k9s via Binary Download :

#### 1. Download the Latest Release:

 ```bash
curl -sSL https://github.com/derailed/k9s/releases/latest/download/k9s_Linux_amd64.tar.gz -o k9s.tar.gz
```
#### 2. Extract the Tarball :
 ```bash
tar -zxvf k9s.tar.gz
```
#### 3. Move the Binary :

 ```bash
sudo mv k9s /usr/local/bin/
```
#### 4. Make it Executable:
 ```bash
sudo chmod +x /usr/local/bin/k9s
```
#### 5. Lancer k9s : 
 ```bash
k9s
```
![image](https://github.com/user-attachments/assets/a936b4a2-25ee-4957-a517-2f9d53f24b78)

#### 6.	Search inside k9s : Ctrl + : 

![image](https://github.com/user-attachments/assets/6a34bf20-a559-4064-9b1e-b2180acd998f)

![image](https://github.com/user-attachments/assets/4bec17ab-8eb2-4fdb-a632-b73084f594bf)

## Configure Master node to launch pods
#### the default taint applied to a master node is : node-role.kubernetes.io/master:NoSchedule

### Method 1: Remove the Taint :
 ```bash
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
```
![image](https://github.com/user-attachments/assets/ab439807-3059-420c-8ba4-8100cdfdb30c)

#### 1. Create yaml file  :
```bash
touch pod.yaml
gedit pod.yaml
```
![image](https://github.com/user-attachments/assets/7c7822dc-8d7b-4234-b5ae-8cbeb35e5379)

#### 2. Deploy the Pod :
```bash
kubectl apply -f pod.yaml
```
![image](https://github.com/user-attachments/assets/8af2f3c7-52c7-4d91-af2a-740c2ecb75a0)

#### 3. Verify the Pod :
```bash
kubectl get pods -o wide
```
![image](https://github.com/user-attachments/assets/46217086-ba71-437d-89c6-173d02465d75)

### Method 2 : Use Node Selector :
```bash
kubectl label nodes master role=master
```
![image](https://github.com/user-attachments/assets/b9a9b45f-7ea4-4f0a-83bd-6552e80cfdd6)

#### 1. Create yaml file  :
```bash
touch podMaster.yaml
gedit podMaster.yaml
```
![image](https://github.com/user-attachments/assets/326a730c-f200-47d7-afa2-09069f21eb9d)

#### 2. Deploy the Pod :
```bash
kubectl apply -f podMaster.yaml
```
![3](https://github.com/user-attachments/assets/7d43b374-1aae-478a-9ab8-b8c2f097cdc7)

#### 3. Verify the Pod :
```bash
kubectl get pods -o wide
kubectl describe pod my-pod-master
```
![image](https://github.com/user-attachments/assets/7b896f13-c455-4271-a361-983a29fcf031)

![image](https://github.com/user-attachments/assets/6cef363d-b8ad-4453-9dcf-dcba98f31559)

#### 4.	Access the pod :
```bash
kubectl port-forward pod/my-pod-master 8081:80
```
![image](https://github.com/user-attachments/assets/542cc494-8c51-4dc2-adb9-9bce7a00ffb4)

![image](https://github.com/user-attachments/assets/6b3c79bf-9064-4329-adb1-00366a1e22d6)

### Method 3 : Use Tolerations
####  keep the master node tainted but still allow specific pods to run on it
#### 1. Create Pod Definition with Tolerations :
```bash
touch podTolerance.yaml
gedit podTolerance.yaml
```
![image](https://github.com/user-attachments/assets/27b0f83b-94f1-4829-90c1-f2bf0f4bda2e)

#### 2. Deploy the Pod :
```bash
kubectl apply -f podTolerance.yaml
```
![image](https://github.com/user-attachments/assets/e7fc2034-462e-409b-801f-562af1e1ae20)

#### 3. Verify the Pod :
```bash
kubectl get pods -o wide
```
![image](https://github.com/user-attachments/assets/7ff62193-e125-4c3d-a793-8160ecd20b59)


## Create a namespace "Development" on your k8s cluster
### 1.	Create namespace 'development' :

 ```bash
kubectl create namespace development
kubectl get namespaces
```
![image](https://github.com/user-attachments/assets/a6427366-a216-49b1-96f4-07c2a8b5f7c0)
### 2.	Switch to the new namespace :
 ```bash
kubectl config set-context --current --namespace= development
```
### 3.	Deploy app with kubectl on namespace « development » :
*	Create a deployment YAML file :
```bash    
touch deployment.yaml
gedit deployment.yaml
 ```
![image](https://github.com/user-attachments/assets/2b2a5bd2-8baa-4a2f-9adf-e12b9a50594f)

* Apply deployment file :
```bash
kubectl apply -f deployment.yaml 
```

* Verify

![image](https://github.com/user-attachments/assets/d4ada9df-f7c9-4f2b-8e5b-0a4e3896e139)
  
![image](https://github.com/user-attachments/assets/17a4fdf0-5dfb-4de0-889c-1951389e6e02)

## Deploy with Helm chart 
 ### 1.	Install Helm : From Script : 
 #### script that will automatically grab the latest version of Helm and install it locally.
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### 2.	Verify Installation :
 ```bash
helm version
 ```
![image](https://github.com/user-attachments/assets/df3e875c-d883-417c-bac1-d2e352dece10)

 ### 3.	Add the Helm Repository :
Helm repo is a collections of Helm charts that provide pre-configured kubernetes resources for popular apps to deploy and manage app on k8s.
 ```bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
 ```
![image](https://github.com/user-attachments/assets/25b8a932-a836-432d-ae33-255df3d57c9f)

![image](https://github.com/user-attachments/assets/ca958bdc-0478-4fea-8bc9-01e9e8ada1fe)

![image](https://github.com/user-attachments/assets/4c9d9a14-bc16-462f-9fde-abbbd1dce80b)

### 4.	Change Type from ClusterIP to NodePort :

 ```bash
KUBE_EDITOR= ‘’nano’’ kubectl edit service my-grafana
 ```
![image](https://github.com/user-attachments/assets/047db804-7d2b-467f-9f1a-5bda17ae5ccd)

### 5.	Verify
 ```bash
kubectl get service -n deployment
 ```
![image](https://github.com/user-attachments/assets/ae4279ee-5853-4fee-aeaa-bb3ce3a02151)

### 6.	Access  to Grafana
![image](https://github.com/user-attachments/assets/2edd4944-12f3-4eeb-bd83-f77bfcfbf9e3)

## Manual Creation of a Helm Chart
### Step 1: Create the Chart Directory
```bash
mkdir mychart
cd mychart
 ```
![image](https://github.com/user-attachments/assets/09265881-3a79-4d7e-906c-f01bba40b7d0)

### Step 2: Create Chart.yaml
```bash
touch Chart.yaml
gedit Chart.yaml
 ```
![image](https://github.com/user-attachments/assets/9098284b-4637-4ab6-b82c-8c6c3f4461f6)

### Step 3: Create values.yaml
```bash
touch values.yaml
gedit values.yaml
```
![image](https://github.com/user-attachments/assets/8780caac-c705-4f14-aafe-799cc5e66c6b)

### Step 4: Create the templates Directory
```bash
mkdir templates
```
### Step 5: Create templates/deployment.yaml
```bash
touch templates/deployment.yaml
gedit templates/deployment.yaml
```
![image](https://github.com/user-attachments/assets/b6584f21-2f07-4e05-bdd6-83a8a8c97757)

### Step 6: Create templates/service.yaml 
```bash
touch templates/service.yaml
gedit templates/service.yaml
```
![image](https://github.com/user-attachments/assets/49681678-dc51-44c7-a35d-a0690678a989)

### Step 7: Install the Chart
```bash
helm install my-nginx ./mychart
```
![image](https://github.com/user-attachments/assets/fc8830da-3b01-4476-9ae4-2818490c399e)

### Step 8: Access the Application
```bash
kubectl port-forward svc/my-nginx-nginx 8080:80
```
![image](https://github.com/user-attachments/assets/1c950c8a-8b39-40ca-a7ab-0ec056128dd4)

![image](https://github.com/user-attachments/assets/a6a9a10c-0cd3-4392-8b94-b776a66b40b6)

### Understand the Chart Structure:
#### The generated chart will have the following structure:
mychart/
- ├── Chart.yaml
- ├── values.yaml
- └── templates/
    - ├── deployment.yaml
    - └── service.yaml

- Chart.yaml: This is the main descriptor file for the Helm chart. It provides metadata about the chart, including its name, version, description, and other relevant information.
- values.yaml: This file contains the default configuration values for the chart. Users can override these values at install time or upgrade time to customize the behavior of the chart.
- templates/: This directory contains Kubernetes manifest templates that define the resources to be created when the chart is installed. Helm uses the Go templating engine to process these files.
    - deployment.yaml: Defines the Deployment resource for the Nginx application.
    - service.yaml: Defines the Service resource to expose the Nginx application.

![image](https://github.com/user-attachments/assets/acfcdae9-9496-431b-b03b-9c9b303b41bf)


## Management of Kubernetes Objects Using Kustomize

### 1.	Installation of kustomize
 ```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
sudo mv kustomize /usr/local/bin
 ```

### 2.	Create Repo ‘Kust’
 ```bash
mkdir Kust
 ```

### 3.	Create Deployment file 'deployment.yaml'
 ```bash
gedit kust/deployment.yaml
 ```
![image](https://github.com/user-attachments/assets/b95e671d-7eea-415f-ae37-3ba1be70589b)

### 4.	Create service file 'service.yaml'
 ```bash
gedit kust/service.yaml
 ```
![image](https://github.com/user-attachments/assets/9a7a05e6-3498-4c64-bcdd-176cce43175a)

### 5.	Create kustomize file 'kustomization.yaml'
 ```bash
gedit kust/kustomization.yaml
```
![image](https://github.com/user-attachments/assets/dfab289c-7902-4fb6-aea8-e52f96114e2b)

 ### Project Tree :
|__ kust
- |__ deployment.yaml
- |__ service.yaml
- |__ kustomization.yaml

![image](https://github.com/user-attachments/assets/ae6c91ae-bd86-463c-b78e-b8548d6b68fd)

### 6.	apply and generate k8s resources from kustomize 
![image](https://github.com/user-attachments/assets/90007755-848c-4ef4-9c9a-b46eb8310701)

