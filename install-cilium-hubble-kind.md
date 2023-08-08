# Cluster creation

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  kubeProxyMode: "none"
  podSubnet: "10.244.0.0/16"
  serviceSubnet: "10.96.0.0/12"
nodes:
 - role: control-plane
 - role: worker
 - role: worker
```

```
kind create cluster --name cilium-hubble --config=cluster.yaml
Creating cluster "cilium-hubble" ...
 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-cilium-hubble"
You can now use your cluster with:

kubectl cluster-info --context kind-cilium-hubble

Thanks for using kind! 😊
```

```
kubectl cluster-info --context kind-cilium-hubble
Kubernetes control plane is running at https://127.0.0.1:46033
CoreDNS is running at https://127.0.0.1:46033/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
kubectl get node
NAME                          STATUS     ROLES           AGE   VERSION
cilium-hubble-control-plane   NotReady   control-plane   56s   v1.27.3
cilium-hubble-worker          NotReady   <none>          34s   v1.27.3
cilium-hubble-worker2         NotReady   <none>          34s   v1.27.3
```

# Cilium installation

```
cilium install
🔮 Auto-detected Kubernetes kind: kind
✨ Running "kind" validation checks
✅ Detected kind version "0.20.0"
ℹ️  Using Cilium version 1.14.0
🔮 Auto-detected cluster name: kind-cilium-hubble
ℹ️  kube-proxy-replacement disabled
🔮 Auto-detected datapath mode: tunnel
ℹ️  Detecting real Kubernetes API server addr and port on Kind
🔮 Auto-detected kube-proxy has not been installed
ℹ️  Cilium will fully replace all functionalities of kube-proxy
```

```
cilium status --wait
    /¯¯\
 /¯¯\__/¯¯\    Cilium:             OK
 \__/¯¯\__/    Operator:           OK
 /¯¯\__/¯¯\    Envoy DaemonSet:    disabled (using embedded mode)
 \__/¯¯\__/    Hubble Relay:       disabled
    \__/       ClusterMesh:        disabled

DaemonSet              cilium             Desired: 3, Ready: 3/3, Available: 3/3
Deployment             cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
Containers:            cilium             Running: 3
                       cilium-operator    Running: 1
Cluster Pods:          3/3 managed by Cilium
Helm chart version:    1.14.0
Image versions         cilium             quay.io/cilium/cilium:v1.14.0@sha256:5a94b561f4651fcfd85970a50bc78b201cfbd6e2ab1a03848eab25a82832653a: 3
                       cilium-operator    quay.io/cilium/operator-generic:v1.14.0@sha256:3014d4bcb8352f0ddef90fa3b5eb1bbf179b91024813a90a0066eb4517ba93c9: 1
```

# Hubble installation

```
cilium hubble enable --ui
```

```
kubectl exec -it -n kube-system ds/cilium -c cilium-agent -- cilium status
KVStore:                 Ok   Disabled
Kubernetes:              Ok   1.27 (v1.27.3) [linux/amd64]
Kubernetes APIs:         ["EndpointSliceOrEndpoint", "cilium/v2::CiliumClusterwideNetworkPolicy", "cilium/v2::CiliumEndpoint", "cilium/v2::CiliumNetworkPolicy", "cilium/v2::CiliumNode", "cilium/v2alpha1::CiliumCIDRGroup", "core/v1::Namespace", "core/v1::Pods", "core/v1::Service", "networking.k8s.io/v1::NetworkPolicy"]
KubeProxyReplacement:    Strict   [eth0 172.18.0.3]
Host firewall:           Disabled
CNI Chaining:            none
Cilium:                  Ok   1.14.0 (v1.14.0-b5013e15)
NodeMonitor:             Listening for events on 2 CPUs with 64x4096 of shared memory
Cilium health daemon:    Ok
IPAM:                    IPv4: 5/254 allocated from 10.244.1.0/24,
IPv4 BIG TCP:            Disabled
IPv6 BIG TCP:            Disabled
BandwidthManager:        Disabled
Host Routing:            Legacy
Masquerading:            IPTables [IPv4: Enabled, IPv6: Disabled]
Controller Status:       32/32 healthy
Proxy Status:            OK, ip 10.244.1.89, 0 redirects active on ports 10000-20000, Envoy: embedded
Global Identity Range:   min 256, max 65535
Hubble:                  Ok              Current/Max Flows: 4095/4095 (100.00%), Flows/s: 8.49   Metrics: Disabled
Encryption:              Disabled
Cluster health:          3/3 reachable   (2023-08-08T05:32:26Z)
```

## Hubble ui

```
cilium hubble ui
```

Alternative:
```
kubectl port-forward -n kube-system svc/hubble-ui  --address 0.0.0.0 8080:80
```

=> 127.0.0.1:8080

## Hubble cli

```
cilium hubble port-forward&
```

```
hubble status
Healthcheck (via localhost:4245): Ok
Current/Max Flows: 10,453/12,285 (85.09%)
Flows/s: 12.47
Connected Nodes: 3/3
```

```
hubble observe --last 10
Aug  8 05:27:17.430: 10.244.0.86:52930 (host) -> 10.244.0.81:4240 (health) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:18.195: 10.244.0.86:52106 (remote-node) <> 10.244.2.71:4240 (health) to-overlay FORWARDED (TCP Flags: ACK)
Aug  8 05:27:18.195: 10.244.0.86:44438 (remote-node) <> 10.244.1.61:4240 (health) to-overlay FORWARDED (TCP Flags: ACK)
Aug  8 05:27:26.678: 10.244.2.244:43376 (remote-node) -> 10.244.0.81:4240 (health) to-endpoint FORWARDED (TCP Flags: ACK, PSH)
Aug  8 05:27:26.678: 10.244.2.244:43376 (remote-node) <- 10.244.0.81:4240 (health) to-overlay FORWARDED (TCP Flags: ACK, PSH)
Aug  8 05:27:26.680: 10.244.2.244 (remote-node) -> 10.244.0.81 (health) to-endpoint FORWARDED (ICMPv4 EchoRequest)
Aug  8 05:27:26.680: 10.244.2.244 (remote-node) <- 10.244.0.81 (health) to-overlay FORWARDED (ICMPv4 EchoReply)
Aug  8 05:27:26.692: 10.244.1.89:58734 (remote-node) <- 10.244.0.81:4240 (health) to-overlay FORWARDED (TCP Flags: ACK, PSH)
Aug  8 05:27:26.692: 10.244.1.89 (remote-node) -> 10.244.0.81 (health) to-endpoint FORWARDED (ICMPv4 EchoRequest)
Aug  8 05:27:26.692: 10.244.1.89 (remote-node) <- 10.244.0.81 (health) to-overlay FORWARDED (ICMPv4 EchoReply)
Aug  8 05:27:29.295: 127.0.0.1:34876 (world) <> kube-system/coredns-5d78c9869d-j6772 (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:29.412: 127.0.0.1:8080 (world) <> kube-system/coredns-5d78c9869d-5wcjl (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:29.412: 127.0.0.1:34878 (world) <> kube-system/coredns-5d78c9869d-5wcjl (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:29.714: kube-system/hubble-ui-6b468cff75-ztjsg:40950 (ID:51211) -> kube-system/hubble-relay-79d64897bd-slsbd:4245 (ID:1539) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:29.714: kube-system/hubble-ui-6b468cff75-ztjsg:40950 (ID:51211) <- kube-system/hubble-relay-79d64897bd-slsbd:4245 (ID:1539) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:29.714: kube-system/hubble-ui-6b468cff75-ztjsg:58054 (ID:51211) <- kube-system/hubble-relay-79d64897bd-slsbd:4245 (ID:1539) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:29.714: kube-system/hubble-ui-6b468cff75-ztjsg:58054 (ID:51211) -> kube-system/hubble-relay-79d64897bd-slsbd:4245 (ID:1539) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:30.295: 127.0.0.1:8080 (world) <> kube-system/coredns-5d78c9869d-j6772 (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:30.295: 127.0.0.1:34892 (world) <> kube-system/coredns-5d78c9869d-j6772 (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:30.412: 127.0.0.1:34900 (world) <> kube-system/coredns-5d78c9869d-5wcjl (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:30.412: 127.0.0.1:8080 (world) <> kube-system/coredns-5d78c9869d-5wcjl (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:31.295: 127.0.0.1:8080 (world) <> kube-system/coredns-5d78c9869d-j6772 (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:31.295: 127.0.0.1:34910 (world) <> kube-system/coredns-5d78c9869d-j6772 (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:31.412: 127.0.0.1:8080 (world) <> kube-system/coredns-5d78c9869d-5wcjl (ID:18872) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:31.506: kube-system/hubble-relay-79d64897bd-slsbd:42914 (ID:1539) -> 172.18.0.4:4244 (host) to-stack FORWARDED (TCP Flags: ACK)
Aug  8 05:27:31.506: kube-system/hubble-relay-79d64897bd-slsbd:42914 (ID:1539) <- 172.18.0.4:4244 (host) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:31.762: kube-system/hubble-ui-6b468cff75-ztjsg:34474 (ID:51211) <- kube-system/hubble-relay-79d64897bd-slsbd:4245 (ID:1539) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:31.762: kube-system/hubble-ui-6b468cff75-ztjsg:34474 (ID:51211) -> kube-system/hubble-relay-79d64897bd-slsbd:4245 (ID:1539) to-endpoint FORWARDED (TCP Flags: ACK)
Aug  8 05:27:31.781: 127.0.0.1:55792 (world) <> kube-system/hubble-relay-79d64897bd-slsbd (ID:1539) pre-xlate-rev TRACED (TCP)
Aug  8 05:27:31.789: kube-system/hubble-relay-79d64897bd-slsbd:57252 (ID:1539) -> 172.18.0.3:4244 (remote-node) to-stack FORWARDED (TCP Flags: ACK, PSH)
```
