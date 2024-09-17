# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

![New Microsoft PowerPoint Presentation](https://github.com/user-attachments/assets/dec01c09-8e5d-4861-9c5a-26e1114a86b7)

Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Requirements:
Java application code hosted on a Git repository, Jenkins server, Kubernetes cluster, Helm package manager, Argo CD

Steps:
1. Create a t2.large Ubuntu Instance on AWS. (We will not be using a free teir resource as the jenkins server and sonarqube will be very resource intensive hencce we choose a t2.large instance)
![image](https://github.com/user-attachments/assets/418cb65f-c7d6-4744-84f6-ad09ccf1cc00)

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


