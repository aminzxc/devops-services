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
  - deploy-stage
  - deploy-production

variables:
  BACKOFFICE_IMAGE_TAG: $CI_COMMIT_SHORT_SHA
  DEPLOY_SERVER_STAGE: USER@IP
  DEPLOY_SERVER_PRODUCTION: USER@IP
  DEPLOY_PATH_STAGE: /home/oto/stack-stage
  DEPLOY_PATH_PROD: /home/oto/services
  GIT_DEPTH: "0"

BACKOFFICE-DOCKER-IMAGE:
  stage: build
  rules:
    - if: '$CI_COMMIT_BRANCH =~ /^(main|develop)$/'
    - when: never
  before_script:
    - echo Logging into $HUB_URL
    - echo "${HUB_PASSWORD}" | docker login --username "${HUB_USER}" --password-stdin "${HUB_URL}"
  script:
    - |
      if [ "$CI_COMMIT_BRANCH" = "main" ]; then
        DOCKERFILE=Dockerfile-prod
        REPO_PATH=production
        IMAGE_NAME=back-office
      elif [ "$CI_COMMIT_BRANCH" = "develop" ]; then
        DOCKERFILE=Dockerfile
        REPO_PATH=frontend
        IMAGE_NAME=back-office
      else
        echo "Unsupported branch: $CI_COMMIT_BRANCH"
        exit 1
      fi

      FULL_IMAGE_NAME="repo.otomob.ir/${REPO_PATH}/${IMAGE_NAME}:${CI_COMMIT_SHORT_SHA}"
      echo "Using Dockerfile: $DOCKERFILE"
      echo "Building and pushing image: $FULL_IMAGE_NAME"
      docker build -f $DOCKERFILE -t $FULL_IMAGE_NAME .
      docker push $FULL_IMAGE_NAME

DEPLOY-BACKOFFICE-STAGE:
  stage: deploy-stage
  before_script:
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  script:
    - echo "Deploying to staging"
    - >
      ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER_STAGE -p 2256 "
      cd $DEPLOY_PATH_STAGE &&
      sed -i 's|image: repo.otomob.ir/frontend/back-office:.*|image: repo.otomob.ir/frontend/back-office:${BACKOFFICE_IMAGE_TAG}|g' docker-compose.yml &&
      echo "${HUB_PASSWORD}" | docker login --username "${HUB_USER}" --password-stdin "${HUB_URL}" &&
      docker stack deploy -c docker-compose.yml --with-registry-auth stage
      "
  rules:
    - if: '$CI_COMMIT_BRANCH == "develop"'

DEPLOY-BACKOFFICE-PROD:
  stage: deploy-production
  when: manual         
  allow_failure: false
  before_script:
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY_PRO" | tr -d '\r' | ssh-add -
  script:
    - echo "Deploying to production"
    - >
      ssh -o StrictHostKeyChecking=no $DEPLOY_SERVER_PRODUCTION -p 2682 "
      cd $DEPLOY_PATH_PROD &&
      sed -i 's|image: repo.otomob.ir/production/back-office:.*|image: repo.otomob.ir/production/back-office:${BACKOFFICE_IMAGE_TAG}|g' docker-compose.yml &&
      echo "${HUB_PASSWORD}" | docker login --username "${HUB_USER}" --password-stdin "${HUB_URL}" &&
      docker compose up -d back-office
      "
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'


```
