# Module 6 - Enabling Encryption in Transit with WireGuard

## Installing WireGuard

Before enabling encryption in transit with Calico, you must first install WireGuard. Please refer to the instructions available here for your OS: [Wireguard Installation]<https://www.wireguard.com/install/>

Before enabling the encryption feature, test to ensure that the wireguard module is loaded in the kernel on each of the 2 worker nodes:

Get the first node's nodemane and save it to a variable

```bash
NODE_NAME=$(kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="Hostname")].address}'| awk '{print $1;}')
```

This command starts a privileged container on your node and connects to it over SSH:

```bash
kubectl debug node/$NODE_NAME -it --image=ubuntu
```

Output will be like:

```bash
Creating debugging pod node-debugger-ip-192-168-4-145.ca-central-1.compute.internal-lx9qd with container debugger on node ip-192-168-4-145.ca-central-1.compute.internal.
If you don't see a command prompt, try pressing enter.
```

Interact with the node session by running chroot /host from the privileged container.

```bash
root@ip-192-168-4-145:/# chroot /host
sh-4.2#
```

Run the following command:

```bash
lsmod | grep wireguard
```

The output should look something like this:

```bash
wireguard             217088  0
curve25519_x86_64      36864  1 wireguard
libcurve25519_generic    49152  2 curve25519_x86_64,wireguard
libchacha20poly1305    16384  1 wireguard
ip6_udp_tunnel         16384  1 wireguard
udp_tunnel             28672  1 wireguard
```

Repeat the process for the other worker nodes if you wish.

Now that we've verified that the wireguard module is enabled on the worker nodes we can enable encrpytion through the `Felixconfiguration` in Calico.

## Enable End to End Encryption

To enable end-to-end encryption, we will patch the `Felixconfiguration` with the `wireguardEnabled` option. This enables Wireguard tunnels to get created on all the worker nodes in the cluster thus securing all inter-node pod-to-pod traffic by default.

```bash
kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"wireguardEnabled":true}}'
```

To validate, we will need to check the node status for Wireguard entries with the following command:

```bash
kubectl get nodes -o yaml | grep 'kubernetes.io/hostname\|Wireguard'
```

Which will give us the following output showing the nodes Wireguard public key

```bash
      projectcalico.org/WireguardPublicKey: 5qukt1/mog1Eq3B3R1AdwZMbrYTgxtseR4doUxFbckY=
      kubernetes.io/hostname: aks-nodepool1-37054976-vmss000002
      projectcalico.org/WireguardPublicKey: Ib4VKvg3cXgA59v66Q2q/+f5VQ9ub7PCj8RPyQsvfDg=
      kubernetes.io/hostname: aks-nodepool1-37054976-vmss000003
```

On each node we can also view the new interface created by Wireguard:

```bash
root@ip-192-168-4-145:/# ip a | grep wireguard
wireguard.cali: flags=209<UP,POINTOPOINT,RUNNING,NOARP>  mtu 1340
```

The Calico Cloud Dashboard should now show the Wireguard stastics.

### Reference Documentation

[Encrypt Data in Transit](https://docs.tigera.io/compliance/encrypt-cluster-pod-traffic)

[Install Wireguard](https://www.wireguard.com/install/)

[:arrow_right: Module 7 - Clean Up](module-7-clean-up.md)  

[:arrow_left: Module 5 - Ingress and Egress access control using NetworkSets](module-5-network-sets.md)  

[:leftwards_arrow_with_hook: Back to Main](../README.md)
