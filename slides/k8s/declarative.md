## Declarative vs imperative in Kubernetes

- With Kubernetes, we do not say: "run this container"
  - (`kubectl run` really just generates a declarative specification...)

- All we can do is write a *spec* and push it to the API server
  - (by creating a resource like e.g. a Pod or a Deployment)

- The API server will validate that spec (and reject it if it's invalid)

- Then it will store it in `etcd`

- A *controller* will "notice" that spec and act upon it

---

## Reconciling state

- Watch for the `state` and `spec` fields in the YAML files later!

  - The `state` describes *what the thing currently is*

  - The `spec` describes *what we want the thing to be*

- Kubernetes controllers monitor objects they are responsible for, *doing something* when the `state` does not match the `spec`

- When we want to change some resource, we update the *spec*

- The controller responsible for that resource will *converge* the change
