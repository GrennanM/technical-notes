# Kubernetes Up and Running

- [Kubernetes Up and Running](#kubernetes-up-and-running)
  - [Chapter 1 - Introduction](#chapter-1---introduction)
    - ["not my monkey, not my circus"](#not-my-monkey-not-my-circus)
  - [Chapter 2 - Creating and running containers](#chapter-2---creating-and-running-containers)
    - [Docker Images](#docker-images)
  - [Chapter 3 - Deploying a Kubernetes cluster](#chapter-3---deploying-a-kubernetes-cluster)
    - [Cluster Status](#cluster-status)
      - [Cluster components](#cluster-components)
  - [Chapter 4 - Common kubectl commands](#chapter-4---common-kubectl-commands)
  - [Chapter 5 - Pods](#chapter-5---pods)
      - [Health checks](#health-checks)
      - [Resource Management](#resource-management)
      - [Ways of using Volumes with Pods](#ways-of-using-volumes-with-pods)
  - [Chapter 6 - Labels and Annotations](#chapter-6---labels-and-annotations)
      - [Labels](#labels)
      - [Annotations](#annotations)
  - [Chapter 7 - Service Discovery](#chapter-7---service-discovery)
    - [Why not DNS?](#why-not-dns)
      - [Service Object](#service-object)
      - [Cluster IP](#cluster-ip)
      - [Readiness Checks](#readiness-checks)
      - [NodePorts](#nodeports)
      - [Endpoints](#endpoints)
      - [`kube-proxy` and cluster IPs](#kube-proxy-and-cluster-ips)
  - [Chapter 8 - HTTP Load Balancing with Ingress](#chapter-8---http-load-balancing-with-ingress)
  - [Chapter 9 - ReplicaSets](#chapter-9---replicasets)
  - [Chapter 10 - Deployments](#chapter-10---deployments)
  - [Chapter 11 - DaemonSets](#chapter-11---daemonsets)
  - [Chapter 12 - Jobs](#chapter-12---jobs)
  - [Chapter 13 - ConfigMaps and Secrets](#chapter-13---configmaps-and-secrets)
  - [Chapter 14 - Role-Based Access Control](#chapter-14---role-based-access-control)

## Chapter 1 - Introduction

- **Velocity** is the number of features that can be shipped while maintaining a highly available service.
- Core concepts:

1. **Immutability**

- In an **immutable** system, rather than a series of incremental updates and changes, an entirely new, complete image is built.
- The update replaces the entire image with the newer image.
- The old image still exists and can be used for a rollback if necessary.

1. **Declarative Configuration**

- Everything in Kubernetes is a declarative configuration object that represents the desired state of the system

1. **Online self-healing systems**

- Kubernetes _continuously_ takes actions to ensure that the current state matches the desired state.

- Kubernetes provides **efficiency** because it allows:
  - multiple applications to run on the same machine
  - sharing of resource usage (CPU, RAM etc.) between applications on the same machine
  - higher utilization of resources
  - easier and cheaper setup of test environment encourages greater testing and hence, quicker feedback cycles

### "not my monkey, not my circus"

- There is a separation of responsibilities in Kubernetes between the application operator and the cluster orchestration operator.
- The application developer relies on the SLA delivered by the container orchestration API.
- The container orchestration engineer focuses on delivering this SLA without worrying about the applications running on top of it.

- What is "infrastructure as code"?
  - Storing declarative configuration in source control.

## Chapter 2 - Creating and running containers

### Docker Images

- Docker packages an executable and pushes it to a remote repository where others can pull the container image
- A **container image** is a binary package that encapsulates all of the files necessary to run a program inside of an OS container.

- Docker Image format:
  - Each layer adds, removes, or modifies files from the preceding layer in the file system. This is an example of an **overlay file system**.
  - Note that if a big file present in an earlier layer is deleted in a later layer, it will still be present in the final image.
  - The image isn’t a single file but rather a specification for a manifest file that points to other files.

- **Multi-stage builds** in docker produce more than one image from a single Dockerfile. This enables you to do the heavy lifting in one image (installing development tools, building binary, etc.) and just have the compiled binary in the second image.

## Chapter 3 - Deploying a Kubernetes cluster

### Cluster Status

- `kubectl get componentstatuses`
- Kubernetes cluster components:
  - `controller-manager` - responsible for running controllers that regulate behaviour - e.g. ensuring all replicas of a service are healthy
  - `scheduler` - placing pods onto different nodes
  - `etcd` - storage for the cluster

- `kubectl get nodes` - lists nodes in a cluster - comprised of master and worker nodes
- `kubectl describe nodes <node-name>`
- Resources requested by a Pod are guaranteed to be present on the node, while a Pod’s limit is the maximum amount of a given resource that a Pod can consume. A Pod’s limit can be higher than its request, in which case the extra resources are supplied on a best-effort basis.

#### Cluster components

- Many of the components that make up a Kubernetes cluster are deployed using Kubernetes. These run in the _kube system_ namespace:
  - `kube-proxy` - responsible for routing network traffic to load-balanced services in the Kubernetes cluster
  - `core-dns` - Kubernetes also runs a DNS server, which provides naming and discovery for the services that are defined in the cluster.


## Chapter 4 - Common kubectl commands

- The `kubectl` command makes HTTP requests to a URL to access the Kubernetes objects that reside at these paths. E.g. https://your-k8s.com/api/v1/name-spaces/default/pods/my-pod
- `kubectl get <resource-name>` - list all resources in current namespace
- `kubectl get <resource-name> <obj-name>` - view a specific resource
- `-o yaml`, `-o wide`, `-o json` provide additional detail
- `kubectl top pods -n edge-squad --context backend-ch1-prod` - display total resource usage by pod

## Chapter 5 - Pods

- Kubernetes groups multiple containers and volumes into a single atomic unit called a **Pod**.
- Pods, not containers, are the smallest deployable artifact in a Kubernetes cluster.
- Applications in the same pod share the same IP address, port space, hostname.
- Applications on the same node but in different pods may as well be on different servers.
- In general, the right question to ask yourself when designing Pods is, “Will these containers work correctly if they land on different machines?”

#### Health checks

- Kubernetes has health checks for application _liveness_. This runs application specific health checks to ensure an application is running correctly. The liveness check is defined in the health manifest.
- **Readiness** describes when a container is ready to serve requests.

#### Resource Management

- **Utilization** is defined as the amount of a resource actively being used divided by the amount of a resource that has been purchased.
- **Resource requests** specify the _minimum_ amount of a resource required to run the application.
- **Resource limits** specify the _maximum_ amount of a resource that an application can consume.
- If a pod is over it's memory request, the OS can't just remove memory from it. If the node needs more memory it will terminate containers whose memory usage is greater than their requested memory.
- Kubernetes measures CPU in cores. E.g. `100m.` The unit suffix `m` stands for “thousandth of a core,” so this resources object specifies that the container process needs 50/1000 of a core (5%).

#### Ways of using Volumes with Pods

- Can be used as a persistant cache (e.g. for thumbnails)
- Can be used as a common filestore to be shared across containers in the same pod
- As persistent data
- If you want data to stay with the pod even if the pod is restarted on a different machine you can use remote disks

## Chapter 6 - Labels and Annotations

#### Labels

- **Labels** are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets.
- Kubernetes uses labels to deal with sets of objects instead of single instances.
- Key/value pairs are strings.
- Labels play a critical role in linking various related Kubernetes objects. E.g. ReplicaSets find the objects they are managing via a selector.

#### Annotations

- **Annotations** are key/value pairs designed to hold non-identifying information that can be leveraged by tools and libraries.
- While labels are used to identify and group objects, annotations are used to provide extra information about where an object came from, how to use it, or policy around that object.
- When in doubt, add information to an object as an annotation and promote it to a label if you find yourself wanting to use it in a selector.
- Annotations are good for small bits of data that are highly associated with a specific resource.

## Chapter 7 - Service Discovery

### Why not DNS?

- Poor load-balancing
- There are limits to the amount of information that a DNS query can return
- Even with short TTL clients can have stale mappings in their cache

#### Service Object

- A service object is a way to create a named label selector.
- A Kubernetes **Service** is a logical abstraction for a deployed group of pods in a cluster. Pods are ephemeral - they come and go - where as a **Service** persists.

#### Cluster IP

- A **cluster IP** is a virtual IP address that is assigned to a __Service__. A cluster IP is a special IP address the system will load-balance across all of the Pods that are identified by the selector. It can be seen with: `kubectl get services`.
- The cluster IP is usually assigned by the API server as the Service is created.

  - *Note*: A virtual IP is often used by NAT to put into practice a one-to-many relationship. For example, a router with NAT will have a virtual IP.. traffic to this router will then be forwarded to many actual IPs. In this way the router can perform load balancing.

- The cluster IP is a stable IP and therefore can be given a DNS address without worrying about caching issues.
- Kubernetes provides a DNS service exposed to Pods running in the cluster. The Kubernetes DNS service provides DNS names for cluster IPs.
- The cluster IP you get when calling `kubectl get services` is the IP assigned to this service _within_ the cluster internally.

#### Readiness Checks

- Readiness check can be set on the deployment object.
- Only ready pods are sent traffic.

#### NodePorts

- A node port exposes the service on a static port on the node IP address.
- Serves as the external entry point for incoming requests.

#### Endpoints

- For every Service object Kubernetes creates a corresponding buddy **Endpoints** object that contains the IP addresses for that service.
- Advanced applications can get the IPs of pods directly from the Kubernetes API. `kubectl describe endpoints --namespace=edge-squad network-defense-manager`

#### `kube-proxy` and cluster IPs

- `kube-proxy` runs on every node in a cluster.
- `kube-proxy` watches for new services in the cluster via the API server. It then programs a set of `iptables` rules in the kernel of that host.

## Chapter 8 - HTTP Load Balancing with Ingress

- The `Service` object operates at `Layer 4` - it only forwards TCP and UDP connections and doesn't look inside of them.
- HTTP is layer 7.
- How to host many HTTP sites on a single IP?
  * Use a load-balancer/reverse proxy to accept incoming requests and based on the `Host` header and `path`, proxy the HTTP call to the upstream service.
  * In Kubernetes this HTTP-based load-balancing system is called **Ingress**.
- The **Ingress controller** is a software system exposed outside the cluster using a service of type: `LoadBalancer`. It then proxies requests to “upstream” servers. The configuration for how it does this is the result of reading and monitoring **Ingress objects**.
- **Ingress Objects** provide configuration for Layer 7 load balancers.

## Chapter 9 - ReplicaSets

- A **ReplicaSet** acts as a cluster-wide Pod **manager**, ensuring that the right types and number of Pods are running at all times.
- A ReplicaSet defines a specification for the Pods we want to create and a desired number of replicas.
- A reconciliation loop compares the desired state with the observed state - it is constantly running observing the current state of the system.
- It's possible to _quarantine_ a misbehaving pod. This can be done by modifying the set of labels on the sick Pod - thus disassociating it from the ReplicaSet.
- In general, ReplicaSets are designed for stateless services.
- ReplicaSets monitor cluster state using a set of Pod labels.
- **Namespaces** provides a mechanism for isolating groups of resources within a single cluster.

## Chapter 10 - Deployments

- The **deployment** object exists to manage the release of new versions.
- The mechanics of the software deployment is controlled by a deployment controller in the Kubernetes cluster.
- Deployments manage **ReplicaSets**.
- Similar to all relationships in Kubernetes, this relationship is defined by labels and a label selector.
- **Deployments** manage ReplicaSets - therefore, setting the number of replicas directly on the ReplicaSet will be overwritten by the number of replicas specified on the deployment.
- The deployment object contains two fields: `OldReplicaSet` and `NewReplicaSet`. These correspond to the ReplicaSets the deployment is currently managing. If a deployment is in progress they will both be populated, otherwise, `OldReplicaSet` will be set to `<none>`.
- The deployment object contains a **strategy** field which dictates how the rollout should proceed. There are two supported rolling strategies:
  - **Recreate** strategy updates the ReplicaSet it is managing to use the new image and terminates all pods. The ReplicaSet then notices that no pods are running are creates them using the new image. Only to be used in test.
  - **RollingUpdate** - worth noting that, with this stratgey, the application will be running new and old versions at the same time. Has two parameters for tuning the RollingUpdate:
    - `maxUnavailable` - can be an absolute number or a percentage. Indicates how much o an application will become unavailable during a rolling deploy.
    - `maxSurge` - In situations where you don't want capacity to fall below 100% you can set `maxUnavailable` to 0% and control the rollout using `maxSurge`. The `maxSurge` dictates how many extra replicas can be created to acheive a rollout.
- You can slow a deployment by using `minReadySeconds` parameter. This indicates to the deployment controller that it must wait this specified time period __after__ a pod becomes healthy before moving onto updating the next pod.
- In addition to waiting a period of time for a Pod to become healthy, you also want to
set a timeout that limits how long the system will wait. Otherwise, if the new version of your service becomes deadlocked, the deployment will stall indefinitely. Timeout is set with the parameter `progressDeadlineSeconds`. The timeout is given in terms of the deployment progress __not__ the overall deployment time. i.e. when a pod is created or deleted the timeout clock is reset.

## Chapter 11 - DaemonSets

- A **DaemonSet** ensures that a copy of a pod is running across a set of nodes in a cluster. This could be a log collector, a monitoring agent etc.
- When to use a DaemonSet vs ReplicaSet?
  - ReplicaSets should be used when your application is decoupled from the node it is run on. i.e. you can run multiple copies of the application on the same node and it doesn't matter
  - If you find that you want a single copy of application per node, then a DaemonSet is the correct choice.
- **NodeSelector** can be used to limit what nodes a pod can run on.

## Chapter 12 - Jobs

- **Jobs** create pods that run until successful termination. Unlike regular K8s pods they are not restarted.
- The Job Object is responsible for creating and managing Pods defined in a Pod template.
- The Job controller will create a new Pod if a pod fails before successful termination.
- Jobs support threes usescases:
  1. Run a pod once
  2. Run multiple pods in parallel to process a batch of work
  3. Run multiple pods to process from a work queue
- There are different restart policies for Jobs:
  - `OnFailure` - restart the pod if it fails
  - `Never` - don't restart the pod if it fails. This can result in a lot of failed pods if not careful.
- A Job that is completed when a certain number of Pods terminate successfully. The **Completions** specifies how many Pods should terminate successfully before the Job is completed.
- **Parallelism** specifies how many pods can run in parallel.

## Chapter 13 - ConfigMaps and Secrets

- **ConfigMaps** are akin to a small file system or a set of environment variables. They are essentially key value pairs. They are combined with the Pod before it's run.
- There are three main ways to use a **ConfigMap**:
  1. Filesystem - mount a ConfigMap as a volume
  2. Environment Variable - a ConfigMap can be used to dynamically set the value of an env var
  3. Command line argument
- Kubernetes secrets are stored on tmps volumes and not written to disk. By default Kubernetes secrets are stored in plaintext in `etcd`.
- Secret data can be exposed to Pods using the secrets volume type. Secret volumes are created at pod creation time.

## Chapter 14 - Role-Based Access Control

- Authentication provides the identity of the caller issuing the request. **Every** request in Kubernetes is authenticated. i.e. Every request has an identity.
- Authorization determines whether the user has the correct permissions to perform an action on a resource.
- Kubernetes makes a distinction between user identities and service account identities. Service accounts are created and managed by Kubernetes itself and are generally associated with components running inside the cluster.
- Authorization is determined using **Roles** and **Role Bindings**. Roles are a set of abstract capabilities. E.g. can create a Pod. Role Bindings assign users to roles.
- **Roles** are described in terms of a resource and a verb that describes an action that can be performed on that resource.
- You can test authorization with `can-i` command. E.g. `kubectl auth can-i create pods -n edge-squad`
