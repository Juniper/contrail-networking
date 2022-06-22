# ContrailReadiness - CN2 Preflight and Postflight Checks

## Description
A collection of automated checks that validate a k8s environment both before and after CN2 is installed. Preflight checks will be run on a vanilla k8s installation and postflight tests will be run after CN2 is installed on top of k8s.

## Note 
1. Preflight tests are not supported on OpenShift platform
2. Preflight checks need to be run on each cluster in a multi-cluster installation in Release 22.2
3. Postflight checks should only be run on the central cluster in Release 22.2
    * The postflight check ensures resources in `core.contrail.juniper.net/v1alpha1` api group are in success state, none of these resources will be present on distributed clusters

## Run Preflight and Postflight Checks in Release 22.2
1. Apply the ContrailReadiness custom resource definitions
    ```bash
    kubectl apply -f crds
    ```
2. Create the ConfigMap from the deployer manifest that you want to apply to this cluster. Name the ConfigMap deployer-yaml.
    ```bash
    kubectl create configmap deployer-yaml --from-file=<path_to_deployer_manifest>
    ```
3. Create the ContrailReadiness controller. Remember to enter your credentials in the secret in [this manifest](contrailreadiness-controller.yaml).
    ```bash
    kubectl apply -f contrailreadiness-controller.yaml
    ```
    Wait for the controller to start.
4. Run the checks. The checks will run on all nodes by default. Look at [contrailreadiness-postflight.yaml](contrailreadiness-postflight.yaml) for a comment that explains how to customize which nodes to run checks on.
  * To run preflight checks:
      ```bash
      kubectl apply -f contrailreadiness-preflight.yaml
      ```
      You run preflight checks after you create the cluster but before you install Contrail.
  * To run the posflight checks:
      ```bash
      kubectl apply -f contrailreadiness-postflight.yaml
      ```
      You run postflight checks after you install Contrail.
5. Read the check results.
    ```bash
    # To get overall results of a preflight / postflight run
    # get the contrailreadiness resource and look at the status 
    # field of the resource
    kubectl get contrailreadiness preflight -o yaml
    kubectl get contrailreadiness postflight -o yaml
    ```
    ```bash
    # To get detailed results of a particular check
    # get the contrailreadinesstest resource and look at the status 
    # field of the resource
    kubectl get contrailreadinesstest preflight-kernel -o yaml
    kubectl get contrailreadinesstest postflight-contrailresources -oyaml
    ```