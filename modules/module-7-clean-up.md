# Module 7 - Clean up

1. Delete the example application stacks.

   ```bash
   kubectl delete -f manifests/01-stars.yaml
   kubectl delete -f manifests/40-catfacts.yaml
   ```

2. Delete EKS cluster.

   ```bash
   source ~/workshopvars.env
   eksctl delete cluster --name $CLUSTERNAME --region $REGION
   ```

3. Delete this repo

   ```bash
   cd .. && rm -Rf cc-eks-zero-trust-workshop
   ```

4. Delete environment variables backup file.

   ```bash
   rm ~/workshopvars.env
   ```

---

[:arrow_left: Module 6 - Enabling Encryption in Transit with WireGuard](module-6-encryption-in-transit.md)  
[:leftwards_arrow_with_hook: Back to Main](../README.md)
