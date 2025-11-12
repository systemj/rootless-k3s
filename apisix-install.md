# apisix install

## install helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

## install apisix
- https://apisix.apache.org/docs/ingress-controller/install/#install-apisix-and-apisix-ingress-controller
```bash
helm repo add apisix https://apache.github.io/apisix-helm-chart
helm repo update
helm upgrade --install apisix apisix/apisix \
  --namespace ingress-apisix \
  --create-namespace \
  --set apisix.ssl.enabled=true \
  --set etcd.persistence.enabled=false \
  --set service.type=LoadBalancer \
  --set service.stream.enabled=true \
  --set service.stream.tcp="{55671}" \
  --set ingress-controller.enabled=true \
  --set ingress-controller.apisix.adminService.namespace=ingress-apisix \
  --set ingress-controller.gatewayProxy.createDefault=true
```

## test app (prometheus)
```bash
helm upgrade --install prometheus oci://ghcr.io/prometheus-community/charts/prometheus \
  --namespace prometheus \
  --create-namespace \
  --values prom-values.yaml
```
### prom-values.yaml:
```yaml
server:
  ingress:
    enabled: true
    ingressClassName: "apisix"
    hosts:
    - prometheus.lab.systemj.net
    tls:
    - secretName: prometheus-server-tls
      hosts:
      - prometheus.lab.systemj.net
  persistentVolume:
    enabled: false
```

## ingress for skupper grant server
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    internal.skupper.io/secured-access: "true"
  name: skupper-grant-server
  namespace: skupper
spec:
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: https
    port: 9090
    protocol: TCP
    targetPort: 9090
  selector:
    app.kubernetes.io/name: skupper-controller
    app.kubernetes.io/part-of: skupper
    application: skupper-controller
    skupper.io/component: controller
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: skupper-grant-server
  namespace: skupper
spec:
  ingressClassName: apisix
  rules:
  - host: rke.systemj.net
    http:
      paths:
      - backend:
          service:
            name: skupper-grant-server
            port:
              number: 9090
        path: /skupper
        pathType: Prefix


```





## streams for skupper
```yaml
apiVersion: apisix.apache.org/v2        # Use APISIX CRD v2
kind: ApisixRoute                       # CRD for defining route
metadata:
  name: tenant1-database-route
  namespace: default     # Route exists in tenant's namespace
spec:
  stream:
    - name: mongo-tcp-proxy            # Unique name for the stream route
      protocol: TCP                    # Protocol is TCP (not HTTP)
      match:
        host: tenant1.mongo.database.com  # Match incoming SNI
        ingressPort: 27017            # Match LB port traffic
      backend:
        serviceName: mongodb          # Internal service to forward traffic
        servicePort: 27017            # Port of internal MongoDB
      plugins:
        - name: ip-restriction        # Plugin for IP-based access control
          enable: true
          config:
            whitelist:
              - 10.228.0.0/16         # Only allow this CIDR range
---
apiVersion: apisix.apache.org/v2        # Use APISIX CRD v2
kind: ApisixRoute                       # CRD for defining route
metadata:
  name: tenant1-database-route
  namespace: tenant1     # Route exists in tenant's namespace
spec:
  stream:
    - name: mongo-tcp-proxy            # Unique name for the stream route
      protocol: TCP                    # Protocol is TCP (not HTTP)
      match:
        host: tenant1.mongo.database.com  # Match incoming SNI
        ingressPort: 27017            # Match LB port traffic
      backend:
        serviceName: mongodb          # Internal service to forward traffic
        servicePort: 27017            # Port of internal MongoDB
      plugins:
        - name: ip-restriction        # Plugin for IP-based access control
          enable: true
          config:
            whitelist:
              - 10.228.0.0/16         # Only allow this CIDR range
```
