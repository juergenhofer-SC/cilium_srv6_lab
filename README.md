# Kubernetes with Cilium and Segmentrouting IPv6 on Containerlab

## Table of Contents

- [About](#about)
- [High Level Overview](#HLD)
- [Getting Started](#getting_started)
- [Usage](#usage)
- [Contributing](../CONTRIBUTING.md)

## About <a name = "about"></a>

Building a lab environment with two Kubernetes cluster, interconnected via SRv6. The environment is based on containerlab.

## High Level Overview <a name = "HLD"></a>

![Hig Level Overview](https://github.com/juergenhofer-SC/cilium_srv6_lab/blob/main/K8s_SRv6_LAB_HLD.png)

## Getting Started <a name = "getting_started"></a>

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See [deployment](#deployment) for notes on how to deploy the project on a live system.

### Prerequisites

**HowTo Prepare Ubuntu 24 for containerlab**
 1. Install containerlab with docker
	- curl -sL <https://containerlab.dev/setup> | sudo bash -s "all"
 2. Add user and perms for Docker
	- sudo groupadd docker
	- sudo usermod -aG docker $MYUSER
 3. Verify Docker installation
	- docker run hello-world
 4. Install kind
	- [ $(uname -m) = x86_64 ] && curl -Lo ./kind <https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64> cmod +x ./kind
 5. Install Helm
	- curl <https://baltocdn.com/helm/signing.asc> | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
	- sudo apt-get install apt-transport-https --yes
	- echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] 		<https://baltocdn.com/helm/stable/debian/> all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
 6. Install kubectx to switch easy beetween cluster context
     (ref.: https://github.com/ahmetb/kubectx/)
     sudo apt install kubectx
 

**HowTo prepare for Cisco XRd**
 1. Download Cisco image form internal repo
     	- docker login $local.artifactory
	- docker pull $local.artifactory/nso/ios-xr/xrd-control-plane:7.10.2

**Create K8s cluster with kind**
1.  kind create cluster --config cluster01.yaml
   	Set kubectl context to "kind-kubernetes-cluster01"
	You can now use your cluster with:

	kubectl cluster-info --context kind-kubernetes-cluster01

3.  kind create cluster --config cluster02.yaml

4.  taahoju3@containerlab01:~/cilium_srv6_lab$ kind get clusters
kubernetes-cluster01
kubernetes-cluster02

### Installing

**Create K8s cluster with kind**

 1. kind create cluster --config cluster01.yaml 
     Set kubectl context to "kind-kubernetes-cluster01" You can now use your cluster with: 
     kubectl cluster-info --context kind-kubernetes-cluster01

 2. kind create cluster --config cluster02.yaml

**Install Cilium CNI**
  (ref: https://docs.isovalent.com/operations-guide/installation/clean-install.html)
 
 1. helm repo add isovalent https://helm.isovalent.com
 2. Create a cilium-enterprise-values.yaml
     This file keep track of your Cilium Enterprise configuration!
 3. Run helm install command to deploy Cilium Enterprise CNI
     helm install cilium isovalent/cilium --version 1.15.6 \
     --namespace kube-system -f cilium-enterprise-values.yaml

Now that the base installation for Cilium Enterprise is complete, you can explore and enable advanced features, like SRv6.

**Cilium SRv6 L3VPN**


## Usage <a name = "usage"></a>

Add notes about how to use the system.

 kind delete clusters --all
  323  kubectx
  324  git pull
  325  kind create cluster --config cluster01.yaml 
  326  helm install cilium isovalent/cilium --version 1.15.6 --namespace kube-system -f cilium-enterprise-values.yaml
  327  kind create cluster --config cluster02.yaml 
  328  kubectx
  329  kubectx kind-cluster02
  330  helm install cilium isovalent/cilium --version 1.15.6 --namespace kube-system -f cilium-enterprise-values.yaml
  331  kubectx kind-cluster01
  332  kubectx



sudo clab --log-level debug -t topology.yaml destroy
sudo clab --log-level debug -t topology.yaml deploy
sudo clab --log-level debug -t topology.yaml deploy --reconfigure


  ----
  339  ip netns list
  340  docker ps
  341  docker inspect --format '{{ .State.Pid }}' <CONTAINER_ID>
  342  docker inspect --format '{{ .State.Pid }}' caa38fbde54a
  343  nsenter -t 12692 -n ip addr
  344  sudo nsenter -t 12692 -n ip addr


taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl get pods --all-namespaces
NAMESPACE            NAME                                                              READY   STATUS    RESTARTS       AGE
kube-system          cilium-gs6zs                                                      1/1     Running   1 (2d8h ago)   16d
kube-system          cilium-jvl8l                                                      1/1     Running   1 (2d8h ago)   16d
kube-system          cilium-operator-5d64f7b64d-4qmzq                                  1/1     Running   2 (2d8h ago)   16d
kube-system          cilium-operator-5d64f7b64d-tf5xz                                  1/1     Running   2 (2d8h ago)   16d
kube-system          coredns-7db6d8ff4d-dkkfh                                          1/1     Running   1 (2d8h ago)   16d
kube-system          coredns-7db6d8ff4d-xjqtx                                          1/1     Running   1 (2d8h ago)   16d
kube-system          etcd-cilium-srv6-lab-cluster01-control-plane                      1/1     Running   0              2d8h
kube-system          kube-apiserver-cilium-srv6-lab-cluster01-control-plane            1/1     Running   0              2d8h
kube-system          kube-controller-manager-cilium-srv6-lab-cluster01-control-plane   1/1     Running   2 (2d8h ago)   16d
kube-system          kube-proxy-ftxv4                                                  1/1     Running   2 (2d8h ago)   16d
kube-system          kube-proxy-phcdq                                                  1/1     Running   2 (2d8h ago)   16d
kube-system          kube-scheduler-cilium-srv6-lab-cluster01-control-plane            1/1     Running   2 (2d8h ago)   16d
local-path-storage   local-path-provisioner-988d74bc-lkh9n                             1/1     Running   1 (2d8h ago)   16d

taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl -n kube-system logs cilium-gs6zs | grep "Allocated SID"
Defaulted container "cilium-agent" out of: cilium-agent, config (init), mount-cgroup (init), apply-sysctl-overwrites (init), mount-bpf-fs (init), clean-cilium-state (init), install-cni-binaries (init)
time="2024-07-25T08:54:17Z" level=info msg="Allocated SID for VRF with export route target." ExportRouteTarget="65001:1" LocatorPool= SID="2001:dead:10:1::adee" VRF=vrf01 component=srv6.Manager.createIngressPathVRFs subsys=srv6-manager


taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl get pods --all-namespaces
NAMESPACE            NAME                                                              READY   STATUS    RESTARTS        AGE
kube-system          cilium-gpq6r                                                      1/1     Running   1 (2d9h ago)    16d
kube-system          cilium-jrcsv                                                      1/1     Running   1 (2d9h ago)    16d
kube-system          cilium-operator-5d64f7b64d-9nvz4                                  1/1     Running   2 (2d9h ago)    16d
kube-system          cilium-operator-5d64f7b64d-crmlv                                  1/1     Running   32 (2d9h ago)   16d
kube-system          coredns-7db6d8ff4d-89xpw                                          1/1     Running   1 (2d9h ago)    16d
kube-system          coredns-7db6d8ff4d-dswbm                                          1/1     Running   1 (2d9h ago)    16d
kube-system          etcd-cilium-srv6-lab-cluster02-control-plane                      1/1     Running   0               2d9h
kube-system          kube-apiserver-cilium-srv6-lab-cluster02-control-plane            1/1     Running   0               2d9h
kube-system          kube-controller-manager-cilium-srv6-lab-cluster02-control-plane   1/1     Running   2 (2d9h ago)    16d
kube-system          kube-proxy-5sgkn                                                  1/1     Running   2 (2d9h ago)    16d
kube-system          kube-proxy-cg9m4                                                  1/1     Running   27 (2d9h ago)   16d
kube-system          kube-scheduler-cilium-srv6-lab-cluster02-control-plane            1/1     Running   2 (2d9h ago)    16d
local-path-storage   local-path-provisioner-988d74bc-tlvct                             1/1     Running   1 (2d9h ago)    16d

time="2024-07-25T13:29:18Z" level=info msg="Allocated SID for VRF with export route target." ExportRouteTarget="65002:2" LocatorPool= SID="2001:db8:20::d80" VRF=vrf02 component=srv6.Manager.createIngressPathVRFs subsys=srv6-manager