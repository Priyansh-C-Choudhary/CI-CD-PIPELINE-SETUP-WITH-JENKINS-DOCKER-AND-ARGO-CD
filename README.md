# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm and Kubernetes

![New Microsoft PowerPoint Presentation](https://github.com/user-attachments/assets/dec01c09-8e5d-4861-9c5a-26e1114a86b7)

Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Requirements:
Java application code hosted on a Git repository, Jenkins server, Kubernetes cluster, Helm package manager, Argo CD

Steps:
1. Create a t2.large Ubuntu Instance on AWS. (We will not be using a free teir resource as the jenkins server and sonarqube will be very resource intensive hencce we choose a t2.large instance)
2. 
