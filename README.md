<h1 align=center>
Kubevirt Community Stack
</h1>
<p align="center">
  <img width="64" src="https://avatars.githubusercontent.com/u/18700703?s=200&v=4">
</p>
<p align=center>
  Create and manage Kubevirt VMs using Helm & ArgoCD.
  <br>
  <br>
  <a href="https://cloudymax.github.io/kubevirt-community-stack/">cloudymax.github.io/kubevirt-community-stack</a>
</p>
<br>

<h1>Components</h1>

<details>
  <summary>Kubervirt Operator</summary>
  <br>
  The operator controls virtual machine instances and provides the CRDs that define them
</details>

<details>
  <summary>Kubevirt CDI</summary>
  <br>The Containerized Data Importer can pull virtual machine images, ISO files, and other types of bootable media from sources like S3, HTTP, or OCI images. This data is then written to PVCs which are mounted as disks. For examples of various ways to use the CDI, see the notes in the <a href="https://github.com/small-hack/argocd-apps/blob/main/kubevirt/examples/disks/Disks.md">argocd-apps repo</a></br></br>
</details>

<details>
  <summary>Cloud-Init</summary>
  <br>
  The Cloud-init helm chart allows the user to define the specification of a linux-based vm's operating system as code. In addition to basec cloud-init functions, his chart provides some extra functionality via an initjob that makes cloud-init more GitOps friendly. 
<br><br>
Additional Features:

  - Regex values using existing secrets or environmental variables via envsubst
  - Create random user passwords or use an existing secret
  - Download files from a URL
  - Base64 encode + gzip your `write_files` content
  - Populate Wireguard configuration values from an existsing secret
  - Track the total size of user-data and check file for valid syntax
  <br>
</details>

<details>
  <summary>Kubevirt VM</summary>
<br>
</details>

<details>
  <summary>Kubevirt Manager</summary>
  <br>
      This is a community-developed web-ui which allows users to create, manage, and interact with virtual machines running in Kubevirt. See their official docs at <a href="https://kubevirt-manager.io/">kubevirt-manager.io</a></br></br>

<p align="center">
  <a href="https://github.com/cloudymax/kubevirt-community-stack/assets/84841307/eeb87969-4dd6-49ce-b25e-37404e05fa72">
      <img src="https://github.com/cloudymax/kubevirt-community-stack/assets/84841307/eeb87969-4dd6-49ce-b25e-37404e05fa72" alt="Screenshot showing the default page of Kubevirt-manager. The screen is devided into 2 sections. On the left, there is a vertical navigation tab with a grey background. The options in this bar are Dashboard, Virtual Machines, VM Pools, Auto Scaling, Nodes, Data Volumes, Instance Types, and Load Balancers.  On the right, there is a grid of blue rectangular icons each representing one of the option in the navigation tab, but with an icon and text representing metrics about that option." width=500>
  </a>
</p>
<br>
</details>

<details>
  <summary>Cluster API Operator & Kubevirt Provider</summary>
<br>
   <a href="https://cluster-api.sigs.k8s.io/">Cluster API</a> provides a standardised kubernetes-native interface for creating k8s clusters using a wide variety of providers. The <a href="https://cluster-api-operator.sigs.k8s.io/">Cluster API Operator</a> can be installed via Helm and configured to bootstrap the <a href="https://github.com/kubernetes-sigs/cluster-api-provider-kubevirt">Cluster API Kubevirt Provider</a> which allows creating k8s clusters from the CLI or as YAML using Kubevirt VMs.

   Example:

   ```bash
   export CAPK_GUEST_K8S_VERSION="v1.23.10"
   export CRI_PATH="/var/run/containerd/containerd.sock"
   export NODE_VM_IMAGE_TEMPLATE="quay.io/capk/ubuntu-2004-container-disk:${CAPK_GUEST_K8S_VERSION}"

   clusterctl generate cluster capi-quickstart \
   --infrastructure="kubevirt:v0.1.8" \
   --flavor lb \
   --kubernetes-version ${CAPK_GUEST_K8S_VERSION} \
   --control-plane-machine-count=1 \
   --worker-machine-count=1 > capi-quickstart.yaml

   kubectl apply -f capi-quickstart.yaml
   ```
</details>

<h1>Dependencies</h1>

<details>
  <summary>libvirt-clients</summary><br>
This utility will audit a host machine and report what virtualisation capabilities are available

  - Installation
      ```bash
      sudo apt-get install -y libvirt-clients
      ```

  - Usage
      ```console
      $ virt-host-validate qemu
      QEMU: Checking for hardware virtualization          : PASS
      QEMU: Checking if device /dev/kvm exists            : PASS
      QEMU: Checking if device /dev/kvm is accessible     : PASS
      QEMU: Checking if device /dev/vhost-net exists      : PASS
      QEMU: Checking if device /dev/net/tun exists        : PASS
      ```
</details>

<details>
  <summary>virtctl</summary><br>
  virtctl is the command-line utility for managing Kubevirt resources. It can be installed as a standalone CLI or as a Kubectl plugin via krew.
    
  - Standalone
      ```bash
      export VERSION=v0.41.0
      wget https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-linux-amd64
      ```

  - Plugin
      ```bash
      kubectl krew install virt
      ```
</details>

<details>
  <summary>clusterctl</summary><br>
  The clusterctl CLI tool handles the lifecycle of a Cluster API management cluster.
    
  ```bash
  curl -L https://github.com/kubernetes-sigs/cluster-api/releases/download/v1.7.2/clusterctl-linux-amd64 -o clusterctl
  sudo install -o root -g root -m 0755 clusterctl /usr/local/bin/clusterctl
  ```
</details>

<h1>
  Install Kubevirt
</h1>

<details>
  <summary>Combined Chart</summary>
<br>

- <a href="https://github.com/cloudymax/kubevirt-charts/blob/main/charts/kubevirt-stack">kubevirt-stack</a>: Installs the combined chart.

    ```bash
    helm repo add kubevirt https://cloudymax.github.io/kubevirt-community-stack
    helm install kubevirt-stack kubevirt/kubevirt-stack \
      --namespace kubevirt \
      --create-namespace
    ```
</details>

<details>
  <summary>Individual Charts</summary>
<br>

- <a href="https://github.com/cloudymax/kubevirt-community-stack/blob/main/charts/kubevirt">kubevirt</a>: Installs the Kubevirt Operator.

    ```bash
    helm repo add kubevirt https://cloudymax.github.io/kubevirt-community-stack
    helm install kubevirt kubevirt/kubevirt \
      --namespace kubevirt \
      --create-namespace
    ```

- <a href="https://github.com/cloudymax/kubevirt-community-stack/blob/main/charts/cluster-api-operator">Cluster API Operator</a>: Installs the Cluster API Operator.

    ```bash
    Work in progress.
    ```

- <a href="https://github.com/cloudymax/kubevirt-community-stack/blob/main/charts/kubevirt-cdi">kubevirt-cdi</a>: Install the Containerized Data Importer.

    ```bash
    helm repo add kubevirt https://cloudymax.github.io/kubevirt-community-stack
    helm install kubevirt-cdi kubevirt/kubevirt-cdi \
      --namespace cdi \
      --create-namespace
    ```

- <a href="https://github.com/cloudymax/kubevirt-community-stack/blob/main/charts/kubevirt-manager">kubevirt-manager</a>: Deploy the Kubevirt-Manager UI

    ```bash
    # Customize your own values.yaml before deploying
    helm repo add kubevirt https://cloudymax.github.io/kubevirt-charts
    helm install kubevirt-manager kubevirt/kubevirt-manager \
      --fnamespace kubevirt-manager \
      --create-namespace
    ```
</details>

<h1>
  Create VMs
</h1>

- <a href="https://github.com/cloudymax/kubevirt-community-stack/blob/main/charts/kubevirt-vm">kubevirt-vm</a>: Installs the Kubevirt Operator.

    ```bash
    helm repo add kubevirt https://cloudymax.github.io/kubevirt-community-stack
    helm install my-vm kubevirt/kubevirt-vm \
      --namespace kubevirt \
      --set virtualMachine.name=my-vm
      --create-namespace
    ```

## Uninstall

In the event that Kubevirt does not uninstall gracefully, you may need to perform the following steps:

```bash
export RELEASE=v0.17.0

# --wait=true should anyway be default
kubectl delete -n kubevirt kubevirt kubevirt --wait=true

# this needs to be deleted to avoid stuck terminating namespaces
kubectl delete apiservices v1.subresources.kubevirt.io

# not blocking but would be left over
kubectl delete mutatingwebhookconfigurations virt-api-mutator

# not blocking but would be left over
kubectl delete validatingwebhookconfigurations virt-operator-validator

# not blocking but would be left over
kubectl delete validatingwebhookconfigurations virt-api-validator

kubectl delete -f https://github.com/kubevirt/kubevirt/releases/download/${RELEASE}/kubevirt-operator.yaml --wait=false

# Find hanging resources
kubectl api-resources --verbs=list --namespaced -o name   | xargs -n 1 kubectl get --show-kind --ignore-not-found -n kubevirt

# If namespace is stuck
kubectl get namespace "kubevirt" -o json   | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/"   | kubectl replace --raw /api/v1/namespaces/kubevirt/finalize -f -
```
