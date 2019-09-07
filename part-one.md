# Kubernetes for Beginners (Part One)

## Introduction

Kubernetes is a container orchestrator designed for applications that runs on containers and need the following things:

- Service Discovery and Load Balancing
- Automatic Bin Packing
- Storage Orchestration
- Self Healing
- Secret and Configuration Management
- Automated Rollouts and Rollbacks
- Batch Execution
- Horizontal Scaling

Kubernetes is used in production environments running applications that should be available at all times, with a lot of variables to monitor. Kubernetes also known as K8s is a big tool with a lot of fancy knobs and buttons, and this article will try to explain and demystify some basics of what Kubernetes is.

### Why you should you it

In the general process of deploying applications, we encounter various issues with the way we want to get it working. One problem with deploying applications in production before Kubernetes and other container orchestrators was the problem of managing resources. If we needed an application, we spun up a thing called a VM (Virtual Machine). A VM is a full-blown machine, we can otherwise refer to it as a Server, you can spin up a server using [MaxiHost Bare Metal Cloud](https://www.maxihost.com/bare-metal) quickly and easily. This machine in principle should only hold one application, including all its dependencies, which would have to be manually deployed via some CI process every time we wish to use it. This process was very stressful and required a lot of technical skills to manage various things such as networking, security, storage and all that which would all go to waste if the VM became unresponsive and stopped working due to varying factors.

### Why Containers

Containers provide a good level of isolation between the application and the machine running it. If the container running the application goes down, it doesn't affect the machine. Since this is the case, it means we can spin up as many containers on a single machine without any hopes of conflict.

Kubernetes provides this container management feature along with the ability to run multiple servers in a cluster setup. Kubernetes can control these multiple machines by running a thing called a Control Plane which defines how the master and slaves communicate with each other.

```
The various parts of the Kubernetes Control Plane, such as the Kubernetes Master and kubelet processes, govern how Kubernetes communicates with your cluster.

The Control Plane maintains a record of all of the Kubernetes Objects in the system and runs continuous control loops to manage those objects’ states.

At any given time, the Control Plane’s control loops will respond to changes in the cluster and work to make the actual state of all the objects in the system match the desired state that you provided.
```
> Source: https://kubernetes.io/docs/concepts/#kubernetes-objects

## Kubernetes Architecture

There are two cluster types in Kubernetes since it is generally for handling distributed workloads:

-  Master
-  Slaves

> **NOTE** </br>
> A Node is a VM in Kubernetes and a Cluster is several nodes working together.

### Master

This is what we use to talk to the slaves. In Kubernetes, the master is the one who we communicate to about what deployments should be on our cluster and how they should be managed. Just as the folklore concept, the master delegates work to the slaves who are then responsible for starting the containers and making sure they function as desired.

The master does this using a collection of three processes:
- API server (kube-apiserver)
- Controller manager (kube-controller-manager)
- Scheduler (kube-scheduler)

The **API server** is what we use to perform duties from a client. The control plane which was earlier explained is accessible to the outside world via this API server. From here, we can perform various operations inside and out of the cluster using a client tool called Kubernetes Control (KubeCtl) .

The **Controller manager** is what does all the checking and validation of the task which we sent to the cluster using the API server. So things like making sure our deployment doesn't fail and that the general health of the cluster and things running on it are in great shape. It does this by checking different resource objects on Kubernetes in a continuous loop to ensure they function as expected.

The **Scheduler** is what enforces rules on the cluster. In Kubernetes, because we run many applications in different containers, we do not want just one to take over all the resources like RAM, CPU, and Storage and undermine others. The Scheduler is, therefore, the one that schedules these checks and ensures that any application joining on any slave can fit there, and if not, schedules it to another one so that the workload on any slave during given instances is fairly shared amongst all of them.

### Slaves

These are the nodes that do the grunt work of running services and all the applications needs. The Scheduler as earlier explained is the one that pushes the requests to them and they are then monitored by the controller manager and interacted with by us through the API Server over the master. In general, the slaves run two things:

- Kubelet
- Kube-Proxy

**Kubelet** is responsible for talking with the master node. Since the slaves are VMs on their own, Kubelet is what the master node uses to communicate and perform various actions such as scheduling and monitoring on such nodes.

**Kube-proxy** exposes the applications on the node. Just as Kubelet communicates with the master, kube-proxy communicates with the applications. It is Kube-proxy which exposes the ports from the containers running to the outside world using things called Services.

## Containers in Kubernetes

If you don't already know how containers work, you can read about it in our [Docker for Beginners]() tutorial which explains step by step on how to configure and build a container using Docker.

![vm vs containers](https://res.cloudinary.com/ichtrojan/image/upload/v1567474875/vm_vs_con_sbj57m.png)

We use containers because as earlier explained, they allow us to run multiple applications on a single host machine with little to no worry about them causing our machine to fail abruptly as we can see in the image above. This is because containers run in sandboxes, hence making them more secure because only the host can see them without one container being able to see or interfere with another container. This is not possible if we were running all our applications on a single server.

## Deployments and Containers

In Kubernetes, because we have different things like Networking, Storage, R.A.M, and C.P.U
to manage amongst various applications that might or might not be similar, we interact with these various resources using things called Kubernetes Objects.

```
Kubernetes contains several abstractions that represent the state of your system: deployed containerized applications and workloads, their associated network and disk resources, and other information about what your cluster is doing.

These abstractions are represented by objects in the Kubernetes API
```
> Source: https://kubernetes.io/docs/concepts/#kubernetes-objects

The basic Kubernetes objects include:

* **Pod** - Running Application
* **Service** - Networking and Port Allocation
* **Volume** - Storage and Disk Management
* **Namespace** - Sandbox For Running Multiple Versions

Also, Kubernetes contains several higher-level abstractions called Controllers. Controllers build upon the basic objects and provide additional functionality and convenience features they include:

* **ReplicaSet** - Similar Pods Running Together
* **Deployment** -  A higher group built on a collection on Pods which may or may not be similar
* **StatefulSet** - An application which has Volume requirements and should not start unless it is offered a separate and persistent space to store files
* **DaemonSet** - Just like a ReplicaSet but this one should be present on all nodes running in the cluster
* **Job** - A short running Pod, good for things like webhooks and email dispatchers


These and much more are the different types of Kubernetes objects. Filtering through all the big talk, we will only focus on Deployments for a start. As earlier stated, a deployment is a collection of pods that may or may not be similar depending on the task it is meant to accomplish. It is a very popular object in Kubernetes and we will be deploying one in the coming session. A deployment is a collection of one or more pods. A pod is a running container which has been allocated resources. A pod on its own is never a good idea as it has limited functionality and cannot restart itself once it dies or an error occurs.

Deployments and various higher-order groups built on Pods allows us to run a certain number and even keep a minimum number going so that our application never dies, even in the influence of a failed pod.

## Exposing a Deployment With Services

![Kubernetes Service Pods](https://res.cloudinary.com/ichtrojan/image/upload/v1567468522/bak2_ndewpp.png)

Just as a deployment runs one or more pods, we have to think about how to access it. Many times when we deploy applications, servers, and the likes, they are accessible from a browser or client through a Port. The Port allows us to access multiple services on the same machine without worrying about conflict; a service allows us to access a deployment spread across different machines without worrying about where it is running. This is made possible by the `kube-proxy` which can forward our request to the machine where it is deployed and send the response back through that same channel.

## Conclusion

In this article, you have learned the basic concepts of Kubernetes and the architecture by which it works. All these may not make sense now until we put it to good practice; in the next segment of this tutorial series, we will be deploying a simple web application on Kubernetes, with the little demo you will be able to fully understand what all the terms used in this article means.
