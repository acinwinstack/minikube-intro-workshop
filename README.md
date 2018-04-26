# Minikube Workshop by Alfie Chen

<p align="center">
<img src="https://avatars0.githubusercontent.com/u/12655890?s=200&v=4" width="70">
&ensp;&ensp;
<img src="https://github.com/kubernetes/minikube/raw/master/logo/logo.png" width="100">
&ensp;&ensp;
<img src="https://avatars2.githubusercontent.com/u/29940671?s=460&v=4" width="70">
</p>

## Description

This is workshop material for beginners looking forward to learning [Docker](https://www.docker.com/) and [Minikube](https://github.com/kubernetes/minikube). 
We're modifying [Get Started Tutorial](https://docs.docker.com/get-started/) from Docker, then deploying the app on Minikube.

## Prerequisites

### Install Docker
* [Docker for Windows](https://download.docker.com/win/stable/Docker for Windows Installer.exe)
  * Requires **Windows 10 Pro with Hyper-V** or **Windows Server 2016**

* [Docker for Mac](https://docs.docker.com/docker-for-mac/install)
  * Requires **Mac 2010 hardware model or newer**
  * Requires **MacOS El Capitan 10.11 or newer**
  * Requires at least **4GB RAM**

  You may choose between **_Stable_** or **_Edge_** versions.

* [Docker Toolbox](https://docs.docker.com/toolbox/oerview)  
  If your Windows or Mac system doesn't meet the above requirements, this is what you'll need.  
  Docker Toolbox comes with VirtualBox and Git, and launches a virtual machine (Docker Machine) where Docker will be installed and run.

* [Docker](https://docs.docker.com/install/#server)
  For  Linux systems, **Docker CE** supports **_CentOS_**, **_Debian_**, **_Fedora_**, and **_Ubuntu_**.  
  The easiest way to intall Docker on these systems is a single **curl** as below:
  ```shell
  $ curl -fsSL "https://get.docker.com/" | sh
  $ sudo systemctl start docker
  ```

### Install VirtualBox  
_If you've installed Docker via **Docker Toolbox**, you may skip this section._  
_**For other release versions of VirtualBox, please refer to this [link](https://www.virtualbox.org/wiki/Downloads).**_

[**VirtualBox**](https://www.virtualbox.org/) is the default hypervisor for running Minikube. 
Although there are multiple hypervisor choices across systems that Minikube [supports](https://github.com/kubernetes/minikube/blob/master/README.md#requirements), 
we'll be using VirtualBox for the simplisity of this workshop.  

Install packages:
* [Windows](https://download.virtualbox.org/virtualbox/5.2.8/VirtualBox-5.2.8-121009-Win.exe) (v5.2.8)
* [Mac](https://download.virtualbox.org/virtualbox/5.2.10/VirtualBox-5.2.10-122088-OSX.dmg) (v5.2.10)
* [Linux](https://www.virtualbox.org/wiki/Linux_Downloads) - Download the package according to your Linux distribution.

### Install kubectl  
_kubectl is the CLI binary for interacting with [**Kubernetes API**](https://kubernetes.io/docs/reference/)._  
_**For specific release versions of kubectl, please refer to further instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).**_  
_**Find out the latest stable version [here](https://storage.googleapis.com/kubernetes-release/release/stable.txt).**_

* Windows  
  Download [executable](https://storage.googleapis.com/kubernetes-release/release/v1.10.0/bin/windows/amd64/kubectl.exe) (**kubectl v1.10.0**).  

  
* MacOS
  ```shell
  $ brew install kubectl
  ```
* Linux
  ```shell
  $ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
  $ chmod +x ./kubectl  
  $ sudo mv ./kubectl /usr/local/bin/kubectl
  ```

### Install Minikube
* Windows  
  Download [executable](https://storage.googleapis.com/minikube/releases/latest/minikube-windows-amd64.exe) and rename it to `minikube.exe`.

  
* MacOS  
  ```shell
  $ brew cask install minikube
  ```

* Linux  
  ```shell
  $ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
  ```

## Quickstart
### Download workshop contents  
* For **Windows**, unzip this [file](https://github.com/acinwinstack/minikube-intro-workshop/archive/master.zip).  
* For **MacOS** or **Linux**, simply
  ```shell
  $ git clone https://github.com/acinwinstack/minikube-intro-workshop.git
  ```
  _If you don't have GIT installed, check out this [link](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)._ 

### Build Docker image
```shell
$ cd minikube-intro-workshop
$ docker build -t helloworld:v1 ./build/
```
### Start Minikube and deploy services
```shell
$ minikube start
Starting local Kubernetes v1.9.4 cluster...
Starting VM...
SSH-ing files into VM...
Setting up certs...
Starting cluster components...
Connecting to cluster...
Setting up kubeconfig...
Kubectl is now configured to use the cluster.

$ kubectl apply -f lab1/redis_pod.yml
pod "redis" created

$ kubectl get po -o wide
NAME                                  READY     STATUS    RESTARTS   AGE       IP           NODE
redis                                 1/1       Running   0          6s        172.17.0.9   minikube
```
Write down the _**pod IP**_ of redis (usually something like **`172.17.0.9`**) then use your favorite text editor to replace **`REDIS_IP`** in `lab2/web_deploy.yml` with the **pod IP**.  

Once **`REDIS_IP`** is configured correctly, `hello-world` is ready to go.

******By default, `web_deploy.yml` uses image `acinwinstack/hello_world:minikube-intro-workshop`, which is an equivalent to `helloworld:v1` we built earlier. Feel free to replace it with `helloworld:v1`.  

```shell
$ kubectl apply -f lab2/web_deploy.yml
deployment "hello-world-deploy" created

$ kubectl get po -o wide
NAME                                  READY     STATUS    RESTARTS   AGE       IP           NODE
hello-world-deploy-59d978b59c-476t2   1/1       Running   0          8s        172.17.0.4   minikube
hello-world-deploy-59d978b59c-kf9s6   1/1       Running   0          8s        172.17.0.2   minikube
hello-world-deploy-59d978b59c-pb5f9   1/1       Running   0          8s        172.17.0.7   minikube
hello-world-deploy-59d978b59c-qpcsk   1/1       Running   0          8s        172.17.0.8   minikube
hello-world-deploy-59d978b59c-tpjh2   1/1       Running   0          8s        172.17.0.5   minikube
redis                                 1/1       Running   0          2m        172.17.0.9   minikube

$ kubectl expose deploy hello-world-deploy --type NodePort --port 80 --target-port 80 --name hello-world-svc
service "hello-world-svc" exposed

$ kubectl get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
hello-world-svc   NodePort    10.100.183.112   <none>        80:30943/TCP   19s
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        26m

$ curl 192.168.99.100:30943
<h3>Hello world!</h3><b>Hostname:</b> hello-world-deploy-5bfb58896b-ptsk5<br/><b>Total Visits:</b> 1<br/><b>Visits since connected to Redis:</b> 1
```

## Notes

1. The content of this workshop is basically a modification of [Docker Get Started](https://docs.docker.com/get-started/) without the visualizer.
2. **LAB1** counts visits upon each hello-world pod, and stores the total value in redis.
3. **LAB2** mounts a persistent local storage from _**`minikube:/webvol/`**_ to each hello-world pod (at location _**`container:/home/app/`**_, and reads/writes the total visit-count in _**`count.txt`**_.
