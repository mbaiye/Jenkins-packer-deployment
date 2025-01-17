﻿# Techpet DevOps challenge part 2
## Jenkins-packer-deployment
For this exercise follow the instructions below.
## Intructions
1. Fork this repo.
2. Deploy Jenkins to an EC2 (with or without docker).
3. After successfully Deploying Jenkins, bake the image with Hashicorp Packer.
4. Deploy the baked custom AMI to an EC2.
5. For each of the proccesses above provide screenshots
6. Organize your code.
7. write a proper Readme.Md to explain your choices and the process.
<hr>


# Steps taken to complete task

1. Fork the repo.
2. Login to your AWS console and launch an Ubuntu t2.micro EC2 instance with ports 8080 and 22 allowed for inbound traffic.
3. ssh into the instance and run the following code line by line to install docker and Jenkins server.


## Docker Installation
1. Download packages to allow apt to use a repository over HTTPS:
```bash
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
```
2. Add Docker’s official GPG key:
```bash
  sudo apt-key fingerprint 0EBFCD88
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
```
3. Install Docker Engine
```bash
  sudo apt-get install docker-ce -y
```
4. Add ubuntu user to docker group 
```bash
  sudo usermod -aG docker ubuntu
```

## Jenkins Installation
1. Install the Java Runtime environment
```bash
  sudo apt install openjdk-11-jre -y
```
2. This is the Debian package repository of Jenkins to automate installation and upgrade. To use this repository, first add the key to your system: 
```bash
  curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```
3. Then add a Jenkins apt repository entry:
```bash
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```
4. Update your local package index, then finally install Jenkins: 
```bash
  sudo apt-get update
  sudo apt-get install jenkins -y
```
5. Reload the daemon,restart and enable Jenkins
```bash
  sudo systemctl daemon-reload
  sudo systemctl restart jenkins
  sudo systemctl enable jenkins
```
You can then navigate to the public ip address of the instance with port 8080 to complete the installation of Jenkins.

## Screenshots of Jenkins configuration

![Jenkins Configure](/images/Jenkins1.png)
![Jenkins Configure](/images/Jenkins5.png)


# Packer Task

Installing packer on the Mac using Homebrew is pretty straight forward.

1. First, install the HashiCorp tap, a repository of all our Homebrew packages.
```bash
  brew tap hashicorp/tap
```
2. Now, install Packer with hashicorp/tap/packer.
```bash
  brew install hashicorp/tap/packer
```
3. To update to the latest, run
```bash
  brew upgrade hashicorp/tap/packer
```
4. After installing Packer, verify the installation worked
```bash
  packer
```

# Create the Packer Template

Afterwards open an IDE of your choice and create a folder structure as below. 

![Packer Setup](/images/packer1.png)


1. A packer.json template file should be created with the code below.
```bash
  {
    "variables": {
        "aws_access_key": "",
        "aws_secret_key": ""
    },
    "builders": [
        {
            "type": "amazon-ebs",
            "access_key": "{{user `aws_access_key`}}",
            "secret_key": "{{user `aws_secret_key`}}",
            "region": "us-east-1",
            "instance_type": "t2.micro",
            "ami_name": "techpet-jenkins-ami-{{timestamp}}",
            "source_ami_filter": {
                "filters": {
                    "virtualization-type": "hvm",
                    "name": "ubuntu/images/*ubuntu-focal-20.04-amd64-server-*",
                    "root-device-type": "ebs"
                },
                "owners": ["099720109477"],
                "most_recent": true
            },
            "ssh_username": "ubuntu"
        }
    ],
    "provisioners": [
        {
            "type": "shell",
            "inline": ["sudo apt update -y && sudo apt upgrade -y"]
        },
        {
            "type": "shell",
            "script": "init.sh"
        }
    ]

}
```

2. Also a bash script init.sh with the code below should be created.
```bash
  #! /bin/bash

  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add
  sudo apt-key fingerprint 0EBFCD88
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
  sudo apt-get install docker-ce -y
  sudo usermod -aG docker ubuntu
  sudo apt install openjdk-11-jre -y
  curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
  sudo apt-get update
  sudo apt-get install jenkins -y
  sudo systemctl daemon-reload
  sudo systemctl restart jenkins
  sudo systemctl enable jenkins
```
3. save your files.

# Test the Template

1. Open the terminal and validate the template:
```bash
  packer validate packer.json 
```
2. Update the aws_access_key, aws_secret_key values in the variables object in the packer.json file with your own credentials.
3. Save your file.
4. Run the build:
```bash
  packer build packer.json
```
It will take a few minutes to complete. 

## Screenshots of created AMI and Launch of baked EC2

![AMI created](/images/packer3.png)
![Launch EC2 from AMI](/images/packer4.png)
![Baking](/images/packer5.png)


# Conclusion

After launching the EC2 instance from the custom built image by packer. Take the public ip address with port 8080 and continue with the installation of Jenkins. Docker also installed!

Encountered a few challenges along the way but it was an awesome learning experience!

I chose to use the bash way of provisioning as it is the simplest, compared to using a configuration manager!


