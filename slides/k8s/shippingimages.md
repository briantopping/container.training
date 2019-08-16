# Distributing images via a shared registry

- Initially, our `docker-compose` app was running on a single node

- We could *build* and *run* in the same place

- Now that we want to run on a cluster, there are *many* nodes with registries to keep updated. We don't want to build on each one!

- The easiest way to distribute container images is to use a registry

---

## How Docker registries work (a reminder)

What happens when we execute `docker run alpine`?

- If the image exists in the machine's registry, it is used

- If the Engine needs to pull the `alpine` image, it expands it into `library/alpine`

- `library/alpine` is expanded into `index.docker.io/library/alpine`

- The Engine communicates with `index.docker.io` to retrieve `library/alpine:latest`

- To use something else than `index.docker.io`, we specify it in the image name

- Examples:
  ```bash
  docker pull gcr.io/google-containers/alpine-with-bash:1.0

  docker build -t registry.mycompany.io:5000/myimage:awesome .
  docker push registry.mycompany.io:5000/myimage:awesome
  ```

---

## Running DockerCoins on Kubernetes

- Create one `Deployment` configuration for each component

  (hasher, redis, rng, webui, worker)

- Expose deployments that need to accept connections, creating `Service` configs

  (hasher, redis, rng, webui)

  - For redis, we can use the official redis image

  - For the 4 others, we need to build images and push them to some registry

---

## Building and shipping images

Standard developer workflows, with additional push to registries

- Manually on each worker node:

  - build (with `docker build` or otherwise) and push to the local registry

- Automatically via Continuous Integration:

  - build and test on developer machine
  - when ready, commit and push a code repository (Git)
  - the code repository notifies an automated build system (Jenkins)
  - CI system gets the code, builds it, pushes the docker image to the shared registry
  - worker nodes are configured to pull from this shared registry

***How does your company do things?***

---

## Which registry do we want to use?

- There are SAAS products like Docker Hub, Quay ...

- Each major cloud provider has an option as well

  (ACR on Azure, ECR on AWS, GCR on Google Cloud...)

- There are also commercial products to run our own registry

  (Nexus, Artifactory, Docker EE, Quay...)

- When picking a registry, pay attention to your ecosystem needs
  - Build system
  - LDAP integration
  - Testing tools (especially automated security tools)
