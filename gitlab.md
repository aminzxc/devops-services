### Install and Configure
```
sudo apt install ca-certificates curl openssh-server postfix tzdata perl
cd /tmp
curl -LO https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh
sudo bash /tmp/script.deb.sh
sudo apt install gitlab-ce
```
### Adjusting the Firewall Rules
```
sudo ufw allow http
sudo ufw allow https
sudo ufw allow OpenSSH
```
### Editing the GitLab Configuration File
```
sudo vim /etc/gitlab/gitlab.rb
external_url 'https://your_domain'
sudo gitlab-ctl reconfigure
```
### Password Login
```
user: root
password: cat /etc/gitlab/initial_root_password
```
### Restricting or Disabling Public Sign-ups
```
hamburger menu ---> Admin ----> Settings ---> Sign-up Restrictions ---> Expand  ---> deselect the Sign-up enabled check box
```
### Backup From Gitlab
```
sudo gitlab-backup create STRATEGY=copy
cd /var/opt/gitlab/backups/
sudo cp /etc/gitlab/gitlab.rb /mnt/backup/gitlab/config/
sudo cp /etc/gitlab/gitlab-secrets.json /mnt/backup/gitlab/config/
```
### Restore Backup
```
sudo chown git:git /var/opt/gitlab/backups/*.tar
sudo gitlab-backup restore BACKUP=1719929494_2025_07_02_15.7.1
sudo gitlab-ctl stop
sudo rm -rf /etc/gitlab
sudo cp -a /mnt/backup/gitlab/config /etc/gitlab
sudo chown -R root:root /etc/gitlab
sudo gitlab-ctl reconfigure
sudo gitlab-ctl start
```
### Test Health
```
sudo gitlab-rake gitlab:check SANITIZE=true
sudo gitlab-rake gitlab:env:info
```
### Upgrade Gitlab
```
apt-cache madison gitlab-ce
apt install gitlab-ce=17.11.1-ce.0
gitlab-ctl reconfigure
```
### sample pipline
```
stages:
  - build
  - deploy

variables:
  F2D_IMAGE_TAG: $CI_COMMIT_SHORT_SHA
  SERVICE_NAME: f2d
  DEVELOPERSIDE: backend
  STACK_NAME: stage
  DEPLOY_SERVER: oto@185.170.8.18
  DEPLOY_PATH: /home/oto/stack-stage
  GIT_DEPTH: "0"

F2D-DOCKER-IMAGE:
  stage: build
  before_script:
    - "echo Logging into $HUB_URL"
    - "docker login --username $HUB_USER --password $HUB_PASSWORD $HUB_URL"
  script:
    - "echo Building ${SERVICE_NAME} image"
    - "docker build -f build/stage/Dockerfile -t $HUB_URL/${DEVELOPERSIDE}/${SERVICE_NAME}:${F2D_IMAGE_TAG} ."
    - "docker push $HUB_URL/${DEVELOPERSIDE}/${SERVICE_NAME}:${F2D_IMAGE_TAG}"
  only:
    - develop

DEPLOY-F2D-IMAGE:
  stage: deploy
  before_script:
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  script:
    - echo "Deploying to stage"
    - |
      ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER -p 2256 -vvv "
        cd $DEPLOY_PATH;

        sed -i 's|image: repo.otomob.ir/backend/f2d:.*|image: repo.otomob.ir/backend/f2d:${F2D_IMAGE_TAG}|g' docker-compose.yml;
        docker login --username ${HUB_USER} --password ${HUB_PASSWORD} ${HUB_URL};
        docker stack deploy -c docker-compose.yml --with-registry-auth stage
      "
  only:
    - develop

```
