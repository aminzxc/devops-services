#!/bin/bash

docker_images=$(docker images --format "{{.Repository}} {{.ID}}" | egrep "^repo\.otomob\.ir/(backend|frontend|production)")

used_images=$(docker ps --format "{{.Image}}" | sort -u)

while read -r repo img_id; do
    if ! echo "$used_images" | grep -q "$img_id"; then
        echo "Deleting unused image: $repo ($img_id)"
        docker rmi "$img_id"
    else
        echo "Skipping in-use image: $repo ($img_id)"
    fi
done <<< "$docker_images"

echo "Running: docker builder prune"
docker builder prune -f
