# END-TO-END CI/CD PIPELINE FOR A SPRING BOOT APPLICATION

END-TO-END CI/CD pipeline for  a Spring Boot application using Jenkins, Maven, SonarQube, Argo CD, Helm and Kubernetes 

## Spring Boot based Java web application


This is a basic Java application built with Spring Boot and managed using Maven. The project's dependencies are defined in the pom.xml file located in the root directory.

The application follows the MVC (Model-View-Controller) architecture, where controller returns a page with title and message attributes to the view.

## Execute the application locally and access it using your browser

Checkout the repo and move to the directory

```
git clone https://github.com/Mp886/CI-CD-END-TO-END-PROJECT/java-maven-sonar-argocd-helm-k8s/sprint-boot-app
cd java-maven-sonar-argocd-helm-k8s/sprint-boot-app
```

Execute the Maven targets to generate the artifacts

```
mvn clean package
```

The Maven build stores the artifacts in the target directory. You can either run the artifact on your local machine or run it as a Docker container.

** Note: To avoid issues with local setup, Java versions and other dependencies, I would recommend the docker way. **

## Execute locally (Java 11 needed) and access the application on http://localhost:8080

```
java -jar target/spring-boot-web.jar
```

## The Docker Way

Build the Docker Image

```
docker build -t ultimate-cicd-pipeline:v1 .
```

Now run the docker image

```
docker run -d -p 8010:8080 -t ultimate-cicd-pipeline:v1
```

Hurray !! Access the application on " http://<ip-address>:8010 ".

This is just to see how the application looks. This is not a part of CI/CD implementation.

## Next Steps

# AWS EC2 Instance

Go to AWS Console, launch an ec2 instance with your desired AMI(Preferable Ubuntu). Select instance type as "t2.large" because, installing all the required tools may take more space. Start installing the below CI/CD tools in your instance.


# Install Jenkins

Pre-Requisites:

- Java (JDK)

## Run the below commands to install Java and Jenkins

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
**Note: By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as shown below.

- EC2 -> click on instance ID
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules (you can just allow TCP 8080 as well, in my case, I allowed All traffic).

## Login to Jenkins using the below URL:

http://:8080 [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

**Note: If you are not interested in allowing All Traffic to your EC2 instance 1. Delete the inbound traffic rule for your instance 2. Edit the inbound traffic rule to only allow custom TCP port 8080.

After you login to Jenkins, Run the below command to copy the Jenkins Admin Password 

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword 
```
## Steps to create and use Jenkins

1) Enter the Administrator password 
2) Click on Install suggested plugins  
3) Wait for the Jenkins to Install suggested plugins 
4) Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user] 5) Jenkins Installation is Successful. You can now starting using the Jenkins.

## Install the Docker Pipeline plugin in Jenkins

- Log in to Jenkins.
- Go to Manage Jenkins -> Manage Plugins.
- Go to Available Plugins -> search for "Docker Pipeline".
- Select the plugin -> click the Install button.
- Restart Jenkins after the plugin is installed.(http://<ec2-instance-public-ip>:8080/restart)

Wait for the Jenkins to be restarted.

# Install Docker

Run the below command to Install Docker

```
sudo apt update
sudo apt install docker.io
```

## Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins
usermod -aG docker ubuntu
systemctl restart docker
```
Once you are done with the above steps, it is better to restart Jenkins again.

```
http://<ec2-instance-public-ip>:8080/restart
```
The docker agent configuration is now successful.

## Install the SonarQube Scanner plugin in Jenkins

- Log in to Jenkins.
- Go to Manage Jenkins -> Manage Plugins.
- Go to Available Plugins -> search for "SonarQube Scanner".
- Select the plugin -> click the Install button.
- Restart Jenkins after the plugin is installed.(http://<ec2-instance-public-ip>:8080/restart)

# Install SonarQube

## Configure a Sonar Server locally

```
sudo su -
adduser sonarqube
```
Give a new password and re-enter the new password. And click on enter till it says "Is the information correct?".

After that, reun the below commands

```
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
apt install unzip
unzip *
chmod -R 755 /home/sonarqube/sonarqube-9.4.0.54424
chown -R sonarqube:sonarqube /home/sonarqube/sonarqube-9.4.0.54424
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start
```
**Now you can access the SonarQube Server on http://<ec2-instance-public-ip>:9000**

- Now to create an sonarqube account, Initially you need to login by giving the username: admin & Password:admin. 
- After logging in, now it asks to update the password. Give the required things and login into your account.

## Authentication between Jenkins and SonarQube

Follow the below steps for the authentication.

1) Go to sonarqube and click on "A" on the top right corner. -> Go to my account
2) Click on security tab -> provide a name to generate token (example: jenkins) and click on generate.
3) Copy the token. -> Go to jenkins dashboard -> Click on manage jenkins -> click on manage credentials or just credentials.
4) click on system -> click on Global credentials(unrestricted) -> Add credentials -> Now add your new credentials.
5) Under "kind"(select secret text). -> Under "secret" paste your sonarqube token. -> Under "ID" give a name (example:sonarqube). -> Click on create.

**DONE*

**NOTE: RESTART JENKINS WHENEVER YOU CONFIGURE SOMETHING OR ADD ANY PLUGINS IN JENKINS**

Now restart the jenkins once again by (http://<ec2-instance-public-ip>:8080/restart)

## Create a Docker Hub Account

Create a docker hub account by giving your email, username and password.

## Authentication between Jenkins and Docker Hub

 Follow the below steps for the authentication.

1) Go to jenkins dashboard -> Click on manage jenkins -> click on manage credentials or just credentials.
2) click on system -> click on Global credentials(unrestricted) -> Add credentials -> Now add your new credentials.
3) Under "kind"(select username and password). -> Under "username" give your dockerhub username.-> Under "Password" give your dockerhub password. -> Under "ID" the input as " docker-cred ". -> Click on create.

## Authentication between Jenkins and Github

1) Go to your github  -> Go to settings -> Click on Developer Settings -> Click on personel access token -> click on classic tokens -> generate a new token.
2) Copy the token. -> Go to jenkins dashboard -> Click on manage jenkins -> click on manage credentials or just credentials.
4) click on system -> click on Global credentials(unrestricted) -> Add credentials -> Now add your new credentials.
5) Under "kind"(select secret text). -> Under "secret" paste your github token. -> Under "ID" give a name (example:github). -> Click on create.

Restart the jenkins once again because you have done some configurations.

# ***Update the required fields in Jenkins file***

The fields that have to be updated in the Jenkins file 

- Under the static code analysis stage. Give the URL of your sonarqube public ip address of instance (example:http://<ec2-instance-public-ip>:9000)

- Under the build and push docker image stage. give your docker hub username (example: <your-dockerhub-username>/ultimate-cicd) 

- Under the update deployment file stage. Give the Github_repo_name and github_user name according to your's repo and user names. and even the user email and name.

Once the fields are updated. Now go to your pipeline and click hte Build now option. Now the building of the pipline starts.

Once you passed all the stages and get succeeded. Your good till the CI part.

Till here, all the configuration part and installation part is completed for CI part. That is for Continuous Integration.

Now lets Start installing the tools required for CD(Continuous delivery or deployment) part.

It is better to install the below tools in other instance or in your local machine because already the ec2 instance has been heavy load.

# Install Kubectl

Install the kubernetes CLI that is "kubectl".

Run the below command to install "kubectl"

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

## Install Minikube 

As we are doing this project in Linux. I will give instructions how to download minikube in linux

Just run the below commands to install minikube

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```

Your installation is successfull.

Now Start your minikube by running the below command

```
minikube start
```

Now, we need to install any GITOPS tool to deploy the application in kubernetes. So, for that we are using ARGOCD.

# Install ArgoCD

Just run the below commands to install argocd

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Now, the Argo cd is intalled.

To check whether the argocd is running and installed. Run the below command and check whether they are running

```
kubectl get pods -n argocd
```

Now, check the argocd services by running the below command

```
kubectl get svc -n argocd
```
Now you will get all the services of the argocd. Now you have to edit the "argocd-server" from cluster Ip to NodePort IP by running the below command. Once you run the below command, you will be taken to an editer where you need to edit the cluster Ip to NodePort IP under Selector -> type:.

```
kubectl edit svc argocd-server
```
Now run the below commands to get the URL of your argo cd server

```
minikube service argocd-server -n argocd
```

You will get the ip address to access the argocd. copy and paste it in your browser.

- Now you need to give username and password. For username:admin and you have to run some commands to get the password.

- Open a new terminal and run the below command

```
kubectl get secret -n argocd
```

All the secrets will be displayed. Now run the below command to see the encrypted password

```
kubectl edit secret argocd-initial-admin-secret -n argocd
```

You will be open an editor. Now copy the encrypted password. and run the below command

```
echo <paste the encrypted password> | base64 --decode
```

You will get a decrypted password. copy that password and paste it under the password field in the browser.

Now you will be logged into the argocd.

## Create an application in argocd

1) Click on create application -> Give the application name (example:test) 
2) Project name: Default
3) Sync Policy: Automatic
4) Source -> github URL: (your github url)
5) path: give the path of your deployment file. for example in my case(java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests)
6) Destination -> Cluster URL: https://kubernetes.default.svc
7) Namespace as "default"
8) Click on create.

Now automatically the application will be synced.

******************************************************************THE END*******************************************************************









