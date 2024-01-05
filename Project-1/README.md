# Jenkins Declarative CI/CD Pipeline Project
<img width="809" alt="Project-1 Diagram" src="https://github.com/MohanPiru/devops-projects/assets/140044323/55cfb9ab-84fb-4033-8ab0-a9484a92fed7">

It is a simple Devops project using jenkins CI/CD pipeline. I am using a Django notes app code [freely available on github ] to deploy it to a Docker container.

## 1. First launch an EC2 instance on AWS
I am usinng here an Ubuntu server [t2.micro]
For help check my repo  **Deploy-Website-on-VM/Simple-Javascript-website/README.md** 

## 2. Update server packages on Ubuntu
```bash
sudo apt Update
```
**step 3 is optional [if you want to run the app in container manually you can by clonning repo]**
## 3. Clone my git repo and go to the correct folder
```bash
git clone https://github.com/MohanPiru/devops-projects.git
cd devops-projects/Project-1/
```
## 4. Install Docker
```bash
sudo apt install docker.io -y
```
## 5. Install Jenkins 
* **Install java first**
Jenkins requires Java to run, yet not all Linux distributions include Java by default. Additionally, not all Java versions are compatible with Jenkins. 
```bash
sudo apt install openjdk-17-jre 
```
**check its install or not**
```bash
java -version
```
* **Install jenkins**
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```
## check jenkins status, it is running..
```bash
sudo systemctl status jenkins
```
## 6.Giving permission of docker group to Ubuntu and jenkins user 
   By default they cannot use docker 
   after changing permission you have to reboot the server to apply changes.
```bash
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
sudo reboot
```
## 7.Download docker-compose
Docker Compose is a tool that helps you define and share multi-container applications. With Compose, you can create a YAML file to define the services and with a single command, you can spin everything up or tear it all down.
```bash
sudo apt-get install docker-compose -y
```
## 8.Configure some set up
* app will be running on port 8000 and jenkins on 8080 , so add **inbound rule** for port 8080 and 8000 in security group of your aws ubuntu server.
* create a **dockerhub** account
* set up credentials of dockerhub in jenkins to login during pipeline process.
  <img width="876" alt="dockerhub cred" src="https://github.com/MohanPiru/devops-projects/assets/140044323/1e2649c1-0838-4ce1-b6f9-9ab233612f99">

  
## 9.Set up jenkins and start a pipeline project
create a job -> select pipeline ->set github project [url : https://github.com/MohanPiru/devops-projects/ ] -> check ( GitHub hook trigger for GITScm polling ) -> paste the "jenkinsfile" file content from my repo -> save it 
## jenkinsfile 
```bash
pipeline {
    agent any
    stages{
        stage("code"){
            steps{
            echo "getting the code from git repo"
            git url:"https://github.com/MohanPiru/devops-projects.git" , branch: "master"
            
            }
        }
        stage("Build"){
            steps{
            echo "Building the image using docker"
            dir('Project-1') {
            sh "docker build -t my-note-app . "
            }
            }
        }
        stage("Push"){
            steps {
            echo "Pushing the code to docker hub"
            dir('Project-1') {
            withCredentials([usernamePassword(credentialsId: "Piru02" , passwordVariable: "P" , usernameVariable: "U")])
            {sh "docker tag my-note-app ${env.U}/my-note-app:latest"
            sh "docker login -u ${env.U} -p ${env.P}"
            sh "docker push ${env.U}/my-note-app:latest"
            }
            }
            
            }
        }
        stage("Deploy"){
            steps{
            echo "deploy to the container"
            dir('Project-1') {
            sh "docker-compose down && docker-compose up -d"
            }
            }
        }
    }
}
```
## 10. Build now
click on build now and you will see your app will be running on port 8000
<img width="657" alt="pipeline1" src="https://github.com/MohanPiru/devops-projects/assets/140044323/7ec92c97-0a5c-48c6-b610-dead23b2c62e">
