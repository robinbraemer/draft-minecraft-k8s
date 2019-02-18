# Proposal: *Running Minecraft Networks in Kubernetes!*


# Goal
The goal of this idea collection is to give a solution on how to
run a Minecraft Network in Kubernetes.
Administrators should be able to easily scale and manage
big Clusters regardless of their size and still be flexible enough
to implement own business logic.

Since Kubernetes is awesome and allows you to deploy any
kind of Apps, you could just go ahead and make your own Minecraft Cluster
architectures. So this is one **opinionated solution!** ;D

## Table of Contents
* [Goal](#goal)
* [Concepts](#concepts)
    * [Cluster](#cluster)
    * [ProxyNet](#proxynet)
    * [Proxy](#proxy)
    * [Server](#server)


# Concepts
All components are coupled using labels and are together making up a Cluster.

## Cluster
A Cluster combines the Server and Proxy components and is
considered the highest-level-resource.

Give all resources ProxyNet/Proxy/Server these labels:
- label `minekube-resource` exists
- and label `minekube-clusterName` set to the desired Cluster name

Optional Fields:
- description: The description of the Cluster.

*Note: A Cluster is a concept and not an actual Custom Resource, it only
exists in form of the `minekube-clusterName` label on each Pod.*

## ProxyNet
A ProxyNet is a network of interconnected Proxies
to balance hundreds and thousands of players concurrently.
One ProxyNet (so the Proxies in it) is meant to be deployed at
one geographical area where your target players are for minimal network latency.
With that being said, there are also Servers in a ProxyNet
also to provide little network latency.
  
Give all Proxy Pods these labels:
- label `minekube-resource=proxy`
- and label `minekube-proxyNetName` set to the desired ProxyNet name
  
Optional Fields:
- description: The description of the ProxyNet.

*Note: A ProxyNet is a concept and not an actual Custom Resource, it only
exists in form of the `minekube-proxyNetName` label on each Proxy Pod.*


## Proxy
A Proxy presents a Minecraft-Proxy.
Proxies join one ProxyNet.

**How to create Proxies and let them join a ProxyNet?**

You decide how Proxy Pods are being created.

Reasonably use
[Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
for stateless and
[StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
for stateful Proxies. Make awesome
benefit of Kubernetes features like
CPU usage based [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
or [Rolling Updates](https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/)!

Finally as described in the [ProxyNet](#proxynet) section to let Proxies join a ProxyNet
give the Pod the label `minekube-resource=proxy` and set `minekube-proxyNetName`
to the desired ProxyNet name.

*Note: A Proxy is a concept and not an actual Custom Resource, it only
exists in form of the `minekube-resource=proxy` label on each Proxy Pod.*

## Server
A Server resource presents one Minecraft Server.

**How to create Servers and let them join a ProxyNet?**
You decide how Server Pods are being created.

For instance you could use a
[Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/),
[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) (replica=1) or
[StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) (replica=1)
for each new Server.

*Note: A Server is a concept and not an actual Custom Resource, it only
exists in form of the `minekube-resource=server` label on each Server Pod.*

##Simplicity is Complicated End
I thought about plenty solutions and they got pretty complex and hard to implement as I
was writing different concepts.
I think this is the best resolution between simplicity, flexibility and extendability.

P.S. Rob Pike is right with saying [Simplicity is Complicated](https://www.youtube.com/watch?v=rFejpH_tAHM) :D