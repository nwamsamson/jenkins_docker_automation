Jenkins Pipeline for Dockerized Web Application
This project demonstrates how to set up a Jenkins pipeline to build and deploy a simple web application using Docker. The pipeline automates the process of fetching code from GitHub, building a Docker image, and running a Docker container to serve an HTML file.

Project Goal
The primary goal of this project is to establish a continuous integration/continuous deployment (CI/CD) pipeline using Jenkins to:

Automatically trigger builds upon code pushes to a GitHub repository.
Build a Docker image containing a web application.
Run the Docker image as a container, exposing the web application.
Prerequisites
Before setting up the Jenkins pipeline, ensure you have:

An EC2 instance running Jenkins.
Docker installed on the Jenkins EC2 instance.
GitHub webhook configured for your repository to trigger Jenkins builds.
Docker Installation on Jenkins EC2 Instance
The following commands were executed on the Jenkins EC2 instance to install Docker and grant necessary permissions:

Bash

sudo yum update -y
sudo yum install docker -y
sudo usermod -a -G docker ec2-user # Add the ec2-user to the docker group
id ec2-user # Verify group membership
newgrp docker # Reload group assignments without logging out
sudo systemctl enable docker.service
sudo systemctl start docker.service
Jenkins Configuration
GitHub Webhook
On the Jenkins UI, the "GitHub hook trigger for GITScm polling" option was enabled for the pipeline job. This ensures that any push events to the configured GitHub repository automatically trigger a new Jenkins build.

Jenkinsfile
The Jenkins pipeline script (Jenkinsfile) defines the stages for the CI/CD process. This script is crucial for automating the build, image creation, and container deployment.

!jenkinsfile

Troubleshooting: Docker Permissions Error
Initially, the Jenkins build failed with a permission denied error when trying to connect to the Docker daemon:

ERROR: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied
Rectification Steps
To resolve this issue, the jenkins user (under which Jenkins runs) needed to be added to the docker group. This grants the Jenkins user the necessary permissions to interact with the Docker daemon.

The following commands were executed on the Jenkins server:

Bash

sudo usermod -aG docker jenkins # Add the jenkins user to the docker group
sudo systemctl restart jenkins # Restart Jenkins to apply the new group permissions
Successful Build and Deployment
After implementing the permission fix and pushing code to the repository, the Jenkins pipeline executed successfully.

The pipeline performs the following actions:

Connect to GitHub: Clones the repository.
Build Docker Image: Creates a Docker image named dockerfile from the Dockerfile in the repository.
Run Docker Container: Starts a Docker container from the dockerfile image, mapping port 8082 on the host to port 80 inside the container.
Console Output of a Successful Build
Started by GitHub push by nwamsamson
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/jenkinspipelinejob
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Connect To Github)
[Pipeline] checkout
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/jenkinspipelinejob/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/andrewmkpanam/jenkinspipelinejob.git # timeout=10
Fetching upstream changes from https://github.com/andrewmkpanam/jenkinspipelinejob.git
 > git --version # timeout=10
 > git --version # 'git version 2.47.1'
 > git fetch --tags --force --progress -- https://github.com/andrewmkpanam/jenkinspipelinejob.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/main^{commit} # timeout=10
Checking out Revision 822ed2b862883f05942c16c45ed88eff66edaa55 (refs/remotes/origin/main)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 822ed2b862883f05942c16c45ed88eff66edaa55 # timeout=10
Commit message: "updates to html"
 > git rev-list --no-walk 8e3d83ffd5b412042c0a53680efb1e3a12c48845 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build Docker Image)
[Pipeline] script
[Pipeline] {
[Pipeline] sh
+ docker build -t dockerfile .
#0 building with "default" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 381B 0.0s done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/nginx:latest
#2 DONE 0.5s

#3 [internal] load .dockerignore
#3 transferring context: 2B done
#3 DONE 0.0s

#4 [1/3] FROM docker.io/library/nginx:latest@sha256:6784fb0834aa7dbbe12e3d7471e69c290df3e6ba810dc38b34ae33d3c1c05f7d
#4 DONE 0.0s

#5 [internal] load build context
#5 transferring context: 505B done
#5 DONE 0.0s

#6 [2/3] WORKDIR  /usr/share/nginx/html/
#6 CACHED

#7 [3/3] COPY index.html /usr/share/nginx/html/
#7 DONE 0.1s

#8 exporting to image
#8 exporting layers 0.0s done
#8 writing image sha256:7407e73828ee605a838759766398f8f81b1af79c1e0e4c455066aa7615df0ccf done
#8 naming to docker.io/library/dockerfile 0.0s done
#8 DONE 0.1s
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Run Docker Container)
[Pipeline] script
[Pipeline] {
[Pipeline] sh
+ docker run -itd -p 8082:80 dockerfile
2f0e0c5759d8e1c30e17f80fc353da6dfc7c071b442d655fc55edc5300c9bf91
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
Served HTML File
After the successful build and container deployment, the HTML file was accessible, verifying the complete functionality of the pipeline.

