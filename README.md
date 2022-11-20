<img src="https://github.com/kura-labs-org/kuralabs_deployment_1/blob/main/Kuralogo.png">
<h1 align="center">kuralabs_deployment_5<h1> 
  
Demonstrate your ability to deploy a containerized application.

## Deployment Document Link:
- Link to instructions: https://github.com/kura-labs-org/kuralabs_deployment_5/blob/main/Deployment-5_Assignment.pdf






<h1 align=center>Deployment 3</h1>
<div align=center>Deploy a flask application through a jenkins agent on a VPC</div>

# Table of contents

1. Set up and configure VPC
2. Install Jenkins on an EC2
3. Create an EC2 on the public subnet of your VPC
4. Configure the Jenkins agent on the VPC
5. Create a pipeline build on Jenkins
6. Additions
7. Diagram

## Set up and configure VPC
* Create a VPC on AWS
* Create 2 subnets a private and a public subnet 
* Configure the internet gateway
* Configure the routing tables

![tables](https://github.com/Antoniorios17/kuralabs_deployment_3/blob/main/images/routing%20tables.PNG)

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





