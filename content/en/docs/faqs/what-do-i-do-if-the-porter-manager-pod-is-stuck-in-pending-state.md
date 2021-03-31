---
title: "What do I do if the porter-manager Pod is stuck in pending state?"
linkTitle: "What do I do if the porter-manager Pod is stuck in pending state?"
weight: 1
---

## Symptom

During the installation of PorterLB, the porter-manager Pod is stuck in pending state.

![porter-manager-pending](/images/en/docs/getting-started/faqs/what-do-i-do-if-the-porter-manager-pod-is-stuck-in-pending-state/porter-manager-pending.jpg)

## Possible Cause

PorterLB uses port 443 by default. If another component in the system has occupied port 443, the porter-manager Pod will be stuck in pending state.

## Solution

Perform the following steps to change port 443 of PorterLB. The namespace in the commands is only an example.

1. Run the following command to edit the porter-manager Deployment:

   ```bash
   kubectl edit deployment porter-manager -n porter-system
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

3. Run the following command to check whether the status of porter-manager is **READY**: **1/1** and **STATUS**: **Running**. If yes, PorterLB has been installed successfully.

   ```bash
   kubectl get po -n porter-system
   ```

   ![verify-porter](/images/en/docs/getting-started/faqs/what-do-i-do-if-the-porter-manager-pod-is-stuck-in-pending-state/verify-porter.jpg)