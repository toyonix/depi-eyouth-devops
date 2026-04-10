# Kubernetes

## Theory Questions

### a) Kubernetes Architecture

#### What are the core components of a Kubernetes cluster (e.g., master, node, etcd, kube-apiserver)? Briefly explain their roles.

So, Let's start with a core components:

- kube-api server

It's a rest api interface for kubernetes control plane and data store.

- etcd

It's where the configuration sets for the whole cluster. It acts as the data store that provides key value store for the whole cluster.

- kube-controller-manager

It is the primary daemon that manages all core components in kubernetes. It monitors the cluster state using the api server and changes the configuration of the cluster as it goes to reach the desired state.

- kube-scheduler

It's the module that monitors the workload for the whole cluster and also attempts to place the new services in nodes in nodes where they have matching resources requirement.

- node

It's a machine, either physical or virtual that kubernetes use to run services aka pods on them. each node contains components like kubelet kube-proxy, and container runtime engine.

- kubelet

It's the node agent that is like the leader that moves the whole node with its services on the host.

- kube-proxy

It's like any proxy server like nginx but packed into kubernetes. It manages the network rules on each node and has proxy features like load balancing and connection forwarding.

- container runtime engine

So, It is what kubernetes relay on to run the services and components. They support multiple CRIs like docker, podman, and Virtlet.

- master and workers

So, The master is a concept not just in kubernetes but in any application that requires multiple machines working together. A master is the machine that controls the configuration and the state of other machines called workers. The master orders something from a worker and the worker has to follow that order. It's for helping managing multiple machines from a single machine and also it helps with the role of everything which makes it easy for every machine to know what it is supposed to do.

- cloud-controller-manager

From its name it helps to connect kubernetes to a cloud provider like AWS and azure. It helps to configure and control the cloud infra.

- cluster DNS

It provides a simple DNS server for the cluster in case needed.

- kube dashboard

It's a simple web interface to view and monitor the cluster.

- Metrics API

It provides metrics like usage and utilization for kubernetes components.

#### What is a pod in Kubernetes, and how does it differ from a Docker container?

So, A pod is a combination of  containers under a single network namespace and storage resources. The main problem that containers were invented for is to isolate services with their own dependencies and they act like VMs. But, it introduces another problem, each service now has it's own localhost or network namespace and if you wanted to group multiple services under the same namespace and storage resources and they for example use different python versions you can't because each one has to live in it's own container and they cannot be with each other. Pods came with the solution to group containers under the same namespace so you could have multiple container with different dependencies but they have the same network namespace and storage if needed. The best of both worlds.

### b) Deployment and Services

#### Explain the purpose of a Kubernetes deployment. How do deployments ensure high availability of applications?

Before anything let's explain the concept of high availability, it's where your application is always available with no down times. To achieve that your application has to be immune to update down time and also be able to withstand high traffic without going down. All of this is considered to high availability.

Kubernetes Deployment is a declarative way of managing pods via something called replica sets. Replica sets from its name it helps to replicate the pods and manages the replicas life cycle. The deployment use for that is if you want to update your application without it going down in your cluster you can replicate it and update some replicas and check if they are good upon testing and then move the whole cluster to it so you update gradually and also if something goes wrong you can go back. Which is called Rolling Update Deployment. 

Also mentioned in the session, The Green Blue deployment model. Which in this case you have two cluster each one has it's own use case and they are interchanging with each other. So, you have a green cluster that is up for production and you have an update you want to push. You start a new cluster and in this case it's the blue cluster and then you deploy on it your new version and you start switching the traffic from your green cluster to your blue one then you switch roles so the green becomes blue and the blue becomes the new green. and then You stop the blue cluster for later use.

All of that helps to make your app highly available and immune to downtime.

#### What are the different types of services in Kubernetes (e.g., ClusterIP, NodePort, LoadBalancer)? When would you use each type?

- cluster IP

It's a virtual ip for the cluster that you define and helps you access the service using that ip without thinking about the internal nodes ip and kubernetes load balance the requests on that ip to the internal nodes.

- Node Post

It's basically the service port that you expose on each node and combined with the cluster ip it gives you a way to access the service.

- Load Balancer

It balances the load (requests) across the cluster nodes so all the requests are not just on one machine it's balances across multiple machines.

### c) Scaling and Auto scaling

#### How does Kubernetes handle scaling? Explain the concept of Horizontal Pod Autoscaler and how it responds to workload changes.

First, What is scaling? The concept of scaling is you add more nodes to your cluster in order to process more requests depending on the demand of your application. For example, you have a cluster with 5 nodes. and the traffic is now hitting these nodes hard. so you add more in the cluster to withstand the high traffic.

Auto scaling is basically scaling but done automatically without user intervention by setting rules and conditions where kubernetes spin up new clusters in the node and also stops then automatically depending on the load. For example if the usage of the current clusters is over 50% kubernetes spins up one more cluster and so on and of they are less than 20% they start stopping nodes and so on.

## Practical Questions

### a) Create Deployment

Requirements:
1. 3 replicas of the web server container from the last assignment.
2. Load balancing is enables across the whole cluster.
3. Describe how you would test load balancing functionality.

Let's start

So, from the last assignment, we did create a nginx service with a custom docker file.

![](/tasks/4/Pasted%20image%2020260311101545.png)

I build the image and called it `nginx-kubernetes` to make it easy for testing later.

Just to test

![](/tasks/4/Pasted%20image%2020260311101825.png)

![](/tasks/4/Pasted%20image%2020260311101843.png)

So, it is working as expected. Let's create the deployment for it.

Since I am using minikube on a VM I copied the local file to the vm using `scp` command

![](/tasks/4/Pasted%20image%2020260311103246.png)

And I will build the image there and start working.

For the replica set, We have two ways of doing the requirements:

1. Create a deployment with the number of the replicas specified.
2. Create a replica set with the deployment as its template.

What I will go with? I will go with option number 1. Looking online, Using replica set for a rolling update is not a good practice since you would need a human intervention to start up another replica and slowly scale down the old replica along side scaling up the other replica. The thing is, Deployment does all of that for you without any human intervention which is a lot better and automated.

This is discussed here https://stackoverflow.com/questions/69448131/kubernetes-whats-the-difference-between-deployment-and-replica-set

For the load balancer, I would just create a simple service for it and expose the node port.

I HAD PROBLEMS!!!

So, This was so funny. I had a lot of test deployments in this vm and I tried to delete them until I found out how to delete them and I started working. The thing is no matter what I did I was not able to get the replicas past pending state. and I found out if you ran the `drain` command (which I did) you are basically saying for the node do not schedule anything until I `uncorden` you. I spend like an hour trouble shooting sooooooo. Let's continue. 

Okay, here we go agaaaain

![](/tasks/4/Pasted%20image%2020260311112910.png)

This is amazing. I was specifying  the docker image to be the name I created for it. But minikube docker doesn't see it! and I have to run this command to get it to run `eval $(minikube docker-env)`.

![](/tasks/4/Pasted%20image%2020260311114003.png)

FINAllY!

![](/tasks/4/Pasted%20image%2020260311114420.png)

So, Here is the deployment file I created:

![](/tasks/4/Pasted%20image%2020260311114447.png)

Okay, What did I do?

I created a simple deployment for the service using 3 replicas. and then created a service of type Load Balancer in order to get load balancing to work with the target port 8080 and node port 30080. and it worked!

Finally, How would I test load balancing?

I would create a load test and observe the cpu usage for all the replicas and theoretically all of them should have the same cpu usage across different loads. Let's do that

From this article https://www.virtuozzo.com/application-management-docs/testing-load-balancing/ I can simulate requests for a url using `apache2-utils` on linux. 

To observe the cpu usage of the service, It was weird, It was not working and then following this fixed it https://medium.com/@cloudspinx/fix-error-metrics-api-not-available-in-kubernetes-aa10766e1c2f 

Then because for some reason the top command doesn't have any way of watching a complete deployment I created a simple bash script for it to observe changes:

I prompted chatgpt to do it tbh 

![](/tasks/4/Pasted%20image%2020260311120755.png)

But, by running this command `ab -n 100000 -c 1000 -g out.txt http://192.168.49.2:30080/` I was able to observe the CPU usage creeping up and it was balanced across all replicas.

Idle:
![](/tasks/4/Pasted%20image%2020260311121753.png)

Under load:
![](/tasks/4/Pasted%20image%2020260311121622.png)

I think that is it for this task.

### b) Service Exposure

Requirements:
1. Expose the service using Node Port. Map port 80 to the cluster.
2. Make sure the service is accessible from the browser.

I already technically did that since the load balancer service is an extension of the node port service but I will do it anyway

![](/tasks/4/Pasted%20image%2020260311125331.png)

The output:

![](/tasks/4/Pasted%20image%2020260311125305.png)

It is a VM, curl works like a browser in the terminal.

### c) Scaling with Auto scaling

Requirements:
1. Set up Kubernetes High Pod Autoscaler up to 10 replicas when CPU utilization exceeds 70%
2. Simulate high CPU usage using kubectl and observe how kubernetes scales the pods.

I would follow this guide from the official docx https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

Ok, so I create the service and run it normally then I create another file to auto scale it. I was trying to modify the same file and it didn't work

so we can just run this command to create the thing:

```bash
kubectl autoscale deployment nginx-deployment --cpu=70 --min=1 --max=10
```

So, For some reason it doesn't want to run because it complains about the api version. So, Let's create a yaml file for it from the docs and use it insead

![](/tasks/4/Pasted%20image%2020260311135814.png)

Ok, So it worked. I will lower the percentage to something like 10% and load test it and see what will happen.

For some reason. No matter what I try I couldn't get the scaler to work. The code from the docs works since modifying the minReplicas spin up or down pods but adding more on its own is not working for me no matter what I did!

![](/tasks/4/Pasted%20image%2020260311142338.png)

I even added limits but still nothing.
