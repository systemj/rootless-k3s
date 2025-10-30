
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
skupper -n default connector create iperf3 5201 --workload service/iperf3
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
skupper -n default listener create iperf3 5201
skupper -n default listener status
skupper -n default connector create test1-kubernetes 443 --host kubernetes
```


# skupper
https://github.com/skupperproject/skupper/blob/93bc6badb1d868ddfa5a89fc47dc06a987bb1dbd/internal/kube/grants/config_test.go#L47

In `install.yaml` comment/delete: `-grant-server-autoconfigure`

```yaml
- name: SKUPPER_GRANT_SERVER_BASE_URL
    value: host.example.com/skupper
- name: SKUPPER_GRANT_SERVER_AUTOCONFIGURE
    value: false
- name: SKUPPER_GRANT_SERVER_TLS_CREDENTIALS
    value: my-secret
```
