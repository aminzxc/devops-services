### on worker mode
```
mkdir -p /etc/containerd/certs.d/ip registery
```
```
vim /etc/containerd/certs.d/ip registery/hosts.toml

server = "http://ip registery"

[host."http://ip registery"]
  capabilities = ["pull", "resolve"]
```
```
sudo cp -a /etc/containerd/config.toml \
                      /etc/containerd/config.toml.bak.$(date +%Y%m%d-%H%M%S)
```
```
vim /etc/containerd/config.toml

[plugins.'io.containerd.cri.v1.images'.registry]
  config_path = '/etc/containerd/certs.d'
```
```
sudo systemctl restart containerd
                sudo systemctl status containerd --no-pager
```
### on master node
```
vim harbor-regcred.yaml

apiVersion: v1
kind: Secret
metadata:
  name: harbor-regcred
  namespace: ns
type: kubernetes.io/dockerconfigjson
stringData:
  .dockerconfigjson: |
    {
      "auths": {
        "ip registery": {
          "username": "admin",
          "password": "Harbor2wsx@WSX"
        }
      }
    }
```
### Add on deployments for pull image
```
    spec:
      imagePullSecrets:
        - name: harbor-regcred
```


