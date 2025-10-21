
# install
both clusters

```bash
kubectl apply -f https://skupper.io/install.yaml
curl https://skupper.io/install.sh | sh
```

# hub cluster
```bash
skupper -n default site create hub --enable-link-access
skupper -n default site status
skupper -n default token issue ${HOME}/hub.token # --redemptions-allowed 2
skupper -n default connector create prom 9090 --workload service/prometheus-service
skupper -n default connector status
skupper -n default listener create test1-kubernetes 443
```

# spoke cluster
```bash
skupper -n default site create spoke
skupper -n default site status
skupper -n default token redeem token.yaml
skupper -n default link status
skupper -n default listener create prom 9090
skupper -n default listener status
skupper -n default connector create kubernetes 443 --host kubernetes
```
