# SRv6 BGP L3VPN Demo

## Topology
```mermaid
flowchart LR
    VPN1("VPN 10.3.0.0/24")
    VPN2("VPN 10.3.0.0/24")
    CE0(("CE0"))
    CE1(("CE1"))
    PE0(("PE0"))
    PE1(("PE1 (Cilium)"))
    Pod1("Pod1 (vrf-0)")
    Pod2("Pod2 (vrf-1)")

    VPN1---CE0
    VPN2---CE1
    CE0<-->|"Advertise VPN 10.3.0.0/24 (vrf-0)"|PE0
    CE1<-->|"Advertise VPN 10.3.0.0/24 (vrf-1)"|PE0
    PE0-->|"Advertise VPNv4 SRv6 L3VPN routes"|PE1
    PE1-->|"Encap pod egress traffic with SRv6 SID"|PE0
    PE1---Pod1
    PE1---Pod2
```

Our demonstration topology consists of a bare-minimum `srv6 l3vpn` configuration.

`PE0` is an `FRR` router configured with two `CE` routers in which it learns VPN
routes from.

`CE0` belongs to `vrf-0` and exports the VPN route `10.3.0.0/24`.

Likewise, `CE1` belongs to `vrf-1` and exports an overlapping `10.3.0.0/24` VPN
route.

`PE1` is a `Cilium` node running `Kubernetes`. This `Cilium` node is acting as a
`PE` in our topology. It runs two pods, `Pod1` belongs to `vrf-0` while `Pod2`
belongs to `vrf-1`.

We will demonstrate that `PE1 (Cilium)`  is capable of both learning the VPN networks of both
`CE` devices and encapsulating/decapsulating the traffic between `CE0` and `Pod1`.

When a ping is issued from `Pod1` to `10.3.0.1`, `PE1 (Cilium)` will encapsulate the traffic
in a `SID` advertised by `PE0`, informing it how to route toward `CE0`.
Likewise, the return traffic will be encapsulated by `PE0` with a  `SID` learned from `PE1 (Cilium)`,
decapulated by `Cilium's datapath` and delivered to `Pod1`.

Note, the traffic will remain in the `vrf-0` path the entire time, as the originator of the traffic `Pod1` belongs to `vrf-0`.

## Prequisites

This demo is confirmed working with the following required software:

[Kind v0.14.0](https://github.com/kubernetes-sigs/kind/releases/tag/v0.14.0) - Automated local kubernetes deployment.

[Containerlab v0.38.0](https://github.com/srl-labs/containerlab/releases/tag/v0.38.0) - Automated local network topology deployment.

[Helm v3.11](https://github.com/helm/helm/releases/tag/v3.11.2) - Installation of Cilium's chart

[Kubectl v1.24.7](https://github.com/kubernetes/kubectl/releases/tag/kubernetes-1.24.7) - Kubectl client for issuing commands.

[Docker v20.10.20](https://github.com/moby/moby/releases/tag/v20.10.21) - Docker which runs Kubernetes cluster on local host.

## Running the demonstration

### Deploy the infrastructure

At the root of the directory this `README.md` file is located issue the following
`make` target.

```shell
$ make deploy
```

You will be asked to `sudo` if you are not `root`.

This will deploy the diagrammed network topology and also program `kubectl` to issue
commands to the local `kind` cluster moving forward.

Confirm that all pods in our demonstration are running:
```
kubectl get pods --all-namespaces
NAMESPACE            NAME                                                           READY   STATUS    RESTARTS   AGE
default              netshoot-vrf0-tqz46                                            1/1     Running   0          8m12s
default              netshoot-vrf1-86s9m                                            1/1     Running   0          8m12s
kube-system          cilium-fld5z                                                   1/1     Running   0          8m28s
kube-system          cilium-operator-556485db6b-sjq75                               1/1     Running   0          8m28s
kube-system          coredns-6d4b75cb6d-dggcl                                       1/1     Running   0          8m28s
kube-system          coredns-6d4b75cb6d-vrl7v                                       1/1     Running   0          8m28s
kube-system          etcd-clab-srv6-vpnv4-simple-control-plane                      1/1     Running   0          8m42s
kube-system          kube-apiserver-clab-srv6-vpnv4-simple-control-plane            1/1     Running   0          8m42s
kube-system          kube-controller-manager-clab-srv6-vpnv4-simple-control-plane   1/1     Running   0          8m42s
kube-system          kube-proxy-8dtjr                                               1/1     Running   0          8m28s
kube-system          kube-scheduler-clab-srv6-vpnv4-simple-control-plane            1/1     Running   0          8m42s
local-path-storage   local-path-provisioner-9cd9bd544-h7pqd                         1/1     Running   0          8m28s
```
Our demo will focus on issuing pings from the `netshoot-vrf0-*` container in `vrf-0`.

Confirm that `FRR` has setup the initial routing table correctly based on its `srv6 l3vpn` configuration.
```
docker exec clab-srv6-vpnv4-simple-pe0 'ip' '-6' 'route' 'show'
b:1::/64 dev net0 proto kernel metric 256 pref medium
b:2:0:0:100:: nhid 19  encap seg6local action End.DT4 dev vrf0 proto bgp metric 20 pref medium
b:2:0:0:200:: nhid 20  encap seg6local action End.DT4 dev vrf1 proto bgp metric 20 pref medium
c::/64 via b:1::2 dev net0 metric 1024 pref medium
2001:172:20:20::/64 dev eth0 proto kernel metric 256 pref medium
fe80::/64 dev eth0 proto kernel metric 256 pref medium
fe80::/64 dev net0 proto kernel metric 256 pref medium
default via 2001:172:20:20::1 dev eth0 metric 1024 pref medium
```

You'll want to confirm the `End.DT4` routes are present, these will handle the decapulation
and delivery of the SID to the associated `vrf` device.

## Apply policy

Two `Kubernetes` policies must be applied, one for configuring a `BGP` peer between
`PE1 (Cilium)` and `PE0`, and the other to configure `Pod1` as belonging to `vrf-0`.

Both policies can be applied with the following `make` target:
```
make policy
```

After these policies are ran `Cilium` will have allocated a `SID` for `vrf-0` and
advertised it forward to `PE1`.

Confirm this in the logs:
```
kubectl -n kube-system logs cilium-fld5z | grep "Allocated SID"
Defaulted container "cilium-agent" out of: cilium-agent, mount-cgroup (init), apply-sysctl-overwrites (init), clean-cilium-state (init)
level=info msg="Allocated SID for VRF with export route target." ExportRouteTarget="65010:1" SID="c::237c" component=srv6.Manager.createIngressPathVRFMapping subsys=srv6 vrfID=1
```

Next, confirm `PE0` has learned of this SID and created the correct `H.Encap` rules for the return traffic.
```
docker exec clab-srv6-vpnv4-simple-pe0 'ip' 'route' 'show' 'table' '100'
10.1.0.0/24 nhid 37  encap seg6 mode encap segs 1 [ c::237c ] via 172.0.0.2 dev net0 proto bgp metric 20
10.3.0.0/24 nhid 31 via 169.254.0.2 dev net1 proto bgp metric 20
10.3.0.1 nhid 31 via 169.254.0.2 dev net1 proto bgp metric 20
169.254.0.0/24 dev net1 proto kernel scope link src 169.254.0.1
local 169.254.0.1 dev net1 proto kernel scope host src 169.254.0.1
broadcast 169.254.0.255 dev net1 proto kernel scope link src 169.254.0.1
```

## Perform end-to-end testing

### Start the ping from Pod1.

Perform a ping from pod `netshoot-vrf0-*` toward `CE0` VPN service `10.3.0.1`.
```
kubectl exec netshoot-vrf0-tqz46 "ping" "10.3.0.1"
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
PING 10.3.0.1 (10.3.0.1) 56(84) bytes of data.
64 bytes from 10.3.0.1: icmp_seq=1 ttl=62 time=0.214 ms
64 bytes from 10.3.0.1: icmp_seq=2 ttl=62 time=0.380 ms
64 bytes from 10.3.0.1: icmp_seq=3 ttl=62 time=0.209 ms
64 bytes from 10.3.0.1: icmp_seq=4 ttl=62 time=0.107 ms
64 bytes from 10.3.0.1: icmp_seq=5 ttl=62 time=0.318 ms
```

Notice the return traffic coming back.

### Show the encap/decap from PE1(Cilium) -> PE0(FRR)
```
docker exec clab-srv6-vpnv4-simple-pe1 "tcpdump" "-ni" "net0"
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on net0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:28:58.423806 IP6 c::237c > b:2:0:0:100::: RT6 (len=2, type=4, segleft=0, last-entry=0, tag=0, [0]b:2:0:0:100::) IP 10.1.0.170 > 10.3.0.1: ICMP echo request, id 30042, seq 163, length 64
16:28:58.423943 IP6 b:1::1 > c::237c: RT6 (len=2, type=4, segleft=0, last-entry=0, tag=0, [0]c::237c) IP 10.3.0.1 > 10.1.0.170: ICMP echo reply, id 30042, seq 163, length 64
```
In the above tcpdump notice traffic from `pod:10.1.0.170` toward `10.3.0.1` is encapsulated in SID learned from `PE0`, `b:2:0:0:100:::` toward `vrf-0 (CE0)`.

### Show decap from PE0(FRR) -> `CE0` via vrf device.

Encapsulated traffic entering `net0` on `pe0`
```
docker exec clab-srv6-vpnv4-simple-pe0 "tcpdump" "-ni" "net0"
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on net0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:34:26.103884 IP6 c::237c > b:2:0:0:100::: RT6 (len=2, type=4, segleft=0, last-entry=0, tag=0, [0]b:2:0:0:100::) IP 10.1.0.170 > 10.3.0.1: ICMP echo request, id 30042, seq 483, length 64
16:34:26.103993 IP6 b:1::1 > c::237c: RT6 (len=2, type=4, segleft=0, last-entry=0, tag=0, [0]c::237c) IP 10.3.0.1 > 10.1.0.170: ICMP echo reply, id 30042, seq 483, length 64
```

Decapsulated traffic leaving `vrf0` device toward `CE0`
```
docker exec clab-srv6-vpnv4-simple-pe0 "tcpdump" "-ni" "vrf0"
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on vrf0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:36:32.055964 IP 10.1.0.170 > 10.3.0.1: ICMP echo request, id 30042, seq 606, length 64
16:36:32.056017 IP 10.3.0.1 > 10.1.0.170: ICMP echo reply, id 30042, seq 606, length 64
```

### Show service traffic on `CE0`
```
docker exec clab-srv6-vpnv4-simple-ce0 "tcpdump" "-ni" "net0"
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on net0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
16:42:24.311924 IP 10.1.0.170 > 10.3.0.1: ICMP echo request, id 30042, seq 950, length 64
16:42:24.311958 IP 10.3.0.1 > 10.1.0.170: ICMP echo reply, id 30042, seq 950, length 64
```
