<h1 align=center>Deployment 3</h1>
<div align=center>Set up a Jenkins CI/CD pipeline with agents for docker and terraform and deploy a infrastructure as well as a containerized application.</div>

# Table of contents

1. Create 3 EC2s on the default VPC
2. Set up a jenkins server on EC2-1
3. Set up docker on EC2-2
4. Set up terraform on EC2-3
5. Set up jenkins agents for docker and terraform
6. Create a pipeline build on Jenkins
7. Additions
8. Diagram


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
  * To facilitate the set up process for jenkins I use [this](https://github.com/Antoniorios17/kuralabs_deployment_5/blob/main/Jenkins-set-up-script.sh) script.

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
      $ echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
* Update the package index
```
      $ sudo apt update && sudo apt install terraform
```
## 5. Set up jenkins agents for docker and terraform
## 6. Create a pipeline build on Jenkins
## 7. Additions
## 8. Diagram





## Install Jenkins on an EC2
The EC2 doesn't need to be part of the vpc, we are trying to connect the jenkins server from outside the VPC with an agent
* Connect to the repository on github using personal token
* Test to verify authentication is successful

## Create an EC2 in your public subnet of your VPC
* Select an ubuntu image and most default settings
* Open ports are 22 and 5000
* Once ubuntu is set up and updated
* Install the following libraries:
  * default-jre
  * python3-pip
  * python3.10-venv
  * nginx

This is the preparation of the EC2 to work with the Jenkins agent coming from the main server.

## Configure the Jenkins agent on the VPC
Keep in mind that you are working on these steps on the main server outside the VPC.

* From the jenkins dashboard select the option for "Build Executor Status"
* To create a new agent "New Node" and permanent agent
* Configure the name and description as necessary
* To connect to the EC2 inside the VPC launch the agent via SSH:
  * Add the public IP of the EC2
  * Add the RSA key and user to access the computer using SSH.
* Keep the availability of the agent at all times.
* Save all the configurations

In the case of the agent disconnecting or if you happen to change the IP address follow the troubleshooting instructions

![offline](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/agent_offline.PNG)
![offline](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/agent_not_connected.PNG)
``` diff
-Troubleshooting: When working with the agent and free EC2s, it is expected that the public IP will change once the computer is turned off.
-You will need to update the public IP and set up the access for ssh to continue to work normally.
```
Once the problems are resolved you can relaunch the agent with the updated IP.

## Create a pipeline build on Jenkins
* Steps to take before startint the build
  * Access the EC2 inside the VPC using ssh or connect through aws console.
  * Modify the default file in nginx
  ```
  /etc/nginx/sites-enabled/default
  ```
  * Update the listening ports:
  ```
  server {
           listen 5000 default_server;
           listen [::]:5000 default_server;
  ```
  * Update the location settings:
    ```
    location / {
             proxy_pass http://127.0.0.1:8000;
             proxy_set_header Host $host;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    ```
``` diff
-Troubleshooting
-I initially encounter errors with working with the documentations and I was not able to run the application successfully and I had an error in the Jenkinsfile
-The documentation was updated for the jenkinsfile and the working and updated file is in the repository
```
* After all the configurations are correct you can create a new build

![deploy](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/deploy.PNG)

If everything is successfull you will see all stages of the build on color green.

# Additions
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


# Diagram

![diagram](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/diagram.PNG)

# Stack

![stack](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/STACK.PNG)





