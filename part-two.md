## Building A Simple Hello World App

In building a simple hello world app on Kubernetes, we require a couple things setup before we can start.

- Docker
- Minikube

Docker is what will run our containers and is referred to as a Container Runtime. For those willing to know more about Container Runtimes, Cgroups and the likes, visit the video [here](youtu.be/sK5i-N34im8).

Minikube is a mini development kubernetes cluster which can be used for testing the features on k8s and performing various functions which we'd like to test such as deploying an application, configuring an ingress controller and all that.

The installation for Docker has already been covered so let's look into installing kubernetes for the major OSes such as Linux, Windows and MacOS.

### Installing K8S
To get started with k8s, we need to be sure we have two applications installed
 - Kubectl (Kubernetes Command Tool)
 - Minikube (Cluster)


#### [Installing Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)
Kubectl is the tool which would be used for interacting with the kubernetes control plane hosted by minikube.
The installation differs depending on the operating system but the installation guide for all major OSes are listed below for better comprehension.

On Linux, we can run the following commands to install the latest kubectl on a neutral binary use basis
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version
```

On MacOS, we can install kubectl using the same approach. I defer from using brew as the binary approach is the least dependency driven approach so there would be few errors for those running. It follows the same approach as that of Linux.
```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version
```

For windows users, kindly download the binary from the link [here](https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/windows/amd64/kubectl.exe)
Move the downloaded exe file to a location and add it to your PATH.

Alternatively, download Docker Desktop on Windows which comes with Kubectl [here](https://docs.docker.com/docker-for-windows/install/)

## Installing MiniKube

To run Minikube, you need an hypervisor to run the cluster VM. An hypervisor is a tool used to control virtual machines.
A common hypervisor is VirtualBox and that is the one which we'd be using in this tutorial.

You can download the software [here](https://www.virtualbox.org/wiki/Downloads).

Once VirtualBox has been installed, you can continue with the minikube installation.

For linux, you can install the minikube application using the current commands:
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube

sudo install minikube /usr/local/bin
```

For MacOS users, you can use the following commands to install minikube.

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \
  && chmod +x minikube

sudo mv minikube /usr/local/bin
```


Windows users should kindly visit the link [here](https://kubernetes.io/docs/tasks/tools/install-minikube/#tab-with-md-2) to familiarize with the applications and installer links for minikube.


## Starting Minikube

To start using minikube, you can run the command `minikube` to get started with the options.

After checking that, you can start the kubernetes cluster on minikube using the command

```bash
minikube start
```

This would start the kubernetes cluster alongside an API on the default 8080 port binded to localhost.


## Deploying an application to Kubernetes

To start off deploying the application, we need to make a deployment. A deployment as previously explained is a kubernetes object for a cluster of pods.

In Kubernetes, we use containers to spin up applications and as such we'd need a container to run in our deployment.

For this example, we'd be using the image from the docker beginner guide which has been uploaded to this  repo: [`ichtrojan/php-hello-world`](https://hub.docker.com/r/ichtrojan/php-hello-world)

When deploying applications, we deploy them to a path called a namespace. A namespace is an abstraction of a sandbox which is used to break down the same application in different zones, we can do this for testing purposes such as A/B testing, where we might use different configurations but need both applications running to infer what the difference in runtime there might be.

If a namespace is not specified, Kubernetes by default uses the `default` namespace.

Here's a sample deployment yaml template using the `ichtrojan/php-hello-world` image.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
#  namespace: random-namespace # it can be anything random
spec:
  selector:
    matchLabels:
      run: hello-world
  replicas: 1 # we want 1 instances of this running
  strategy:
    rollingUpdate: # this ensures that if we add a new update, we only have one application down
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  template:
    metadata:
      labels:
        run: hello-world
    spec:
      containers:
      - name: php-hello-world
        image: ichtrojan/php-hello-world
        imagePullPolicy: IfNotPresent
        readinessProbe:
          httpGet: # Calls an endpoint and check if returns a status code < 400
            path: /
            port: liveness-port
          initialDelaySeconds: 5
          timeoutSeconds: 1
          periodSeconds: 15
        livenessProbe: # Checks if the port is open on the container
          tcpSocket:
            port: liveness-port
          initialDelaySeconds: 5
          timeoutSeconds: 1
          periodSeconds: 15
        resources: # We can restrict how much resources we'd like to give to containers also
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - name: liveness-port
          containerPort: 80
```

You can apply this by running the command:

```bash
kubectl apply -f php-hello-world-deployment.yaml
```

You can likewise check the status using the following commands:

![List of deployments](deployment.png)

And we can view further the pods which constitute the deployment

![List of pods](pods.png)

## Exposing our Deployment using Services

When we create a deployment, we start our application using containers that run within our cluster and will do so using the configuration which we have specified. To access the deployment, we need to create a service.

A service is an entrypoint to our application which defines which port should be open and how it can be accessed. What this means is that a service is binded to a port, which is very true. So for each port which is open in our application, we have an accompanying service to open it to the world in the Kubernetes cluster.

If you rememeber that kubernetes is a cluster, that means that we can access our application from any node using the service name. This defeats the purpose of knowing which IP and port the application is deployed on.

If we have an application that we don't need exposed, we  don't create a service for the application and it's securely hosted on the cluster.

When it comes down to services, we can have the following types:

NodePort - This is the default host mapping, we basically host the port of our application on some arbitrary port in the range of 30000-32767. All pods in our cluster have a node port which all requests to the service get forwarded to.

ClusterIP - When we have more than one replica which we'd like to forward traffic to, we use a clusterIP. This assigns the entire deployment a clusterIP which internally load balances all the requests to the pods which were previously created by the deployment, this service type has no NodePort. This ClusterIP can only be accessed internally and is not open to the world

LoadBalancer - This is similar to the ClusterIP service type but exposes the traffic outside the cluster. For a loadbalancer to work, we need a load balancer controller installed on the network. There are some software controllers such as [MetalLB](https://metallb.universe.tf) which allow us use load balancers on-prem without purchasing load balancing equipment like F5-Big IP. In general, such controllers are provided by cloud providers so most load balancer setups would happen on a cloud service such as GKE or AKS.

Enough of the talk, let's write a service for our hello-world application.


```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
# namespace: random-namespace # you can likewise add a namespace to a service too
spec:
  selector:
    run: hello-world
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    name: http
  type: LoadBalancer # We use a load balancer cause we want to access it outside the cluster
```

Once you've written the service, you can apply it using:

```bash
kubectl apply -f php-hello-world-service.yaml
```

![Hello World Service](service.png)

To test the service as we have no load balancer controller, we can use the minikube application to send requests to our service

```bash
minikube service hello-world-service
```

If you want to try it some other way, you can always curl the ClusterIP too.
```bash
curl $(kubectl get svc hello-world-service -o=jsonpath='{.spec.clusterIP}')
```

This would return the response from the container which has the format
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Hello World</title>
  </head>
  <body>
    Hello World. The time is "<time>"  </body>
</html>
```

 where \<time> is the current time.

If that worked, then you've successfully created a deployment and matched it to a service.

For those on linux wanting to experiment a lot more than the previous guide offers, another one was made with linux in particular. All the steps for setting it up on a linux environment are outlined here also:

[https://github.com/Tiemma/devfest-kubernetes-demo](https://github.com/Tiemma/devfest-kubernetes-demo)

## Conclusion

You've successfully learnt about the internals of kubernetes and hopefully tested and deployed an application both on the cloud and on your computer with minikube.

For those willing to cover a lot more content than just the basics: Check out the slides [here](https://docs.google.com/presentation/d/1dz5KB_ojadj4paf2H6TJvcj6QmcU6lZQMytdGkwUo9M/edit?usp=sharing) and don't be afraid to leave a comment or two.
