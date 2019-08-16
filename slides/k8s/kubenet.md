# Kubernetes network model assumptions

- We have a bunch of *ephemeral* containers: *they don't last long*

- If a container dies, it shouldn't (quickly) reuse an address: *services might keep trying that address and fail on the replacement service.*

- Containers might last only a few seconds and we might launch a lot of them over time: *we need a lot of addresses!*

- We can't consume a lot of addresses without disrupting where we are deployed.

**BUT**

- Nodes and pods must be able to reach each other directly in this large address space and be self-aware!

---

## Enter [Container Network Interface](https://github.com/containernetworking/cni/blob/master/SPEC.md)

  **Q:** *What if our premises are wrong?*
--

  <br/>**A: *Make the implementation pluggable!***

--

CNI is a simple interface that allows a myriad of subtle means to connect short-lived (*ephemeral!*) addresses in virtual environments between cluster nodes:
  - Many implementations use `iptables`: *Kubernetes quickly found the folly of that path!*
  
  - Newer implementations started using `ipvs`: *better!*
  
  - [One implementation](https://cilium.io) uses [BPF](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter): *wow!*
  
  - [Another implementation](https://github.com/intel/multus-cni) allows multiple separate CNI implementations to be loaded: *whoa...*
 
***The possibilities are endless!!***

---

## Kubernetes network model: the good

- Everything can reach everything

- No address translation

- No port translation

- No new protocol

- The network implementation can decide how to allocate addresses

- IP addresses don't have to be "portable" from a node to another

  (We can use e.g. a subnet per node and use a simple routed topology)

- The specification is simple enough to allow many various implementations

---

## Kubernetes network model: the less good

- Everything can reach everything

  - if you want security, you need to add network policies

  - the network implementation that you use needs to support them

- There are literally dozens of implementations out there

  (15 are listed in the Kubernetes documentation)

- Pods have level 3 (IP) connectivity, but *services* are level 4 (TCP or UDP)

  (Services map to a single UDP or TCP port; no port ranges or arbitrary IP packets)

- `kube-proxy` is on the data path when connecting to a pod or container,
  <br/>and it's not particularly fast (relies on userland proxying or iptables)

---

## Kubernetes network model: in practice

- The nodes that we are using have been set up to use [Weave](https://github.com/weaveworks/weave)

- We don't endorse Weave in a particular way, it Just Works

- Don't worry about the warning about `kube-proxy` performance

- Unless you:

  - routinely saturate 10G network interfaces
  - count packet rates in millions per second
  - run high-traffic VOIP or gaming platforms
  - do weird things that involve millions of simultaneous connections
    <br/>(in which case you're already familiar with kernel tuning)

- If necessary, there are alternatives to `kube-proxy`; e.g.
  [`kube-router`](https://www.kube-router.io)

---

class: extra-details

## CNI Details

- Kubernetes clusters always use a CNI plugin to implement networking, (even if that plugin delegates to other plugins or to a upstream hosting provider like AWS)

- When a pod is created, Kubernetes gives the abstract command to the plugin

- Typically, CNI plugins will:

  - allocate an IP address (by calling an IPAM plugin)

  - add a network interface into the pod's network namespace

  - configure the interface as well as required routes etc.

---

class: extra-details

## Multiple moving parts

- The "pod-to-pod network" or "pod network":

  - provides communication between pods and nodes

  - is generally implemented with CNI plugins

- The "service-to-pod network" or "service network":

  - provides internal communication and load balancing

  - is generally implemented with kube-proxy (or e.g. kube-router)

- Network policies:

  - provide firewalling and isolation

  - can be bundled with the "pod network" or provided by another component

---

class: extra-details

## Even more moving parts

- Inbound traffic can be handled by multiple components:

  - something like kube-proxy or kube-router (for NodePort services)

  - load balancers (ideally, connected to the pod network)

- Some solutions can fill multiple roles

  (e.g. kube-router can be set up to provide the pod network and/or network policies and/or replace kube-proxy)

---

## So many features, how do I choose one?

Answer: Don't choose your CNI, let it choose you!

- You already started with Weave. Maybe stick with it?

- No matter what you start with, you'll find reasons another one is better:
  - Maybe your internal infrastructure is already using BGP (for instance)
  - Production is seeing problems related to specific capabilities of your current CNI
  - ???
  
***CNI is a deep rabbit hole. Expect to change once you know better!***