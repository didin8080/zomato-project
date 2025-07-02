# 🚀 **DevOps Project: ZOMATO Clone App Deployment**

In this **DevOps project**, I demonstrate how to **deploy a ZOMATO Clone App** using a variety of modern DevOps tools and services.


## **Phase 1: Initial setup and Deployment**

**Step 1: Launch EC2 (ubuntu 22.04)**

- Provision an EC2 instance on AWS with Ubuntu 22.04.
- Connect to the instance using SSH.

**Step 2: update the Package**
```bash
sudo su
sudo apt-get update
```

**Step 3: Clone the repository**

```bash
git clone https://github.com/didin8080/zomato-project.git
```

**Step 4: Install awscli on ubuntu**
```bash
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

**Step 5: Install Jenkins on ubuntu**

```bash
sudo apt update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins
```
Open Port No. 8080 for VM and access Jenkins
Setup Jenkins by following the necessary steps

`publicIp:8080`

**Step 6: Install Docker on Ubuntu**

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker ubuntu
sudo chmod 777 /var/run/docker.sock
newgrp docker
sudo systemctl status docker
```

**Step 7: Install Trivy on Ubuntu**

```bash
sudo apt-get install wget apt-transport-https gnupg
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb generic main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

**step 8: Install Docker Scout**

```bash
docker login -u docker-username  #it asks for a password provide dockerhub password
curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin
sudo chmod 777 /var/run/docker.sock
```

**step 9:Install SonarQube using Docker**

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
docker ps #You can see SonarQube container
```
Open Port No. 9000 for VM and access SonarQube. (by default username & password is admin)

`publicIp:8080`

**step 10: Install necessary plugins on Jenkins**

Goto Jenkins dashboard--> Manage Jenkins --> Plugins

Install below plugins
- Eclipse temurim installer
- sonarqube scanner
- nodej
- OWASP dependency-check 5.5.1
- docker 1.6.2
- Docker commons
- docker pipeline
- docker api
- docker-build-step
- pipeline:stage view
- promethieus metrics
- email extension template


**step 11: Tools configuration in jenkins**

Goto Jenkins dashboard--> Manage Jenkins --> tools 
- add jdk,give name as jdk17 ,install automatically ,add installer, install from adoptium.net, select jdk-17.0.8.1+1 
- add sonarqube scanner--> name= sonar-scaner,insall automatically,
- add node js--> name=node23 
- add dependency check --> name= DP-Check ,install automatically ,add installer, install from github.com
- add docker --> name=docker,install automatically,add installer ,download from docker.com
- apply and save

**step 12: ADD Credentials on Jenkins**

- On sonarqube -> admininstration-->security--> user--> rightside 3dots--> give any name --> generate token
- On jenkins --> manage jenkins-->credentials-->global--> add credentials--> kind=secret text --> on secret paste the token and give any name and description-->give any id and description-->create
- On jenkins --> manage jenkins-->credentials-->global--> add credentials-->kind=username with password--> username=give the username of dockerhub--> password=password of dockerhub-->create

**step 13: System Configuration in Jenkins**

- On jenkins--> manage jenkins--> system--> add sonarqube server -->give name as sonar-server--> on server url= paste the url of sonarqube -->select the authentication --> apply and save .

**step 14: Create webhook in sonarqube**

- on jenkins--> manage jenkins--> system--> add sonarqube server -->give name as sonar-server--> on server url= paste the url of sonarqube with sonarqube-webhook eg:**(192.168.20.173:8080/sonarqube-webhook)** -->select the authentication --> apply and save .


**step 15: Configure CI/CD pipeline in jenkins**

Before pasting the pipeline script, do the following changes in the script
1. In the stage 'Tag and Push to DockerHub', give your docker-hub username. Similar thing you should do in 'DockerScoutImage', 'Deploy to container' stages
2. In post actions stage in pipeline, make sure to give the email id you have configured in jenkins.


```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git 'https://github.com/KastroVKiran/Zomato-Project-Kastro.git'
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage("Code Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --update -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag zomato didin8080/zomato:latest "
                        sh "docker push didin8080/zomato:latest "
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview didin8080/zomato:latest'
                       sh 'docker-scout cves didin8080/zomato:latest'
                       sh 'docker-scout recommendations didin8080/zomato:latest'
                   }
                }
            }
        }
        stage ("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 didin8080/zomato:latest'
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'didin123@gmail.com',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
```

If the build stage of "OWASP FS SCAN" shows 'UNSTABLE BUILD' replace the below script in OWASP FS SCAN stage

```groovy
stage('OWASP FS SCAN') {
    steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit --update -n', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
```
### **Replace the checout repo and generate the link using Pipeline Syntax**

Goto jenkins dashboard → select pipeline → scrolldown select pipeline Syntax → on sample step → select git:Git → paste the repo url → verify the branch → generate pipeline script → copy the link and paste it on checkout

**BUILT IT.**

**Copy the public ip and paste it in the new tab with port number 3000** 

**You will see the output like this** 

`<public-ip:3000>`


<div align="center">
<img src="img/homepage.png" alt="Logo" height="100%" weight="100%">
</div>

**Application deployed in docker**


## **Phase 2: Moniter the Application**

**Step 1: Launch EC2 (Ubuntu 22.04)**

- Provision an EC2 instance on AWS Name: Monitoring Server, Ubuntu 24.04, t2.large, Select the SG created in the Step 1, EBS: 30GB.
- Connect to the instance using SSH.

We will install Grafana, Prometheus, Node Exporter in the above instance and then we will monitor

**Step 2: Update the packages**

```bash
sudo apt-get udate
```


**Step 3: Installing Prometheus**

```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
#Extract Prometheus files, move them, and create directories:
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
#Set ownership for directories:
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

**Create a systemd unit configuration file for Prometheus:**

```bash
sudo vi /etc/systemd/system/prometheus.service
```

**Add the following content to the prometheus.service file:**

```plaintext
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

**Enable and start Prometheus:**

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

**Verify Prometheus's status:**

```bash
sudo systemctl status prometheus
```

Press Control+c to come out



Access Prometheus in browser using your server's IP and port 9090:

`http://<your-server-ip>:9090`

Click on 'Status' dropdown ---> Click on 'Targets' ---> You can see 'Prometheus (1/1 up)'



**Step 4: Installing Node Exporter**

```bash
cd 
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
#Extract Node Exporter files, move the binary, and clean up:
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

**Create a systemd unit configuration file for Node Exporter:**

```bash
sudo vi /etc/systemd/system/prometheus.service
```

**Add the following content to the node_exporter.service file:**

```plaintext
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
```


**Enable and start Node Exporter:**

```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

**Verify Node Exporter status:**

```bash
sudo systemctl status node_exporter
```

Press Control+c to come out


**Step 5: Configure Prometheus Plugin Integration**

As of now we created Prometheus service, but we need to add a job in order to fetch the details by node exporter. So for that we need to create 2 jobs, one with 'node exporter' and the other with 'jenkins' as shown below;


Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

Prometheus Configuration:


**To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file.**

```bash
cd /etc/prometheus/
ls -l
#You can see the "prometheus.yml" file
sudo vi prometheus.yml
```

**You will see the content and also there is a default job called "Prometheus" Paste the below content at the end of the file;**

```yaml
- job_name: 'node_exporter'
    static_configs:
      - targets: ['<MonitoringVMip>:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
```

In the above, replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate IPs ----> esc ----> :wq


**Check the validity of the configuration file:**

```bash
promtool check config /etc/prometheus/prometheus.yml
```

You should see "SUCCESS" when you run the above command, it means every configuration made so far is good.

**Reload the Prometheus configuration without restarting:**

```bash
curl -X POST http://localhost:9090/-/reload
```

Access Prometheus in browser (if already opened, just reload the page):


`http://<your-prometheus-ip>:9090/targets`


Open Port number 9100 for Monitoring VM

You should now see "Jenkins (1/1 up)" "node exporter (1/1 up)" and "prometheus (1/1 up)" in the prometheus browser.


**step 6: Install Grafana**

You are currently in /etc/Prometheus path.


```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
cd
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
#You should see OK when executed the above command.
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get -y install grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

Press control+c to come out


Access grafana in browser using your server's IP and port 3000:

`http://<your-server-ip>:3000`

Default username and password is "admin"

You can Set new password or you can click on "skip now".

Click on "skip now" (If you want you can create the password)


**Step 7: Add Prometheus Data Source:**

To visualize metrics, you need to add a data source. Follow these steps:

- Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.

- Select "Data Sources."

- Click on the "Add data source" button.

- Choose "Prometheus" as the data source type.

- In the "HTTP" section:
  - Set the "URL" to `http://localhost:9090` (assuming Prometheus is running on the same server).
  - Click the "Save & Test" button to ensure the data source is working.

  **Step 8: Import a Dashboard:**

  To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:

- Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

- Select "Dashboard."

- Click on the "Import" dashboard option.

- Enter the dashboard code you want to import (e.g., code 1860).

- Click the "Load" button.

- Select the data source you added (Prometheus) from the dropdown.

- Click on the "Import" button.

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.

Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.

That's it! You've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.


## **Phase 3: Kubernetes**

### Deploy the same Application on kubernetes.

**Step 1: create kubectl on local system**

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

**Step 2: create awscli on local system**

```bash
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

**Step 3: create eksctl on local system**

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```


**Step 4: create Kubernetes Cluster** 

```bash
eksctl create cluster --name zomatocluster --region eu-north-1 --zones eu-north-1b,eu-north-1c --without-nodegroup
```

It will take atleast 20-25 minutes for the cluster to create.

**Step 5: Create & Associate IAM OIDC Provider for our EKS Cluster**

```bash
eksctl utils associate-iam-oidc-provider --region eu-north-1 --cluster zomatocluster --approve
```


**Step 6: create Kubernetes Nodegroup**

```bash
eksctl create nodegroup --cluster zomatocluster --region eu-north-1 --name zomatodemo-ng-public1 --node-type t3.medium --nodes 2 --nodes-min 2 --nodes-max 4 --node-volume-size 20 --ssh-access --ssh-public-key sonu --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```
Replace "sonu" with your key pair

**Verify Cluster & Nodes**

Goto EKS Service in AWS and check for the cluster creation

**step 7: Install HELM**

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

**step 8: ArgoCD installation**

Inorder to monitor k8s with Prometheus, we need to install ArgoCD.

```bash
aws eks update-kubeconfig --name zomatocluster --region eu-north-1
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname')
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
kubectl create namespace prometheus-node-exporter
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname')
echo $ARGOCD_SERVER   #gives an adress copy it and paste it in the new tab. this is the argocd link.(eg: a4f5b673fa5a04274bbb06884d487745-74365602.eu-north-1.elb.amazonaws.com)
export ARGOCD_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo $ARGOCD_PWD     #it gives the password copy it and paste it on the argocd page username is "admin" (eg:XefIYWOviBUSfiCO)
```

In ArgoCD page login it.
- Goto settings, select repository, select connect repo using https, on type= git, project= default, repository URL = (repo url)
- connect it
- Go back to ArgoCD homepage
- select new app
- provide application name given in the repo file
- project name = default
- sync policy = automatic
- on source = paste the repo link
- path = (the file contain the deployment.yml file ,service.yml file and node-service.yml file)
- on destination = click on cluster url a link will popup select that link
- namespace = default
- save it
- select the application and sync , enable force and then ok.


Goto node instance on security group allow port 30001 (because it is on service.yml file).

Copy the `public-ip` of node instance and paste it on the new terminal with portnumber 30001

`eg:(http://51.20.61.179:30001)`


## **Phase 4: Moniter kubernetes with Prometheus**

Goto to Prometheus terminal

```bash
cd /ect/prometheus
ls
vi prometheus.yml
```

**Inside this**

Paste the below commands at the bottom of screen

```yaml
- job_name: 'k8s'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['nodeIP:9100']
```


**Check the validity of the configuration file:**

```bash
promtool check config /etc/prometheus/prometheus.yml
```

You should see "SUCCESS" when you run the above command, it means every configuration made so far is good.

**Reload the Prometheus configuration without restarting:**

```bash
curl -X POST http://localhost:9090/-/reload
```

Access Prometheus in browser (if already opened, just reload the page):


`http://<your-prometheus-ip>:9090/targets`


Open the port number 30001

`<nodeip:30001>`

**You can access the application.**

<div align="center">
<img src="img/zomato.png" alto="Logo" height="100%" width="100%">
<p align="center">Swiggy page </p>
</div>