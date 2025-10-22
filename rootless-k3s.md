# rootless k3s
https://docs.k3s.io/advanced#running-rootless-servers-experimental

## host details
- amd64
- 4x cpu
- 8g mem
- 25g disk
- Ubuntu 24.04 (minimal, +ssh only)

## configure host

### install required packages
Install OS packages required for rootless operation.
```bash
sudo apt install -y uidmap iptables
```

### user setup
Required (...helpful) for systemd things
```bash
sudo loginctl enable-linger k3s
```

### configure cgroups v2 delegation
https://rootlesscontaine.rs/getting-started/common/cgroup2/#enabling-cpu-cpuset-and-io-delegation
```bash
sudo mkdir -p /etc/systemd/system/user@.service.d
cat <<EOF | sudo tee /etc/systemd/system/user@.service.d/delegate.conf
[Service]
Delegate=cpu cpuset io memory pids
EOF
sudo systemctl daemon-reload
```

### configure sysctls
```bash
cat << EOF | sudo tee /etc/sysctl.d/20-k3s-rootless.conf
# allow user created namespaces - https://github.com/k3s-io/k3s/issues/12711#issuecomment-3152639172
kernel.apparmor_restrict_unprivileged_unconfined=0
kernel.apparmor_restrict_unprivileged_userns=0

# enable ip forwarding
net.ipv4.ip_forward=1

# enable icmp for non-root user groups (gid:1000)
net.ipv4.ping_group_range=1000 1000
EOF
sudo sysctl -p /etc/sysctl.d/20-k3s-rootless.conf
```

### configure iptables rules
```bash
# loopback interface
sudo iptables -t nat -A OUTPUT -o lo -p tcp --dport 514 -j REDIRECT --to-port 31234
sudo iptables -t nat -A OUTPUT -o lo -p udp --dport 514 -j REDIRECT --to-port 31234


# external interfaces
sudo iptables -t nat -A PREROUTING -i enp1s0 -p tcp --dport 514 -j REDIRECT --to-ports 31234
sudo iptables -t nat -A PREROUTING -i enp1s0 -p udp --dport 514 -j REDIRECT --to-ports 31234


# SNAT for ping/traceroute traffic to specific interface
sudo iptables -t nat -A POSTROUTING -m owner --uid-owner 1000 -p icmp -j SNAT --to-source 192.168.122.78
sudo iptables -t nat -A POSTROUTING -m owner --uid-owner 1000 -p udp --destination-port 33441:33500 -j SNAT --to-source 192.168.122.78

```

## k3s install

### external requirements
```bash
mkdir -p ${HOME}/.local/bin
curl -Lo ${HOME}/.local/bin/slirp4netns --fail -L https://github.com/rootless-containers/slirp4netns/releases/download/v1.3.3/slirp4netns-$(uname -m)
chmod +x ${HOME}/.local/bin/slirp4netns
```


```bash
export K3S_VERSION="v1.33.5+k3s1"

# download binary
mkdir -p ${HOME}/.local/bin
curl -Lo ${HOME}/.local/bin/k3s https://github.com/k3s-io/k3s/releases/download/${K3S_VERSION}/k3s
chmod +x ${HOME}/.local/bin/k3s
for cmd in kubectl crictl ctr
  do
  ln -s k3s ${HOME}/.local/bin/${cmd}
done


# download images
mkdir -p ${HOME}/.rancher/k3s/agent/images/
curl -Lo ${HOME}/.rancher/k3s/agent/images/k3s-airgap-images-amd64.tar.zst "https://github.com/k3s-io/k3s/releases/download/${K3S_VERSION}/k3s-airgap-images-amd64.tar.zst"

# configure systemd user service
mkdir -p ${HOME}/.config/systemd/user/
cat << EOF > ${HOME}/.config/systemd/user/k3s-rootless.service
# systemd unit file for k3s (rootless)

[Unit]
Description=k3s (Rootless)

[Service]
Environment=PATH=${HOME}/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
Environment=K3S_ROOTLESS_PORT_DRIVER=slirp4netns
Environment=K3S_ROOTLESS_MTU=1500
ExecStart=${HOME}/.local/bin/k3s server --rootless --snapshotter=fuse-overlayfs --disable=traefik --tls-san test1-kubernetes
ExecReload=/bin/kill -s HUP \$MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
Type=simple
KillMode=mixed

[Install]
WantedBy=default.target
EOF

# these must be set if switching from 'sudo su - user'
export XDG_RUNTIME_DIR="/run/user/$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=${XDG_RUNTIME_DIR}/bus"
systemctl --user daemon-reload
systemctl --user enable --now k3s-rootless
```

## user env setup
```bash
cat << EOF >> .bashrc
export XDG_RUNTIME_DIR="/run/user/\$(id -u)"
export DBUS_SESSION_BUS_ADDRESS="unix:path=\${XDG_RUNTIME_DIR}/bus"
export KUBECONFIG=\${HOME}/.kube/k3s.yaml
export PATH=\${HOME}/.local/bin:\${PATH}
export CONTAINER_RUNTIME_ENDPOINT=/run/user/$(id -u)/k3s/containerd/containerd.sock
export CONTAINERD_ADDRESS=\${CONTAINER_RUNTIME_ENDPOINT}
alias k="kubectl"
EOF
```

## operation
```bash
systemctl --user stop k3s-rootless.service
systemctl --user stop k3s-rootless.service
journalctl --user k3s-rootless.service
```

## testing
```bash
# general (curl, etc)
k run -i -t --rm --image dtzar/helm-kubectl:latest testing -- /bin/bash

# rsyslog
logger --udp --server 127.0.0.1 --port 514 -p local0.crit "test message 1 udp"
logger --tcp --server 127.0.0.1 --port 514 -p local0.crit "test message 1 tcp"
logger --tcp --server 192.168.122.36 --port 31234 -p local0.crit "test message 1 tcp remote for real"
logger --udp --server 192.168.122.36 --port 31234 -p local0.crit "test message 1 udp remote for real"

# iperf client pod
kubectl run -i -t --image ubuntu:latest test1 -- /bin/bash
apt update && apt install -y iperf3
iperf3 -c iperf3^C
```

## rsyslog test pod/service
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rsyslog
  labels:
    app.kubernetes.io/name: rsyslog
spec:
  containers:
  - name: rsyslog
    image: rsyslog/syslog_appliance_alpine:8.36.0-3.7
    ports:
    - containerPort: 514
      protocol: UDP
    - containerPort: 514
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: rsyslog-udp
spec:
  externalTrafficPolicy: Local
  internalTrafficPolicy: Local
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: rsyslog
  ports:
    - name: udp
      protocol: UDP
      port: 514
      targetPort: 514
      nodePort: 31234
    - name: tcp
      protocol: TCP
      port: 514
      targetPort: 514
      nodePort: 31234
```

## iperf test pod/service
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: iperf3
  labels:
    app.kubernetes.io/name: iperf3
spec:
  containers:
  - name: iperf3
    image: networkstatic/iperf3:latest
    args:
      - -s
      - -V
    ports:
    - containerPort: 5201
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: iperf3
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: iperf3
  ports:
    - name: tcp
      protocol: TCP
      port: 5201
      targetPort: 5201
```