# Bankapp - A multi account bank 

# Bankapp Project End to End Implementation

### In this demo, we will see how to deploy an end to end three tier MAVEN application on EKS cluster.

### <mark>Project Deployment Flow:</mark>

![Screenshot 2025-04-28 111514](https://github.com/user-attachments/assets/57e0d697-4a21-4b1d-bdf0-2e81056da196)

## Tech stack used in this project:
- GitHub (Code)
- Docker (Containerization)
- Jenkins (CI)
- OWASP (Dependency check)
- SonarQube (Quality)
- Trivy (Filesystem Scan)
- ArgoCD (CD)
- Redis (Caching)
- AWS EKS (Kubernetes)
- Helm (Monitoring using grafana and prometheus)

### Pre-requisites to implement this project:

> [!Note]
> This project will be implemented on United States(Ohio) region (us-west-1).

- <b>Create 1 Master machine on AWS with 2CPU, 8GB of RAM (t2.large) and 29 GB of storage and install Docker on it.</b>

- <b>Open the below ports in security group of master machine and also attach same security group to Jenkins worker node (We will create worker node shortly)</b>
![361331879-4e5ecd37-fe2e-4e4b-a6ba-14c7b62715a3](https://github.com/user-attachments/assets/d5330aa1-b66e-4656-b15b-cb96e1bbfac4)

> [!Note]
> We are creating this master machine because we will configure Jenkins master, eksctl, EKS cluster creation from here.

```bash
sudo apt-get update
```
```bash
sudo apt-get install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker
```

#
- <b id="Jenkins">Install and configure Jenkins (Master machine)</b>
```bash
sudo apt update -y
sudo apt install fontconfig openjdk-17-jre -y

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
  
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  
sudo apt-get update -y
sudo apt-get install jenkins -y
```
- <b>Now, access Jenkins Master on the browser on port 8080 and configure it</b>.
#
- <b id="EKS">Create EKS Cluster on AWS (Master machine)</b>
  - Clone this Repository for creating EKS Cluster.
  ```bash
  git clone https://github.com/krunalp1908/EKS-Terraform.git
  ```
  ```bash
  cd EKS-Terraform
  ```
  - Change the ssh-public-key in variable.tf according to your region and also update the region in main.tf file accorfing yo your region.
  ```bash
  terraform init
  ```
  ```bash
  terraform apply --auto-approve
  ```

  > [!Note]
>  Make sure the ssh-public-key "eks-nodegroup-key is available in your aws account"

#
- <b id="Jenkins-worker">Setting up jenkins worker node</b>
  - Create a new EC2 instance (Jenkins Worker) with 2CPU, 8GB of RAM (t2.large) and 29 GB of storage and install java on it
  ```bash
  sudo apt update -y
  sudo apt install fontconfig openjdk-17-jre -y
  ```

  - Create an IAM role with <mark>administrator access</mark> attach it to the jenkins worker node <mark>Select Jenkins worker node EC2 instance --> Actions --> Security --> Modify IAM role</mark>
  ![361370600-1a9060db-db11-40b7-86f0-47a65e8ed68b](https://github.com/user-attachments/assets/9e567faf-2bc6-4908-ac6f-236382920e1d)

  - Configure AWSCLI (<a href="https://github.com/DevMadhup/DevOps-Tools-Installations/blob/main/AWSCLI/AWSCLI.sh">Setup AWSCLI</a>)
  ```bash
  sudo su
  ```
  ```bash
  curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
  sudo apt install unzip
  unzip awscliv2.zip
  sudo ./aws/install
  aws configure
  ```

#
  - <b>generate ssh keys (Master machine) to setup jenkins master-slave</b>
    •	Now assign private and public key to respective master server and agent node. 
    •	Now on master server got to : - cd ~/.shh folder and write ssh-keygen 
    •	This will generate one private and public key i.e : - id_rsa id_rsa.pub
    •	For copying public key on agent node do below option
    •	On master node cat id_rsa.pub and copy the kye 
    •	On agent node go to: - cd .ssh and vim authorised keys
    •	And paste the keys in that file and save it.
    •	On master node while creating node. You will get option name launch method.
    •	In that option select option launch agent via SSH
    •	In host paste the ip address of agent node ip
    •	In credentials : - SSH Usernames with private key
    •	And enter the private key there.
    •	Save it and relaunch agent.

#
  - And your jenkins worker node is added
  ![357403596-cab93696-a4e2-4501-b164-8287d7077eef](https://github.com/user-attachments/assets/a37ed689-0131-4b49-ba26-d3c51c9db0f0)






