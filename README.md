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

# 
- <b id="docker">Install docker (Jenkins Worker)</b>

```bash
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu && newgrp docker
```
#
- <b id="Sonar">Install and configure SonarQube (Master machine)</b>
```bash
docker run -itd --name SonarQube-Server -p 9000:9000 sonarqube:lts-community
```
#
- <b id="Trivy">Install Trivy (Jenkins Worker)</b>
```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update -y
sudo apt-get install trivy -y
```
#
- <b id="Argo">Install and Configure ArgoCD (Master Machine)</b>
  - <b>Create argocd namespace</b>
  ```bash
  kubectl create namespace argocd
  ```
  - <b>Apply argocd manifest</b>
  ```bash
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  ```
  - <b>Make sure all pods are running in argocd namespace</b>
  ```bash
  watch kubectl get pods -n argocd
  ```
  - <b>Install argocd CLI</b>
  ```bash
  sudo curl --silent --location -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.7/argocd-linux-amd64
  ```
  - <b>Provide executable permission</b>
  ```bash
  sudo chmod +x /usr/local/bin/argocd
  ```
  - <b>Check argocd services</b>
  ```bash
  kubectl get svc -n argocd
  ```
  - <b>Change argocd server's service from ClusterIP to NodePort</b>
  ```bash
  kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
  ```
  - <b>Confirm service is patched or not</b>
  ```bash
  kubectl get svc -n argocd
  ```
  - <b> Check the port where ArgoCD server is running and expose it on security groups of a worker node</b>
  ![Screenshot 2025-04-28 122145](https://github.com/user-attachments/assets/d58ae72c-acc0-4763-8c27-c56e62125a94)

  - <b>Access it on browser, click on advance and proceed with</b>
  ```bash
  <public-ip-worker>:<port>
  ```
  ![Screenshot 2025-04-28 122204](https://github.com/user-attachments/assets/d0393a09-60cd-4406-8213-64052abcd1f0)
  ![Screenshot 2025-04-28 122226](https://github.com/user-attachments/assets/8fe98148-f343-439c-be18-2ba9c5cca8f7)
  ![Screenshot 2025-04-28 122236](https://github.com/user-attachments/assets/22be503b-1022-4873-abe3-84cb2e3c7984)

  - <b>Fetch the initial password of argocd server</b>
  ```bash
  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
  ```
  - <b>Username: admin</b>
  - <b> Now, go to <mark>User Info</mark> and update your argocd password
#
#
## Steps to add email notification
- <b id="Mail">Go to your Jenkins Master EC2 instance and allow 465 port number for SMTPS</b>
#
- <b>Now, we need to generate an application password from our gmail account to authenticate with jenkins</b>
  - <b>Open gmail and go to <mark>Manage your Google Account --> Security</mark></b>
> [!Important]
> Make sure 2 step verification must be on

  ![Screenshot 2025-04-28 122629](https://github.com/user-attachments/assets/78859961-73c0-4166-ad84-bd26b4f90638)


  - <b>Search for <mark>App password</mark> and create a app password for jenkins</b>
  ![Screenshot 2025-04-28 122809](https://github.com/user-attachments/assets/be9241e0-4608-4acd-a8a6-9f444ad1bff4)
  ![Screenshot 2025-04-28 122907](https://github.com/user-attachments/assets/15201cc3-b033-4b9a-87f6-7b8e9eb9ccaa)

#
- <b> Once, app password is create and go back to jenkins <mark>Manage Jenkins --> Credentials</mark> to add username and password for email notification</b>
![Screenshot 2025-04-28 123003](https://github.com/user-attachments/assets/dd6ea14c-060c-4507-9b36-8b8562986491)

# 
- <b> Go back to <mark>Manage Jenkins --> System</mark> and search for <mark>Extended E-mail Notification</mark></b>
![image](https://github.com/user-attachments/assets/bac81e24-bb07-4659-a251-955966feded8)
#
- <b>Scroll down and search for <mark>E-mail Notification</mark> and setup email notification</b>
> [!Important]
> Enter your gmail password which we copied recently in password field <mark>E-mail Notification --> Advance</mark>



  





  




