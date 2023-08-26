# Deploy K3s and Rancher Manager

Lab duration: 20 - 30 minutes

## The goals for the lab

1. Use [Rancher K3s](https://www.rancher.com/products/k3s) to deploy a single node Kubernetes environment.
2. Deploy a containerized application, [Rancher Manager](https://www.rancher.com/products/rancher), which provides a web based way of managing Kubernetes clusters.
---

### Prep work

Use any of the K3s supported Linux distributions available in the [IBM LinuxONE Community Cloud](https://developer.ibm.com/articles/get-started-with-ibm-linuxone/).
The following steps will guide you to find the K3s version that will be used and the supported Linux distributions for the K3s version.

1. To find the highest version of K3s that is supported with the latest version of Rancher Manager.

    1. Go to the [latest Rancher Manger version](https://www.suse.com/suse-rancher/support-matrix/).
    2. Find the K3s row in the *Supported Kubernetes Platforms for Rancher Manager* section which is at the top of the page.
    3. Make note of the K3s version listed in the column labeled *Highest Version certified on*.

2. Use the K3s version identified from the previous set of steps to find the K3s supported Linux distributions.

    1. Find K3s by scrolling down on the left under *Support Matrix*.
    2. Expand the list of K3s versions and select the version identified previously.
    3. Find your Linux distro of choice and make note of the K3s supported Linux distro versions.

Deploy your Linux distro of choice by following several articles in the [LinuxONE Basics Hands on Lab](https://github.com/jacobemery/linux1-lab) from the [Fast Start Guides for the LinuxONE Community Cloud](https://www.ibm.com/community/z/linuxone-cc/faststart/).

1. If you have not registered for a LinuxONE Community Cloud trial account then follow [Registering for IBM LinuxONE Community Cloud Account](https://github.com/jacobemery/linux1-lab/blob/general/instructions/1_register.md#registering-for-ibm-linuxone-community-cloud-account).

2. Provision a Linux distro of your choice by following [Provisioning your IBM LinuxONE Virtual Server](https://github.com/jacobemery/linux1-lab/blob/general/instructions/2_provision.md#provisioning-your-ibm-linuxone-virtual-server).

    **SUGGESTION:** Name the instance *k3s_rancher*.

3. Use ssh from [Windows](https://github.com/jacobemery/linux1-lab/blob/general/instructions/3_windows_connect.md#connecting-to-your-server---windows), [Mac](https://github.com/jacobemery/linux1-lab/blob/general/instructions/3_mac_connect.md#connecting-to-your-server---mac) or **Linux** to log into your LinuxONE Community Cloud instance.

### Use Rancher K3s to deploy a single node Kubernetes environment

Let's deploy K3s using the steps below which are from [Quick-Start Guide](https://docs.k3s.io/quick-start) with a couple of [configuration options](https://docs.k3s.io/cli/server#admin-kubeconfig-options). 
**NOTE:** Any Linux distro specific information will be a note specific to the the distro and version below each step.

1. Execute the following command to deploy the single node K3s Kubernetes cluster. Replace *<HIGHEST_K3S_VERSION_SUPPORTED>* with the version that was noted earlier.

    ```
    sudo curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL="<HIGHEST_K3S_VERSION_SUPPORTED>" K3S_KUBECONFIG_MODE="644" sh -
    ```

    For example:

    `sudo curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL="v1.26" K3S_KUBECONFIG_MODE="644" sh -`

    **RHEL9.1:** Execute `sudo yum makecache` before deploying K3s with the command above.


2. Verify that the K3s Kubernetes cluster is active.

    ```
    sudo systemctl status k3s
    ```

3. Confirm the Kubernetes cluster is running. It may take one to two minutes for the Kubernetes cluster to fully configure itself.
    
    ```
    kubectl get nodes
    ```

    Continue waiting when the following message is reported because Kubernetes is not fully configured.

    `E0826 16:36:38.189678   13687 memcache.go:287] couldn't get resource list for metrics.k8s.io/v1beta1: the server is currently unable to handle the request`

4. View the namespaces and pods that have been deployed. It may take one to two minutes for the Kubernetes cluster to fully configure itself.

    ```
    kubectl get namespaces
    kubectl get pods -A
    ```

### Deploy a containerized application, Rancher Manager, which provides a web based way of managing Kubernetes clusters

Let's deploy Rancher Manager using the steps below which are from [Install Rancher with Helm](https://ranchermanager.docs.rancher.com/getting-started/quick-start-guides/deploy-rancher-manager/helm-cli#install-rancher-with-helm).

1. Helm is the package manager for Kubernetes and will be used in the following steps. Deploy helm with the following command. Any WARNING messages can be ignored.

    ```
    sudo curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    ```

2. Use the following commands to deploy the prerequisites for Rancher Manager.

    ```
    mkdir -p $HOME/.kube

    cp /etc/rancher/k3s/k3s.yaml $HOME/.kube/config

    chmod 600 $HOME/.kube/config

    helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

    kubectl create namespace cattle-system

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.crds.yaml

    helm repo add jetstack https://charts.jetstack.io

    helm repo update

    helm install cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --create-namespace \
    --version v1.11.0
    ```

3. Use the following `ip` command and make note of the IP address of your LinuxONE instance which will be used in the following step.

    **SLES15SP4:** `ip a show dev eth1000`
    
    **RHEL9.1:** `ip a show dev enc1000`

    **ubuntu22.04:** `ip a show dev enc1000`

4. Deploy Rancher Manager using the following command. Replace <IP_OF_LINUX_NODE> with what was noted in the previous step.

    ```
    helm install rancher rancher-latest/rancher \
    --namespace cattle-system \
    --set hostname=<IP_OF_LINUX_NODE>.sslip.io \
    --set replicas=1
    ```

    Rancher Manager will take one to two minutes to fully initialize.

5. Find, copy and execute the command in the output from deploying Rancher.  The command will look similar to what is shown below.

    `echo https://<IP_OF_LINUX_NODE>.sslip.io/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')`

    Continue waiting when the following message is reported because Rancher Manager is not fully deployed.

    `Error from server (NotFound): secrets "bootstrap-secret" not found`

6. Copy the URL from the output of the above comand that will be used to access the Rancher Manager Web UI. The output will look similar to:

    `https://<IP_OF_LINUX_NODE>.sslip.io/dashboard/?setup=bvflm5ngp49g5qq6t8tn8lgvhphz622w8jz6bp8tdjsm6dtrtrlsh2`

7. Open a local web browser and paste the URL copied from the previous step.  The self-signed certificate will generate a browser warning which can be ignored.

8. Set a password leaving the other settings unchanged. Accept the End User License Agreement & Terms & Conditions by clicking the checkbox.  Press the Continue button.

CONGRATULATIONS!!  You have successfully

- deployed a K3s single node Kubernetes environment, 
- deployed the containerized Rancher Manager Kubernetes management application,
- accessed the Rancher Manager web UI.  

If you logout and want to access the web UI, go to *https://<IP_OF_LINUX_NODE>.sslip.io* and login as admin with the password that you chose.