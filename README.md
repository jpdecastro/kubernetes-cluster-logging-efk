# Kubernetes Cluster Logging with EFK
This project has been created to demo cluster logging with EFK using Docker Desktop. Enabling Kubernetes support on 
Docker Desktop gives us an alternative to Minikube and Kind. The installation is quite simple, and you can have your
cluster running in seconds.

The Kubernetes cluster that runs within Docker Desktop is a single-node cluster and is primarily for testing purposes.
This is great as it allows you to experiment with the technology faster as there are no costs like when running a
cluster in the cloud. Furthermore, you don't have to install any other tools like virtualbox to get up-and-running.

Docker Desktop with Kubernetes support is available for:
* Windows 10 (Professional or Enterprise)
* Mac OS (Sierra 10.12 minimum)

Before starting, let's have a look at what we will be creating.
<paste image>

## Enable Kubernetes Support
The following page https://docs.docker.com/desktop/kubernetes/ contains information on how to enable Kubernetes on
Docker Desktop. Follow the instructions to get started. Keep in mind that if you have installed `kubectl` and are using it
for other projects it could be that it is pointing to another context. If that's the case then you must make sure it is
pointing to the `docker-desktop` context, otherwise you will have issues connecting to your local Kubernetes cluster.

## Simplify life with Kubectl Autocomplete
Working with kubectl to create resources is quite easy, just apply a YAML file, and your good to go. Actually `kubectl` has 
a lot more to offer. You will eventually need to know if your deployment is successful, read the running configuration of 
a resource, etc. Typing all those commands could become quite a chore, so thankfully we got autocompletion.

If you're running a ZSH terminal this is quite simple to set up, all you need to do is add the following to the beginning 
of `~/.zshrc`:

```shell
autoload -Uz compinit
compinit
source <(kubectl completion zsh)
```
Information on the ZSH configuration can be found [here](https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-zsh/).
The folks at Kubernetes.io have also provided us with a [Kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

## Verify Single-Node Kubernetes cluster is up
Once Kubernetes is enabled, we can start interacting with it using `kubectl`.

Let's check how many nodes our cluster has. The result should contain 1 (single-node cluster): 

`kubectl get nodes`

To get information about our cluster:

`kubectl describe node docker-desktop` or `kubectl cluster-info`

## Creating the cluster namespace
A Kubernetes cluster comes with a set of predefines namespaces (i.e. kube-system, kube-public...) containing resources 
to function properly. Namespaces are a way to divide cluster resources between multiple users. It is not required to 
create namespaces when working on a small project, but it helps to create some structure.

To view the current active namespaces: 

`kubectl get namespaces`

Now that we have seen the list, let us start by creating our first own namespace:

`kubectl apply -f 1-efk-logging-ns.yaml`

Let us confirm the namespace is created:

`kubectl get namespaces`

If everything looks as expected we can continue with the creation of our Elasticsearch cluster.

## Creating the Elasticsearch cluster
We will be creating a 3-node Elasticsearch cluster in this section. Why a 3-node cluster you may ask? It's a best practice, 
and prevents the "split-brain" situation in a highly available multi-node cluster. In short, "split-brain" arises when one or more
nodes can't communicate with each other (i.e. node failure, network disconnect, etc) and a new master can't be elected. 
With a 3-node cluster if one of the nodes get disconnected then by election a new master can be assigned from the
remaining nodes.

For more information read [Designing for resilience](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability-cluster-design.html).

### Headless Service
Before we create our 3-node Elasticsearch cluster, we will need to create a Kubernetes headless service. This headless
service will not act as a loadbalancer and will not return a single IP address. Instead, it will return all the IP's 
of the associated Pod's. Headless Services do not have a ClusterIP allocated, will not be proxied by kube-proxy but in this
case it will allow Elasticsearch to handle service discovery. We achieve this by setting the `ClusterIP` configuration to `None`.

Create the headless service:

`kubectl apply -f 2-elasticsearch-svc.yaml`

Read [Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) for more
information.

### Elasticsearch StatefulSet
Rolling out the 3-node Elasticsearch cluster in Kubernetes requires a different type of resource also known as the
StatefulSet. It provides pods with a stable identity and grants them stable persistent storage. Elasticsearch requires 
stable persistent storage to persist data between Pod rescheduling and restarts. To learn more about 
[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) visit the kubernetes.io website.



## Creating the Service & Kibana Deployment

## Creating the FluentD DeamonSet

## Launch Counter Pod