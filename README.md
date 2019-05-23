# Hybrid Multi Cloud Kubernetes Cluster using Citrix ITM
This is a solution for managing your Hybrid Multi Cloud Kubernetes Cluster using [Citrix
ITM](https://www.citrix.com/en-in/products/citrix-intelligent-traffic-management/). We use Citrix ITM to perform GLB
decisions for your applications hosted inside the Kubernetes cluster. 
For example, a user trying to access your application from US should not routed to access the application from a cluster
in India. This Global Load-Balancing (GLB) of the application traffic is taken care by Citrix ITM.


## Assumptions:
1. You have a Hybrid multi Cloud Application running in Kubernetes
2. The Application is configured fine and working in all the Kubernetes clusters in your Hybrid multi Cloud deployments
3. It is a 2-tier or a single tier deployment with Citrix ADC VPX in the first tier.
4. You have a publically available DNS domain for your application and your Ingress is configured accordingly for that
domain

## NOTE:
The Hybrid Multi Cloud Kubernetes cluster is not
[Federation](https://kubernetes.io/docs/concepts/cluster-administration/federation/) enabled. Currently the apps, ingress, etc are manually
configured in every cluster.

## Deployment Details:
Currently for illustration I just have two Kubernetes cluster in GKE on different regions

1. Kubernetes cluster in Asia
    * GKE cluster with 3 nodes
    * *Deployment:* 2-Tier deployment with Citrix ADC VPX in Tier-1 and Citrix ADC CPX in Tier-2
2. Kubernetes cluster in US Central
    * GKE cluster with 3 nodes
    * *Deployment:* 2-Tier deployment with Citrix ADC VPX in Tier-1 and Citrix ADC CPX in Tier-2

## Topology:

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/multicluster-itm-topology.png" width="500">

## Pre-requisites:

1. Access to Citrix ITM portal. Sign-up if you are new to Citrix ITM.
2. Kubernetes clusters completely setup with Apps and Ingress UP and Running
3. A publically available DNS domain for your application
4. Publish the Application


## Steps to configure ITM for your application

1. Add every Kubernetes cluster as a Platform in ITM
2. Create an OpenMix Application in ITM
3. Update the ITM provided CNAME for your application in your DNS server

### Add every Kubernetes cluster as a Platform in ITM

In order for ITM to perform GLB, we need to add our Kubernetes clusters as platforms to ITM. This enables ITM to collect
real-time stats about our cluster and perform intelligent GLB decisions.

Navigate to the Platforms page in ITM portal and create 2 platforms. One for the cluster in Asia and another for the
cluster in US.

##### Adding Platform for Kubernetes cluster in Asia

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/platform-asia.png" width="500">

##### Adding Platform for Kubernetes cluster in US

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/platform-us.png" width="500">

### Create an OpenMix Application in ITM

Now that we have added our Kubernetes cluster as ITM platforms, let's create an OpenMix ITM application that actually
takes the GLB decision. 

In the ITM portal navigate to Openmix section, click "Application Configuration" and create a new Application.

Fill in the details appropriately (refer below)

**Step 1:**
Protocol: DNS
Application Type: Optimal RTT
Name: k8s-multi-cluster

Click NEXT

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/step-1.png" width="500">

**Step 2:**

For Fallback URL, I am going to provide the public IP of VPX of India Kubernetes cluster

Click NEXT

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/step-2.png" width="500">

**Step 3:**

Add the newly created platforms to the Application

In the CNAME, provide the public IP of the VPX (or public DNS of VPX if available) of the respective clusters.

After adding one platform, DON'T CLICK NEXT.

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/step-3-1.png" width="500">

Click "ADD PLATFORM" and add the second cluster

When you are done adding all the platforms, click NEXT

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/step-3-2.png" width="500">

**Step 4:**

This is the final step. If you want to configure any weights for a platform, you can do it here. Else click COMPLETE

### Update the ITM provided CNAME for your application in your DNS server

Once you have clicked COMPLETE, you would receive the CNAME for your application in the next screen.

Make a note of it and configure your DNS server accordingly. In my case, Route 53 is my DNS provider. I would update the
Route 53 to point my public domain to the CNAME provided by ITM.

### Publish the Application

Once you have updated the CNAME in your DNS server, you can PUBLISH the newly created ITM application.

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/cname-update.png" width="500">

## Testing your Hybrid Multi Cluster Application

Inorder to test application, we may need users from different geo-locations of the world so that we can see that the
users are landing on the nearest proximity sites.

In my example, I have two clients: 
1. In Bangalore, India
2. In US

If the user from India, tries to access my Application, he should land on the Kubernetes cluster that is created on GCP
Asia Region.
If the user from US tries to access my Application, he should land on the Kubernetes cluster that is created on GCP US
Central Region.

**Inorder to identify where the response is coming from, I have made use of our [Rewrite Responder
CRD](https://github.com/citrix/citrix-k8s-ingress-controller/blob/master/docs/crds/rewrite-responder.md). This Rewrite
Responder CRD would echo back the cluster location for us.**

When a user from India tries to access my Application, below is the response:

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/working-India.png" width="500">

When a user from US tries to access my Application, below is the response:

<img src="https://raw.githubusercontent.com/christus02/k8s-multicluster-ITM/master/images/working-US.png" width="500">

**We saw it working !!!**

### Citrix Ingress Controller

For more information on Citrix ingress controller and its features, please visit the official GitHub repo
[citrix-k8s-ingress-controller](https://github.com/citrix/citrix-k8s-ingress-controller)

### Hope this tutorial helps !!
