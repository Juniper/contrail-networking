<img width="1433" alt="image" src="https://user-images.githubusercontent.com/34732034/170180009-e573ea2f-e9aa-4005-b52e-ee6d9b22b211.png">



# Augmenting Visibility with Lens
[Lens](https://k8slens.dev/) is an open multi-cluster GUI for Kubernetes, augmented further with the Prometheus metrics “built-in” (a Prometheus pack). We have developed an extension for CN2 that will allow Contrail users to gain visibility and management of CN2 network functionality directly from Lens. Let's install Lens and the the monitoring stack including Prometheus and Grafana.

👉 To learn more about extensions in Lens see https://docs.k8slens.dev/main/extensions/

👉 The CN2 extension for Lens is tech preview at this time.

## Getting Started with Lens

1. Lens is a desktop application. To install the latest version on your machine, downloading it from the [Lens app repository](https://github.com/lensapp/lens/tags) (this is source code only recently and you may need to ask the Lens community for a specific download link) for a specific version like [Lens v5.4.4](https://api.k8slens.dev/binaries/Lens-5.4.4-latest.20220324.1.dmg) for Mac OS which is tested and supported with the CN2 extension. 

2. When you open Lens for the first time, there will be no clusters. It will also ask you to auto-upgrade. Do not do that. You need to add clusters, but first, let’s add the CN2 extension.

<img width="1433" alt="Screen Shot 2022-05-24 at 8 50 00 PM" src="https://user-images.githubusercontent.com/34732034/170176222-c9b6c196-a7a5-4a4e-9985-3e516ecbee8b.png">

3. Download the CN2 extension for Lens to your local machine. 

4. To install an extension, open the extensions manager by selecting it from the dropdown in the menubar. 

<img width="240" alt="Screen Shot 2022-05-24 at 8 34 47 PM" src="https://user-images.githubusercontent.com/34732034/170174864-278845fd-0881-4403-93f3-5beb514b8eee.png">

5. From the main Lens application navigate to and select the `CN2_lens_extension-latest.tar` file you downloaded to your local machine 

**👉 It will take a 1-2 minutes for the plug-in to install, but there is not a progress bar. Be patient and it will eventually show up in the list of installed extensions.**

<img width="984" alt="Screen Shot 2022-05-24 at 9 22 16 PM" src="https://user-images.githubusercontent.com/34732034/170179381-8db2e4d2-ccea-424d-8a9f-a8fbd89fb2fc.png">

Upon successful completion you will see the k8slens/cn2-ui extension listed under Installed extensions. You can now close the extension manager by clicking the X above ESC

6. With a default minikube installation on your machine, Lens will usually automatically find your [kubeconfig](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/) file. If it doesn't, then you can add a cluster manually and feed it the kubeconfig file that will be in your user home directory. 

<img width="468" alt="image" src="https://user-images.githubusercontent.com/34732034/170180716-de11739c-4a2e-45bb-892a-e25528842dfb.png">

👉 If you're not using minkube with Lens and mainstream Kubernetes with CN2, refer to your Kubernetes distribution or managed service for the kubeconfig file, which you can save locally and specify or insert directly into Lens `paste as text` tab.



## Installing Prometheus and Grafana to populate the Lens GUI

Even if you already have Prometheus installed into a Kubernetes cluster, this will add a new instance in a Contrail namespace with just the right settings that the Contrail Lens extension will work automatically without any extra configuration tweaks. However, for best results, we recommend installing using this method into a fresh cluster without other Prometheus instances, and certainly none that Lens is configured to use.

1. Installing Prometheus into the cluster, most easily you’ll need helm on your local system. On Mac OS, you can install helm with:
>
```
brew install helm
```

And if you have already installed helm, then it's a good idea to update: 
>
```
brew upgrade helm
```

2. [Download](https://support.juniper.net/support/downloads/?p=contrail-networking) and install the Contrail analytics "contrail-analytics*.tgz" package and install it in helm. 

👉 The full analytics package can be installed following the [official Juniper documentation](https://www.juniper.net/documentation/us/en/software/cn-cloud-native22/cn-cloud-native-k8s-install-and-lcm/cn-cloud-native-upstream-install-and-lcm/topics/task/cn-cloud-native-k8s-install-analytics.html). Herein, we will turn off influxDB and Emissary Ingress. 
>
```
helm install analytics contrail-analytics-22.1.0.93.tgz -n contrail-analytics --timeout=8000s -f values.yaml -f image-pull-secret-values.yaml --debug
```
