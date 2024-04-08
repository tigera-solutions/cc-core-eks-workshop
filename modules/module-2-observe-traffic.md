# Module 2 - Observe traffic flows in Calico Cloud

## Install sample application stacks

1. Clone this repository into the AWS Cloudshell:

   ```bash
   git clone 
   ```

2. From the cloned directory, execute:

    ```bash
    kubectl apply -f manifests
    ```
  
    (Optional) Also install the metrics-server on EKS to get an idea as to the resource consumption on the cluster

    ```bash
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ```

    Connect to Calico Cloud GUI. From the menu select `Service Graph > Default`. Explore the options.

3. There are two applications that will get installed, ```Stars``` and ```Cat Facts```

   ```Cat Facts``` is deployed into a single namespace ```catfacts``` and has the following architecture:

   ![catfacts-application](https://github.com/tigera-solutions/cc-aks-zero-trust-workshop/assets/104035488/868c7ccf-e215-41d6-91ab-635832700c50)

   ```Stars``` is deployed across a few namespaces
  
## Check traffic flows

1. Try sending some traffic between the pods

   a. From the ```management-ui``` pod in ```management-ui``` namespace to the ```customer``` pod in the ```yaobank``` namespace  

    ```bash
    kubectl exec -it -n management-ui deploy/management-ui -- sh -c 'curl -m3 -sI http://facts.catfacts 2>/dev/null | grep -i http'
    ```

   The expected result should be ```HTTP/1.0 200 OK``` indicating that the request succeeded between the namespaces

   b. From the service graph and the application, we can also see that there is traffic correctly flowing within each microservice   application pods.

2. Each application also has a webserver with a ```LoadBalancer``` service to access from the internet to check that the application is functioning normally.

   a. Validate and access the ```management-ui``` svc of the Stars application via the ```Loadbalancer``` service in a browser

    ```bash
    kubectl get svc -n management-ui
    ```

    The output gives the external-IP of the AWS LB that can be used to access the svc

    ```bash
    NAME            TYPE           CLUSTER-IP       EXTERNAL-IP                                                                  PORT(S)        AGE
    management-ui   LoadBalancer   10.100.154.186   a2b94c3b1192d490f8c4b1b9caf30589-1684915063.ca-central-1.elb.amazonaws.com   80:31996/TCP   4h48m
    ```

    In a browser, the following should be seen:
    ![stars_gui](https://github.com/tigera-solutions/cc-eks-observability-workshop/assets/117195889/7774d604-361c-4fe9-928f-18b45a4bb948)

   b. Validate and access the ```facts``` svc of the ```Cat Facts``` application via the ```Loadbalancer``` service in a browser

   ```bash
   kubectl get svc -n catfacts facts
   ```

   The output gives the external-IP of the AWS LB that can be used to access the svc

   ```bash
   NAME    TYPE           CLUSTER-IP    EXTERNAL-IP                                                                        PORT(S)        AGE
   facts   LoadBalancer   10.100.1.72   acc9343e00125492dafc74b1de34f7d4-eb6bb4b945442cbb.elb.ca-central-1.amazonaws.com   80:32267/TCP   32m
   ```

[:arrow_right: Module 3 - Zero-Trust Workload Access Control with Namespace Isolation Recommendation](module-3-ztac-ns-isolation.md)  

[:arrow_left: Module 1 - Connect your EKS cluster to Calico Cloud](module-1-connect-calicocloud.md)

[:leftwards_arrow_with_hook: Back to Main](../README.md)  
