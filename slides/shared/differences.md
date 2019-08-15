## Kubernetes Roadmap

- It's helpful to ponder what we just did in the context of where we are going.
  - Built a microservice app from source
  - Deployed it in Docker containers
  - Did it on the **same** machines we'll run with Kubernetes.
  
- What we need to learn is how to abstract out the concrete resources of these machines:
  - CPU / VM
  - Memory
  - Network
  - Disk
  
---

## Abstracting

- With Docker Compose, we brought our app up on a single machine. The machine has specific allocations of CPU, memory and network resources:

  - CPU / Memory - Allocated when the VM was created. Is it enough? What to do if not?
  
  - Network - Has a specific IP address that won't be the same across a cluster. How can containers reach each other and be reached by the outside world?
  
  - Disk - Isn't replicated across machines. If the machine isn't accessible, data on the disk isn't either!
  
--

- How do we solve these problems?
 
---

## Core Kubernetes value proposition

- Early versions of Kubernetes really only solved these basic abstractions. It [wasn't very secure](https://www.cvedetails.com/cve/CVE-2016-1906/) in a world of advanced threats in multitenant environments. 

- Early versions also didn't handle these abstractions with today's flexibility. Containers with specific requirements were harder (or even impossible) to deploy in older versions.

- Much of the complexity that's been added is to securely support containers of increasingly hostile nature and do so with increasingly fewer demands on the container structure itself.

--

- **Result:** We can deploy a wider variety of workloads more securely and quickly than we ever could in the past!
- **Trend:** More of the same!!

---

## So what's the plan?

- In order to extract the Kubernetes core value proposition, what we are really doing is **describing each app to the orchestrator**.
- As we declare each aspect of the application that needs to be exposed or supported (CPU, memory, disk, network), we let Kubernetes deal with the "how":

  - Which machine(s) in our cluster has the basic resources to run the app?

  - If the app is described to be reachable over the network, what ports need to be exposed? Where should they be exposed?

  - Provide the means to transparently find other apps

  - Provide the means to transparently firewall *from* other apps

--

- **Let's do this!**
