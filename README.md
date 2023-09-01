
# Deploying a Static Website to an EC2 Instance Using GitLab CI

Deploying a Static Website to an EC2 Instance Using GitLab CI

##  Prerequisites

Before getting started, make sure you have the following:

1) An EC2 instance running on AWS (Amazon Web Services).
2) Your website's static files ready for deployment.
3) GitLab account and a GitLab repository containing your website's source code.

##  Step 1: Configure Your EC2 Instance

Ensure that your EC2 instance is set up correctly with SSH access. You'll need to have your instance's public IP address and an SSH key pair (.pem) for authentication.

Once connected to your EC2 instance, you can perform the following tasks:

//Update the system package list:

sudo yum update -y

//Install a web server (e.g., Apache) if not already installed:

sudo yum install httpd -y

//Start the web server:

sudo systemctl start httpd

//Check status of web server 

sudo systemctl status httpd

//check location 

pwd

//giving full access to html file 

chmod 777 ./html 



##  Step 2: Prepare Your GitLab Repository

1) In your GitLab repository, go to the "Settings" section and navigate to "CI/CD > Variables".
2) Add a variable named ibrahimkey and paste your SSH private key as the value. Add user and paste your user name (ec2-user) and add ip and paste your instance ip. This key will be used for secure access to your EC2 instance.Similarly user and ip will be used to connect to your EC2 instance.

##  Step 3: Create a .gitlab-ci.yml File

In your GitLab repository, create a .gitlab-ci.yml file with the following content:

stages:
  - deploy

deploy:

  stage: deploy

  script:

    # Update the package list on the server quietly (-q) and automatically confirm (-y) any prompts.
    - apt-get update -qy

    # Install sshpass which is a tool for non-interactive SSH authentication.
    - apt-get install -y sshpass

    # Create an SSH private key file named id_rsa (with the generic key name where rsa is a key format).
    # Write the contents of the $ibrahimkey variable to it.
    - echo "$ibrahimkey" > id_rsa

    # Set permissions for the private key to ensure it's secure (chmod = change mode).
    - chmod 600 id_rsa

    # Create a directory ~/.ssh if it doesn't already exist to store SSH configuration.
    - mkdir -p ~/.ssh

    # Use ssh-keyscan to add the SSH host key of $ip to the known_hosts file in ~/.ssh.
    - ssh-keyscan -H $ip >> ~/.ssh/known_hosts

    # Use scp (Secure Copy) to recursively (-r) copy all files and directories (*) 
    # using the private key (-i id_rsa) to the $user@$ip server under /var/www/html/.
    - scp -r -i id_rsa * $user@$ip:/var/www/html/

    # This job should only be executed when changes are pushed to the "main" branch.
  
  only:
    - main

This YAML file defines a CI/CD pipeline to deploy your website.

##  Step 4: Trigger the Deployment

1) Push changes to your GitLab repository's "main" branch.

2) GitLab CI/CD will automatically trigger the deployment job specified in the .gitlab-ci.yml file.

3) Your static website files will be copied to your EC2 instance under the /var/www/html/ directory.

##  Step 5: Access Your Deployed Website

Once the CI/CD job completes successfully, you can access your deployed static website using your EC2 instance's public IP address in a web browser.
##  Problems 

1) In EC2 instance we check the access of files because I am facing this error 
 
## error

$ scp -r -i id_rsa * $user@$ip:/var/www/html/

scp: dest open "/var/www/html/README.md": Permission denied

scp: failed to upload file README.md to /var/www/html/

scp: dest open "/var/www/html/id_rsa": Permission denied

scp: failed to upload file id_rsa to /var/www/html/

scp: dest open "/var/www/html/index.html": 

Permission denied

scp: failed to upload file index.html to /var/www/html/

## solution 

So I checked the permission of my ec2 instance by 

cd /var/www/html/ 

//then list the items in this file by 

ls -al 

// then we move one step back by 
 
cd ../

// again listing all item in that list of cd /var/www/

ls -al 

// we find that html has some limited permissions
// Then we enchance permission of html and give it full access 

chmod 777 ./html

2) yml file issues 

##error

I am facing problem in connecting my git with ec2 because fingerprint was not matching 

## soltuion 

We set public ip in a known host by in yml file of git 

- ssh-keyscan -H $ip >> ~/.ssh/known_hosts

