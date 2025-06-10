###  What is `Hadolint`?
```
hadolint is a static analysis (Lint) tool designed to check Dockerfiles. It helps to:
Write Dockerfiles in a more optimized and maintainable way
Detect common mistakes in Dockerfile syntax and structure
Follow best practices for Dockerfile authoring
Ultimately reduce the image size and improve security and stability
```
### proje
```
https://github.com/hadolint/hadolint
```
### install & use `hadolint`
```
docker run --rm -i hadolint/hadolint < Dockerfile
```

### What is `dive`?
```
A tool for exploring a Docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image
```
### proje
```
https://github.com/wagoodman/dive
```
### install & use
```
docker run --rm -it \
 -v /var/run/docker.sock:/var/run/docker.sock \
 wagoodman/dive:latest \
 <image-name>/<tag>
```
### What is `slimtoolkit`?
```
With this project, you can optimize the size of the created images and create a new image
```
### install & use
```
wget https://github.com/slimtoolkit/slim/releases/download/1.40.11/dist_linux.tar.gz
tar -xzvf dist_linux.tar.gz
./docker-slim build \
  --tag front:slim \
  --http-probe=true \
  --http-probe-cmd "GET /" \
  --include-path /app/.next \
  --include-path /app/node_modules \
  --include-path /app/package.json \
  --include-path /etc/passwd \
  --include-path /etc/group \
  --continue-after=probe \
  <image>:<tag>

```
### Docker Image Reduction Project
```
https://github.com/slimtoolkit/slim
```
