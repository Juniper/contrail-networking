# Augmenting Visibility with Lens
[Lens](https://k8slens.dev/) is an open multi-cluster GUI for Kubernetes, augmented further with the Prometheus metrics “built-in” (a Prometheus pack). We have developed an extension for CN2 that will allow Contrial users to gain visibility and management of CN2 network functionality directly from Lens.

**👉 The CN2 extension for Lens is tech preview at this time.**

## Getting Started with Lens

1. Let’s install Lens and the stack including Prometheus and Grafana.
Lens is a desktop application. To install the latest version on your machine, downloading it from the [Lens app repository](https://github.com/lensapp/lens/tags) (this is source code only recently and you may need to ask the Lens community for a specific download link) for a specific version like [Lens v5.4.4](https://api.k8slens.dev/binaries/Lens-5.4.4-latest.20220324.1.dmg) for Mac OS which is tested and supported with the CN2 extension. 

2. When you open Lens for the first time, there will be no clusters. It will also ask you to auto-upgrade. Do not do that. You need to add clusters, but first, let’s add the CN2 extension.

<img width="1431" alt="Screen Shot 2022-05-24 at 8 13 42 PM" src="https://user-images.githubusercontent.com/34732034/170172235-1683ca57-ff4f-452e-94c1-9bd2825c3888.png">

To install an extension, open the extensions manager by selecting it from the dropdown in the menubar.
