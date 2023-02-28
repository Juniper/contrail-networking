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
*	You need enterprise-hub.juniper.net credentials for CN2. If you do not have credentials, fill out the [free-trial form](https://www.juniper.net/us/en/forms/cn2-free-trial.html) using your company email 📧 to gain access

## Getting Started 

1.	Install the latest [hyperkit](https://minikube.sigs.k8s.io/docs/drivers/hyperkit/) as the container/vmm:   
> 
```
brew install hyperkit
```
>
2.	Download and install the minikube binary supported with CN2. In CN2 22.1 minikube v1.22 is required.  Downloads are available [here](https://github.com/kubernetes/minikube/releases/tag/v1.22.0), and [minikube-darwin-amd64](https://github.com/kubernetes/minikube/releases/download/v1.22.0/minikube-darwin-amd64) would be the right v1.22 download for Macbooks with the intel i7 CPU. 

**👉 M1 ARM chips won’t support CN2 at this time.**

If the downloaded file is indeed minikube-darwin-amd64, then to install it, use the command: 
> 
```
sudo install minikube-darwin-amd64 /usr/local/bin/minikube 
minikube version
```

```The first command will require your local password. The second `minikube version` command will usually flag a security warning on your Macbook. You may    see an OSX pop-up error because you downloaded this software outside the App store. To allow it, open the menu _“<Apple> / System Preferences /            Security & Privacy”_ and go to the General tab. You should see a warning about   minikube, and a button to allow it. Click that and enter your password.    After this, re-run the `minikube version` command. Ensure it runs and presents the version.```

3. Next we will first bring up minikube without the CN2 cni. The goal of this step is to get into the minikube hyperkit shell and use the docker inside        that shell to get the docker registry credentials from https://enterprise-hub.juniper.net. The reason for using the docker installed on minikube            hyperkit VM to fetch registry authentication token and not from the docker installed on the MacOS is because with MacOS the docker registry token is        stored in the OSX keychain by default and this can create issues. Please follow the sub-steps as follows to fetch the docker registry token required to    fetch CN2 docker containers.

   Please note that ❯ is the prompt on the MACOS and $ is the prompt inside minikube VM.

```
    3.1 Git clone the CN2 deployer github repo and navigate to the minikube deployer folder for 22.4 release.
        
        ❯ git clone https://github.com/Juniper/contrail-networking.git
        ❯ cd contrail-networking/releases/22.4/minikube                <<< This will be our working directory.
        ❯ ls
          deployer.yaml
        
    
    3.2 Now bring up minikube with the default cni ( bridge ). Run the below command and wait for minikube to come up.
   
        ❯ minikube start --driver hyperkit --container-runtime cri-o --memory 7g --kubernetes-version stable –force
    
    3.3 Login to the minikube vm and from there authenticate with https://enterprise-hub.juniper.net docker registry using the username and the   
        access_token received in the email.
        
        ❯ minikube ssh
        
        $ docker login enterprise-hub.juniper.net <<< This command should be run inside the minikube VM
          Username: << your email id used for login >>
          Password:                        <<< Paste the access-token recieved in the email from enterprise-hub-util@juniper.net
          WARNING! Your password will be stored unencrypted in /home/docker/.docker/config.json.
          Configure a credential helper to remove this warning. See
          https://docs.docker.com/engine/reference/commandline/login/#credentials-store

          Login Succeeded
        $
        
     3.4 Log out of the minikube and on the local MacOS machine copy the ~/.docker/config.json from the minikube VM to the local MacOS working directory.
     
         ❯ sudo scp -i $(minikube ssh-key) docker@$(minikube ip):~/.docker/config.json.

           Password:
           The authenticity of host '192.168.64.11 (192.168.64.11)' can't be established.
           ED25519 key fingerprint is SHA256:upwPzZEyEN8cz3nxndznsq0/16vndbVV0+LdlxWfxjI.
           This key is not known by any other names
           Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
           Warning: Permanently added '192.168.64.11' (ED25519) to the list of known hosts.
           config.json                                                                                                   100% 1190     1.4MB/s   00:00
           
     3.5 Content of config.json should look something like below
     
         ❯ cat config.json 
           {
	           "auths": {
		           "enterprise-hub.juniper.net": {
			            "auth": "<encrypted-auth-key>"
		            }
	           }
          }
          
      3.6 Now do a base64 encoding of the above config.json file which we will provide as a secret to CN2 deployer manifest file in the next step.
      
        ❯ ENCODED_CREDS=$(sudo base64 -i config.json)
        
      3.7 The next step is to replace the occurance of <base64-encoded-credential> in deployer.yaml with the base64 encoded value of the docker registry             authentication credentials.
      
        ❯ sudo sed -i '' s/'<base64-encoded-credential>'/$ENCODED_CREDS/ deployer.yaml
        
      3.8 Now we can destroy the minikube VM so that we can bring up it back up using CN2 as the CNI which is shown in the next step.
      
        ❯ sudo minikube delete --purge   
```


5. Next step is to deploy minikube using CN2 as the CNI as shown below.

```
         ❯ minikube start --driver hyperkit --container-runtime cri-o --memory 7g --cni deployer.yaml --kubernetes-version stable –force
```

  If you intend to later use Lens with Prometheus, Grafana and such, included in the optional [CN2 Analytics](https://support.juniper.net/support/downloads/?p=contrail-networking) package, then please give minikube 7g of memory (or more if deploying Influx and Emissary Ingress also in the analytics helm chart package), but 5g is fine in general.

6. After the containers have been pulled down and started we can look at the container images running in the cluster. 
>
```
kubectl get pods -A

NAMESPACE            NAME                                                     READY   STATUS             RESTARTS   AGE
contrail-deploy      contrail-k8s-deployer-6c5bcd444f-2ntrq                   1/1     Running            0          39m
contrail-system      contrail-k8s-apiserver-cdbb5b9bd-ftw42                   1/1     Running            0          38m
contrail-system      contrail-k8s-controller-76fdd87f4-vn5w7                  1/1     Running            0          38m
contrail             contrail-control-0                                       2/2     Running            0          38m
contrail             contrail-k8s-kubemanager-54b58947c4-wwcw6                1/1     Running            0          38m
contrail             contrail-vrouter-masters-pvlmb                           3/3     Running            0          38m
kube-system          coredns-74ff55c5b-cr7ps                                  1/1     Running            0          39m 
kube-system          etcd-minikube                                            1/1     Running            0          39m
kube-system          kube-apiserver-minikube                                  1/1     Running            0          39m
kube-system          kube-controller-manager-minikube                         1/1     Running            0          39m
kube-system          kube-proxy-mcc2g                                         1/1     Running            0          39m
kube-system          kube-scheduler-minikube                                  1/1     Running            0          39m
kube-system          storage-provisioner                                      1/1     Running            0          39m
```

**👉 Because CoreDNS came up before CN2, it will not be on CN2’s default pod network. Once CN2 is fully running, restart the coreDNS pod. It is easy to kill it in Lens with the trash-can icon. It will automatically respawn and be in the right network.**
  

7. The contrail/CN2 containers can be seen in the contrail-deploy, contrail-system, and contrail namespaces 
>
```
kubectl get pods -A
  
NAMESPACE         NAME                                       READY   STATUS    RESTARTS   AGE
contrail-deploy   contrail-k8s-deployer-6c5bcd444f-h96lv     1/1     Running   0          65m
contrail-system   contrail-k8s-apiserver-7596bcffbc-kc9cb    1/1     Running   0          65m
contrail-system   contrail-k8s-controller-99b8ff694-phv5p    1/1     Running   0          64m
contrail          contrail-control-0                         2/2     Running   0          64m
contrail          contrail-k8s-kubemanager-ccc4dcd66-f4nxx   1/1     Running   0          64m
contrail          contrail-vrouter-masters-485r9             3/3     Running   0          64m
kube-system       coredns-558bd4d5db-2c6n7                   1/1     Running   0          65m
kube-system       etcd-minikube                              1/1     Running   0          65m
kube-system       kube-apiserver-minikube                    1/1     Running   0          65m
kube-system       kube-controller-manager-minikube           1/1     Running   0          65m
kube-system       kube-proxy-xvr46                           1/1     Running   0          65m
kube-system       kube-scheduler-minikube                    1/1     Running   0          65m
kube-system       storage-provisioner                        1/1     Running   0          65m
```


8. Check the APIs and kubectl commands now include the CN2 extensions
>
```
kubectl api-resources

NAME                              SHORTNAMES       APIVERSION                                  NAMESPACED   KIND
bindings                                           v1                                          true         Binding
componentstatuses                 cs               v1                                          false        ComponentStatus
configmaps                        cm               v1                                          true         ConfigMap
endpoints                         ep               v1                                          true         Endpoints
events                            ev               v1                                          true         Event
limitranges                       limits           v1                                          true         LimitRange
namespaces                        ns               v1                                          false        Namespace
nodes                             no               v1                                          false        Node
persistentvolumeclaims            pvc              v1                                          true         PersistentVolumeClaim
persistentvolumes                 pv               v1                                          false        PersistentVolume
pods                              po               v1                                          true         Pod
podtemplates                                       v1                                          true         PodTemplate
replicationcontrollers            rc               v1                                          true         ReplicationController
resourcequotas                    quota            v1                                          true         ResourceQuota
secrets                                            v1                                          true         Secret
serviceaccounts                   sa               v1                                          true         ServiceAccount
services                          svc              v1                                          true         Service
mutatingwebhookconfigurations                      admissionregistration.k8s.io/v1             false        MutatingWebhookConfiguration
validatingwebhookconfigurations                    admissionregistration.k8s.io/v1             false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds         apiextensions.k8s.io/v1                     false        CustomResourceDefinition
apiservices                                        apiregistration.k8s.io/v1                   false        APIService
controllerrevisions                                apps/v1                                     true         ControllerRevision
daemonsets                        ds               apps/v1                                     true         DaemonSet
deployments                       deploy           apps/v1                                     true         Deployment
replicasets                       rs               apps/v1                                     true         ReplicaSet
statefulsets                      sts              apps/v1                                     true         StatefulSet
tokenreviews                                       authentication.k8s.io/v1                    false        TokenReview
localsubjectaccessreviews                          authorization.k8s.io/v1                     true         LocalSubjectAccessReview
selfsubjectaccessreviews                           authorization.k8s.io/v1                     false        SelfSubjectAccessReview
selfsubjectrulesreviews                            authorization.k8s.io/v1                     false        SelfSubjectRulesReview
subjectaccessreviews                               authorization.k8s.io/v1                     false        SubjectAccessReview
horizontalpodautoscalers          hpa              autoscaling/v1                              true         HorizontalPodAutoscaler
cronjobs                          cj               batch/v1                                    true         CronJob
jobs                                               batch/v1                                    true         Job
certificatesigningrequests        csr              certificates.k8s.io/v1                      false        CertificateSigningRequest
apiservers                                         configplane.juniper.net/v1alpha1            true         ApiServer
controllers                                        configplane.juniper.net/v1alpha1            true         Controller
kubemanagers                                       configplane.juniper.net/v1alpha1            true         Kubemanager
controls                                           controlplane.juniper.net/v1alpha1           true         Control
leases                                             coordination.k8s.io/v1                      true         Lease
addressgroups                     ag               core.contrail.juniper.net/v1alpha1          true         AddressGroup
applicationpolicysets             aps              core.contrail.juniper.net/v1alpha1          true         ApplicationPolicySet
bgpasaservices                    bgpaas           core.contrail.juniper.net/v1alpha1          true         BGPAsAService
bgprouters                        br               core.contrail.juniper.net/v1alpha1          true         BGPRouter
firewallpolicies                  fp               core.contrail.juniper.net/v1alpha1          true         FirewallPolicy
firewallrules                     fr               core.contrail.juniper.net/v1alpha1          true         FirewallRule
floatingips                       fip              core.contrail.juniper.net/v1alpha1          false        FloatingIP
globalsystemconfigs               gsc              core.contrail.juniper.net/v1alpha1          false        GlobalSystemConfig
globalvrouterconfigs              gvc              core.contrail.juniper.net/v1alpha1          false        GlobalVrouterConfig
instanceips                       iip              core.contrail.juniper.net/v1alpha1          false        InstanceIP
mirrordestinations                md               core.contrail.juniper.net/v1alpha1          false        MirrorDestination
routetargets                      rt               core.contrail.juniper.net/v1alpha1          false        RouteTarget
routinginstances                  ri               core.contrail.juniper.net/v1alpha1          true         RoutingInstance
subnets                           sn               core.contrail.juniper.net/v1alpha1          true         Subnet
tags                              t                core.contrail.juniper.net/v1alpha1          false        Tag
tagtypes                          tt               core.contrail.juniper.net/v1alpha1          false        TagType
virtualmachineinterfaces          vmi              core.contrail.juniper.net/v1alpha1          true         VirtualMachineInterface
virtualmachines                   vm               core.contrail.juniper.net/v1alpha1          false        VirtualMachine
virtualnetworkrouters             vnr              core.contrail.juniper.net/v1alpha1          true         VirtualNetworkRouter
virtualnetworks                   vn               core.contrail.juniper.net/v1alpha1          true         VirtualNetwork
virtualrouters                    vr               core.contrail.juniper.net/v1alpha1          false        VirtualRouter
vrouters                                           dataplane.juniper.net/v1alpha1              true         Vrouter
endpointslices                                     discovery.k8s.io/v1                         true         EndpointSlice
events                            ev               events.k8s.io/v1                            true         Event
ingresses                         ing              extensions/v1beta1                          true         Ingress
flowschemas                                        flowcontrol.apiserver.k8s.io/v1beta1        false        FlowSchema
prioritylevelconfigurations                        flowcontrol.apiserver.k8s.io/v1beta1        false        PriorityLevelConfiguration
pools                                              idallocator.contrail.juniper.net/v1alpha1   false        Pool
network-attachment-definitions    net-attach-def   k8s.cni.cncf.io/v1                          true         NetworkAttachmentDefinition
ingressclasses                                     networking.k8s.io/v1                        false        IngressClass
ingresses                         ing              networking.k8s.io/v1                        true         Ingress
networkpolicies                   netpol           networking.k8s.io/v1                        true         NetworkPolicy
runtimeclasses                                     node.k8s.io/v1                              false        RuntimeClass
poddisruptionbudgets              pdb              policy/v1                                   true         PodDisruptionBudget
podsecuritypolicies               psp              policy/v1beta1                              false        PodSecurityPolicy
clusterrolebindings                                rbac.authorization.k8s.io/v1                false        ClusterRoleBinding
clusterroles                                       rbac.authorization.k8s.io/v1                false        ClusterRole
rolebindings                                       rbac.authorization.k8s.io/v1                true         RoleBinding
roles                                              rbac.authorization.k8s.io/v1                true         Role
priorityclasses                   pc               scheduling.k8s.io/v1                        false        PriorityClass
csidrivers                                         storage.k8s.io/v1                           false        CSIDriver
csinodes                                           storage.k8s.io/v1                           false        CSINode
csistoragecapacities                               storage.k8s.io/v1beta1                      true         CSIStorageCapacity
storageclasses                    sc               storage.k8s.io/v1                           false        StorageClass
volumeattachments                                  storage.k8s.io/v1                           false        VolumeAttachment
```





  

 



