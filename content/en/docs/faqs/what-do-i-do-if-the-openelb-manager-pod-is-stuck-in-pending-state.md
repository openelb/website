---
title: "What do I do if the openelb-manager Pod is stuck in pending state?"
linkTitle: "What do I do if the openelb-manager Pod is stuck in pending state?"
weight: 1
---

## Symptom

During the installation of OpenELB, the openelb-manager Pod is stuck in pending state.

```bash
root@node1:~/openelb# kubectl get pod -n openelb-system
NAME                               READY   STATUS      RESTARTS   AGE
openelb-admission-create-m2p52     0/1     Completed   0          30s
openelb-admission-patch-qmvnq      0/1     Completed   0          30s
openelb-manager-74c5467674-pgtmh   0/1     Pending     0          30s
```



## Possible Cause

OpenELB uses port 443 by default. If another component in the system has occupied port 443, the openelb-manager Pod will be stuck in pending state.

## Solution

Perform the following steps to change port 443 of OpenELB. The namespace in the commands is only an example.

1. Run the following command to edit the openelb-manager Deployment:

   ```bash
   kubectl edit deployment openelb-manager -n openelb-system
   ```

2. Change port 443 to a different value (for example, 30443) to avoid the port conflict:

   ```yaml
   spec:
     template:
       spec:
         containers:
         - args:
           - --webhook-port=443 # Change the port number.
           ports:
           - containerPort: 443 # Change the port number.
             hostPort: 443 # Change the port number.
   ```

3. Run the following command to check whether the status of openelb-manager is **READY**: **1/1** and **STATUS**: **Running**. If yes, OpenELB has been installed successfully.

   ```bash
   root@node1:~/openelb# kubectl get pod -n openelb-system
   NAME                               READY   STATUS      RESTARTS   AGE
   openelb-admission-create-m2p52     0/1     Completed   0          7m22s
   openelb-admission-patch-qmvnq      0/1     Completed   0          7m22s
   openelb-manager-74c5467674-pgtmh   1/1     Running     0          30s
   ```