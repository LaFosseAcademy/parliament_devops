# Introduction to Kubernetes

An introduction to Kubernetes, the terminology and how to utilise the technology

## Organisation

### Duration

2 hours (includes a 10 min break)

### Set-up

#### For Trainers
**For the slidee presentation**

**Open** the [Slidee](https://github.com/mklilley/slidee) presentation:

- Run `npx slidee` from within the `curriculum` folder to see the available slide decks (those with extension `.slides.md`)
- Click on `See your presentations`
- Open `Weeks 6 7 > CDO > Kubernetes`
- Hit `s` to open the presenter notes
- Resource to be kept open on a tab throughout lecture to refer to

**Check out** [FAQs](/FAQs/README.md) for help with using Slidee

#### For Students
- **Direct** students to the starter code for this modile
  - [Student Facing Repo](https://github.com/LaFosseAcademy/cloud-devops-student-repos/tree/main)
- **Use** 
  - **kubernetes/intro-to-kubernetes/starter-code**
- **Make sure** 
  - You clone the entire repo

## Learning objectives

- **Understand** how Kubernetes builds on the work of Docker
- **Understand** that Kubernetes makes deploying applications, quickly, easily and effectively
- **Apply** multiple `kubectl` commands to influence the Kubernete Cluster
- **Create** configuration files based off the current state of the K8S Cluster

## Sequence

### Getting started with Docker, Kubernetes and Azure Kubernetes Service

So let's give an overview of Docker and Kubernetes. 

*REFER TO RESOURCE 1 - SLIDEE* <br>
![intro-to-kubernetes-1](./resources/intro-to-kubernetes-1.png)

To remind ourselves of the usefulness of Docker, let's use a scenario of a tech team:

We're a Dev and we want to quickly deploy an application, we can go to someone in the operations team. 

Because of Docker they can easily ask us what our Docker image is, which we provide. They'll be able to run a command similar to:

*REFER TO RESOURCE 2 - SLIDEE* <br>
```
docker run -p 8080:8080 lfacademy/hello-world-rest-api:0.0.1.RELEASE
```

After one command, we've built a container and it's deployed on whichever machine the command is run on. 

Our operations team didn't need to know:
* the language the application was built in
* any frameworks we were using
* the OS the application needed to be deployed on 
* any other config

*REFER TO RESOURCE 1 - SLIDEE* <br>

Docker enables the developer to build an image and the operations team can deploy it wherever a container run time is installed, whether that's on a local machine or in the cloud. 

During the **taster-day** we spoke about DevOps breaking down a figurative **Wall of confusion** and this is a really good example of it. 

Docker also allows us to standardise how we package and deploy applications irrespective of languages, frameworks or platforms we wish to deploy. 

At this stage we're happy with the deployment but maybe there's things we need to change later down the road. 

* We could want to ensure the application to always be running
* We could expect more load on the application at a certain points in the year
  * Therefore want to increase the number of instances or computing power
* There's other applications and new versions of those applications in the pipeline which will need deploying too

This isn't what Docker is built for. The solution we'll look at is **Kubernetes** which is a **container orchastration** tool. 

*REFER TO RESOURCE 3 - SLIDEE* <br>
![intro-to-kubernetes-2](./resources/intro-to-kubernetes-2.png)

We can create a **Kubernetes cluster** (which is a group of servers that are managed together) to handle some of these requirements. 

If we were connected to a **Kubernetes cluster** we could:

*REFER TO RESOURCE 4 - SLIDEE* <br>
#### Quickly deploy an application with a specific image
* `kubectl create deployment hello-world-rest-api --image=lfacademy/hello-world-rest-api:0.0.1.RELEASE`
* `kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080`

With Docker this can be done in one command, with Kubernetes it's taken two but there's more power we can add.

*REFER TO RESOURCE 5 - SLIDEE* <br>
#### Add further instances 
* `kubectl scale deployment hello-world-rest-api --replicas=3`

Now we have 2 more instances created. 

With a deployment like this on a **kubernetes cluster**, the load balancing is being handled automatically for us between the **3 instances**.

*REFER TO RESOURCE 6 - SLIDEE* <br>
#### Keep instances running 
We may want 3 instances to be running continuously, even if one were to fail, another is quickly loaded up. 

* `kubectl delete pod hello-world-rest-api-<instace-id>` <br>

This would delete one of the instances, the application will still work and in a few seconds a new instance will be loaded up, so behind the scenes we'll always have 3 different instances of our application running. 

*REFER TO RESOURCE 7 - SLIDEE* <br>
#### Increase number of instances when higher load anticipated
We can also expect certain applications to have higher loads during the weekends or during specific holidays etc.

Imagine the traffic:
* Amazon receives during the Christmas holiday/Black Friday sales/Valentines Day etc.
* Glastonbury receives when their ticket sales are released
* `kubectl autoscale deployment hello-world-rest-api --max=10 --cpu-percent=70`
  * Now we can go up to a max of 10 instances when the previous instance reaches 70% CPU capacity

*REFER TO RESOURCE 8 - SLIDEE* <br>
#### Deploy a new release
We need to specify the image for new release's version: **0.0.2.RELEASE**
* `kubectl set image deployment hello-world-rest-api hello-world-rest-api=lfacademy/hello-world-rest-api:0.0.2.RELEASE`



So with these commands we can see Kubernetes can help us:
- Auto-scale based on load
- Auto load balanced between available instances
- Monitoring and if instance goes down, it'll load another
- Release new version of application with no downtime 

Kubernetes, especially in the cloud is a great tool to impliment DevOps. We get all of these features pretty much out the box.

Imagine old operation teams having to monitor every service to load it back up if it goes down or scaling the capacity of a service up or down constantly based on CPU usage. 

We can scale thousands of microservices with thousands of instances in a declarative way. i.e. we can make statements like **"we need 100 instances of this service running"** and Kubernetes will action it for us. 

This is where we'll start to bring in our Cloud Providers too. <br>
Kubernetes can be deployed on:
- Azure
- AWS
- Google Cloud Platform 
  - and we'll see examples on each of them. 

Setting up Kubernetes and managing Kubernetes clusters outside of the cloud is possible, we can do it locally or in a data centre but we'll be using the cloud.

What's often used are the different Kubernetes Engines which do the heavy lifting:
* Azure
  * **Azure Kubernetes Service (AKS)**
  * Used by Siemens, HP, Heineken
* AWS
  * **Amazon Elastic Kubernetes Service (EKS)**
  * Used by Netflix, GoDaddy
* Google Cloud Platform
  * **Google Kubernetes Engine (GKE)**
  * Used by YouTube / Google Maps / Google Search

These Engines provide us with the underlying configuration and the Kubernete managed servers for us to access. We'll be using **AKS** for this course.

<br>
<br>

### Creating an Azure Account

An Azure free account gives you a starting credit (around $200 / equivalent local currency) to spend over your first 30 days, plus a set of services that stay free for 12 months. 

You'll need:
* Personal Details
* Valid Credit/Debit Card (used for identity verification — you won't be charged unless you explicitly upgrade)
* A Microsoft Account to sign in with
* Navigate to: https://azure.microsoft.com/free
* Fill out all the information 

**NOTE FOR TRAINERS** <br>
The exact free credit amount and offer terms change from time to time — worth double-checking the current offer on the page above before class, rather than promising a specific figure to students. <br>
**END OF NOTE**

### Creating Kubernetes Clusters with Azure Kubernetes Service (AKS)


What's a cluster?

First let's define Kubernetes… it's a resource manager. <br>
**ASK** <br>
What's the resources that Kubernetes manages? <br>
**ANSWER** <br>
Servers

The servers are in the cloud, so we can refer to them as virtual servers. 

Different cloud providers have different names for these generic virtual servers:
* **Azure**: **Virtual Machines**
* **AWS**: **AWS EC2 (Elastic Compute Cloud)**
* **GCP**: **Compute Engine**

**Kubernetes** calls them **nodes**

*REFER TO RESOURCE 9 - SLIDEE* <br>
![intro-to-kubernetes-3](./resources/intro-to-kubernetes-3.png)

Kubernetes can manage thousands of these virtual servers or **nodes**. <br>
With thousands of things to oversee, Kubernetes makes that process easier by introducing managers, called **Master Node(s)**, more commonly referred to today as the **control plane**.

**NOTE FOR TRAINERS** <br>
This is a genuinely nice thing about AKS worth calling out directly: unlike some other managed Kubernetes offerings, Azure fully manages and hosts the control plane for you **at no extra cost** — you don't see it, configure it, or pay for it as a resource. You only see and pay for the **worker nodes** in your **node pool**. This is a real difference from providers where you can pick a "zonal" vs "regional" control plane, or pay a per-cluster management fee — with AKS there's simply nothing to configure there. <br>
**END OF NOTE**

So a **cluster** is just a combination of **Nodes**
* **Worker Nodes** (commonly referred to as **nodes**): are the nodes that do the work
* **Control Plane**: Oversee management of nodes (fully managed by Azure, hidden from us)
  * Makes sure nodes are available 
  * Makes sure nodes are doing some work 

##### Creating a cluster 
* So we should go to: [portal.azure.com](https://portal.azure.com)

We want to create a Kubernetes cluster. <br>
If we search for **Kubernetes services** in the top search bar and click the matching result.

![intro-to-kubernetes-5](./resources/intro-to-kubernetes-5.png)

This will take us to the Kubernetes services landing page. <br>
![intro-to-kubernetes-6](./resources/intro-to-kubernetes-6.png)

If you ever want to get back to this page we can again just search for **Kubernetes services**

On the left hand panel, once inside a cluster, we'll be able to see a few of the tools available:
* Node pools: Where our worker nodes live
* Workloads: The applications we run in the cluster
* Secrets: Where we can store secrets tied to the cluster
* Services and ingresses: How traffic reaches our applications

Let's start by creating a **Cluster**. <br>
On the Kubernetes services page, click **Create > Create a Kubernetes cluster**

At the top of the **Basics** tab, Azure lets us choose between a **cluster preset configuration**. There's a newer, more hands-off option called **Automatic** (broadly comparable to GKE's Autopilot — Azure manages node pools, scaling and security settings for you), and a **Standard** preset where we configure and manage the nodes ourselves.

![intro-to-kubernetes-7](./resources/intro-to-kubernetes-7.png)

* with **Standard**
  * We configure and manage all our nodes
* **Automatic**
  * Very little configuration, Azure manages most of it for us

We're going to choose **Standard**, so we can see what's actually being configured.

This will take us through the **Create Kubernetes cluster** wizard.

- **Subscription**: Pay-As-You-Go
- **Resource group**: I'll create a new one, `rg-aks-emilesherrott-devops`
- Give a name: I'll be using **emilesherrott-cluster**
- **Region**: I'll choose **UK South**
- **Availability zones**: I'll leave the default selection (zones 1, 2, 3)
- The rest I'm going to leave as default 

*HIT REVIEW + CREATE, THEN CREATE*

It should take a little while to complete this. 

<br>
<br>

### Review Kubernetes Cluster and Learn a few fun facts about Kubernetes

While the **Kubernetes Cluster** is being created let's look over some facts about **Kubernetes**

* You may see the **Abbreviation**: **K8S** -> `K <eight-letters> S`
* Logo is about providing direction 
* Kubernetes on Cloud
  * Azure: AKS
  * AWS: Amazon EKS
  * GCP: GKE

Now our cluster should be up and running, we can see there's a few pieces of information we've got back on the top level. The cluster:
* name
* location
* Kubernetes version
* number of nodes

#### Overview
If we click onto the **cluster** name we can see more information about it. 

From **Overview**
* **Kubernetes version**: Version of Kubernetes being used
* **API server address**: how tools like `kubectl` reach the control plane
* **Region**: where our nodes are located

**NOTE FOR TRAINERS** <br>
Unlike GKE, where the Portal shows you a "control plane zone" alongside your "default node zones", AKS doesn't expose control-plane location details at all — that's Azure's infrastructure to manage, not ours to configure or inspect. All we can see and configure is where our **nodes** live. <br>
**END OF NOTE**

#### Node pools
We can also switch over to **Node pools** to get some specific information about them

**Node Pool**
- The key information
- The **VM size** (Azure's equivalent of GCP's "Machine Type")

If we click into the node pool and look at an individual node's details, we can see
- Amount of CPU currently being requested
  - mCPU stands for "milliCPU" which is the unit of measurements for cloud computing — this is a Kubernetes-level concept, so it's identical no matter which cloud we're running on. 
  - `<number mCPU>` is the portion out of 1 whole core being utilised
* How much CPU available: **CPU allocatable**
* Memory available for new applications on these nodes

When we created the nodes, we created them each with a specific VM size, for example a machine with:
* 2 virtual CPUs
* 4GB of memory

However when we look into the cluster there's less available. <br>
So it looks like we're down a certain amount of memory. 

This is Kubernetes working in the background. It takes some CPU and Memory to manage these nodes. 

<br>
<br>

### Deploy Your First Docker Container to a Kubernetes Cluster

The first step to deploy an application to a Kubernetes cluster is to connect to the Kubernetes cluster. 

We'll use the CLI to do this. 

Azure gives us access to a CLI called **Azure Cloud Shell**. 

To access it we need to make sure we're in the Azure Portal, viewing our cluster.

From this window, in the top right of the Portal (not scoped to the cluster specifically) we can see a terminal icon — if we hover over it, it says: `Cloud Shell`

![intro-to-kubernetes-8](./resources/intro-to-kubernetes-8.png)

When we click on it we can see we get access to a terminal. It'll take a little while to configure, and the first time you use it, it'll ask you to create a small storage account to persist your Cloud Shell files between sessions.

Right now it takes up one portion of our screen — there's usually an option to maximise it, which will allow us to work in it more comfortably.

So now we have a CLI and a complete UI for our cloud platform which will allow us to interact with our Azure resources. 

So to connect to the Cluster we need to click on the **Connect** button on our cluster's Overview page. 

![intro-to-kubernetes-9](./resources/intro-to-kubernetes-9.png)

Then we can copy the command shown under **Azure CLI**, and paste that command into our **Cloud Shell**

* **e.g.**: `az aks get-credentials --resource-group rg-aks-emilesherrott-devops --name emilesherrott-cluster`
* If we look at the command we can see it's an Azure CLI command to connect to the cluster. We:
  * Connect to the cluster: `--name emilesherrott-cluster`
  * Within the resource group: `--resource-group rg-aks-emilesherrott-devops`

We can run that command:
* if we're prompted to authenticate, we can approve it.
* run the command again if there were any issues

#### kubectl

Let's look at the command then we saw earlier **kubectl**

Pronounced "cube-cuttle" or "cube-control", it means Kube Controller.

It's a Kubernetes command to interact with our Clusters. It's a great command because it will work with clusters in our local machine, data centre or cloud — including AKS, EKS, or GKE equally, since `kubectl` itself doesn't care which cloud provider is underneath it.

Once we've connected to the cluster we can execute commands against any cluster using **Kubectl**
* What we saw earlier with the demo commands, **kubectl** can:
  * deploy a new application
  * increase number of instances
  * deploy new version 

**kubectl** is already available for us to use in the Cloud Shell. 

* **inside Cloud Shell**, run: `kubectl version`
* It returns the Client Version and Server Version 

![intro-to-kubernetes-10](./resources/intro-to-kubernetes-10.png)

The Server Version is the cluster we're connecting **Kubectl** to. 

#### Deploying application to kuberenentes cluster
* `kubectl create deployment hello-world-rest-api --image=lfacademy/hello-world-rest-api:0.0.1.RELEASE`

It fair to ask where this image being referred to: `--image=lfacademy/hello-world-rest-api:0.0.1.RELEASE` is coming from. <br>
This is a Docker image for the api we want to deploy. Which is being kept on the Docker repository.

*REFER TO RESOURCE 10 - SLIDEE* <br>
![intro-to-kubernetes-11](./resources/intro-to-kubernetes-11.png)

- Docker images take a few steps before they're created:
  - We have the application code.
  - We then use a Dockerfile to create an image
  - We then can push that image to the Docker Repository. 

When we run: `kubectl create deployment hello-world-rest-api --image=lfacademy/hello-world-rest-api:0.0.1.RELEASE` an image on the Docker hub repository is found and pulled. 


*REFER TO RESOURCE 11 - SLIDEE* <br>
**NOTE FOR STUDENTS** <br>
When I've been building images previously, by default they've been built using the platform **linux/arm64/v8** which is fine for my local machine. 
  - The nodes underlying our AKS cluster typically run on a different platform to my own laptop, **linux/amd64**, so if we're building images for the cloud you'll need to update your build commands to something like: `docker build --platform linux/amd64 -t lfacademy/hello-world-rest-api:0.0.1.RELEASE .` where we specify the platform. 
  - By the platform, all I really mean is the OS and the CPU architecture.
  - Some CPU architecture is more commonly found on Laptops and Mobiles or on larger servers. AKS does also support ARM-based node pools if we specifically wanted to match an ARM-built image, but we won't be using that today.
**END OF NOTE**

Anyway, from our previous command we should get a response from the **Cloud Shell** giving us a confirmation the deployment has been created. 

Our deployment's been created. We now need to expose this to the outside world. 
* `kubectl expose deployment <name-of-deployment> --type=<type-of-deployment> --port=<port-number>`

* `kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080`

It should respond which a service being exposed. <br>
![intro-to-kubernetes-12](./resources/intro-to-kubernetes-12.png)

We should really find out the status of creation of this service. For this we'll use the UI. 

- On our cluster's page, if we look on the left hand side we can see a tab: **Services and ingresses**
- We created a **Service** when we exposed our **Deployment**

![intro-to-kubernetes-13](./resources/intro-to-kubernetes-13.png)

We can see:
* the name of the service: `hello-world-rest-api`
* status of **OK**
* the type: `LoadBalancer`
* the external IP

Let's click on that external IP. 

We should see some JSON coming back.

If we go to the actual endpoint where the API is exposed from **/hello-world** we should get a **hello-world** response. 

* http://20.108.19.134:8080/hello-world
* `<this-part-will-change>/hello-world`

Essentially what we've done quite easily is:
* Take an image from Docker Hub
* Deploy to Kubernentes
* Expose as a service to the outside world 

If part of the DevOps philosophy is getting applications deployed easily which I believe we've achieved. 

<br>
<br>

### Quick Look at Kubernetes concepts - Pods, Replica Sets and Deployment

Getting an application up and running on Kubernetes is the easy part though. 

Understanding what's happening behind the scenes is more complicated and there's lots of new concepts for us to explore. 

With our service created and exposed let's go to the Cloud Shell and run:
* `kubectl get events`

It'll return a lot of events. All these events are to do with creating the cluster. 


We can see a **pod** which was created, we've pulled an image and a sequence of steps to start a container. <br>
![intro-to-kubernetes-14](./resources/intro-to-kubernetes-14.png)

We can see something called a **replica set** and a **deployment** as well.<br>

Finally we can see something called a **service**

![intro-to-kubernetes-15](./resources/intro-to-kubernetes-15.png)

We create and exposed a simple deployment. Kubernetes itself created a port, a replica set and a service. 

What are these and why are they being created? Let's run a few more commands.

#### pods
* `kubectl get pods`
* named **hello-world-rest-api** which is our **deployment** name
  - the numbers on the end are the id of the **replica set** followed by the id of the **pod**

![intro-to-kubernetes-16](./resources/intro-to-kubernetes-16.png)

This command returns all the **pods** that have been created. 

#### replica set
* `kubectl get replicaset`
* named **hello-world-rest-api** followed by the id.

![intro-to-kubernetes-17](./resources/intro-to-kubernetes-17.png)

#### deployment
* `kubectl get deployment`
* named **hello-world-rest-api**

![intro-to-kubernetes-18](./resources/intro-to-kubernetes-18.png)


#### `kubectl get <option>`

`kubectl get` is a generic command, we can pass in different `<options>`
* `kubectl get service`
* We have our **load balancer** service which we created earlier
* Then **kubernetes** which is an internal service we don't need to worry about

![intro-to-kubernetes-19](./resources/intro-to-kubernetes-19.png)


#### Single Responsibility Principle 

We have a few concepts to learn because Kubernetes uses something called a: **Single Responsibility Principle**.

This basically means, **one concept, one responsibility**
So a:
* pod
* replica set
* deployment
* service <br>

All have an important and unique roles to play. 

When we ran: `kubectl create deployment` we made the: **pods, replica set and deployment**

When we ran: `kubectl expose deployment` we made the: **service**

Each concept has a role and responsibility which makes Kubernetes a great tool to:
* manage workloads
* provide external access to workloads
* enable scaling
* enable zero downtime deployments

If we think back to **AWS's** definition of DevOps. They said:<br>
*"DevOps is the combination of cultural philosophies practices, and tools that increases an organisatons ability to deliver applications and services"* 

Kubernetes is definitely a tool helping us achieve this. 

<br>
<br>

### Understanding Pods in Kubernetes 
Let's take a deeper look into understanding **pods**

*REFER TO RESOURCE 13 - SLIDEE* <br>
![intro-to-kubernetes-21](./resources/intro-to-kubernetes-21.png)

**Pods** in Kubernetes are the smallest deployable unit we have access to. 


If we want to create a container in Kubernetes, we can't do so without a **pod** to run it on. 

All containers live inside **pods**.

* Run: `kubectl get pods -o wide` <br>

![intro-to-kubernetes-22](./resources/intro-to-kubernetes-22.png)
* **IP**: One of the important things we're seeing is the **IP address** which is unique to it. 
* **READY**: This is returning `1/1`: The number of containers present in pod and how many are ready. 
  * A pod can contain multiple containers. 
  * All containers share the resources of that pod. 
  * Containers in a pod can talk to each other using **localhost**


* Run: `kubectl explain pods`

![intro-to-kubernetes-23](./resources/intro-to-kubernetes-23.png)

In the **description** it says:

```
Pod is a collection of containers that can run on a host. This resource is created by clients and scheduled onto hosts. 
```

When we say **host** what we mean is a **node** or our servers on the kubernete cluster. 

So a Kubernetes **node** can contain multiple **pods** and each **pod** can contain multiple **containers**. 

Each **pod** can be related to the same application or to different applications. 

* Run: `kubectl get pods` -> copy the **id or name**
* Run: `kubectl describe pod <pod-full-name>`

Show the details of the **pods**

![intro-to-kubernetes-24](./resources/intro-to-kubernetes-24.png)

We can see the:
* **name**
* **ip**
* the specific **node** its running on
* **namespace** which is set to default
  * namespaces are important: they provide isolation from one part of the cluster to other parts of the cluster. 
  * We could have the **Dev and QA environments** running inside the same Cluster and wish to separate those resources. 
  * We can create separate namespaces for **Dev and QA** and associate each resources with that namespace. 
* **labels**: Has the value `app=hello-world-rest-api`which is linked to this specific pod. 
  * We've spoken about **pods, replica sets, service** we link all these together using **selectors** and **labels**.
* **Annotations**: Typically meta data:
  * author name
  * build id
  * etc.
* **Status**: The status of the pod, is it running etc. 

Essentially a **pod** is a resource to keep our **containers**.

**NOTE FOR STUDENTS**
If we're not using a Cluster, we should delete it, just to preserve some of our free credits. We can do this from the **Azure Portal: Kubernetes services** in the **Overview** or by deleting the resource group we created. <br>
**END OF NOTE**

<br>
<br>

### Understanding ReplicaSets in Kubernetes

* `kubectl get replicaset` <br>

This will return the **replicaset** made with the deployment

![intro-to-kubernetes-25](./resources/intro-to-kubernetes-25.png)

* An alternative command is: `kubectl get rs`

What's the role of a **replica set**? <br>
They ensure that a specific number of pods are running at all times. <br> 
We can see from the response of our command that we have: <br>
- **Desired**: Number of pods desired
- **Current**: We have one pod
- **Ready**: Number of containers running 

* `kubectl get pods -o wide`
![intro-to-kubernetes-26](./resources/intro-to-kubernetes-26.png)

We've used this command before and it's just returning information about the pods we have running. 
Shows the:
* Node where it's running
* The IP address
* etc.

We can see there's one pod. Under the key of **Name** we've got the name of the **pod** and an **ID** on the end. 

e.g.: `hello-world-rest-api-5b78b5c566-m8jh4`

Let's delete the **pod**. We'll need to remember the value of the **pod id**
* `kubectl delete pod <pod-full-name>`
* e.g.: `kubectl delete pod hello-world-rest-api-5b78b5c566-m8jh4`

Let's run our previous commands to see the pods we have running again and see if we have any pods working. 
* `kubectl get pods -o wide`

![intro-to-kubernetes-27](./resources/intro-to-kubernetes-27.png)

We can see that the previous **pod** no longer exists. 
* The earlier **pod** was: e.g. `hello-world-rest-api-5b78b5c566-m8jh4`
* This **pod** we have now is: e.g. `hello-world-rest-api-5b78b5c566-2njdd`
- We can see the first part of the **id** is the same, that's because it's part of the same **deployment** and **replica set**

So we deleted the last **pod** but a new one was spun up for us after around a minute. 

The URL is still working. 

This is because of the **replicaset**, which is always monitoring the amount of **pods**, if the number of **pods** is fewer than the amount we requested, the **replicaset** will start a new **pod** for us. 

If we wanted 3 **pods**, we can tell the **replicaset** to maintain a higher number of **pods**.
* `kubectl scale deployment <deployment-name> --replicas=<number-of-instances>`
* e.g. `kubectl scale deployment hello-world-rest-api --replicas=3`

![intro-to-kubernetes-28](./resources/intro-to-kubernetes-28.png)

The output is telling us that the deployment has now been scaled. 

Now when we run: `kubectl get pods -o wide` we should expect to see 3 instead of the previous one. 
* Run: `kubectl get pods `

![intro-to-kubernetes-29](./resources/intro-to-kubernetes-29.png)

We can see the previous **pod**:
* e.g. **hello-world-react-api-5b78b5c566-2njdd** is older than the other two which just got spun up. 

Now with 3 pods we have multiple instances of the application running and load distribution between the 3 of them. 

* Run: `kubectl get replicaset` <br>
![intro-to-kubernetes-30](./resources/intro-to-kubernetes-30.png)

Now the **desired** is set to three. <br>
After we ran: `kubectl scale deployment hello-world-rest-api --replicas=3` the **replica set** would have noticed the current was 1 and then would have loaded two up. 

Let's see what's happening in the background.
* Run: `kubectl get events` <br>
On the left hand column we can see the events aren't sorted by time so we can run a new command to see the information a bit more clearly. 
* `kubectl get events --sort-by=.metadata.creationTimestamp`

Near the bottom we can see that we've scaled up the number of pods to three. 

The following events are pulling the image and then running them onto the pods. 

* Run: `kubectl explain replicaset`
![intro-to-kubernetes-31](./resources/intro-to-kubernetes-31.png)

*READ* <br>
```
ReplicaSet ensures that a specied number of pod replicas are running at any given time.
```

That's the job of **replicaset**, when we scale a **deployment**, the **deployment** updates the **replicaset** and if the number of **pods** is less than that desired the number, the **replicaset** will automatically scale for us. 

So just to reiterate: <br>
**PODS**: Where are containers run <br>
**REPLICA SET**: Ensures specified number of pods is running

<br>
<br>

### Understand Deployment with Kubernetes

Why do we need the **Deployment** resource in Kubernetes. <br>
Imagine we have a specific version of an application deployed. <br>

**Version 1** and we want to update to **Version 2**. When updating an application the main thing we want is zero down time so people can still use our app when it's transitioning between versions. 

* `kubectl get replicaset` or `kubectl get rs`

![intro-to-kubernetes-32](./resources/intro-to-kubernetes-32.png)

So we have our deployment and the number of pods up and running. We can see more information, similar to pods with: `kubectl get rs -o wide`
* Run: `kubectl get rs -o wide`

![intro-to-kubernetes-33](./resources/intro-to-kubernetes-33.png)

We can see the details of the **image** related with the **replicaset**. Within this information we can see the **tag** which is basically the version of the release. 

With any application we'll get to the point where we wish to deploy a new version. 

* `kubectl set image deployment <deployment-name> <container-name>=<image-name>` <br>


*DON'T RUN*
- I'll just write in a command to update the image to a fake one
  * `kubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:test`
    * `kubectl set image deployment hello-world-rest-api` the name of the deployment in Kubernetes
    * `hello-world-rest-api`: name of the container
    * `DUMMY_IMAGE:test` we set the path of the container to an image

In this case the **path to the image** could cause an error.

Basically I want to see if the deployment will go down. 

* Run: `kubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:test`

We can see the response is that the **image** has been updated. 

This is concerning because we know the image is a fake.

If we go back to the location of the the deployment's endpoint and refresh. <br>

*VISIT DEPLOYMENT*

Things still seem to be working even though we made a mistake with our deployment. The old version is still working. 

* Run: `kubectl get rs -o wide`
![intro-to-kubernetes-34](./resources/intro-to-kubernetes-34.png)

We can see there's now two replica sets. 
1. The first is the one associated with the genuine deployment, we can see the:
   * **image**
   * the number of **pods**
   * the age
2. The second **replicaset** created much more recently has the DUMMY image associated with it. 
   * We can see, there's **1 Desired Pod**
   * 0 are **Ready** though

They're not Ready because the image has an error. 

* Run: `kubectl get pods`

![intro-to-kubernetes-35](./resources/intro-to-kubernetes-35.png)

We can again see the new **Pod** has a **status** of **InvalidImageName**. 

If we're interested in the details we can run: `kubectl describe pod <pod-full-name>` <br>
If we trawl the information we can see it's got an invalid image name. 

* Run: `kubectl get events --sort-by=.metadata.creationTimestamp` <br>
![intro-to-kubernetes-36](./resources/intro-to-kubernetes-36.png) <br>
What we can see is we've:
* Started a new **Deployment**
* The **Deployment** has created a new **Replica Set**
* the **Replica Set** said we need one instance
* Which in turns has launched a new **Pod** 
* the **Pod** failed to start 

*REFER TO RESOURCE 12 - SLIDEE* <br>
![intro-to-kubernetes-37](./resources/intro-to-kubernetes-37.png)

So we currently have a **deployment** with a **replica set** with three **pod instances** running.

The **replica set** was associated with **version 1** when we tried to update the application, we wanted to create a new **replica set** which managed new **pods** linked to new **containers** built from new **images**.

That **replica set** is trying to launch a **pod** for that **replica set**. The **deployment** said to start with one **pod instance**. This process failed because of the image. 

Let's try again using a proper image. 

`kubectl set image deployment hello-world-rest-api hello-world-rest-api=lfacademy/hello-world-rest-api:0.0.2.RELEASE`

* `kubectl set image deployment <deployment-name>`
  * `<container-name>`
  * `=<path-to-hosted-image>:<tag>`

* Run: `kubectl set image deployment hello-world-rest-api hello-world-rest-api=lfacademy/hello-world-rest-api:0.0.2.RELEASE`

We've got our response that the image is updated again.

Let's check the deployment was successful.

* Run: `kubectl get pods`

![intro-to-kubernetes-38](./resources/intro-to-kubernetes-38.png)

**POSSIBLE NOTE FOR STUDENTS** <br>
If we were quicker we'd have seen that the three original **pods** would be in **Status: Terminating** <br>
**END OF NOTE** <br>

We can see our new **pods** are very new in terms of age. 

* Run: `kubectl get rs`

![intro-to-kubernetes-39](./resources/intro-to-kubernetes-39.png)

So the:
1. First **replica set** is the original
2. Second **replica set** is the one with the fake image
3. Third **replica set** is the new release of the original
   * **Desired**: Number of pods desired
   * **Current**: Number of pods available
   * **Ready**: Number of containers running 

If we visit our end point again. 

We can see we're now on **v2**. 

* Run: `kubectl get events --sort-by=.metadata.creationTimestamp`

So what happens?

* Deployment creates a new **replica set** for **v2**
* Gives it **one pod**
* The **pod** runs up a **v2 container** 
* Scales down the **v1 replica set** to 2 **instances / pods**
* Scales up the **v2 replica set** to 2 **instances / pods**
* Scales down the **v1 replica set** to 1 **instances / pods**
* Scales up the **v2 replica set** to 3 **instances / pods**
* Scales down the **v1 replica set** to 0 **instances / pods**

What we can take away from this is that it updates **1 pod** at a time. 

<br>
<br>

### Quick review of Kubernetes Concepts Pods, Replica Sets, Deployments

*REFER TO RESOURCE 13 - SLIDEE* <br>
![intro-to-kubernetes-40](./resources/intro-to-kubernetes-40.png)

Let's summarise what we've just covered. 

#### Pods
**pods** are wrappers for a set of **containers**

It has:
* An IP address
* Labels / Annotations

#### Replica Set
Ensures a specific number of **pods** are running. <br>   
**Replica set's** in practice are tied with a specific release version. <br>
We can have a **replica set** for a **v1** release and that'll be maintaining a specific number of **pod instances** for that release. 

#### Deployment
A deployment ensures a release upgrade without downtime. 

There's a variety of deployment strategies when releasing a new version:
* We could send 50% traffic to **v1** and 50% traffic to **v2**
* We could do what's called a rolling upgrade:
  * Create one **pod instances** of **v2** ensure it works
  * Reduce number of **pod instances** of **v1**
    * Create another **pod instance** of **v2** etc. 

The default method of **deployment updates** is the **rolling upgrade** which is what we saw in our earlier update. 

We'll look more into **deployment updates** later. 

Kuberentes can be tricky, there's lots more intricacy to the technology but the more we familiarise ourselves with it the easier it becomes to do really interesting and useful things well.

Ultimately it should enable us to achieve some of our DevOps goals. <br> 
Reliable technology, quick deployment at low cost and effort. 

This is why Kubernetes is utilised on pretty much every cloud platform and is used by lots of enterprise companies. 

<br>
<br>

### Understanding Services in Kubernetes

Let's understand the need for a **service**

* Run: `kubectl get pods -o wide` <br>

![intro-to-kubernetes-41](./resources/intro-to-kubernetes-41.png)

**NOTE FOR TRAINER**: *DON'T CLEAR TERMINAL* <br>

We can see that each **pod** has a different **IP address** associated with it. 

Again on the **Azure Portal** in **Services and ingresses** we know we can access this application on our **endpoint** that doesn't match any of the **IP address** our **Pods** have. 

Our **deployment** has a load balancer which is splitting the traffic between the three **pod instances** 

If we grab one **pod name/id** and run:
* `kubectl delete pod <pod-name/id>`

Then we'll give it a few moments and re-run our command: `kubectl get pods -o wide`

**ASK** <br>
What's different? <br>
**ANSWER** <br>
Our newly created pod has a new **IP address** assigned to it. <br>

Each time a **pod** starts up it gets a new **IP address**.

This could be problematic, each time a **pod** changes, we don't want our users to have to navigate to a new **IP address**. 

We want to have a specific endpoint we can reach which we don't want to have to change.

So this is why we we'll turn our attention to **services**

**Service**

As far as Kubernetes is concerned, a **pod** is a throwaway resource. 

They're built expecting to be removed, to go down, then be loaded back up. <br>
The role of a Kubernetes **service** is to provide an always available interface to the applications running inside the **pods**. <br>
They'll always be able to receive traffic through a permanent **IP address**.

To reiterate we created our **pods, replicates and deployments** by running:

 `kubectl create deployment <deployment-name> --image=<image-name>`

We created our **service** running:

`kubectl expose deployment <deployment-name> --type=<deployment-type> --port=<port>`

From **Services and ingresses** let's click onto our **service**

![intro-to-kubernetes-42](./resources/intro-to-kubernetes-42.png)

From this point we can see:
* the labels associated with the service
* the deployments associated with it
* the external IP
* the pods running and serving the requests for our service

We can see how this is implemented in **Azure**, from the top search bar if we search for **Load balancers**.

![intro-to-kubernetes-43](./resources/intro-to-kubernetes-43.png)

What we see is a specific **Load Balancer** resource which was automatically created for the **service** — this is Azure's **Standard Load Balancer**, sitting in the same resource group as our cluster's node pool.

* Under **Frontend IP configuration**, we can see the **IP** which is the one we've been using in the browser.
* Under **Backend pools**, we can see our three **pod instances** (via the underlying node pool)

So to summarise, a **service** is basically there to provide a **frontend** to maintain a present IP address available, regardless of the **pod instances** on the backend. 

**In Cloud Shell** run: `kubectl get services`

![intro-to-kubernetes-44](./resources/intro-to-kubernetes-44.png)

We see our **Load Balancer** service we have called **hello-world-rest-api**

We also have a **kubernentes Cluster IP service**. 

* A **Cluster IP** service can only be accessed from within the **Cluster**
* We can see there's no **External-IP** for anyone to connect onto it.
- It create a connector for node-to-node communcation which we can use if we have a specific application where nodes need to talk with each other. 

There's one more type of **service** we'll introduce you to called a **NodePort** which we'll talk about later. 

<br>
<br>

### Understand Azure Regions and Availability Zones
When we set up our Cluster we chose a specific region. Let's just spend a moment talking about why we have regions as we'll see this more and more through the deep dive. 

Let's do a Google Search for: *Azure global infrastructure*

https://azure.microsoft.com/en-us/explore/global-infrastructure/geographies/

Azure runs datacentres across 60+ regions worldwide — more than any other major cloud provider — grouped into geographies to help meet data residency requirements. If we look at the map, we can see regions clustered in populated, developed parts of the world.

So there's regions set up in developed or populated parts of the world. <br>
One of the reason we have these variety of locations is do with something called **latency**

**ASK** <br>
What is **latency**? <br>
**ANSWER** <br>
The time it takes for requests and responses to traverse a network <br>

If all of our users for an application are in **Wolverhampton**, let's say for Wolverhampton Football Club but we choose to deploy our application in **Melbourne** (Azure's **Australia Southeast** region) we'd have some latency issues. 

That's one of the reasons we'd want to set up our **Clusters** close to our users. 

Another reason we have multiple **locations** is due to **availability**

**ASK** <br>
What do I mean by **availability**? <br>
**ANSWER** <br>
The ability to access resources <br>

Imagine there was flooding in **London**, perhaps now all of our users in **Wolverhampton** won't be able to access to application. 

So we want to **distribute** our application through lots of regions to ensure if something happens in **UK South (London)**, we'd be able to run it out of **France Central (Paris)** and still have relativly good performance. 

Another reason we'd want to **distribute** our application is to do with legal reasons. 

Some countries won't allow data related to its citizens to be kept outside of that country. 

There's also something called **Availability Zones** — Azure uses the same term as AWS here (GCP just calls them "zones"). Most Azure regions that support Availability Zones have 3 of them, though — unlike GCP, where every region we looked at had a uniform number of zones — **not every Azure region supports Availability Zones at all**. It's worth checking the [Azure regions page](https://azure.microsoft.com/en-us/explore/global-infrastructure/geographies/) for whichever region you're planning to use.

Within each **region** that does support zones, we have a number of different physically isolated **Availability Zones** again to help with keeping our application available. <br>

If one area of **London** has a power cut, we hope it wouldn't affect other **zones**.

Again, this is so if a **zone** goes down within a **region** you'd still have availability.

<br>
<br>

### Installing the Azure CLI
In the previous steps we were using **Cloud Shell**, we could and should start to use our:
* terminal
* command prompt
* git bash

We just need to install:
* **Azure CLI**: which is a CLI interface to Azure, essentially allowing us to access our resources on the Cloud platform
* **Kubectl**: we'll also need **kubectl**

Azure CLI: [Install the Azure CLI | Microsoft Learn](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

#### Logging into Azure CLI
`az login`

This opens a browser window to authenticate, the same as it did earlier.

If you have access to more than one subscription, you'll want to make sure the right one is active:

#### Changing Subscription
`az account set --subscription <subscription-id-or-name>` <br>
- i.e. `az account set --subscription "Pay-As-You-Go"`
- You can list what's available first with `az account list -o table` if you don't remember the exact name.

**NOTE FOR STUDENTS**
Unlike `gcloud`, which asks you up front whether you want to configure a default region/zone, `az` doesn't prompt for this by default — if you want one, you can optionally set it yourself with `az configure --defaults group=<resource-group> location=<region>`. Not required for today, just useful to know it exists.
**END OF NOTE**

<br>
<br>

### Installing Kubectl 

Let's install **kubectl**, we may already have it from Docker. 
- Go to the Docker App
- Go to Settings
- Go to Kubernetes 
- Click 'Enable Kubernetes' 

Check with: `kubectl version`

If you get a response, you don't need the additional install. 

If not, the Azure CLI can actually install it for us directly, which is the quickest route if we've already got `az` installed:

* `az aks install-cli`

Otherwise, go onto Google and search: **Install and Set up Kubectl**. 

I'd go for **Homebrew** on Mac or **Chocolatey** on Windows.

Once we've installed, let's play around with our project.

We can run the usual command from the GUI to connect to the project. <br>

**From our cluster** <br>

![intro-to-kubernetes-52](./resources/intro-to-kubernetes-52.png)

Run that in terminal.

**NOTE FOR STUDENTS** 
If your cluster has Azure AD / Microsoft Entra ID authentication enabled, `kubectl` may ask you to install an extra credential plugin called **kubelogin** the first time you try to run a command — this is the AKS equivalent of the extra `gke-gcloud-auth-plugin` step needed on GKE. `az aks install-cli` installs this alongside `kubectl` for exactly this reason, so it's worth running that command even if you already have `kubectl` from Docker.
**END OF NOTE**


If done those steps, copy and run the previous command we grabbed from the GUI.

So now we can connect locally. This is good because we now have more flexibility. 
* We can create files more easily
* Put them into version control
* We can introduce YAML files

<br>
<br>

### Generate Kubernetes YAML Configuration for Deployment and Service

Right... so far we've been using lots of different commands to deploy and scale with Kubernetes. 

Another reason Kubernetes is popular because it's **declarative**, we can define our **desired state** in a **YAML** file and Kubernetes will ensure it's up and running to that spec. 

Let's start using them. In the repo we downloaded navigate to: <br>
**kubernetes/intro-to-kubernetes/starter-code/kubernetes/01-hello-world-rest-api** <br>
Run: `code .` <br>
It's from here we'll define our **YAML** file. <br>

So far we've been:
* deploying **hello-world-rest-api**
  * `kubectl get deployment hello-world-rest-api`
  * 3 pods, 3 running, 3 containers inside

![intro-to-kubernetes-57](./resources/intro-to-kubernetes-57.png)

So we can retrieve information from our **deployment**

We can also retrieve information as a YAML file.
* `kubectl get deployment hello-world-rest-api -o yaml`

A lot of information
* API version
* The Kind

We know we can take an output and return it somewhere specifically from our lecture in LAP 1.
* From inside **/01-hello-world-rest-api**: `kubectl get deployment hello-world-rest-api -o yaml > deployment.yaml`

We should now see a **deployment.yaml** file created. 

Let's also get the **service** details. 

`kubectl get service hello-world-rest-api -o yaml > service.yaml`

Both of these files contain all the definitions based on all the commands we've been running.
* **deployment.yaml**
  * In **status** we have:
    * **availableReplicas: 3**
    * **updatedReplicas: 3**
  * In **spec/template/spec** we have:
    * **containers / -image** which shows the image we're using.


*STAY ON deployment.yaml* <br>

The same is true of the **service.yaml** file.


Instead of trying to understand every line, let's play around and see what we can change. 

Inside **spec** you'll see a key of **replicas** set to **3**. Let's update it to **2**

**deployment.yaml**
```yaml
spec:
  progressDeadlineSeconds: 600
  # UPDATED YAML
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: hello-world-rest-api
```

Let's save that file and from the same terminal directory run:

`kubectl apply -f deployment.yaml`

Now if we run: `kubectl get pods`

![intro-to-kubernetes-58](./resources/intro-to-kubernetes-58.png)

We can see we only have two **pods** running now. 

So now we're at a point where we can use the YAML file to make updates to our application.

**NOTE FOR TRAINER**
You'll encouter errors if you try to apply more changes before deleting a lot of the meta data contained within the **YAML configuration**

<br>
<br>

### Conclusion

- **Inform** students this marks the end of the lecture for this afternoon
- **Tell** students we will be working with YAML files again this afternoon
- **Inform** students not to run more `kubectl apply` commands as it'll cause an error until we change some more configuration
- **Direct** students to the exercises for the rest of the morning

### Exercises

You'll see the exercises in the student repo. Only focus on Exercise 1 for today because exercise two is for tomorrow once we've finished kubernetes. 

---

[Back](./README.md)

---

