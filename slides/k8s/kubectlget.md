## `kubectl describe`

- `kubectl describe` needs a resource type and (optionally) a resource name

- It is possible to provide a resource name *prefix*

  (all matching objects will be displayed)

- `kubectl describe` will retrieve some extra information about the resource

.exercise[

- Look at the information available for `node1` with one of the following commands:
  ```bash
  kubectl describe node/node1
  kubectl describe node node1
  ```

]

(We should notice a bunch of control plane pods.)

---

## Example object: `services`

- A *service* is a stable endpoint to connect to "something"

  (In the initial proposal, they were called "portals")

.exercise[

- List the services on our cluster with one of these commands:
  ```bash
  kubectl get services
  kubectl get svc
  ```

]

--

There is already one service on our cluster: the Kubernetes API itself.

---

## `describe` the `service`

- Let's contrast `get` and `describe`

- Both take an extra argument, the name of the object

- `describe` without a name can be overwhelming, but it's there...

.exercise[

- Compare the output of both styles:
  ```bash
  kubectl get services kubernetes
  kubectl describe service kubernetes
  ```

]

- Try it with the following examples as you go.

--

*The exercise doesn't have misuse of plurality, it's there to demonstrate the plural aliases are not semantically significant (they don't mean anything)...*

---

## Tab completion

- You can move faster with `kubectl` by using the tab key.

.exercise[

- Let `kubectl` type the name of the service for you:
  ```bash
  kubectl des`<tab>` ser`<tab>` `<tab>`
  ```
  
- If the terminal beeps after hitting `<tab>`, hit it a second time to disambiguate options
]

- This is so convenient, you'll want to put your terminal in "silent alert" mode (flashes the screen instead of making a sound).

---

## Example object: ClusterIP services

- A `ClusterIP` service is internal, available from the cluster only

- This is useful for introspection from within containers

.exercise[

- Try to connect to the API:
  ```bash
  curl -k https://`10.96.0.1`
  ```
  
  - `-k` is used to skip certificate verification

  - Make sure to replace 10.96.0.1 with the CLUSTER-IP shown by `kubectl get svc`

]

--

The error that we see is expected: the Kubernetes API requires authentication.

---

## Example object: Listing running containers

- Containers are manipulated through *pods*

- A pod is a group of containers:

 - running together (on the same node)

 - sharing resources (RAM, CPU; but also network, volumes)

.exercise[

- List pods on our cluster:
  ```bash
  kubectl get pods
  ```

]

--

*Where are the pods that we saw just a moment earlier?!?*

---

## Example object: Namespaces

- Namespaces allow us to segregate resources

.exercise[

- List the namespaces on our cluster with one of these commands:
  ```bash
  kubectl get namespaces
  kubectl get namespace
  kubectl get ns
  ```
  
  - Don't forget about the tab key to type the word `namespaces`

]

--

*You know what... This `kube-system` thing looks familiar.*

*In fact, I'm pretty sure it showed up earlier, when we did:*

`kubectl describe node node1`

---

## Accessing namespaces

- By default, `kubectl` uses the `default` namespace

- We can see resources in all namespaces with `--all-namespaces`

.exercise[

- List the pods in all namespaces:
  ```bash
  kubectl get pods --all-namespaces
  ```

- Since Kubernetes 1.14, we can also use `-A` as a shorter version:
  ```bash
  kubectl get pods -A
  ```

]

*Here are our system pods!*

---

## What are all these pods?

Kubernetes has control logic, packaged as pods so it can be managed and monitored: 

- `etcd` is the database that stores all configuration objects

- `kube-apiserver` is the API server we have been talking to the whole time

- We'll cover `kube-controller-manager` and `kube-scheduler` shortly

- `coredns` is an internal dynamic DNS server that is synchronized to current state 

- `kube-proxy` sets up NAT mappings for container interfaces

- `weave` is a pluggable component that manages networking between cluster nodes

- the `READY` column indicates the number of containers in each pod

  (1 for most pods, but `weave` has 2, for instance)

---

## Scoping another namespace

- We can also look at a different namespace (other than `default`)

.exercise[

- List only the pods in the `kube-system` namespace:
  ```bash
  kubectl get pods --namespace=kube-system
  kubectl get pods -n kube-system
  ```

]

---

## Namespaces and other `kubectl` commands

- We can use `-n`/`--namespace` with almost every `kubectl` command

- Example:

  - `kubectl create --namespace=X` to create something in namespace X

- We can use `-A`/`--all-namespaces` with most commands that manipulate multiple objects

- Examples:

  - `kubectl delete` can delete resources across multiple namespaces

  - `kubectl label` can add/remove/update labels across multiple namespaces

---

class: extra-details

## What about `kube-public`?

.exercise[

- List the pods in the `kube-public` namespace:
  ```bash
  kubectl -n kube-public get pods
  ```

]

Nothing!

`kube-public` is created by kubeadm & [used for security bootstrapping](https://kubernetes.io/blog/2017/01/stronger-foundation-for-creating-and-managing-kubernetes-clusters).

---

class: extra-details

## Whoa wait, `kubeadm`?

- There are a lot of projects that start with `kube`, it can be hard to keep track of them all. Pay close attention, they are all quite distinct!

- `kubeadm` is the set of installers and initial configurators that are provided and supported by the Kubernetes team. [This is a good explanation](https://www.weave.works/technologies/kubernetes-installation-options/).

- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) is the canonical reference for how to install Kubernetes manually. It's not meant so much as a production guide as it is a means to learn exactly what is installed and where the parts come from.

- `kubectl` is the command line for controlling a finished Kubernetes installation and will *always* be required, no matter which installer you use.

---

class: extra-details

## Exploring `kube-public`

- The only interesting object in `kube-public` is a ConfigMap named `cluster-info`

.exercise[

- List ConfigMap objects:
  ```bash
  kubectl -n kube-public get configmaps
  ```

- Inspect `cluster-info`:
  ```bash
  kubectl -n kube-public get configmap cluster-info -o yaml
  ```

]

Note the `selfLink` URI: `/api/v1/namespaces/kube-public/configmaps/cluster-info`

We can use that!

---

class: extra-details

## Accessing `cluster-info`

- Earlier, when trying to access the API server, we got a `Forbidden` message

- But `cluster-info` is readable by everyone (even without authentication)

.exercise[

- Retrieve `cluster-info`:
  ```bash
  curl -k https://10.96.0.1/api/v1/namespaces/kube-public/configmaps/cluster-info
  ```

]

- We were able to access `cluster-info` (without auth)

- It contains a `kubeconfig` file

---

class: extra-details

## Retrieving `kubeconfig`

- We can easily extract the `kubeconfig` file from this ConfigMap

.exercise[

- Display the content of `kubeconfig`:
  ```bash
    curl -sk https://10.96.0.1/api/v1/namespaces/kube-public/configmaps/cluster-info \
         | jq -r .data.kubeconfig
  ```

]

- This file holds the canonical address of the API server, and the public key of the CA

- This file *does not* hold client keys or tokens

- This is not sensitive information, but allows us to establish trust

---

class: extra-details

## What about `kube-node-lease`?

- Starting with Kubernetes 1.14, there is a `kube-node-lease` namespace

  (or in Kubernetes 1.13 if the NodeLease feature gate is enabled)

- That namespace contains one Lease object per node

- *Node leases* are a new way to implement node heartbeats

  (i.e. node regularly pinging the control plane to say "I'm alive!")

- For more details, see [KEP-0009] or the [node controller documentation]

[KEP-0009]: https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/0009-node-heartbeat.md
[node controller documentation]: https://kubernetes.io/docs/concepts/architecture/nodes/#node-controller
