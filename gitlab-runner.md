### Registering a GitLab Runner
```
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner
sudo gitlab-runner register -n --url https://your_gitlab.com --registration-token project_token --executor docker --description "Deployment Runner" --docker-image "docker:stable" --tag-list deployment --docker-privileged
```
### Config Runner
```
concurrent = 2
check_interval = 0
connection_max_age = "15m0s"
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "harbor"
  helper_image = "registry.gitlab.com/gitlab-org/gitlab-runner/gitlab-runner-helper:x86_64-v17.11.0"
  url = "https://git.metakhodro.ir"
  id = 3
  token = "t1_zdNEHZsdfwb4niRANH2uxk-"
  token_obtained_at = 2025-04-20T13:10:41Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  clone_url = "http://45.149.71.173"
  [runners.environment]
    GIT_DEPTH = "0"
    GIT_CURL_VERBOSE = "1"
    GIT_TRACE = "1"
    GIT_TRACE_PACKET = "1"
  [runners.cache]
    MaxUploadedArchiveSize = 0
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "repo.otomob.ir/stable-image/docker:stable"
    pull_policy = "if-not-present"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
    network_mtu = 0

[[runners]]
  name = "front-office&back-office"
  url = "https://git.metakhodro.ir"
  id = 5
  token = "t3_xc1fdgzsrdVpttSWHrz6niy"
  token_obtained_at = 2025-07-01T06:23:36Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  clone_url = "https://git.metakhodro.ir"
  [runners.cache]
    MaxUploadedArchiveSize = 0
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "repo.otomob.ir/stable-image/docker:stable"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]
    shm_size = 0
    network_mtu = 0
    dns = ["8.8.8.8", "1.1.1.1"]

```
### test conection repo with gitlab-runner on server runner
```
git ls-remote http://45.149.71.175/backend/msfront2db.git
curl -I http://45.149.71.175/users/sign_in
```
