# AWS ECR (Elastic Container Repoistory)
Amazon ECR uses Amazon S3 for storage to make your container images highly available and accessible, allowing you to reliably deploy new containers for your applications. Amazon ECR transfers your container images over HTTPS and automatically encrypts your images at rest. Amazon ECR is integrated with Amazon Elastic Container Service (ECS), simplifying your development to production workflow.

## Steps to Create ECR 

<b>Step1:-</b> Create ECR Repo in AWS lets say testrepo 

<b>Step2:-</b> Create A Role wihich has EC2RepositoryFullAccess Role

<b>Step3:-</b> Create An EC2 instance ( I am using AWS Ubuntu) and attach above role to this instance and also install docker in it.

<b>Step4:-</b> Install aws cli ( on ubuntu)
    
        apt install unzip -y
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        unzip awscliv2.zip
        sudo ./aws/install
        export PATH=/usr/local/bin:$PATH
        aws --version

<b>Step5:-</b> Follow the steps given on Push Commands

    aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 670154208195.dkr.ecr.ap-south-1.amazonaws.com
    docker pull ubuntu
    docker image tag ubuntu testrepo
    docker tag testrepo:latest 670154208195.dkr.ecr.ap-south-1.amazonaws.com/testrepo:latest
    docker images
    docker push 670154208195.dkr.ecr.ap-south-1.amazonaws.com/testrepo:latest
    docker rmi testrepo ubuntu
    docker rmi 670154208195.dkr.ecr.ap-south-1.amazonaws.com/testrepo
    docker pull 670154208195.dkr.ecr.ap-south-1.amazonaws.com/testrepo

