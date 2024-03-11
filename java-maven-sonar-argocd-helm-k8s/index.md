# Jenkins Pipeline for Java based application using Maven, SonarQube, Argo CD, Helm, and Kubernetes

<img width="1000" alt="Screenshot 2023-03-28 at 9 38 09 PM" src="https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/f017f64d-bd32-434b-a632-60f0db82e4b3">
Here are the step-by-step details to set up an end-to-end Jenkins pipeline for a Java application using SonarQube, Argo CD, Helm, and Kubernetes:

Prerequisites:

   -  Java application code hosted on a Git repository
   -  Jenkins server
   -  Kubernetes cluster
   -  Helm package manager
   -  Argo CD
     
### Step 1: Launched an Ec2 instance with the latest Ubuntu image, t2.large instance type, and installed Java JDK and Jenkins.
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/98a6f0cb-216a-4bee-8b3c-27ce6adbddff)

#### Commands to install Java and Jenkins
Install Java JDK and Jenkins

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

### Step 2: Create a new Pipeline job, configure it with the Git repository, and define the JenkinsFile path
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/350966db-1e74-4898-a212-44d358046606)

Jenkinsfile path: java-maven-sonar-argocd-helm-k8s/spring-boot-app/JenkinsFile

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/19c0fd8d-f59d-4181-b57f-ab1d73e80147)

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/9e18866f-5e16-4acb-b6b1-6714c2c1e96c)

#### Step 2.1: Go to the Jenkins server Plugin section and install the `Docker Pipeline` and `Sonarqube scanner` plugin

### Step 3: Install and Configure a Sonarqube on the Jenkins server:
```
sudo su –
apt install unzip
adduser sonarqube

sudo su – sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *

chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/         //Inside sonarqube-9.4.0.54424/bin/ can see different type of machines like windows, Linux, choose accordingly

./sonar.sh start
```
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/546db28d-b3c8-4a1a-a43e-2b215498853b)


Hurray !! Now, you can access the SonarQube Server at http://3.85.177.164:9000   
Wait for the server to load, and then enter the username and password as admin/admin and update it with a new strong password.

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/dbd09b1e-a344-4c58-8c88-ad3b03d309cf)

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/a44d88ac-fe46-4412-877c-390bd8e7725f)

#### Step 3.1: Now need to create a security token for Sonarqube to communicate with the Jenkins server
Steps to configure: 
Go to Sonarqube server 
```
>> My account
>> Security
>> Enter a Token name
>> Generate and copy 
```
Now go to the Jenkins server
```
>> Manage Jenkins
>> Manage credentials
>> System
>> Global credentials >> Add
>> Select Kind: Secret text
>> Paste the copied token in the Secret section and ID = "sonarqube"
>> Click on Create
```
### Step 4: Install Docker and Docker Slave Configuration
Run the below command to Install the Docker
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
http://3.85.177.164:8080/restart
```
The docker agent configuration is now successful.

### Step 5: Configure Kubernetes cluster on our local machine
Run the below commands to setup the K8S cluster 
```
minikube start –memory=4098 –driver=virtualbox
```
Go to the `operatorhub.io` website, search for **ArgoCD** Click on install. It will display the commands to run.
```
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.27.0/install.sh | bash -s v0.27.0
kubectl create -f https://operatorhub.io/install/argocd-operator.yaml
kubectl get pods –n operators
```
#### Step 5.1: Meanwhile, add ``Github and Docker credentials`` to the Jenkins server:
**For Docker:**
```Kind: Username and password for docker
Username of Docker hub and password
ID: "docker-cred" as in Jenkins file
Click on Create
```
**For Github:**
```
Kind: secret text
Secret: Enter the GitHub token
ID: github
Click on Create
```
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/d6626aa8-325a-40a7-afe4-884597ea11f2)






