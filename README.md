Github repo:

https://github.com/abhigit84/CICD-project-jenkins-shared-library1.git

https://github.com/abhigit84/CICD-Project-with-ansible.git

Region:us-west-2
ubuntu,t2.medium


1)Install jdk on AWS EC2 Instance 

sudo apt-get update

sudo apt install openjdk-11-jre-headless -y

java --version


2)Install and Setup Jenkins

#!/bin/bash

sudo apt update -y

sudo apt upgrade -y 

sudo apt install openjdk-17-jre -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y 
sudo apt-get install jenkins -y
sudo systemctl status jenkins

Change the security group

security groups-edit inbound rules-
add rule-all traffic-anywhere on ipv4-add rule
Now u can see 8080 port predefined.u dont have to configure port.

http://ec2 public ip:8080
this is master url

 cat /var/lib/jenkins/secrets/initialAdminPassword
put password-which is diplayed by above command in jenkins
install all suggested plugins
all admin(username,password)
abhinoi84@gmail.com
jenkins url-http://ec2 ip:8080/
Note for jenkins login-all admin(username,password)


3)Process to make Jenkins as rootuser

Update visudo and assign administration privileges to jenkinsuser - Open the file /etc/sudoers in vi mode 
sudo vi /etc/sudoers 
-Add the following 2 lines at the end of the file 

jenkins ALL=(ALL) NOPASSWD: ALL

@includedir /etc/sudoers.d

-After adding the line save and quit the file(esc:wq!).
Now we can use Jenkins as rootuser and for that run the following command:

sudo su - jenkins


4)Install Docker with user jenkins

Make sure u are at jenkins user,if no run sudo su - jenkins again (jenkins@ip-172-31-22-7) and not ubuntu.
basically adding in sudo cat /etc/group jenkins user to docker group to use docker commands without sudo.

sudo apt install docker.io
docker--version
docker ps
sudo usermod -aG docker jenkins

5)Install and Setup AWSCLI

AWSCLI:-

sudo apt install unzip

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

6)Install Eksctl

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

eksctl version

7)Configure AWSCLI so that it can authenticate and communicate with AWS enviroments
Create role with administraor access and generate access keys after clicking on security credentials tab.
Create a role with admin user rights and update in ec2.
Always better to create role only so that u dont need to do aws configure and all.
aws configure and give all 4 things.
you can validate if its done by doing aws s3 ls

8)Install and setup kubectl

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin 

kubectl version

9)Creating an Amazon EKS cluster using eksctl(unlike document m creating t2.medium and not t2.micro)

eksctl create cluster --name first-eks-cluster --version 1.24 --region us-west-2 --nodegroup-name worker-nodes --node-type t2.medium --nodes 2

always run this:
aws eks update-kubeconfig --region us-west-2 --name first-eks-cluster


10)Create IAM OIDC Provider(We do this as Ec2 access to EKS and from EKS also it needs to talk to EC2 and thats why we create OpenId Connect..we use these shell scripts create id,pass id,aatach openid to our cluster)

oidc_id=$(aws eks describe-cluster --name first-eks-cluster --region us-west-2 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

eksctl utils associate-iam-oidc-provider --cluster first-eks-cluster --approve --region us-west-2


11)Create IAM service account with role(This service account is to access ebs volume and Backend systems,create iam role for service accounts under kube-system so that kubernetes can talk to amazon systems):-

eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster first-eks-cluster --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve --role-only --role-name AmazonEKS_EBS_CSI_DriverRole --region us-west-2

12)Then attach role to eks by running the following command(so basically just like ec2 u gave iam role,u give kubernetes also a iam role so that both communicate with each other):-

Go to IAM-See role created with name-AmazonEKS_EBS_CSI_DriverRole-Click on it-copy account no-027183218661 from arn (and use in below command)-arn:aws:iam::027183218661:role/AmazonEKS_EBS_CSI_DriverRole


eksctl create addon --name aws-ebs-csi-driver --cluster first-eks-cluster --service-account-role-arn arn:aws:iam::027183218661:role/AmazonEKS_EBS_CSI_DriverRole --force --region us-west-2

13)Add Dockerhub and Github Credentials on Jenkins(so repeat these process 2 times one for dockerhub and one for github)

Dockerhub:

jenkins dashboard -manage jenkins-credentials-system-click on global credentials-
add kind-use username with password-username- and password--id-docker_cred-Create


Github:

jenkins dashboard -manage jenkins-credentials-system-click on global credentials-
add kind-use username with password-username-and password--id-GIT_HUB_CREDENTIALS-Create

14)Add maven in global tool configuration(as u not installing maven on ec2 so u using maven from jenkins like this)

Manage jenkins-Tools-maven installations-add maven-name-maven3-version-2.0.1

15)Add Jenkins shared library

dashboard-manage jenkins-configure system-ctrl f(global pipeline library)-
name(jenkins-shared-library)this is same name as mentioned in ur main Jenkinsfile otherwise will not work in github repo-default version-main
SCM-git-repo url-https://github.com/abhigit84/CICD-project-jenkins-shared-library1.git

16)Build,deploy and test CI/CD pipeline

Create Pipeline:-

dashboard-new item-name-pipeline-ok

Now in configure-Pipeline-Pipeline script-copy jenkinsfile code from https://github.com/abhigit84/CICD-Project-with-ansible.git
and paste here.

Note:make sure ur dockerhub id is given in Jenkinsfile.

17)Ansible Python Setup

sudo apt update 
sudo apt install software-properties-common 
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
sudo apt install python3
sudo apt install python3-pip
pip3 install Kubernetes

docker login
uername
password


18)select build with parameters in jenkins

Action-Create
Imagename-kubernetes-configmap-reload
Imagetag-v1
Appname-kubernetes-configmap-reload
Docker_repo-abhinoi84

Click on Piepline-you will get option-build with parameters.


When you get JAVA_HOME error in pipeline at mvn stage:-
on aws console run:-
echo "JAVA_HOME='java installation path'" >> .bashrc


19)Add Webhook

On Github:-

select this repo-
https://github.com/abhigit84/CICD-project-jenkins-shared-library1.git
and go to Settings-webhooks-Payload URL-http://52.25.255.24:8080/github-webhook/
Just the Push event
Add webhook
Note:above ip is jenkins instance public ip and add url exactly like this.

On Jenkins:-

select your pipeline-Configure-General tab-Build Triggers-select checkbox(GitHub hook trigger for GITScm polling)

20)delete cluster

eksctl delete cluster --name first-eks-cluster


Note:After jenkins installation please ensure all steps run with jenkins user only(and not ubuntu user)
sudo su - jenkins
10,11,12 steps done just for safe side.
