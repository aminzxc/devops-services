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
