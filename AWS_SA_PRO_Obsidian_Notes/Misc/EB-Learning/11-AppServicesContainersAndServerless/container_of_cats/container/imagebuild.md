---
aliases:
- "Build the docker image"
- "Container of cats docker image"
- "Download, Install and Configure docker and tools"
- "Test the image by running a container"
- "Upload image to docker hub"
- "Use Git to get the lesson files"
---

# Container of cats docker image

Ideally run this on an [[ec2]] instance within AWS (running amazon linux)
Needs internet access for image upload to dockerhub
You will need a dockerhub account
To test the container ... tcp/80 will need to be open on the instance [[appsync|security]] group

## Download, Install and Configure docker and tools

sudo amazon-linux-extras install docker
sudo service docker start
sudo usermod -a -G docker ssm-user

## Use Git to get the lesson files

sudo yum install git
git clone [[github/acantril/aws-sa-pro.git]] [🔗](https://github.com/acantril/aws-sa-pro.git)

## Build the docker image

[[cicd|cd]] aws-sa-pro/11-AppServicesContainersAndServerless/container_of_cats/container
docker build -t containerofcats .
docker images --filter reference=containerofcats

## Test the image by running a container
docker run -t -i -p 80:80 containerofcats

## Upload image to docker hub

docker login --username YOUR_USER
docker images
docker tag IMAGEID YOUR_USER/containerofcats
docker push YOUR_USER/containerofcats
