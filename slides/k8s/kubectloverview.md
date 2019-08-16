# First contact with `kubectl`

- `kubectl` is (almost) the only tool we'll need to talk to Kubernetes

- It is a rich CLI tool that talks to the Kubernetes API

  (Everything you can do with `kubectl`, you can do directly with the API)

- **Everything** is configured with `~/.kube/config`:

  - the Kubernetes API address

  - the path to our TLS certificates used to authenticate

- You can also use the `--kubeconfig` flag to pass a config file

- Or directly `--server`, `--user`, etc.

- `kubectl` can be pronounced "Cube C T L", "Cube cuttle", "Cube cuddle"...

---

## `kubectl get`

- Let's look at our `Node` resources with `kubectl get`!

.exercise[

- Look at the composition of our cluster:
  ```bash
  kubectl get node
  ```

- These commands are equivalent:
  ```bash
  kubectl get no
  kubectl get node
  kubectl get nodes
  ```

]

---

## Where can we use it?

- `kubectl` only needs two things to run:

  - The config file / keys

  - Firewall access to the control plane

- Any machine that has these two things can run `kubectl`. But for the most part, one only needs it where they are typically on shell. In our case, this is `node1`.

--

- As we'll see later, it's easy to run it from your own laptop, etc.

--

- You and your team need to decide if this is a good idea or not **because the API port needs to be accessible to your laptop**.
  - ...Works great over a VPN though!!

---

## Obtaining machine-readable output

- `kubectl get` can output JSON, YAML, or be directly formatted

.exercise[

- Give us more info about the nodes:
  ```bash
  kubectl get nodes -o wide
  ```

- Let's have some YAML:
  ```bash
  kubectl get no -o yaml
  ```
  See that `kind: List` at the end? It's the type of our result!

]

---

## (Ab)using `kubectl` and `jq`

- It's super easy to build custom reports

.exercise[

- Show the capacity of all our nodes as a stream of JSON objects:
  ```bash
    kubectl get nodes -o json | 
            jq ".items[] | {name:.metadata.name} + .status.capacity"
  ```

]

--

- Note the key difference there with the output type of `json`. That provides a usable input format to `jq`. 

---

class: extra-details

## Exploring types and definitions

- We can list all available resource types by running `kubectl api-resources`
  <br/>
  (In Kubernetes 1.10 and prior, this command used to be `kubectl get`)

- We can view the definition for a resource type with:
  ```bash
  kubectl explain type
  ```

- We can view the definition of a field in a resource, for instance:
  ```bash
  kubectl explain node.spec
  ```

- Or get the full definition of all fields and sub-fields:
  ```bash
  kubectl explain node --recursive
  ```

---

class: extra-details

## Introspection vs. documentation

- We can access the same information by reading the [API documentation](https://kubernetes.io/docs/reference/#api-reference)

- The API documentation is usually easier to read, but:

  - it won't show custom types (like Custom Resource Definitions)

  - we need to make sure that we look at the correct version

- `kubectl api-resources` and `kubectl explain` perform *introspection*

  (they communicate with the API server and obtain the exact type definitions)

--

- Google is your friend. Sometimes blog entries by independent developers are better at explaining subtle concepts and accellerating learning...
---

## Type names

- The most common resource names have three forms:

  - singular (e.g. `node`, `service`, `deployment`)

  - plural (e.g. `nodes`, `services`, `deployments`)

  - short (e.g. `no`, `svc`, `deploy`)

- Some resources do not have a short name

---

## Viewing details

- We can use `kubectl get -o yaml` to see all available details

- However, YAML output is often simultaneously too much and not enough

- For instance, `kubectl get node node1 -o yaml` is:

  - too much information (e.g.: list of images available on this node)

  - not enough information (e.g.: doesn't show pods running on this node)

  - difficult to read for a human operator

--

- Instead, we can use `kubectl describe`

