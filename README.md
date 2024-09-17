# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

![New Microsoft PowerPoint Presentation](https://github.com/user-attachments/assets/dec01c09-8e5d-4861-9c5a-26e1114a86b7)

Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Requirements:
Java application code hosted on a Git repository, Jenkins server, Kubernetes cluster, Helm package manager, Argo CD

Steps:
1. Create a t2.large Ubuntu Instance on AWS, this will serve Jenkins, Maven, SonarQube, Docker. (We will not be using a free teir resource as the jenkins server and sonarqube will be very resource intensive hencce we choose a t2.large instance)
![image](https://github.com/user-attachments/assets/833c7dce-6322-4a65-8c22-ddd6cbccee4d)


2. Install Java and Jenkins

### Run the below commands to install Java and Jenkins

Install Java

```
sudo apt update
sudo apt install openjdk-17-jre
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

![image](https://github.com/user-attachments/assets/11af656c-f180-4bd2-9f9d-ad56b07c4a49)


**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

Check if jenkins is running
```
ps -ef | grep jenkins
```

Login to Jenkins, 
- Run the command to copy the Jenkins Admin Password - 
```
 sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![image](https://github.com/user-attachments/assets/f6220134-9834-4698-8a48-3d24c07f6722)

- Enter the Administrator password
- Click on Install suggested plugins
      
3. In Jenkins:

New Item -> Pipeline (Grovy Scripting) -> Pipeline Configuration -> Pipeline script from SCM -> Add the Path

![image](https://github.com/user-attachments/assets/c292fa87-56d4-45ec-ab76-9f950c445bf2)

4. Install Sonarqube

### Install the Sonarqube Plugin in Jenkins

![image](https://github.com/user-attachments/assets/cbde73da-0dd5-4c6d-b5f7-d4a09a5b7b1c)

### Configure a Sonar Server

```
apt install unzip
adduser sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```

Now access the `SonarQube Server` on `http://<ip-address>:9000` 

Generate a Sonarqube token through My Account -> Security -> Generate Token -> Copy -> Move to Jenkins -> Manage Credentails -> System -> Global Crdentails -> add credentails -> Secret Text -> Paste the token

5. Install Docker

Logout and go to root user and use the command:

```
sudo apt install docker.io
```
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

**Note: ** Use Docker Containers as Agent

Reason: Traditionally, when using Jenkins, you would need multiple worker EC2 instances (agents) to handle various build and test jobs. These EC2 instances would often remain running and consume resources even when not in use. This approach leads to unnecessary costs and underutilized resources since EC2 instances are typically up and running continuously. Even if you use Jenkins auto-scaling with EC2 instances, there's still overhead involved. Each new EC2 instance would require configuration, such as installing the necessary tools, dependencies, and build environment settings. This process can be time-consuming and error-prone, especially when scaling up or down frequently. Hence using Docker as Agent is better as it ensures Dynamic provisioning, Resource efficiency, Fast Provisioning, Consistency, Scalability, Portability.

### Install Docker Plugin
![image](https://github.com/user-attachments/assets/e682cbd0-dd8d-40c1-a59b-01ac732a2633)
 
We use the abhishekf5/maven-abhishek-docker-agent:v1 image as the agent, it already contains maven

6. Create a t2.large Ubuntu Instance on AWS, this will serve MiniQube and ArgoCD.

![image](https://github.com/user-attachments/assets/69c64034-009c-4d9e-990e-79dcfbba6ad0)

7. Install Minikube(K8) on the new instance

Run the following commands:
```
sudo su -
sudo apt update
```

### Install Docker
```
sudo apt install docker.io
```

### Install Minikube
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

### Setup Minikube
First Exit Root User and run these commands to install driver and give the permissions to group
```
sudo usermod -aG docker $USER && newgrp docker
minikube start --driver=docker
```
![image](https://github.com/user-attachments/assets/8e027877-6079-4045-80b9-34a4bb236111)

### Install Kubectl
```
sudo snap install kubectl --classic
```

8. Installing Kubernetes Controller(ArgoCD) through Operators

**Note: ** Installing Kubernetes Controllers through Operators is a best practice because Operators automate the management and lifecycle of Kubernetes controllers/applications, including deployment, scaling, upgrading, and recovery. Operators provide automated management of controllers, handling installation, upgrades, configuration changes, and health monitoring without manual intervention. We will use operatorhub.io 

### Install ArgoCD Operator

![image](https://github.com/user-attachments/assets/8f37780a-5be5-4b03-b39d-18a2a03d3d56)


Install Operator Lifecycle Manager (OLM), a tool to help manage the Operators running on your cluster.
```
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.28.0/install.sh | bash -s v0.28.0
```
This is one time for an instance
![image](https://github.com/user-attachments/assets/4381ccbc-7674-46ce-a15f-cf4f98137c48)

Install the operator by running the following command:
```
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
```
This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster. (This basically installs the ArgoCD Operator)


Verify the install using this:
```
kubectl get pods -n operators
```
![image](https://github.com/user-attachments/assets/7e71628e-3b10-45d4-be3b-4c65d8f0f51d)

9. Make the Jenkins File/Pipeline

**Note: ** Now what are the different stages in a Jenkins Pipeline, they are nothing but the different blocks that we are trying to build using the pipeline.

First, Checkout Stage (which is not required in our case as our jenkins file is already in the scm, if it was not in the scm and written in Jenkins UI then we would need the checkout stage)

Second, Build and Test, as maven is already installed in the docker container we only need to run
```
mvn clean package
```
After executing the command, a target folder is created which stores the web archive file and the docker file is just configured to take the jar file and run it on port 8080

Third, Static Code Analysis, we have configured sonar token and we will execute the static code analysis using the command:
```
mvn sonar:sonar -Dsonar.login=$Sonar_AUTH_TOKEN -Dsonar.host.url=${Sonar_URL}
```

Fourth, Docker Build and Push, we will be passing docker credentials, building the image and pushing to DockerHub using the ususal docker commands.

Fifth, Update Deployment File, in this stage a shell script will be executed to to replace the image tag in deployment.yaml with with the currecnt build number/version number of the image. It requires github credential as we are then pushing the new deployment file to the scm.
```
sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" CI-CD-PIPELINE-SETUP-WITH-JENKINS-DOCKER-AND-ARGO-CD/spring-boot-app-manifests/deployment.yml
```

### Add Docker and GitHub Credentials

Add the credentials in the same way we added the SonarQube credentials
![image](https://github.com/user-attachments/assets/191e9d61-28b0-40b3-972a-6e5b4ce1c4eb)

10. Run The CI Pipeline (Without the ArgoCD Part)

![image](https://github.com/user-attachments/assets/269ede1d-f1d1-45d5-b907-53bd74821cb6)

