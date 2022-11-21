<h1 align=center>Deployment 5</h1>
<div align=center>Set up a Jenkins CI/CD pipeline with agents for docker and terraform and deploy a infrastructure as well as a containerized application.</div>

# Table of contents

1. Create 3 EC2s on the default VPC
2. Set up a jenkins server on EC2-1
3. Set up docker on EC2-2
4. Set up terraform on EC2-3
5. Set up jenkins agents for docker and terraform
6. Create a pipeline build on Jenkins
7. Build a Jenkinsfile
8. Additions
9. Diagram


## 1. Create 3 EC2s on the default VPC

* Log in to AWS and provision 3 EC2 instances.
* The EC2s will be allocated for Jenkins, Docker and Terraform.
* The computers will be running Ubunut linux
* Most settings can be left on default
* Security groups
  * The Jenkins server will needs the ports 22, 80 and 8080.
  * The docker EC2 will only require 22 for ssh.
  * The terraform machine will only require 22 for ssh.

## 2. Set up a jenkins server on EC2-1

* Select an ubuntu image and most default settings
* Open ports are 22, 80 and 8080
* Once ubuntu is set up and updated
* Install the following libraries:
  * default-jre
  * python3-pip
  * python3.10-venv
  * nginx
* Alternative:
  * To facilitate the set up process for jenkins I use [this script](https://github.com/Antoniorios17/kuralabs_deployment_5/blob/main/Jenkins-set-up-script.sh)

## 3. Set up docker on EC2-2

* to install docker follow this instructions
   * update the apt package index and install packages for allowing the use of other repositories over HTTPS

      ```
      $sudo apt-get update
      $sudo apt-get install \
          ca-certificates \
          curl \
          gnupg \
          lsb-release
      ```
   * Add docker's oficial key:
      ```
      $ sudo mkdir -p /etc/apt/keyrings
      $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      ```
   * Set up the repository
     ```
     $echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null 
     ```
   * Update the package index
     ```
     $ sudo apt-get update
     ```
   * Install the latest version of docker
     ```
     $ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
     ```
   * You can use docker now
## 4. Set up terraform on EC2-3

* To set up terraform run the following commands:
  * Install the terraform's official key

    ``` 
     $ wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
    ```
  * Set up the repository
  
    ```
     $ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
     https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    ```
        
  * Update the package index

    ``` 
     $ sudo apt update && sudo apt install terraform
    ```
        
## 5. Set up jenkins agents for docker and terraform

* Install the proper libraries for Jenkins to work
  * Jenkins run on java and will need default-jre
* Make sure to copy the public IP addresses of both EC2s
* The IP will be used by Jenkins to remote on the EC2s
* Once remote access is set up Jenkins will install the agents
* For a more detailed guide on how to set up the agents [click here](https://github.com/Antoniorios17/kuralabs_deployment_3#configure-the-jenkins-agent-on-the-vpc)
* For setting up the credentials to work with jenkins and AWS [click here](https://github.com/Antoniorios17/kuralabs_deployment_4/edit/main/README.md#configure-credentials-on-jenkins)



## 6. Create a pipeline build on Jenkins

* Create a new multibranch pipeline
* Add source: github
* Create a new access token on github for this deployment
* Add github credentials and use the token for password
* Validate the credential by adding the repository
* Apply and save the configuration


## 7. Build a Jenkinsfile
* The Jenkinsfile creates all the stages of production:
* The following stages run on the jenkins server:
  * Build
    * The application will be created in a virtual environment.
  * Test
    * The application will run unit tests to make sure it's working as expected. 
* Stages performed by the Jenkins agent set up on the EC2-2(Docker)
  * Create Container
    * The dockerfile will be copied from the repository to create an image of the flask application
  * Push to DockerHub
    * The image will be push to dockerHub to later be used in terraform
    
``` diff
-Troubleshooting: I had many problems trying to work push my image to dockehub running the commands as part of the stages.
the solution was to login to docker manually on the docker machine before hand and that way the credentials would be saved on the computer.

It is not very efficient but it fixed the issue and continued with the next stages.
```
* Stages to be performed by the Jenkins agent on the EC2-3(Terraform)
  * Init
    * This stage will initialize terraform
  * Plan
    * This stage will plan and make sure that the terraform files are working as expected without erros
    * Details will be presented of the work performed by terraform if it was to be applied.
  * Apply-Deploy to ECS
    * This stage will deploy the terraform files and configurations
    * The insfrastructure will be created
      * The VPC will have the following configurations
      * 2 availability zones
      * 2 subnets, a private and public
      * internet gateway
      * load balancer
      * routing tables
    * ECS will use the image from dockerhub
      * The application will be deployed in the private subnet
      * The traffic will be managed by the load balancer 
  * Destroy
    * This stage is not necessary as part of the production environment.
    * The only purpose of this stage is to take down the infrastructure to avoid charges from amazon.

## 7. Additions

## Webhook
* To make the current configuration more automated you can connect jenkins to the github repository:
  * Log in to github and access the repository online
  * "Settings" of the repository
  * Select "Webhoooks"
  * Add Webhook
  * ```
    http://{Ip-address}:8080/github-webhook/
    ```
## Update the front end
* this is a very simple change to the UI
* I added my name as part of the website page
* Updated the color of the front page
* This is the original page:

![UI-before](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/UI-before.PNG)

* This is the website after the changes:

![UI-after](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/UI-after.PNG)

## 8. Diagram
![Diagram](https://github.com/Antoniorios17/kuralabs_deployment_5/blob/main/Pictures/deployment5-diagram.png)








