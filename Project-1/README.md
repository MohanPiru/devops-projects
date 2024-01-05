# Jenkins Declarative CI/CD Pipeline Project
<img width="809" alt="Project-1 Diagram" src="https://github.com/MohanPiru/devops-projects/assets/140044323/55cfb9ab-84fb-4033-8ab0-a9484a92fed7">

It is a simple Devops project using jenkins CI/CD pipeline. I am using a Django notes app code [freely available on github ] to deploy it to a Docker container.

## 1. First launch an EC2 instance on AWS
I am usinng here an Ubuntu server [t2.micro]
For help check my repo  **Deploy-Website-on-VM/Simple-Javascript-website/README.md** 

## 2. Update server packages on Ubuntu
```bash
sudo apt-get Update
```
## 3. Clone my git repo and go to the correct folder
```bash
git clone https://github.com/MohanPiru/devops-projects.git
cd devops-projects/Project-1/
```
## 3. Install Docker
```bash
sudo apt install docker.io -y
```
## 4. Install Jenkins 
* **Install java first**
Jenkins requires Java to run, yet not all Linux distributions include Java by default. Additionally, not all Java versions are compatible with Jenkins. 
```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre
```
* **Install jenkins**
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
## 5. Giving permission of docker group to Ubuntu and jenkins user 
   By default they cannot use docker 
   after changing permission you have to reboot the server to apply changes.
```bash
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
sudo reboot
```
## configure some set up
* app will be running on port 8000 and jenkins on 8080 , so add inbound rule for port 8080 and 8000 in security group of your aws ubuntu server.
* create a dockerhub account
* set up credentials of dockerhub in jenkins to login during pipeline process.
  
## set up jenkins and start a pipeline project
create a job -> select pipeline ->set github project [url : https://github.com/MohanPiru/devops-projects/tree/master/Project-1 ] -> check ( GitHub hook trigger for GITScm polling ) -> paste the "pipeline-script" file content from my repo -> save it 

## Build now
click on build now and you will see your app will be running on port 8000
