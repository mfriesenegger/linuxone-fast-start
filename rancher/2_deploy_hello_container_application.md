# Deploy a hello containerized application

Lab duration: 5 - 15 minutes

## The goals for the lab

1. Use [Rancher Manager](https://www.rancher.com/products/rancher) to deploy a containerized application.
2. Verify and manage the deployed containerized application.
---

### Prep work

This lab requires a working single node K3s Kubernetes environment running on the IBM LinuxONE Community Cloud.
Follow the [Deploy K3s and Rancher Manager](https://github.com/mfriesenegger/linuxone-fast-start/blob/main/rancher/1_deploy_k3s_and_rancher_manager.md#deploy-k3s-and-rancher-manager) lab guide before starting this lab.

The steps below will 

1. Log into the Rancher Manager web UI.
2. Click on `local` which is the Kubernetes cluster that is available to be managed.

### Use Rancher Manager to deploy a containerized application

This guide offers two methods to deploy a hello containerized application.

- Walk through several screens within the Rancher Manager web UI. 
This method requires no knowledge of Kubernetes. 
This method will take more time to complete. 
Click [here](#method-1-walk-through-several-screens-within-the-rancher-manager-web-ui) to jump to this method.
- Use a Kubernetes YAML file with the Rancher Manager Import YAML UI. 
Basic reading and editing of a Kubernetes YAML file is required. 
This method will take very little time to complete.
Click [here](#method-2-use-a-kubernetes-yaml-file-with-the-rancher-manager-import-yaml-ui) to jump to this method.

#### Method 1: Walk through several screens within the Rancher Manager web UI

1. Click on `Workloads`.
2. Click the `Create` button.
3. Select `Deployment`.
4. Enter the following in the fields.
    - `Name`: s390x-hello
    - `Container Name`: s390x-hello
    - `Container Image`: mfriesenegger/s390x-hello:latest
5. Click `Add Port or Service`.
6. Enter the following in the Add Port or Service fields.
    - `Service Type`: Node Port
    - `Name`: s390x-hello
    - `Private Container Port`: 8080
    - `Listening Port`: 32080
7. Click the `Create` button.
8. Click the `Service Discovery` dropdown.
9. Click `Ingresses`.
10. Click `Create`.
11. Enter the following in the fields.
    - `Name`: s390x-hello
    - `Request Host`: <IP_OF_LINUX_NODE>.sslip.io
        - This is the hostname in the browser URL used to connect to Rancher Manager.
    - `Path`:
        - Prefix
        - /hello
    - `Target Service`: s390x-hello-nodeport
    - `Port`: 8080
12. Click the `Create` button.

Jump to the [Verify and manage the deployed containerized application](#verify-the-deployed-containerized-application) section.

#### Method 2: Use a Kubernetes YAML file with the Rancher Manager Import YAML UI

1.  Copy the following prepared Kubernetes YAML file.

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: s390x-hello
  namespace: default
  labels:
    workload.user.cattle.io/workloadselector: apps.deployment-default-s390x-hello
spec:
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: apps.deployment-default-s390x-hello
  replicas: 1
  template:
    metadata:
      creationTimestamp: null
      labels:
        workload.user.cattle.io/workloadselector: apps.deployment-default-s390x-hello
      namespace: default
    spec:
      containers:
        - image: mfriesenegger/s390x-hello:latest
          imagePullPolicy: Always
          name: s390x-hello
          ports:
            - containerPort: 8080
              name: s390x-hello
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: s390x-hello
  namespace: default
spec:
  ports:
    - name: s390x-hello
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    workload.user.cattle.io/workloadselector: apps.deployment-default-s390x-hello
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: s390x-hello-nodeport
  namespace: default
spec:
  ports:
    - name: s390x-hello
      nodePort: 32080
      port: 8080
      protocol: TCP
      targetPort: 8080
  selector:
    workload.user.cattle.io/workloadselector: apps.deployment-default-s390x-hello
  type: NodePort
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: s390x-hello
  namespace: default
spec:
  ingressClassName: traefik
  rules:
    - host: <IP_OF_LINUX_NODE.sslip.io>
      http:
        paths:
          - backend:
              service:
                name: s390x-hello-nodeport
                port:
                  number: 8080
            path: /hello
            pathType: Prefix
```
2. Click the `Import YAML` icon in the upper right portion of the Manager Manager UI.
3. Paste the YAML file.
4. Replace <IP_OF_LINUX_NODE.sslip.io> where it is the hostname in the browser URL used to connect to Rancher Manager
5. Scroll up and down in the YAML editor verfying no warning icons are to the left of any line.
6. Click the `Import` button.

### Verify the deployed containerized application

1. Open a browser tab.
2. In the browser tab, go to `http://<IP_OF_LINUX_NODE.sslip.io>/hello` where <IP_OF_LINUX_NODE.sslip.io> is the hostname in the browser URL used to connect to Rancher Manager.

You should see something similar to

```
Hello from s390x-hello-6fb94664f7-tjnj7 at [10.42.0.30]!
PRETTY_NAME="SUSE Linux Enterprise Server 15 SP3"
Linux s390x-hello-6fb94664f7-tjnj7 5.14.21-150400.24.33-default #1 SMP Fri Nov 4 13:55:06 UTC 2022 (76cfe60) s390x s390x s390x GNU/Linux
```

CONGRATULATIONS!!  You have successfully

- deployed a hello containerized application, 
- verified the deployed containerized application.

Take some time to explore the Rancher Manager web UI to see additional options available to manage the deployed application.

Use the following steps to delete the application so you try deploying using either [method](#use-rancher-manager-to-deploy-a-containerized-application).

1. Click on the `Workloads` dropdown.
2. Click on `Deployments`.
    1. Checkmark `s390x-hello`.
    2. Click the `Delete` button.
3. Click the `Service Discovery` dropdown.
4. Click `Ingresses`.
    1. Checkmark `s390x-hello`.
    2. Click the `Delete` button.
5. Click `Services`.
    1. Checkmark any services with `s390x-hello` in the name.
    2. Click the `Delete` button.