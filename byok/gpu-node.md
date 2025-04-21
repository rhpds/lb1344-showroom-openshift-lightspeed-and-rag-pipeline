---
title: ITOCP-AI | Adding GPU Node Pool to AI Clusters
---

# Adding a GPU Node Pool to AI Clusters

## Overview

This HowTo guide explains the process of adding Node/Machine Pools for new GPU Nodes in the ITOCP AI clusters.

## Prerequisites

- `Cluster editor` Role under `OCM Roles and Access` at console.redhat.com/openshift
- A ticket capturing the request
  - Budget approval is required from both customer and provider if instance size is larger than `g5.4xlarge` and needs to persist indefinitely.
- Repo Editor permission on https://gitlab.acme.com/cloud-openshift
- If using `rosa` cli tool, make sure to setup the AWS and Rosa tools.  Details on how to do this located here: https://gitlab.acme.com/cloud-openshift/documentation/.  You will also need access to the relevant AWS accounts.

### How To Add a New GPU Node Pool

1. First, a daemonsets toleration in the nvidia gpu operators `clusterpolicy` needs to be added, which will ensure that the new nodes install the necessary media drivers.
    1.  Edit the ai clusters policy, located in the plaform-management repo under `overlays/$CLUSTER_NAME/ai/nvidia-gpu-operator/clusterpolicy.yaml`
    2.  Add a new toleration under spec > daemonsets > tolerations.  For example:
        ```
        - effect: NoSchedule
          key: tenant
          value: example
        ```
2. Once the above change is merged, you can create the new node pool
    1. go to https://console.redhat.com/openshift
    2. Filter for the ai cluster you wish to edit.  For example, `stc-ai-e1-dev`
    3. Once in the cluster, select the `Machine pools` tab
    4. Click the button that says `Add machine pool`
    5. Provide the following information:
        1. Machine pool name - generally, we use the name worker-$INSTANCETYPE
        2. Compute node instance type - See prerequisites to verify manager approval isn't needed
        3. Compute node count - this is per zone, for every node you add you need to multiply by x3
        4. Edit node labels and taints - Here is where you'll specify the taint you used in the toleration specified in the `clusterpolicy`.  You may also want to provide a label in case the customer wants to use NodeSelectors.
        5. Click the `Add machine pool` button
3. The new nodes should now be spinning up in the AI cluster.  Log into the cluster, and verify that they are building.
4. Once the new nodes are ready, update the Ticket request and notify the customer of which Toleration and Label they need to use in order to land pods on the new nodes.
5. If the new node pool has a decom date, keep the ticket open in `pending change` state, until the new nodes have been cleaned up.  If the nodes are permanent, the ticket can be closed.

### How To Add a Single Node only

Using the process above, you will get 1 node per zone, which can get expensive.  In order to save in cost, often you may need to ignore zoning and just spin up a single node.  To do this follow these steps.

1.  Overall, the process is nearly identical to the one above.  However, when it comes to actually creating the node pool, you will now be using `rosa` cli rather than using https://console.redhat.com/openshift.  
    1. log into aws with `aws-saml`
    1. log into rosa `rosa login`
1. Create the new machinepool. For example, adding one g5 node in the dev cluster would look like this:
   ```
   rosa create machinepool \
   --cluster=stc-ai-e1-dev \
   --name=<name of the node pool> \
   --replicas=1 \
   --instance-type=g5.4xlarge \
   --disk-size=300GiB \
   --availability-zone=us-east-1a
   ```

## Links

- Openshift Console: https://console.redhat.com/openshift
- Platform Management Repo: https://gitlab.acme.com/cloud-openshift
- AWS instance-type g5 chart reference: https://aws.amazon.com/ec2/instance-types/g5/
- AWS A10 chart reference: https://aws.amazon.com/blogs/aws/new-ec2-instances-g5-with-nvidia-a10g-tensor-core-gpus/
