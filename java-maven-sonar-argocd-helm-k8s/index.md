# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm, and Kubernetes

<img width="1000" alt="Screenshot 2023-03-28 at 9 38 09 PM" src="https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/f017f64d-bd32-434b-a632-60f0db82e4b3">
Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Prerequisites:

   -  Java application code hosted on a Git repository
   -  Jenkins server
   -  Kubernetes cluster
   -  Helm package manager
   -  Argo CD
     
### Step 1: Launched an Ec2 instance with latest Ubuuntu image, t2.large instance type and installed Java jdk and Jenkins.
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/98a6f0cb-216a-4bee-8b3c-27ce6adbddff)

#### Commands to install Java and Jenkins

Install Java jdk and Jenkins

```
sudo apt update
sudo apt install openjdk-11-jre

curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
Launched Jenkins application at http://3.85.177.164:8080

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/e641df66-9c1c-427f-9344-21a0f1f4d232)

### Step 2: Create a new Pipeline job and configure it with Git repository and define JenkinsFile path
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/350966db-1e74-4898-a212-44d358046606)

Jenkinsfile path: java-maven-sonar-argocd-helm-k8s/spring-boot-app/JenkinsFile

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/9b4b8354-b2d8-4da5-ba1b-000c9faa936d)
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/f3a1e177-4d61-4131-9957-840083d4a442)

## Install the Docker Pipeline plugin in Jenkins:

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline".
   - Select the plugin and click the Install button.
   - Restart Jenkins after the plugin is installed.
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.


## Docker Slave Configuration

Run the below command to Install Docker

```
sudo apt update
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




