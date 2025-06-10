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
