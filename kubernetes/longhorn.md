### install longhorn with helm
### on worker node
```
sudo apt update

sudo apt install -y \
  open-iscsi \
  nfs-common \
  curl \
  bash \
  gawk \
  grep \
  util-linux
```
```
sudo modprobe iscsi_tcp

echo iscsi_tcp | \
  sudo tee /etc/modules-load.d/longhorn-iscsi.conf

sudo systemctl enable --now iscsid
```
### check enabled iscsid
```
systemctl status iscsid
lsmod | grep iscsi_tcp
which iscsiadm
```
```
sudo mkdir -p /var/lib/longhorn

df -h /var/lib/longhorn
```
### For a raw, unpartitioned disk
```
sudo parted -s /dev/sdb \
  mklabel gpt \
  mkpart primary ext4 1MiB 100%

sudo partprobe /dev/sdb

sudo mkfs.ext4 /dev/sdb1

sudo mkdir -p /var/lib/longhorn
UUID=YOUR-UUID /var/lib/longhorn ext4 defaults 0 2
sudo mount -a

findmnt /var/lib/longhorn
df -h /var/lib/longhorn
```
### create values-longhorn.yaml
```
defaultSettings:
  # Host directory used for Longhorn data
  defaultDataPath: /var/lib/longhorn

  # Initial number of replicas
  defaultReplicaCount: 1

  # Rebalance when new nodes become available
  replicaAutoBalance: least-effort

  # Conservative capacity allocation
  storageOverProvisioningPercentage: 100
  storageMinimalAvailablePercentage: 25

persistence:
  # Do not replace another default StorageClass
  defaultClass: false

  # Replica count for the generated Longhorn StorageClass
  defaultClassReplicaCount: 1

  # Keep data if PVC is deleted
  reclaimPolicy: Retain

  # Filesystem of dynamically created volumes
  defaultFsType: ext4
```
### on master node
```
helm repo add longhorn https://charts.longhorn.io
helm repo update

helm search repo longhorn/longhorn --versions

helm upgrade --install longhorn longhorn/longhorn \
  --namespace longhorn-system \
  --create-namespace \
  --version 1.13.0 \
  --values values-longhorn.yaml \
  --wait \
  --timeout 15m
```
### check install 
```
helm list -n longhorn-system
kubectl get pods -n longhorn-system -o wide
kubectl get storageclass
```
### test pv & pvc
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-test-pvc
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 1Gi

---
apiVersion: v1
kind: Pod
metadata:
  name: longhorn-test
  namespace: default
spec:
  containers:
    - name: test
      image: busybox:1.36
      command:
        - /bin/sh
        - -c
        - |
          if [ ! -f /data/hello.txt ]; then
            echo "Longhorn storage is working" > /data/hello.txt
          fi
          sleep 3600
      volumeMounts:
        - name: data
          mountPath: /data

  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: longhorn-test-pvc
```
```
kubectl apply -f longhorn-test.yaml
kubectl get pvc longhorn-test-pvc

kubectl get pod longhorn-test

kubectl get pv
```
### access dashboard
```
kubectl port-forward --address 0.0.0.0 service/longhorn-frontend -n longhorn-system 8090:80
```
### access with URL
```
kubectl get crd middlewares.traefik.io
```
```
sudo apt install -y apache2-utils
htpasswd -cB auth longhornadmin
kubectl create secret generic longhorn-basic-auth \
  --from-file=auth \
  -n longhorn-system
```
### create longhorn-middleware.yaml
```
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: longhorn-auth
  namespace: longhorn-system
spec:
  basicAuth:
    secret: longhorn-basic-auth
    realm: "Longhorn Administration"
    removeHeader: true
```
### create longhorn-ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-ingress
  namespace: longhorn-system
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: web
    traefik.ingress.kubernetes.io/router.middlewares: "longhorn-system-longhorn-auth@kubernetescrd"

spec:
  ingressClassName: traefik

  rules:
    - host: longhorn.30bime.ir
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: longhorn-frontend
                port:
                  number: 80
```
```
kubectl get ingress -n longhorn-system
```
