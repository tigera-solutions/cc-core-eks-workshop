# EKS and Calico Cloud: Hands-on workshop for the Truist Engineering Teams

## Welcome

In this EKS-focused workshop, you will work with AWS and Calico Cloud to learn how to utilize:

- The **Observability** plane of Calico to visualize traffic and help with tracing and troubleshooting:
  - Pod to pod traffic within the cluster
  - Egress traffic from cluster workloads to external IPs/FQDNs
  - Ingress traffic to cluster workloads from external IPs
  - The effects of network policy chain on a workload (once applied), and understand the decision of the path of a packet and why it was ```Allowed``` or ```Denied```
- Calico Cloud offers 3 observability tools, *Service Graph*, *FlowViz* and *Kibana*, that we will look at in more detail in this workshop

- The **Network Security** features of Calico to secure the workload traffic by utilizing:
  - Workload isolation by Kubernetes ```Namespaces```
  - Global threatfeeds to deny traffic to and from malicious IPs external to the cluster
  - Zero-trust/Default-deny network policy posture for cluster workloads to only explicitly allow required traffic and fulfil compliance stadard control criteria requirements (like PCI or SOC2)
  - Using Calico Policy tiers to segregate network policy hierarchy by allowing different teams to have different levels of control over the cluster security posture by responsibility, and prevent misconfigurations
  - Using ```NetworkSets``` to establish scalable workload access controls for egress using FQDN/DNS policy to create an allow-list to access specific 3rd party services

### Time Requirements

A timeslot of 4 hours has been allocated to complete this workshop and leave room for questions and any debugging.

## Workshop Environment Preparation

> [!WARNING]
> **For this workshop, you are expected to have access to a previously created EKS cluster.**

- Please, follow the instructions on the repository below if you don't have it ready:

  [Calico Cloud on EKS - Workshop Environment Preparation](https://github.com/tigera-solutions/eks-workshop-prep.git)

- We will run this workshop from the AWS CloudShell, as described in that repository.

- To start your cluster, we will scale the nodegroup up to 2 nodes using ```eksctl```. Reload the environment variables that were created in your AWS CloudShell first and then scale the nodegroup up.
  
- Ensure the nodegroup variable is populated into the ```workshopvars.env``` file:

   ```bash
   export NGNAME=$(eksctl get nodegroups --cluster $CLUSTERNAME --region $REGION | grep $CLUSTERNAME | awk -F ' ' '{print $2}') && \
   echo export NGNAME=$NGNAME >> ~/workshopvars.env
   ```

- Use the following command:

  ```bash
  source ~/workshopvars.env
  eksctl scale nodegroup $NGNAME \
  --cluster $CLUSTERNAME \
  --region $REGION \
  --nodes 2 \
  --nodes-max 2 \
  --nodes-min 2
  ```

## Modules

This workshop is organized in sequential modules. One module will build up on top of the previous module, so please, follow the order as proposed below.

Module 1 - [Getting Started](modules/module-1-getting-started.md)  
Module 2 - [Deploy an AWS EKS cluster](modules/module-2-deploy-eks.md)  
Module 3 - [Connect the AWS EKS cluster to Calico Cloud](modules/module-3-connect-calicocloud.md)  
Module 4 - [Observe traffic flows in Calico Cloud](modules/module-4-observe-traffic.md)  
Module 5 - [Secure pod traffic using Calico Policy Recommender](modules/module-5-secure-pod-traffic.md)  
Module 6 - [Zero-trust security for pod traffic](modules/module-6-zero-trust-security.md)</br>
Module 7 - [Use Observability to Troubleshoot Connectivity Issues](modules/module-7-troubleshooting.md)</br>
Module 8 - [Clean up](/modules/module-8-clean-up.md)  

---

### Useful links

- [Project Calico](https://www.tigera.io/project-calico/)
- [Calico Academy - Get Calico Certified!](https://academy.tigera.io/)
- [O’REILLY EBOOK: Kubernetes security and observability](https://www.tigera.io/lp/kubernetes-security-and-observability-ebook)
- [Calico Users - Slack](https://slack.projectcalico.org/)

### Follow us on social media

- [LinkedIn](https://www.linkedin.com/company/tigera/)
- [Twitter](https://twitter.com/tigeraio)
- [YouTube](https://www.youtube.com/channel/UC8uN3yhpeBeerGNwDiQbcgw/)
- [Slack](https://calicousers.slack.com/)
- [Github](https://github.com/tigera-solutions/)
- [Discuss](https://discuss.projectcalico.tigera.io/)

> **Note**: The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how Calico Cloud can be configured to build a functional solution. These examples are not intended for use in production environments.
