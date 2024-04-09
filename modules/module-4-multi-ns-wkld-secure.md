# Module 4 - Workload Isolation with Microsegmentation

Calico provides methods to enable fine-grained access controls between your microservices and external databases, cloud services, APIs, and other applications by using the workload access control with namespace isolation recommendation, as learned in the previous module.

If you need more restrictive policies, you can enforce controls on a fine-grained, per-pod basis using workload isolation with microsegmentation. This is particularly useful for applications spread across multiple namespaces.

Let's use the `Stars` application to demosntrate how to implement the microsegmentation by creating policies for each of the workloads that exist across different namespaces.

## Security Policies

Calico Security Policies provide a richer set of policy capabilities than the native Kubernetes network policies, including:  

- Policies that can be applied to any endpoint: pods/containers, VMs, and/or to host interfaces
- Policies that can define rules that apply to ingress, egress, or both
- Policy rules support:
  - Actions: allow, deny, log, pass
  - Source and destination match criteria:
    - Ports: numbered, ports in a range, and Kubernetes named ports
    - Protocols: TCP, UDP, ICMP, SCTP, UDPlite, ICMPv6, protocol numbers (1-255)
    - HTTP attributes
    - ICMP attributes
    - IP version (IPv4, IPv6)
    - IP or CIDR
    - Endpoint selectors (using label expression to select pods, VMs, host interfaces, and/or network sets)
    - Namespace selectors
    - Service account selectors

### Creating the policies

1. Let's use the Calico Cloud UI to create a policy manually to microsegment the traffic to/from the ```management-ui``` as an example:

   - Go to the `Policies Board`
   - On the bottom of the tier box `platform` click on `Add Policy`
     - In the `Create Policy` page enter the policy name: `management-ui`
     - Change the `Scope` from `Global` to `Namespace` and select the namespace `management-ui`
     - On the `Applies To` section, click `Add Label Selector`
       - Select Key... `projectcalico.org/namespace`
       - =
       - Select Value... `management-ui`
       - Click on `SAVE LABEL SELECTOR`
     - On the field `Type` select the checkbox for Ingress.
     - Click on `Add Ingress Rule`
       - On the `Create New Policy Rule` window,
         - Click on the `dropdown` with `Any Protocol`
         - Change the radio button to `Protocol is` and select `TCP`
         - In the field `To:` click on `Add Port`
         - `Port is` 9001 - Save
         - In the field `From:`, under `Namespace Selectors` select the `Namespaced` radio button, then `Add Label Selector`
           - Select Key... `projectcalico.org/name`
           - =
           - Select Value... `stars`
           - Click on `SAVE LABEL SELECTOR`  
           - On `OR`, click `Add Label Selector`
           - Select Key... `projectcalico.org/name`
           - =
           - Select Value... `client`
           - Click on `SAVE LABEL SELECTOR`
         - Click on the button `Save Rule`
     - Click on `Add Egress Rule`
       - On the `Create New Policy Rule` window,
         - Click on the `dropdown` with `Any Protocol`
         - Change the radio button to `Protocol is` and select `TCP`
         - In the field `To:` click on `Add Port`
         - `Port is` 9000 - Save
         - In the field `From:`, under `Namespace Selectors` select the `Namespaced` radio button, then `Add Label Selector`
           - Select Key... `projectcalico.org/name`
           - =
           - Select Value... `client`
           - Click on `SAVE LABEL SELECTOR`
         - Click on the button `Save Rule`
     - Click on `Add Egress Rule`
       - On the `Create New Policy Rule` window,
         - Click on the `dropdown` with `Any Protocol`
         - Change the radio button to `Protocol is` and select `TCP`
         - In the field `To:` click on `Add Port`
         - `Port is` 80 - Save
         - Click on `Add Port` again
         - `Port is` 6379 - Save
         - In the field `From:`, under `Namespace Selectors` select the `Namespaced` radio button, then `Add Label Selector`
           - Select Key... `projectcalico.org/name`
           - =
           - Select Value... `stars`
           - Click on `SAVE LABEL SELECTOR`
         - Click on the button `Save Rule`
     - You are done. Click `Enforce` on the top-right of your page.

2. Now, let's use the `Recommend a Policy` feature to create the policies for the other workloads.

   Let's start with the `client` workload/namespace.
  
3. Next create fine-grained microsegmentation policies for the `stars` namespace and the ```frontend``` and ```backend``` workloads from a YAML file.

```yaml
kubectl apply -f - <<EOF
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: platform.frontend
  namespace: stars
spec:
  tier: platform
  order: 20
  selector: role == "frontend"
  ingress:
    - action: Allow
      protocol: TCP
      source:
        selector: role == "backend"
      destination:
        ports:
          - '80'
    - action: Allow
      protocol: TCP
      source:
        selector: role == "frontend"
      destination:
        ports:
          - '80'
    - action: Allow
      protocol: TCP
      source:
        selector: role == "client"
        namespaceSelector: projectcalico.org/name == "client"
      destination:
        ports:
          - '80'
    - action: Allow
      protocol: TCP
      source:
        selector: role == "management-ui"
        namespaceSelector: projectcalico.org/name == "management-ui"
      destination:
        ports:
          - '80'
  egress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        selector: role == "backend"
        ports:
          - '6379'
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        selector: role == "frontend"
        ports:
          - '80'
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        selector: role == "client"
        namespaceSelector: projectcalico.org/name == "client"
        ports:
          - '9000'
  types:
    - Ingress
    - Egress
---
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: platform.backend
  namespace: stars
spec:
  tier: platform
  order: 20
  selector: role == "backend"
  ingress:
    - action: Allow
      protocol: TCP
      source:
        selector: role == "frontend"
      destination:
        ports:
          - '6379'
    - action: Allow
      protocol: TCP
      source:
        selector: role == "backend"
      destination:
        ports:
          - '6379'
    - action: Allow
      protocol: TCP
      source:
        selector: role == "client"
        namespaceSelector: projectcalico.org/name == "client"
      destination:
        ports:
          - '6379'
    - action: Allow
      protocol: TCP
      source:
        selector: role == "management-ui"
        namespaceSelector: projectcalico.org/name == "management-ui"
      destination:
        ports:
          - '6379'
  egress:
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        selector: role == "backend"
        ports:
          - '6379'
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        selector: role == "frontend"
        ports:
          - '80'
    - action: Allow
      protocol: TCP
      source: {}
      destination:
        selector: role == "client"
        namespaceSelector: projectcalico.org/name == "client"
        ports:
          - '9000'
  types:
    - Ingress
    - Egress
EOF
```

If you create all the policies correctly, at some point, you will start seeing zero traffic being denied by your default-deny staged policy. At that point, you can go ahead and enforce the default-deny policy.

> [!TIP]
>
> - After enforcing the default-deny policy, if you need to troubleshoot, use the Service Graph or Kibana to see what traffic is being blocked.

### Bonus - About Tiers

Tiers are a hierarchical construct used to group policies and enforce higher-precedence policies that other teams cannot circumvent, providing the basis for **Identity-aware micro-segmentation**.

All Calico and Kubernetes security policies reside in tiers. You can start “thinking in tiers” by grouping your teams and the types of policies within each group, such as security, platform, etc.

Policies are processed in sequential order from top to bottom.

![policy-processing](https://user-images.githubusercontent.com/104035488/206433417-0d186664-1514-41cc-80d2-17ed0d20a2f4.png)

Two mechanisms drive how traffic is processed across tiered policies:

- Labels and selectors
- Policy action rules

For more information about tiers, please refer to the Calico Cloud documentation [Understanding policy tiers](https://docs.calicocloud.io/get-started/tutorials/policy-tiers)

---

[:arrow_right: Module 5 - Ingress and Egress access control using NetworkSets](module-5-network-sets.md)  

[:arrow_left: Module 3 - Zero-Trust Workload Access Control with Namespace Isolation Recommendation](module-3-ztac-ns-isolation.md)  
[:leftwards_arrow_with_hook: Back to Main](../README.md)  
