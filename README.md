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

Topology Diagram coming soon

## Pre-requisites:

1. Access to Citrix ITM portal. Sign-up if you are new to Citrix ITM.
2. Kubernetes clusters completely setup with Apps and Ingress UP and Running
3. A publically available DNS domain for your application


## Steps to setup ITM for your application

1. Add every Kubernetes cluster as a Platform in ITM
2. Create an OpenMix Application in ITM
3. Update the ITM provided CNAME for your application in your DNS server

### Add every Kubernetes cluster as a Platform in ITM

In order for ITM to perform GLB, we need to add our Kubernetes clusters as platforms to ITM. This enables ITM to collect
real-time stats about our cluster and perform intelligent GLB decisions.

Navigate to the Platforms page in ITM portal and create 2 platforms. One for the cluster in Asia and another for the
cluster in US.


### Create an OpenMix Application in ITM
### Update the ITM provided CNAME for your application in your DNS server
