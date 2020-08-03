# Introduction
I need to show some HPA (Horizontal Pod Autoscaler) on a TKG 1.0.0 Guest cluster. 
With helm 3 already available, I installed the helm repo and subsequently metrics-server.
However, the metrics-server is not working as intended.

# Testing method
- kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes" should yield results
- using `kubectl top pods` should show some utilization results
- for additional confirmation, can add namespace, such as `kubectl top pods -n default`

# Solution
1. Install metrics-server chart with PSP
```
helm install metric-server bitnami/metrics-server
```

2. Edit the deployment to include `--kubelet-insecure-tls` and `--kubelet-preferred-address-types=InternalIP`
```
    spec:
      containers:
      - command:
        - metrics-server
        - --secure-port=8443
        - --kubelet-insecure-tls
        - --kubelet-preferred-address-types=InternalIP
```
3. Wait for few minutes, lets say 5 minutes.

4. Try retrieve the results
```
$ kubectl top nodes
NAME                                            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
ip-10-0-0-120.ap-southeast-1.compute.internal   123m         6%     938Mi           50%       
ip-10-0-0-29.ap-southeast-1.compute.internal    50m          2%     629Mi           33%       

$ kubectl top pods
NAME                                           CPU(cores)   MEMORY(bytes)   
metric-server-metrics-server-8787f5cd4-gwjgh   2m           11Mi       
```
