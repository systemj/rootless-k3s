# metallb

## install
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.2/config/manifests/metallb-native.yaml
```


```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: test
  namespace: metallb-system
spec:
  addresses:
  - 192.168.86.101/32
  - 192.168.86.101/32
  - 192.168.86.101/32
  - 192.168.86.101/32
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: test
  namespace: metallb-system
spec:
  ipAddressPools:
  - test
```