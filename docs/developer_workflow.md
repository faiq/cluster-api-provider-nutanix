# Developer workflow

## Prerequisites

This project requires the following to be installed on the developer's workstation.

1. [devbox](https://www.jetpack.io/devbox/docs/installing_devbox/)
1. [direnv](https://direnv.net/docs/installation.html)

To install the required packages, please follow the directions provided for your system.

NOTE: The first time you `cd` into the directory, the required dependencies for the project will start to download and will take some time depending on your connection.

1. Download source code:

    ```shell
    git clone https://github.com/nutanix-cloud-native/cluster-api-provider-nutanix.git
    cd cluster-api-provider-nutanix
    ```

1. Build a container image with the source code:

    ```shell
    make docker-build
    ```

## Create a local management cluster

1. Create a [KIND](https://kind.sigs.k8s.io/) cluster:

   ```shell
    make kind-create
    ```

This will configure [kubectl](https://kubernetes.io/docs/reference/kubectl/) for the local cluster.

## Build a CAPI Image for Nutanix

1. Follow the instructions [here](https://image-builder.sigs.k8s.io/capi/providers/nutanix.html#building-capi-images-for-nutanix-cloud-platform-ncp) to build a CAPI image for Nutanix.
   The image name printed at the end of the build will be needed to create a Kubernetes cluster.

## Prepare local clusterctl

1. Make a copy of the [clusterctl](https://cluster-api.sigs.k8s.io/clusterctl/overview) configuration file:

    ```shell
    cp -n clusterctl.yaml.tmpl clusterctl.yaml
    ```

1. Update `./clusterctl.yaml` with appropriate configuration.

1. Setup `clusterctl` with the configuration:

    ```shell
    make prepare-local-clusterctl
    ```

## Deploy cluster-api-provider-nutanix provider and CRDs on local management cluster

1. Deploy the provider, along with CAPI core controllers and CRDs:

    ```shell
    make deploy
    ```

1. Verify the provider Pod is `READY`:

    ```shell
    kubectl get pods -n capx-system
    ```

## Create a test workload cluster without topology

1. Create a workload cluster:

    ```shell
    make test-cluster-create
    ```

   Optionally, to use a unique cluster name:

    ```shell
    make test-cluster-create TEST_CLUSTER_NAME=<>
    ```

1. Get the workload cluster kubeconfig. This will write out the kubeconfig file in the local directory as `<cluster-name>.workload.kubeconfig`:

    ```shell
    make test-kubectl-workload 
    ```

   When using a unique cluster name set `TEST_CLUSTER_NAME` variable:

    ```shell
    make test-kubectl-workload TEST_CLUSTER_NAME=<>
    ```

## Create a test workload cluster with topology

1. Create a workload cluster:

    ```shell
    make test-cc-cluster-create
    ```

   Optionally, to use a unique cluster name:

    ```shell
    make test-cc-cluster-create TEST_TOPOLOGY_CLUSTER_NAME=<>
    ```

## Upgrade test workload cluster's k8s version

1. Upgrade workload cluster's k8s version

    ```shell
    make test-cc-cluster-upgrade TEST_TOPOLOGY_CLUSTER_NAME=<> UPGRADE_K8S_VERSION_TO=<vx.x.x>
    ```


## Debugging failures

1. Check the cluster resources:

    ```shell
    kubectl get cluster-api --namespace capx-test-ns
    ```

1. Check the provider logs:

    ```shell
    kubectl logs -n capx-system -l cluster.x-k8s.io/provider=infrastructure-nutanix
    ```

1. Check status of individual Nodes by using the address from the corresponding `NutanixMachine`:

    ```shell
    ssh capiuser@<address>
    ```

    * Check cloud-init bootstrap logs:

        ```shell
        tail /var/log/cloud-init-output.log
        ```

    * Check journalctl logs for Kubelet and Containerd
    * Check Containerd containers:

        ```shell
        crictl ps -a
        ```

## Cleanup

1. Delete the test workload cluster without topology:

    ```shell
    make test-cluster-delete
    ```

   When using a unique cluster name set `TEST_CLUSTER_NAME` variable:

    ```shell
    make test-cluster-delete TEST_CLUSTER_NAME=<>

1. Delete the test workload cluster with topology:

    ```shell
    make test-cc-cluster-delete
    ```

   When using a unique cluster name set `TEST_CLUSTER_NAME` variable:

    ```shell
    make test-cluster-delete TEST_CLUSTER_NAME=<>

1. Delete the management KIND cluster:

    ```shell
    make kind-delete
    ```
