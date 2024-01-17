
# Realtime CI/Cd Project
<img width="978" alt="Project-2" src="https://github.com/MohanPiru/devops-projects/assets/140044323/ce31d771-4aa2-43f7-b575-ddec9eb2d8ed">

**Here I am using 3 servers to implement this project**

<img width="661" alt="servers-project-2" src="https://github.com/MohanPiru/devops-projects/assets/140044323/d2c4584f-7993-4fcd-8f21-9b52cafca056">


## 1. Setup server1
**Install docker**
```bash
sudo apt-get Update
sudo apt-get install docker.io -y
```
**Install jenkins**
 
 it needs java
```bash
sudo apt-get install openjdk-17-jre 

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins
```
Now you can access the jenkins Server on ```http://<ip-address>:8080```

* **Install maven**
```bash
sudo apt-get install maven
```
## 2. Setup server 2 
* **Install Sonarqube**
```bash
sudo apt-get Update
sudo apt-get install openjdk-17-jre 
sudo apt-get install unzip
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip *
cd sonarqube-9.4.0.54424/bin/linux-x86-64/
./sonar.sh start 
```
Now you can access the SonarQube Server on ``` http://<ip-address>:9000 ```

## 3. Setup server 3
Here I am using minikube kubernetes cluster (although it is not used in product environment)
* **Install minikube**
```bash
sudo apt-get update
sudo apt-get install docker.io

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

minikube start --driver=docker
```
you can see pods are running
```bash 
kubectl get pods -A
```
* **Install kubectl**
 ```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

#to check 
kubectl version --client
```
* **Install argocd using operators**
```bash
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/download/v0.26.0/install.sh | bash -s v0.26.0

kubectl create -f https://operatorhub.io/install/argocd-operator.yaml

kubectl get csv -n operators
```
create a file 'argocd-basic.yml' and put the following inside it
```bash
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
  labels:
    example: basic
spec: {}
```
run
```bash
kubectl apply -f argocd-basic.yml
```

change the service type of argocd from 'ClusterIp' to 'NodePort' to access that on your browser

```bash 
kubectl get svc
kubectl edit svc <argocd-server>
```
after argocd pods are up and running.

minikube will generate a url for accesing the argocd server from locally
```bash
minikube service argocd-server
minikube service list
```
get the argocd password 
```bash
kubectl get secret
kubectl edit secret <argocd-cluster-secret>
echo <admin.password> | base64 -d
```
* **Install Argocd from ArgoCD github repository(simple Process from manifests)**
 ```bash
kubectl create ns argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.5.8/manifests/install.yaml
```
change the service type of argocd from 'ClusterIp' to 'NodePort' to access that on your browser

```bash 
kubectl get svc
kubectl edit svc argocd-server -n argocd
```
make sure pods are up and running...

`kubectl get all -n argocd`
we need to do a port forwarding. For that we will use the argocd-server service (But make sure that pods are in a running state before running this command).
```bash
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443
```
argocd is runing on `http://<localhost/public-ip-of-vm>:8080
username is 'admin' and get the password running the following
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
Before Installing Prometheus and Grafana using helm we have to install helm first (From Script)
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh
./get_helm.sh
```
* **Install Prometheus using helm**
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install prometheus prometheus-community/prometheus
```
### Expose Prometheus Service

This is required to access prometheus-server using your browser.

```bash 
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext
```

open prometheus on your browser

`kubectl get svc`

`minikube ip`

* **Install Grafana**
```bash
helm repo add grafana https://grafana.github.io/helm-charts

helm repo update

helm install grafana grafana/grafana

kubectl expose service grafana —-type=NodePort —-target-port=3000 —-name=grafana-ext

kubectl get svc
```
* get the secret and login to Grafana
* Link your prometheus and create first dashboard by putting code `3662`

## 4. Configure jenkins
after generating secret token on sonarqube and access token on github...
* **Add 3 global credentials --> dockerhub(as username and password) , sonarqube(token as secret text) and github(developer settings --> access token)**
* **download 'sonarqube scanner' plugin**
* **global tool configuration --> add sonarqube scanner (automatically install)**
* **change environmental variables of jenkinsfile acording to yours**


## 5. Build the pipeline 
  Hurray! The docker image is pushed to dockerhub

## create a new app with argocd providng the github url and path of the yml manifests it will automatically deploy the application
**Argocd will take github as a single source of truth**
<img width="945" alt="argocd-nodejs-website" src="https://github.com/MohanPiru/devops-projects/assets/140044323/08d95073-e0cc-4826-8c46-a038d657b010">
