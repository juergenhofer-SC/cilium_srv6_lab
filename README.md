# Kubernetes with Cilium and Segmentrouting IPv6 on Containerlab

## Table of Contents

- [About](#about)
- [High Level Overview](#HLD)
- [Getting Started](#getting_started)
- [Functional tests](#tests)

## About <a name = "about"></a>

Building a lab environment with two Kubernetes cluster, interconnected via SRv6. The environment is based on containerlab.

## High Level Overview <a name = "HLD"></a>

![Hig Level Overview](https://github.com/juergenhofer-SC/cilium_srv6_lab/blob/SRv6_base_config/K8s_SRv6_LAB_detail.png)

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
- echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg]   <https://baltocdn.com/helm/stable/debian/> all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

 6. Install kubectx to switch easy beetween cluster context
     (ref.: <https://github.com/ahmetb/kubectx/>)
     sudo apt install kubectx

**HowTo prepare for Cisco XRd**

 1. Download Cisco image form internal repo
      - docker login $local.artifactory

- docker pull $local.artifactory/nso/ios-xr/xrd-control-plane:7.10.2

### Installing

**Create K8s cluster with kind**

1. kind create cluster --config cluster01.yaml
    Set kubectl context to "kind-kubernetes-cluster01"
 You can now use your cluster with:

 kubectl cluster-info --context kind-kubernetes-cluster01

3. kind create cluster --config cluster02.yaml

4. taahoju3@containerlab01:~/cilium_srv6_lab$ kind get clusters
kubernetes-cluster01
kubernetes-cluster02

**Install Containerlab Environment**
 sudo clab deploy -t Containerlab/topology.yaml

```console
Install Output:
INFO[0000] Containerlab v0.56.0 started
INFO[0000] Parsing & checking topology file: topology.yaml
INFO[0000] Creating docker network: Name="clab", IPv4Subnet="172.20.20.0/24", IPv6Subnet="2001:172:20:20::/64", MTU=1500
INFO[0000] Creating lab directory: /home/taahoju3/cilium_srv6_lab/clab-cilium-srv6-lab
INFO[0001] Creating container: "pe1"
INFO[0001] Creating container: "pe2"
INFO[0001] Creating container: "rr"
INFO[0002] Created link: pe1:Gi0-0-0-0 <--> pe2:Gi0-0-0-0
INFO[0002] Created link: pe2:Gi0-0-0-1 <--> rr:Gi0-0-0-1
INFO[0002] Created link: pe1:Gi0-0-0-1 <--> rr:Gi0-0-0-0
INFO[0005] Created link: cilium-srv6-lab-cluster01-control-plane:net0 <--> pe1:Gi0-0-0-2
INFO[0005] Created link: cilium-srv6-lab-cluster02-control-plane:net0 <--> pe2:Gi0-0-0-2
INFO[0005] Executed command "ip addr add 172.200.200.2/24 dev net0" on the node "cilium-srv6-lab-cluster02-control-plane". stdout:
INFO[0005] Executed command "ip addr add fd00:172:200:200::2/64 dev net0" on the node "cilium-srv6-lab-cluster02-control-plane". stdout:
INFO[0005] Executed command "ip route add 10.0.0.0/8 via 172.200.200.1 dev net0" on the node "cilium-srv6-lab-cluster02-control-plane". stdout:
INFO[0005] Executed command "ip route add fd00::/16 via fd00:172:200:200::1 dev net0" on the node "cilium-srv6-lab-cluster02-control-plane". stdout:
INFO[0005] Executed command "ip route add fc00:0:3333::1/128 via fd00:172:200:200::1 dev net0" on the node "cilium-srv6-lab-cluster02-control-plane". stdout:
INFO[0005] Executed command "ip addr add 172.100.100.2/24 dev net0" on the node "cilium-srv6-lab-cluster01-control-plane". stdout:
INFO[0005] Executed command "ip addr add fd00:172:100:100::2/64 dev net0" on the node "cilium-srv6-lab-cluster01-control-plane". stdout:
INFO[0005] Executed command "ip route add 10.0.0.0/8 via 172.100.100.1 dev net0" on the node "cilium-srv6-lab-cluster01-control-plane". stdout:
INFO[0005] Executed command "ip route add fd00::/16 via fd00:172:100:100::1 dev net0" on the node "cilium-srv6-lab-cluster01-control-plane". stdout:
INFO[0005] Executed command "ip route add fc00:0:3333::1/128 via fd00:172:100:100::1 dev net0" on the node "cilium-srv6-lab-cluster01-control-plane". stdout:
INFO[0005] Adding containerlab host entries to /etc/hosts file
INFO[0005] Adding ssh config for containerlab nodes
INFO[0005] 🎉 New containerlab version 0.57.0 is available! Release notes: <https://containerlab.dev/rn/0.57/>
Run 'containerlab version upgrade' to upgrade or go check other installation options at <https://containerlab.dev/install/>
+---+-----------------------------------------+--------------+----------------------------------------------------------------------------------------------+---------------+---------+-----------------+--------------------------+
| # |                  Name                   | Container ID |                                            Image                                             |     Kind      |  State  |  IPv4 Address   |       IPv6 Address       |
+---+-----------------------------------------+--------------+----------------------------------------------------------------------------------------------+---------------+---------+-----------------+--------------------------+
| 1 | cilium-srv6-lab-cluster01-control-plane | c5fada159da2 | kindest/node:v1.30.0@sha256:047357ac0cfea04663786a612ba1eaba9702bef25227a794b52890dd8bcd692e | ext-container | running | 172.18.0.2/16   | fc00:f853:ccd:e793::2/64 |
| 2 | cilium-srv6-lab-cluster01-worker        | d4222b3e8fe9 | kindest/node:v1.30.0@sha256:047357ac0cfea04663786a612ba1eaba9702bef25227a794b52890dd8bcd692e | ext-container | running | 172.18.0.3/16   | fc00:f853:ccd:e793::3/64 |
| 3 | cilium-srv6-lab-cluster02-control-plane | 62dbdd8e7848 | kindest/node:v1.30.0@sha256:047357ac0cfea04663786a612ba1eaba9702bef25227a794b52890dd8bcd692e | ext-container | running | 172.18.0.4/16   | fc00:f853:ccd:e793::4/64 |
| 4 | cilium-srv6-lab-cluster02-worker        | 93f74dc49be7 | kindest/node:v1.30.0@sha256:047357ac0cfea04663786a612ba1eaba9702bef25227a794b52890dd8bcd692e | ext-container | running | 172.18.0.5/16   | fc00:f853:ccd:e793::5/64 |
| 5 | clab-cilium-srv6-lab-pe1                | 9dedea52a10c | nso-docker-local.artifactory.swisscom.com/nso/ios-xr/xrd-control-plane:7.10.2                | cisco_xrd     | running | 172.20.20.36/24 | 2001:172:20:20::4/64     |
| 6 | clab-cilium-srv6-lab-pe2                | 04f1022819b9 | nso-docker-local.artifactory.swisscom.com/nso/ios-xr/xrd-control-plane:7.10.2                | cisco_xrd     | running | 172.20.20.37/24 | 2001:172:20:20::3/64     |
| 7 | clab-cilium-srv6-lab-rr                 | b7df49b3dd0f | nso-docker-local.artifactory.swisscom.com/nso/ios-xr/xrd-control-plane:7.10.2                | cisco_xrd     | running | 172.20.20.46/24 | 2001:172:20:20::2/64     |
+---+-----------------------------------------+--------------+----------------------------------------------------------------------------------------------+---------------+---------+-----------------+--------------------------+
```

**Install Cilium CNI**
NOTE: it is important that the CNI install is done after clab config!

  (ref: <https://docs.isovalent.com/operations-guide/installation/clean-install.html>)

 1. helm repo add isovalent <https://helm.isovalent.com>
 2. Create a cilium-enterprise-values.yaml
     This file keep track of your Cilium Enterprise configuration!
 3. Run helm install command to deploy Cilium Enterprise CNI
     helm install cilium isovalent/cilium --version 1.15.8 \
     --namespace kube-system \
  --set operator.replicas=1 \
  -f cilium-enterprise-values.yaml

Now that the base installation for Cilium Enterprise is complete, you can explore and enable advanced features, like SRv6.

**Cilium SRv6 L3VPN**

# Apply Config parameter yamls per Cluster

####

NOTE: Only after splitting locator pool to two seperate files, get's correct advertised.

####

```console
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl apply -f cilium-bgp-peering-policies.yaml
ciliumbgppeeringpolicy.cilium.io/cilium-srv6-lab-cluster01-control-plane created
ciliumbgppeeringpolicy.cilium.io/cilium-srv6-lab-cluster02-control-plane created
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl apply -f locator-pool1.yaml 
isovalentsrv6locatorpool.isovalent.com/pool1 created
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl apply -f vrf-policy-cluster1.yaml
isovalentvrf.isovalent.com/vrf01 created
isovalentvrf.isovalent.com/vrf02 created
```

```console
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectx kind-cilium-srv6-lab-cluster02
✔ Switched to context "kind-cilium-srv6-lab-cluster02".
taahoju3@containerlab01:~/cilium_srv6_lab$
taahoju3@containerlab01:~/cilium_srv6_lab$ 
taahoju3@containerlab01:~/cilium_srv6_lab$  kubectl apply -f cilium-bgp-peering-policies.yaml
ciliumbgppeeringpolicy.cilium.io/cilium-srv6-lab-cluster01-control-plane created
ciliumbgppeeringpolicy.cilium.io/cilium-srv6-lab-cluster02-control-plane created
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl apply -f locator-pool2.yaml 
isovalentsrv6locatorpool.isovalent.com/pool2 created
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl apply -f vrf-policy-cluster2.yaml
isovalentvrf.isovalent.com/vrf01 created
```

### Rebuild section

# delete all nodes / destroy and rebuild clab

 kind delete clusters --all
  
 sudo clab destroy --cleanup -t Containerlab/topology.yaml
 sudo clab deploy -t Containerlab/topology.yaml

## Functional tests <a name = "tests"></a>

```console
taahoju3@containerlab01:~/cilium_srv6_lab/Containerlab$ kubectl get pods --all-namespaces
NAMESPACE            NAME                                                              READY   STATUS    RESTARTS   AGE
kube-system          cilium-operator-6568655999-2zcqs                                  1/1     Running   0          9h
kube-system          cilium-operator-6568655999-v6qh6                                  1/1     Running   0          9h
kube-system          cilium-tsccc                                                      1/1     Running   0          9h
kube-system          coredns-7db6d8ff4d-46q4g                                          1/1     Running   0          9h
kube-system          coredns-7db6d8ff4d-lk62r                                          1/1     Running   0          9h
kube-system          etcd-cilium-srv6-lab-cluster01-control-plane                      1/1     Running   0          9h
kube-system          kube-apiserver-cilium-srv6-lab-cluster01-control-plane            1/1     Running   0          9h
kube-system          kube-controller-manager-cilium-srv6-lab-cluster01-control-plane   1/1     Running   0          9h
kube-system          kube-proxy-6ddbm                                                  1/1     Running   0          9h
kube-system          kube-proxy-p9smn                                                  1/1     Running   0          9h
kube-system          kube-scheduler-cilium-srv6-lab-cluster01-control-plane            1/1     Running   0          9h
local-path-storage   local-path-provisioner-988d74bc-pjj7x                             1/1     Running   0          9h
```

```console
taahoju3@containerlab01:~/cilium_srv6_lab/Containerlab$ kubectl -n kube-system logs cilium-462ww | grep "Allocated SID"
Defaulted container "cilium-agent" out of: cilium-agent, config (init), mount-cgroup (init), apply-sysctl-overwrites (init), mount-bpf-fs (init), clean-cilium-state (init), install-cni-binaries (init)
time="2024-08-05T21:21:09Z" level=info msg="Allocated SID for VRF with export route target." ExportRouteTarget="65001:1" LocatorPool=pool1 SID="fd00:1:0:e3f9:2baf::" VRF=vrf01 component=srv6.Manager.createIngressPathVRFs subsys=srv6-manager
```

```console
taahoju3@containerlab01:~/cilium_srv6_lab/Containerlab$ kubectl -n kube-system logs cilium-tsccc | grep "Allocated SID"
Defaulted container "cilium-agent" out of: cilium-agent, config (init), mount-cgroup (init), apply-sysctl-overwrites (init), mount-bpf-fs (init), clean-cilium-state (init), install-cni-binaries (init)
time="2024-08-05T21:21:09Z" level=info msg="Allocated SID for VRF with export route target." ExportRouteTarget="65001:1" LocatorPool=pool1 SID="fd00:1:0:b479:71f4::" VRF=vrf01 component=srv6.Manager.createIngressPathVRFs subsys=srv6-manager

```

```console
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl get sidmanager -o custom-columns="NAME:.metadata.name,ALLOCATIONS:.spec.locatorAllocations"
NAME                                      ALLOCATIONS
cilium-srv6-lab-cluster01-control-plane   [map[locators:[map[behaviorType:uSID prefix:fd00:1:0:b479::/64 structure:map[argumentLenBits:0 functionLenBits:32 locatorBlockLenBits:32 locatorNodeLenBits:16]]] poolRef:pool1]]
```

```console
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectx kind-cilium-srv6-lab-cluster02
✔ Switched to context "kind-cilium-srv6-lab-cluster02".
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl get sidmanager -o custom-columns="NAME:.metadata.name,ALLOCATIONS:.spec.locatorAllocations"
NAME                                      ALLOCATIONS
cilium-srv6-lab-cluster02-control-plane   [map[locators:[map[behaviorType:uSID prefix:fd00:2:0:5abc::/64 structure:map[argumentLenBits:0 functionLenBits:32 locatorBlockLenBits:32 locatorNodeLenBits:16]]] poolRef:pool2]]
```

```console
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl get sidmanager -o custom-columns="NAME:.metadata.name,ALLOCATIONS:.status.sidAllocations"
NAME                                      ALLOCATIONS
cilium-srv6-lab-cluster02-control-plane   [map[poolRef:pool2 sids:[map[behavior:uDT4 behaviorType:uSID metadata:vrf02 owner:srv6-manager sid:map[addr:fd00:2:0:5abc:44ef:: structure:map[argumentLenBits:0 functionLenBits:32 locatorBlockLenBits:32 locatorNodeLenBits:16]]]]]]
```

```console
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectx kind-cilium-srv6-lab-cluster01
✔ Switched to context "kind-cilium-srv6-lab-cluster01".
taahoju3@containerlab01:~/cilium_srv6_lab$ kubectl get sidmanager -o custom-columns="NAME:.metadata.name,ALLOCATIONS:.status.sidAllocations"
NAME                                      ALLOCATIONS
cilium-srv6-lab-cluster01-control-plane   [map[poolRef:pool1 sids:[map[behavior:uDT4 behaviorType:uSID metadata:vrf01 owner:srv6-manager sid:map[addr:fd00:1:0:b479:71f4:: structure:map[argumentLenBits:0 functionLenBits:32 locatorBlockLenBits:32 locatorNodeLenBits:16]]]]]]
```

```console
taahoju3@containerlab01:~$ kubectl -n kube-system get pods -l k8s-app=cilium
NAME           READY   STATUS    RESTARTS   AGE
cilium-tsccc   1/1     Running   0          8h <--- Control-plane
```

```console
taahoju3@containerlab01:~$ kubectl -n kube-system exec cilium-tsccc -- cilium-dbg bgp peers
Defaulted container "cilium-agent" out of: cilium-agent, config (init), mount-cgroup (init), apply-sysctl-overwrites (init), mount-bpf-fs (init), clean-cilium-state (init), install-cni-binaries (init)
Local AS   Peer AS   Peer Address              Session       Uptime    Family          Received   Advertised
65001      65000     fc00:0:3333::1:179        established   8h3m51s   ipv4/mpls_vpn   2          1
65001      65000     fd00:172:100:100::1:179   established   8h26m7s   ipv6/unicast    3          1
```
