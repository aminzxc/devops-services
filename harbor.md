### First, download the latest version of `Harbor`
```
wget https://github.com/goharbor/harbor/releases/download/v2.10.0/harbor-offline-installer-v2.10.0.tgz
tar xzvf harbor-offline-installer-v2.10.0.tgz
cd harbor
```
### Configuring the `harbor.yml` file
```
hostname: registry.example.com  #DomainName

http:
  port: 80

external_url: https://registry.example.com   #The final address that users will use

harbor_admin_password: yourStrongPassword

database:
  password: changeme

data_volume: /data/harbor

```
### install & run `harbor`
```
./install.sh
```
### change config & restart `harbor`
```
cd /path/to/harbor
./prepare      #Checks and updates configurations
docker-compose down
docker-compose up -d

```
### let’s Encrypt for `harbor`
```
apt update
apt install certbot
``` 
### Obtaining a temporary certificate
```
certbot certonly --standalone -d repo.domain.ir
# After successful execution, the certificates will be located in the following path
Certificate: /etc/letsencrypt/live/repo.domain.ir/fullchain.pem

Private key: /etc/letsencrypt/live/repo.domain.ir/privkey.pem

```
### config harbor.yml
```
hostname: repo.domain.ir

https:
  port: 443
  certificate: /etc/letsencrypt/live/repo.domain.ir/fullchain.pem
  private_key: /etc/letsencrypt/live/repo.domain.ir/privkey.pem

harbor_admin_password: YourStrongPassword

```
### add config service nginx
```
  nginx:
    image: goharbor/nginx-photon:v2.10.0
    container_name: nginx
    ...
    volumes:
      - /etc/letsencrypt:/etc/letsencrypt:ro
      - type: volume
        source: harbor-nginx-conf
        target: /etc/nginx
      ...

```
### let’s Encrypt is only valid for 90 days. To automatically renew, create this cronjob
```
sudo crontab -e
0 3 * * * certbot renew --pre-hook "docker-compose -f /path/to/harbor/docker-compose.yml down" --post-hook "docker-compose -f /path/to/harbor/docker-compose.yml up -d"

```
### For disk management, Retention Policy and Garbage Collection must be enabled
## ✅ Recommended Settings for Production Use

Here are some best-practice recommendations for optimizing Harbor in a production environment:

| Setting                    | Recommended Value                          | Description                                                                 |
|----------------------------|--------------------------------------------|-----------------------------------------------------------------------------|
| **Tag Retention Policy**   | Enabled per project                        | Keep only the latest 3–10 image versions depending on your CI/CD workflow. |
| **Garbage Collection (GC)**| Scheduled daily (e.g., 2:00 AM)            | Reclaims disk space by removing unreferenced blobs.                         |
| **Untagged Artifacts**     | ✅ Enabled                                 | Allows GC to remove orphaned/untagged images.                               |
| **Log Rotation**           | 100MB size, 3 backups, 7 days              | Prevents logs from consuming disk space.                                    |
| **Audit Log Cleanup**      | 7–14 days, purge frequent event types      | Keeps the database light by purging unnecessary event logs.                 |
| **GC Workers**             | 2–4 (based on available CPU/RAM)           | Controls the number of concurrent GC operations.                            |

> ℹ️ It's important to always schedule GC after retention policies to ensure actual disk space is freed.
### Setting to keep the last 3 images and delete older images
![config](Practical-tools/images/project-policy.png)

