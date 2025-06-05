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