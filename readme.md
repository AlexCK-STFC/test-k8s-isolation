# Description/Considerations
- Central tenet is labelling namespaces with `network-zone=internal-net` or `network-zone=external-net`
  - Can also target by pod label etc, any k8s selector
- We then have a global default-deny network policy, only allowing traffic with nodes with the same network-zone
- We deploy two ingress controllers with different ingressclasses, with the same network-zones

- Would require applying a label to all application namespaces
- Requires Calico CNI (which we use by default) for GlobalNetworkPolicy support
- Would require RBAC access controls to prevent adding `network-zone` label where it shouldn't be
# Install
Set up a k8s/capi cluster, set it as your default kubeconfig, then:

```sh
cd ingress-controllers
kubectl -apply namespaces.yaml
kubectl apply -f namespaces.yaml
help dependencies update .
helm install test-ingress . -f values.yaml 

cd httpd-test
kubectl apply -f namespace.yaml 
helm install httpd-test . -f values.yaml
```

# Test
```sh
kubectl create namespace busybox-ns
kubectl label namespace busybox-ns network-zone=internal-net
kubectl run busybox-temp --rm -it --image=busybox --namespace=busybox-ns --restart=Never -- sh
wget -O- httpd.httpd-test.svc.cluster.local
```
wget should show the index.html successfully

```sh
exit
kubectl label namespace busybox-ns network-zone=external-net --overwrite
kubectl run busybox-temp --rm -it --image=busybox --namespace=busybox-ns --restart=Never -- sh
wget -O- httpd.httpd-test.svc.cluster.local
```
wget should now fail, and time out eventually

```sh
exit
```

# Uninstall
```sh
cd httpd-test
helm uninstall httpd-test
kubectl delete -f namespace.yaml

cd ../ingress-controllers
helm uninstall test-ingress
kubectl delete -f namespaces.yaml
```
