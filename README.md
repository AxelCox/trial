# A Simple Python Flask Application built and deployed to Kubernetes

## Code Overview and Understanding the application

app.py

This file contains the code for the Flask application. It application has one endpoint:

/ - returns a JSON response with a status of "OK".

To run the application locally, use the code below:
```
python3 app.py
```

## Starting services with AWS Cloud

This is a training project to deploy the application to Amazon EKS Cluster using build and deployment Pipeline with Jenkinsfile. In order to deploy the application, you need to run Jenkins Server using .tf files in Jenkins-Server folder. Clone this repository to your local machine, and use tf files in Jenkins-Server folder to launch Jenkins Server on AWS Cloud. Connect to the Jenkins Server (Amazon Linux 2) using ssh, and use prepare your Pipeline with Jenkinsfile.

Do NOT Forget!!

The GUI of Jenkins Server running on: 
```
http://[Public_IP_of_EC2]:8080
```

Jenkinsfile uses this repository during running CD Pipeline. Main, Dev and Feature branches are created to simulate a real project. Main branch is created for the production environment, dev branch is created for the development environment and feature branches are used to develop new features for the upcoming or a distant future release. The application is regarded as final product and it is deployed to the Cluster by using main branch.

### Installations in Jenkins Server:

* Git
* Jenkins
* Docker
* Docker-Compose
* CLI 2
* Python3
* Terraform
* eksctl which is a simple command line tool for creating Kubernetes clusters on Amazon EKS
* Amazon EKS vended kubectl binary to manage Cluster


### Dockerfile

This file contains the instructions for building a Docker image that can run the Flask application. It starts with an official Python runtime image, installs the required dependencies with requirements.txt. The container expose port is 3333.

### Manifest File

This file includes both deployment and service configuration in this single yaml file for deploying the application in AWS EKS Cluster. The application will run in a container using deployment and service object will create an ALB endpoint.

### Jenkinsfile

This file runs the CD pipeline to build and deploy the application in EKS Cluster. With this file, 

* AWS ECR is created for the image built with Dockerfile
* A dynamic tag is prepared for the Docker Image which will be built in the next steps
* Docker Image is built using Docker Image Tag
* The Image is pushed to AWS ECR Registry
* EKS Cluster is created
* EKS Cluster and Nodes are waited to be ready before deploying
* The application is deployed using Amazon EKS vended kubectl binary
* ALB endpoint is provided if deployment is successfully completed
* Local images, ECR Registry and EKS is deleted if there is a failure in CD Pipeline 




