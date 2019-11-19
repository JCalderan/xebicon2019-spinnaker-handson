# Application configuration
In the [previous exercise](../exercise2/README.md) we learnt to parametrize our pipeline, 
and we added a new parameter "version".

This version v2 of xebicon-frontend use a Back-End application in order to fetch the Xebicon slots to display.  
In order to get it working, we'll need to deploy two new applications:
- xebicon-backend: the API consumed by xebicon-frontend
- xebicon-database: the database hosting the Xebicon slots data 

## Seting up the backend application
Create a new application "**xebicon-backend**", and set up a new pipeline in order to:
- deploy a new Service named "xebicon-backend"
- deploy a Kubernetes ReplicaSet (use the docker image ```jcalderan/xebicon-backend:v1```)

This application is a simple Node.js application which **listen on port 8080**, and exposes **two endpoints**:
- /slots: fetch all slots stored in database
- /health: return ```200 OK {"status": "up"}``` when the application is up and running 

## Setting up the database
Create a new application "**xebicon-database**", and set up a new pipeline in order to:
- deploy a new Service named "xebicon-database"
- deploy a Kubernetes replicaSet (use the docker image ```jcalderan/xebicon-database:v1```)

This application is a single node MongoDB database which **listen on port 27017**.

## Wiring up everything
We need to connect all three components.  

Both the Front-End and the Back-End applications have to be exposed to the Internet through a domain name, 
while the database must only be accessible from the inside of the Cluster.  
The following diagram illustrate this setup.
![Target deployment diagram](./xebia-stack.svg)

In order to access both applications from the Internet, we will use a domain name which resolve to our kubernetes cluster:
- Route53 is a DNS web service by AWS, which has already been provisioned in order to resolve the domain name ```*.my-cluster.com```
- Elastic Load Balancer, is an AWS Load Balancer which has been provisioned in order to forward traffic to the Kubernetes cluster nodes.  
- Ingress Controller is a Kubernetes components which implements Ingress rules in order to root incoming traffic to relevant Services.

Thus, route53 will resolve any sub-domain of ```*.my-cluster.com``` and forward the traffic to the ELB which in turn will forward the traffic to the kubernetes cluster node.
 
Services and Pods names can be resolved [using DNS names](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
inside a Kubernetes cluster, using a name of the form:
- my-service.my-namespace.svc.cluster-domain.example
- my-pod.my-namespace.svc.cluster-domain.example

This means the application **xebicon-backend** can connect to the **xebicon-database** Service using the DNS name ```xebicon-database.your-namespace.svc.cluster.local```.

### Exercise
- Bake a set of Ingress Rules ???
- Update xebia-frontend service (TODO: give more insights)
- Update xebia-backend service(TODO: give more insights)
- Update xebia-database configmap(TODO: give more insights)

