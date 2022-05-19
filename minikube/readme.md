# Welcome to the CN2 for Minikube guide

Cloud Native Contrail Networking (CN2) natively supports minikube so you can run a kubernetes cluster on your laptop using a single command: minikbe start. This guide walks you you through the ability to install CN2 in a Kubernetes cluster using minikube. (For instructions on a PC, there is not much different but some downloads will be for Windows.)

## Overview 

This guide is intended to provide hands on exposure to learn CN2 in an easy to consume manner using just your laptop. For installation instructions intended for large scale deployments refer to the [Juniper installation documentation](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/index.html).

## Environment
This procedure was performed on an Intel-based MacBook Pro but could be used for other platforms that support Minikube as well. Unfortunately, Minikube v1.23+ newer does not work at this time with CN2, therefore we will use Minikube v1.22, known to work with CN2, as we add in support for newer versions of minikube in the future. 

## Requirements
*	2 CPUs or more
*	7GB of free memory (5GB w/o analytics)
*	20GB of free disk space
*	Internet connection
*	[Podman installed](https://podman.io/getting-started/installation)
*	Download the deployer.yaml from the manifests folder to your local home directory. 
*	You need hub.juniper.net credentials for CN2. If you do not have credentials, fill out the [free-trial form](https://www.juniper.net/us/en/forms/cn2-free-trial.html) using your company email 📧 to gain access

## Getting Started 

1.	Install the latest [hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/) as the container/vmm:   
> ``brew install hyperkit``
>
2.	Download and install the minikube binary supported with CN2. In CN2 22.1 minikube 1.22 is required.  Downloads are available here, and minikube-darwin-amd64 would be the right v1.22 download for Macbooks with the intel i7 CPU. 
> Note that M1 ARM chips won’t support CN2 at this time.



