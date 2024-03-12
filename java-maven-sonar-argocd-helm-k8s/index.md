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

Hurray !! Now, we can access the SonarQube Server at http://3.85.177.164:9000   
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

#### Step 5.2: Update the Sonarqube IP address of Jenkinsfile and the Image Tag name on the deployment.yml file.
- We have to update the `SONAR_URL` on JenkinsFile (`java-maven-sonar-argocd-helm-k8s/spring-boot-app/JenkinsFile`) with the EC2 instance public IP address or IP address where the Sonarqube is installed so that the Git source code can be analyzed by Sonarqube server.
- The Docker Image tag-name on the deployment.yml file (`java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml`) as it is pulled from the Docker Hub so that can be replaced by the `Build-Number` when .jar file is generated during the Jenkins Pipeline execution.

### Step 6: The next step is to execute/run the Jenkins Pipeline
Try running, Jenkins Pipeline, monitor the pipeline stages, and see if all the stages pass without any error.

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/20f5ffb9-c0e6-48b7-b186-44dfa87575cb)
Screenshot of Console Output with Success message of the Jenkins Pipeline:

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/fdc01365-360b-49f7-a8dd-a67f0818f246)

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/e51e38e1-4c71-4db2-b95a-debb1082ceea)

Will check if the Maven build has pushed the report to Sonarqube
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/4198be74-41ca-44a9-bd63-deacfa2df7dc)


#### Step 6.1: Configuring the ArgoCD server
Screenshot of ArgoCD Operator status
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/73b3ec57-deb5-4f33-9159-9d17d6e75251)

Creating the Argo CD cluster with the default configuration
```vim argocd-basic.yml```       
```
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: example-argocd
  labels:
    example: basic
spec: {}
```
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/52579b1e-8368-4580-b5b2-6aeecb5349a2)

Now, can see the ArgoCD cluster pod's status:
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/1d4dc8d5-7d05-4a97-b75a-a4aac2092f77)
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/a0f75291-e1f7-49fb-bd25-012183b8c6b7)

Screenshot of ArgoCD services With the `example-argocd-server` status as ClusterIP:
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/f88c9c87-eaf9-4b6e-bf3e-8f88c4c49238)

In the above screenshot, we can see `example-argocd-server` status as ClusterIP in which we won't be able to access the ArgoCD server on the website, so to make it accessible on the website using the IP address we will change the **TYPE** to ``NodePort`` mode.

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/52f644e1-8910-4de8-96fc-42bb6c1ce5dc)
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/432af796-d68f-47cf-9c63-35cbe474d354)

Now, we see the changes in the TYPE mode
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/1fd96cff-a2b0-4c7c-b244-e128f41a2359)

Run the command on Minikube to generate the URL to execute the ArgoCD server on the browser:

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/4c4cba07-9642-4978-b1b6-7c97aac4deca)

Using the Minikube-generated URL, now we can access the ArgoCD server on the browser. Before that make sure that all ArgoCD pods are up and running

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/cfcf0da4-3356-411c-a466-d5c0c1ad980c)

Click on Advanced >> Accept Risk and continue.

Enter the username as Admin and for Password need to get it from ``example-argocd-cluster`` file.

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/8a5dbd6d-b0cb-4902-95fe-195ffebb526b)

Run the command ` kubectl edit secret example-argocd-cluster ` and copy the password from ``admin.password`` and execute the command ` echo 'copiedpassword' | base64 -d ` to decrypt the password and use the newly created password to login to ArgoCD server.


![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/dd84643e-f74c-412f-bf0e-2f7a10802246)

Click on Create Application to pull the data and manage the resouce on the cluster

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/37d3c71f-6772-436e-bbe8-0d260fe2418b)

![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/bd3f9675-9f9c-4dc1-960e-406774e9c044)
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/8fc4c482-95e7-4589-94b0-3b250bfd8a16)

Here can see the deployed `spring-boot-app` on the cluster with healthy state
![image](https://github.com/Jayalakshmi-i/Jenkins-End-to-End-CICD/assets/141424247/07d8e9ac-7fce-4ab4-8ca0-06034d446737)











