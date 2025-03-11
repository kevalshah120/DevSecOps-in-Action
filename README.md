## **Introduction: Why DevSecOps?**

In today’s fast-paced tech world, speed and security go hand in hand. You can’t just build and deploy apps quickly—you need to **keep them secure** from day one. That’s where **DevSecOps** comes in! It blends **development, security, and operations** into one seamless process, ensuring that security is baked into every stage of the pipeline instead of being an afterthought.

This **Ultimate DevSecOps Project** is all about deploying a **three-tier application** on **AWS EKS** with a fully automated **CI/CD pipeline**. The goal? To make sure every piece of code is **secure, high-quality, and production-ready** before it even goes live.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741698758056/91258b41-3a89-4900-8afa-031fb2fbcc77.webp)

### **What’s Inside This Project?**

We’ll be using some of the **best DevSecOps tools** out there to make this happen:

✅ **Jenkins** – Automates the entire CI/CD pipeline.  
✅ **SonarQube & OWASP Dependency-Check** – Keep the code clean, secure, and compliant.  
✅ **Trivy** – Scans container images for security vulnerabilities before deployment.  
✅ **Terraform** – Automates infrastructure setup on AWS.  
✅ **ArgoCD** – Ensures Kubernetes deployments stay in sync with Git (GitOps).  
✅ **Prometheus & Grafana** – Provide real-time monitoring and insights.

By the end of this project, you’ll have a **fully functional, security-first DevSecOps pipeline** that not only deploys applications but also keeps them **safe, scalable, and efficient**.

🚀 **Ready to dive in? Let’s build something amazing!**

---

## **Step 1: Set Up a Jenkins Server on AWS EC2**

#### **1\. Log in to AWS and Launch an EC2 Instance**

1. Go to the **AWS Console**.
    
2. Navigate to **EC2** (Elastic Compute Cloud).
    
3. Click **Launch Instance** to create a new virtual machine.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741292059477/784e1068-1aea-4399-b999-83d1a0bafb8f.png)

#### **2\. Configure the EC2 Instance**

* **AMI (Amazon Machine Image):** Choose **Ubuntu Server**.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741292194265/b0f3b2c7-084b-4a03-a1a5-becb4af20c3b.png)
    
    **Instance Type:** Select **t2.2xlarge** (8 vCPUs, 32GB RAM) for better performance.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741292259900/ee5e29eb-0610-4cb0-a91f-eb48eef384c9.png)
    
    **Key Pair:** **No need to create a key pair** (proceed without key pair).
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741292295866/66776b0e-ac60-4f13-ba85-0632c9fd4f35.png)
    

#### **3\. Configure Security Group (Firewall Rules)**

Set up inbound rules to allow required network traffic:

| **Port** | **Protocol** | **Purpose** |
| --- | --- | --- |
| 8080 | TCP | Jenkins Web UI (Restrict access to trusted IPs or internal network). |
| 50000 | TCP | Communication between Jenkins Controller and Agents (for distributed builds). |
| 443 | TCP | HTTPS access (if Jenkins is secured with SSL). |
| 80 | TCP | HTTP access (if using an Nginx reverse proxy for Jenkins). |
| 9000 | TCP | SonarQube Access (for code analysis). |

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741292610377/9547ca61-0423-44d1-ad79-eb09e4a10d7d.png)

> **Note:** For security, avoid opening all these ports to the public. Instead, restrict access to trusted IPs or internal networks.

* **Choose the created** `security group` **in Network setting**
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741292807123/55c5a524-3441-463e-bd3d-f64df1b65455.png)

#### **4\. Configure Storage and IAM Role**

* **Storage:** Set at least **30 GiB**.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293161695/8f46ae7e-c591-407b-b8c6-748be37d1837.png)
    
    **IAM Role:** Attach an **IAM profile with administrative access** to allow Jenkins to manage AWS resources.
    
    > Create an IAM profile with Administrator Access and attach it to EC2 instance
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293299421/fbdcfb68-26ae-4247-ab9c-a65826835a50.png)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293467284/08898a1a-152b-40b4-8de5-547875fd01cd.png)
    

#### **5\. Automate Installation with User Data**

Instead of manually installing required tools, you can automate it using a **User Data script**. This script will automatically install:

* Jenkins
    
* Docker
    
* Terraform
    
* AWS CLI
    
* SonarQube (running in a container)
    
* Trivy (for security scanning)
    

```bash
#!/bin/bash
# For Ubuntu 22.04
# Intsalling Java
sudo apt update -y
sudo apt install openjdk-17-jre -y
sudo apt install openjdk-17-jdk -y
java --version

# Installing Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y

# Installing Docker 
#!/bin/bash
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock

# If you don't want to install Jenkins, you can create a container of Jenkins
# docker run -d -p 8080:8080 -p 50000:50000 --name jenkins-container jenkins/jenkins:lts

# Run Docker Container of Sonarqube
#!/bin/bash
docker run -d  --name sonar -p 9000:9000 sonarqube:lts-community


# Installing AWS CLI
#!/bin/bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install

# Installing Kubectl
#!/bin/bash
sudo apt update
sudo apt install curl -y
sudo curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client


# Installing eksctl
#! /bin/bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Installing Terraform
#!/bin/bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y

# Installing Trivy
#!/bin/bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy -y


# Intalling Helm
#! /bin/bash
sudo snap install helm --classic
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293562191/ca3dc12d-b66a-4296-9d4d-ce3aa9d5d3fc.png)

> **Why use User Data?**
> 
> * Automates setup.
>     
> * Ensures all required tools are installed before first login.
>     
> * Saves time compared to manual installation.
>     

#### **6\. Launch the Instance**

* Click **Launch Instance** to start your Jenkins server.
    

#### **7\. Connect to the Instance**

Since SSH is disabled for security reasons, use the **EC2 Instance Connect** feature:

* Go to the **AWS EC2 Console**.
    
* Select your Jenkins instance.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293679169/40de1163-3fb8-4b77-a17d-2c4511faaba7.png)
    
    Click the **"Connect"** button at the top.
    
* Choose the **"EC2 Instance Connect"** tab.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293744180/7b851c53-7f58-41fc-80c5-0e508a6d3496.png)
    
    Click **"Connect"** to open a web-based terminal directly in your browser.
    

#### **8\. Monitor Running Processes**

To check what commands are running inside the instance, use:

```bash
htop
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741293861459/ac005bc4-b040-4c06-9bdf-c41dc7a7f6cf.png)

This command provides a real-time view of system performance and running processes.

---

## **Step 2: Set Up a Jenkins Pipeline to Deploy EKS and Networking Services**

#### **1\. Access Jenkins**

1. Open your browser and go to:
    
    ```bash
    http://<public-ip-of-jenkins-server>:8080
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741294434215/b9c6c2dc-6c2b-494c-a273-09f79a5bd77d.png)
    
2. Retrieve the initial Jenkins admin password by running the following command in the **jenkins Instance**:
    
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
    
3. Follow the setup wizard:
    
    * Install **Suggested Plugins**.
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741294679400/dc21cab8-5540-4831-a9e3-c329abda050a.png)
        
        Create an **Admin Username & Password**.
        
    * Complete the basic configuration.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741294906516/f9efff05-a7f4-49ac-9c24-7151cf978adc.png)
        

> Now, **Jenkins is ready to use**.

#### **2\. Install Required Plugins**

To enable Jenkins to work with AWS and Terraform, install the following plugins manually:

1. Click on **"Manage Jenkins"** &gt; **"Plugins Manager"**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741295083525/5e02f6ce-0f48-485a-8f6e-ab06c873b1b8.png)
    
2. Search for and install these plugins:
    
    * **AWS Credentials** → Securely stores AWS access keys.
        
    * **Pipeline: AWS Steps** → Adds built-in AWS-specific pipeline steps.
        
    * **Terraform** → Enables Terraform automation in Jenkins.
        
    * **Pipeline: Stage View** → Provides a visual representation of pipeline stages.
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741295264252/2beb8213-17ed-4720-b46b-b03a575f10fd.png)
    

#### **3\. Configure AWS Credentials in Jenkins**

1. Go to **"Manage Jenkins"** &gt; **"Credentials"**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741295411239/6e5f0a6e-323b-430c-a8f5-cf1bdfd36d73.png)
    
    Click **"Global"** &gt; **"Add Credentials"**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741295457576/9180c439-d5c1-4cf1-960d-ed83d794bb71.png)
    
3. Fill in the details:
    
    * **Kind:** AWS Credentials
        
    * **Scope:** Global
        
    * **ID:** `aws-creds`
        
    * **Access Key ID:** `From your AWS IAM user`
        
    * **Secret Access Key:** `From your AWS IAM user`
        
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741295930076/17d4c666-df84-406e-b68d-094ad5623a60.png)
    
    Click **"Create"**.
    

> **Note:** Create an **IAM User** in AWS with **Administrator Access** and use its credentials here.

#### **4\. Configure Terraform in Jenkins**

1. Go to **"Manage Jenkins"** &gt; **"Tools"**.
    
2. Scroll to **Terraform Installation**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741296144218/23525410-81af-4563-b85d-e364d87ee90f.png)
    
3. Provide:
    
    * **Name:** `terraform`
        
    * **Installation Directory:** Find Terraform’s installation path by running below command in jenkins instance:
        
        ```bash
        whereis terraform
        ```
        
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741296359742/94569b47-fd79-4dde-a4fa-a7059e81f220.png)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741296416592/0e9a2f0c-6ef8-4b15-9f66-802082ab658e.png)
    
    Click **Save**.
    

#### **5\. Create a New Jenkins Pipeline**

> In this pipeline i will use this repository here all the terraform source code present to create production grade infrastructure

%[https://github.com/praduman8435/Production-ready-EKS-with-automation] 

1. In Jenkins, go to **Dashboard** &gt; **New Item**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741296577029/43590b03-97a2-4ac5-8635-5b1af97e5b1f.png)
    
    Enter an **item name**.
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741296664743/a6d5d08b-c877-464b-99fb-f5a57ece067b.png)
    
    Select **Pipeline** and click **OK**.
    
4. Scroll to the **Pipeline** section:
    
    * Under **Definition**, select **Pipeline script**.
        
    * Copy and paste the following pipeline script:
        
    
    ```bash
    properties([
        parameters([
            string(
                defaultValue: 'dev',
                name: 'Environment'
            ),
            choice(
                choices: ['plan', 'apply', 'destroy'], 
                name: 'Terraform_Action'
            )
        ])
    ])
    
    pipeline {
        agent any
        stages {
            stage('Preparing') {
                steps {
                    sh 'echo Preparing'
                }
            }
            stage('Git Pulling') {
                steps {
                    git branch: 'main', url: 'https://github.com/praduman8435/Production-ready-EKS-with-automation.git'
                }
            }
            stage('Init') {
                steps {
                    withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                        sh 'terraform -chdir=eks/ init'
                    }
                }
            }
            stage('Validate') {
                steps {
                    withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                        sh 'terraform -chdir=eks/ validate'
                    }
                }
            }
            stage('Action') {
                steps {
                    withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                        script {    
                            if (params.Terraform_Action == 'plan') {
                                sh "terraform -chdir=eks/ plan -var-file=${params.Environment}.tfvars"
                            } else if (params.Terraform_Action == 'apply') {
                                sh "terraform -chdir=eks/ apply -var-file=${params.Environment}.tfvars -auto-approve"
                            } else if (params.Terraform_Action == 'destroy') {
                                sh "terraform -chdir=eks/ destroy -var-file=${params.Environment}.tfvars -auto-approve"
                            } else {
                                error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                            }
                        }
                    }
                }
            }
        }
    }
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741296777138/1d14929d-0327-43a5-8f69-de27a5392673.png)
    

#### **6\. Finalize Pipeline Setup**

1. **Enable Groovy Sandbox:** Check the box for **"Use Groovy Sandbox"**.
    
2. Click **"Save"**.
    

#### **7\. Run the Pipeline**

1. Wait for a minute, then click **"Build with Parameters"**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741297477046/1323bbd7-1f49-42d0-8765-55d05921759f.png)
    
    Select a **Terraform action** (`plan`, `apply`, or `destroy`).
    
3. Click **"Build"**.
    
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741410767616/eb6fe041-5fde-47a1-9e2a-0f1b79940766.png)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741411925695/520fa628-1ecf-4f6c-8b98-106d22daef72.png)
    
    Navigate to the **Console Output** to track progress.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741411958533/aed6e5a2-39cd-467a-ab0d-f916847afac1.png)
    

> ### What This Pipeline Does?
> 
> * **Connects Jenkins to AWS** using stored credentials.
>     
> * **Fetches Terraform code** from GitHub.
>     
> * **Initializes Terraform** for EKS and networking setup.
>     
> * **Validates Terraform code** before deployment.
>     
> * **Executes Terraform actions** based on user selection (`plan`, `apply`, or `destroy`)
>     

---

## **Step 3: Set Up the Jump Server**

#### **Why Do You Need a Jump Server?**

Since your **EKS cluster is inside a VPC**, it cannot be accessed directly from the internet. A **Jump Server (Bastion Host)** acts as a secure gateway, allowing access to private resources within your VPC.

#### **How It Works:**

* Your **EKS cluster** and other private resources don’t have public IPs, so they can't be accessed directly.
    
* Instead of exposing these private resources, you connect to a **Jump Server** first.
    
* The Jump Server has a **public IP** and is placed in a **public subnet**, acting as an intermediary to access the private cluster securely.
    

### 1\. Create a Jump Server in AWS

1. **Go to AWS EC2 Console** and click **"Launch Instance"**.
    
2. **Configure the Instance:**
    
    * **Instance Name:** `jump-server`
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741413617948/7077e8a2-2498-4916-80ac-6776735ac98d.png)
        
        **AMI:** Ubuntu
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741413676683/8bcb1a0c-3641-4dfe-90a1-b2f90bf68f4b.png)
        
    * **Instance Type:** `t2.medium`
        
    * **Key Pair:** *No need to attach a key pair (SSH disabled for security).*
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741413746424/c1aa733e-4fd1-47b3-a7f8-ce4c584f4119.png)
        
    * **Network Settings:**
        
        * **VPC:** Select the VPC created by the **Jenkins Terraform pipeline**.
            
        * **Subnet:** Choose **any public subnet**.
            
    * **Storage:** At least **30 GiB**.
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741414061185/43d4cf74-da34-423d-a9a9-e05c3699c3be.png)
        
        **IAM Role:** Attach an **IAM profile with administrative access**.
        
3. **Install** required tools on the **Jump Server** automatically, by adding the following script in the **User Data** field:
    
    ```bash
    sudo apt update -y
    
    # Installing AWS CLI
    #!/bin/bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip -y
    unzip awscliv2.zip
    sudo ./aws/install
    
    # Installing Kubectl
    #!/bin/bash
    sudo apt update
    sudo apt install curl -y
    sudo curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
    sudo chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    kubectl version --client
    
    # Intalling Helm
    #! /bin/bash
    sudo snap install helm --classic
    
    # Installing eksctl
    #! /bin/bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo mv /tmp/eksctl /usr/local/bin
    eksctl version
    ```
    
4. **Launch the Instance** by clicking **"Launch Instance"**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741414254891/71281416-52f5-4948-9df7-f5d4f1958756.png)
    

### 2\. Connect to the Jump Server and Verify Access

Once the instance is running, access it using **EC2 Instance Connect**:

1. Go to **AWS EC2 Console**.
    
2. Select your **Jump Server** instance.
    
3. Click **"Connect"** &gt; **"EC2 Instance Connect"**.
    
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741414558786/c70eb5cb-7b03-4381-a9d1-c058ff6e5f1b.png)
    
    Click **"Connect"** to open a web terminal.
    

### 3\. Configure AWS Credentials on the Jump Server

To allow the Jump Server to interact with AWS services, configure AWS CLI:

```bash
aws configure
```

* **Enter AWS Access Key ID** (from IAM user).
    
* **Enter AWS Secret Access Key** (from IAM user).
    
* **Default region:** Set your AWS region (e.g., `us-east-1`).
    
* **Output format:** Press Enter (default is `json`).
    

### 4\. Update kubeconfig to Access the EKS Cluster

Run the following command to configure `kubectl` for your EKS cluster:

```bash
aws eks update-kubeconfig --name <your-eks-cluster-name> --region <your-region>
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741437752403/45b020f4-9bae-4abd-9493-62a0272a8da3.png)

### 5\. Verify the Cluster Connection

Check if the Jump Server can access the EKS cluster by listing the worker nodes:

```bash
kubectl get nodes
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741437762798/e61dd85d-5ec6-491c-bf1f-3007eb1f8093.png align="center")

If you see the nodes, **your Jump Server setup is successful!** 🎉

---

## **Step 4: Set Up an AWS Load Balancer for EKS**

In order to configure an AWS **Load Balancer** in our **EKS cluster**, we need a **Service Account** that allows Kubernetes to create and manage the load balancer automatically.

### 1\. Create an IAM-Backed Service Account

The AWS Load Balancer Controller requires an IAM role with the necessary permissions to create and manage Elastic Load Balancers (ELB) in AWS.

Run the following command to create a **service account** with the required IAM role inside the EKS cluster:

```bash
eksctl create iamserviceaccount \
  --cluster=<eks-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerRole \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=ap-south-1
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741437810310/e6353e33-cc05-43a5-bf33-29b63e1b8336.png align="center")

📌 **Explanation:**

* `--cluster=<eks-cluster-name>` → Name of your EKS cluster.
    
* `--namespace=kube-system` → Deploys the service account in the `kube-system` namespace.
    
* `--name=aws-load-balancer-controller` → Creates a service account with this name.
    
* `--role-name AmazonEKSLoadBalancerRole` → Assigns an IAM role to the service account.
    
* `--attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy` → Attaches the AWS Load Balancer Controller IAM policy.
    
* `--approve` → Automatically applies the changes.
    

### 2\. Add the AWS EKS Helm Repository

To deploy the **AWS Load Balancer Controller**, we use **Helm**, a package manager for Kubernetes.

Add the official AWS **EKS Helm repository**:

```bash
helm repo add eks https://aws.github.io/eks-charts
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741437839956/94e69cd0-5a42-4d35-8c50-07879f0ccd8f.png align="center")

This repository contains pre-packaged Helm charts for essential AWS EKS components such as:  
✅ **AWS Load Balancer Controller**  
✅ **EBS CSI Driver** (for dynamic volume provisioning)  
✅ **VPC CNI Plugin** (for networking enhancements)  
✅ **Cluster Autoscaler** (for automatic scaling)

**Update the repository to get the latest charts:**

```bash
helm repo update
```

### 3\. Install the AWS Load Balancer Controller

Now, install the **AWS Load Balancer Controller** using **Helm**:

```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  --namespace kube-system \
  --set clusterName=<eks-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

📌 **Explanation:**

* `--namespace kube-system` → Deploys the controller in the `kube-system` namespace.
    
* `--set clusterName=<eks-cluster-name>` → Associates it with your EKS cluster.
    
* `--set serviceAccount.create=false` → Uses the existing IAM-backed service account.
    
* `--set serviceAccount.name=aws-load-balancer-controller` → Specifies the service account created earlier.
    

### 4\. Verify the AWS Load Balancer Controller

Check if the **Load Balancer Controller** is running correctly:

```bash
kubectl get pods -n kube-system | grep aws-load-balancer-controller
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741438471377/b51b2dd3-a22c-471c-a331-e42bfa9a5025.png align="center")

### 4\. Fixing Pods in Error or CrashLoopBackOff

If your AWS Load Balancer Controller pods are in **Error** or **CrashLoopBackOff**, it’s likely due to misconfiguration. To fix this, upgrade the Helm release with the correct settings:

```bash
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id> \
  -n kube-system
```

* `<your-vpc-id>`: VPC ID where your EKS cluster runs (e.g., `vpc-0123456789abcdef0`).
    
* `<cluster-name>`: Your EKS cluster name (e.g., `dev-medium-eks-cluster`).
    
* `<your-region>`: AWS region of your cluster (e.g., `us-west-1`).
    

> #### **What This Does**?
> 
> * **Upgrades/Installs**: Updates the Helm release or installs it if missing.
>     
> * **Configures Correctly**: Ensures the controller uses the right cluster, service account, region, and VPC.
>     

**Check if the pods are running:**

```bash
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741438829300/9c190c92-9313-490c-865b-76980b11b797.png align="center")

🚀 **Your AWS Load Balancer Controller is now ready to manage Kubernetes services!** 🎉

---

## **Step 5: Set Up and Configure ArgoCD on EKS**

ArgoCD is a **GitOps** continuous delivery tool that automates the deployment of applications to Kubernetes. We will install ArgoCD in our **EKS cluster** and expose its UI for external access.

### 1\. Create a Separate Namespace for ArgoCD

To keep ArgoCD components organized, create a dedicated **namespace**:

```bash
kubectl create namespace argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439351489/a185aa2d-b571-4d90-a295-814f033521aa.png align="center")

### 2\. Install ArgoCD Using Manifests

Apply the official **ArgoCD installation YAML** to deploy its components:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439449977/9b3f488b-cfe9-48e5-b1c0-92b1f320c716.png align="center")

This will install all necessary ArgoCD components inside the `argocd` namespace.

### 3\. Verify ArgoCD Installation

Check if all ArgoCD **pods** are running:

```bash
kubectl get pods -n argocd
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439486327/2f4c8c30-7c14-4d64-be54-078824843f5b.png align="center")

### **4\. Expose ArgoCD Server**

By default, ArgoCD runs as a **ClusterIP service**, meaning it is only accessible inside the cluster. To access the UI externally, change it to a **LoadBalancer** service:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

This will assign a public IP to the ArgoCD server, making it accessible via an **Elastic Load Balancer (ELB)** in AWS.

### 5\. Retrieve the External IP of ArgoCD

Run the following command to get the external URL:

```bash
kubectl get svc -n argocd argocd-server
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439573002/3ace2789-147d-4e85-8802-4ecb013c8104.png align="center")

Look for the **EXTERNAL-IP** in the output. This is the URL you’ll use to access the ArgoCD UI.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439768269/e971887f-7d2a-41c4-8db7-bec16fb15f2c.png align="center")

### 6\. Get the ArgoCD Admin Password

By default, ArgoCD generates an **admin password** stored as a Kubernetes secret. Retrieve it using:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
```

Use this password to log in as the `admin` user.

### 7\. Access the ArgoCD UI

Now, you can access the UI of argoCD using the elastic loadbalancer created on the aws

Login using:

* **Username:** `admin`
    
* **Password:** *(retrieved from the secret above)*
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439804593/2056b752-4073-4cee-9ede-564f76f193e8.png align="center")

> **ArgoCD is now ready!** You can start managing your Kubernetes deployments using GitOps. 🚀

---

## **Step 6: Configure SonarQube for DevSecOps Pipeline**

SonarQube is a crucial tool for **static code analysis**, ensuring **code quality and security** in your DevSecOps pipeline. We will configure it within Jenkins for automated code scanning.

### 1\. Verify if SonarQube is Running

Since SonarQube is running as a **Docker container** on the Jenkins server, check its status with:

```bash
docker ps
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439908245/ccf2d8c4-e366-4e4d-8d31-c338c0278c9e.png align="center")

You should see a running SonarQube container **exposed on port 9000**.

### 2\. Access SonarQube UI

Open any web browser and visit:

```bash
http://<public-ip-of-jenkins-server>:9000
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741439966061/2da33e4c-a391-460e-8341-c802c580ceb9.png align="center")

Log in using the default credentials:

* **Username:** `admin`
    
* **Password:** `admin`
    

Once logged in, **set a new password** for security.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440050172/dfbf8897-cc8d-4972-8f8d-3ee275516d7b.png align="center")

### 3\. Generate an Authentication Token

Jenkins needs a **token** to authenticate with SonarQube for automated scans.

1. Go to **Administration** → **Security** → **Users**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440120078/c7d2deac-510e-4e4b-b72e-5ba3e1d2143a.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440353109/8c32814d-561f-4123-8c3a-b66337004e57.png align="center")
    
    Click on **Update Token**.
    
3. Provide a **name** and set an **expiration date** (or leave it as "No Expiration").
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440489989/ee5fe744-8bfd-4624-976d-f29b45867663.png align="center")
    
4. Click **Generate Token**.
    
5. **Copy and save** the token securely (you will need it for Jenkins).
    

### 4\. Create a Webhook for Jenkins Notifications

A webhook will notify Jenkins once SonarQube completes an analysis.

1. Navigate to **Administration** → **Configuration** → **Webhooks**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440664143/69b7eb5b-ee81-4683-8c7e-7373df3fc454.png align="center")
    
    Click **Create Webhook**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440702895/a9a62e5f-e128-4d39-a3b2-e2240abaef79.png align="center")
    
3. Enter the details:
    
    * **Name:** `Jenkins Webhook`
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741440860439/75c3c487-a15c-4203-b8c9-bb783e37ade6.png align="center")
        
        **URL:** `http://<public-ip-of-jenkins-server>:8080/sonarqube-webhook`
        
    * **Secret:** (Leave blank)
        
4. Click **Create**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741487965585/b3bc2554-674a-4921-a496-4c20e9f263a4.png align="center")
    

> Now, the webhook will trigger Jenkins when a project analysis is complete.

### 5\. Create a SonarQube Project for Code Analysis

SonarQube will analyze the **frontend and backend** code separately.

#### **Frontend Analysis Configuration**

1. Go to **Projects** → **Manually → Create a New Project**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441258669/81ccfc7c-1d6e-4e94-a5ef-3ec53f318b4b.png align="center")
    
    Fill in the required details (Project Name, Key, etc.).
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441418605/aa78604f-da77-47e3-a230-22e2c66b0bd5.png align="center")
    
    Click **Setup**.
    
4. Choose **Analyze Locally**.
    
5. Select **Use an Existing Token** and **paste the token** generated earlier.
    
6. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441582221/166bedc7-2765-438c-a1a8-7073cc4f9d3a.png align="center")
    
    Choose **Other** if your build type is not listed.
    
7. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441621232/65fb0958-141a-4aa4-82b9-2d039822166c.png align="center")
    
    Select **OS: Linux**.
    
8. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441672905/b5162b3a-e481-4d86-be98-141387ef29c0.png align="center")
    
    SonarQube will generate a command for analysis—**copy and save it**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441699085/e31e9567-a89e-45b3-8370-4f13957ae9a8.png align="center")
    
9. Add the command to your **Jenkins pipeline**
    

#### **Backend Analysis Configuration**

Repeat the **same steps** for the backend project:

1. Go to **Projects** → **Create a New Project**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441258669/81ccfc7c-1d6e-4e94-a5ef-3ec53f318b4b.png align="center")
    
2. Fill in the required details.
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441891639/8c0388dc-3ca1-4151-bc34-af7098e0e53f.png align="center")
    
    Click **Setup** → **Analyze Locally**.
    
4. Use the **previously generated token**.
    
5. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741441967533/88db0b7d-43a7-4c29-a4a2-2f8f1df8b9b2.png align="center")
    
    Choose **Other** as the build type if needed.
    
6. Select **OS: Linux**.
    
7. **Copy the generated analysis command** and save it.
    
8. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442008154/019dc47a-6987-4f33-981c-d61b8cef3887.png align="center")
    
    Add the command to your **Jenkins pipeline**
    

### 6\. Final Verification

At this point:  
✅ **SonarQube is running** and accessible.  
✅ **Jenkins has an authentication token** to interact with SonarQube.  
✅ **A webhook is set up** to notify Jenkins about completed scans.  
✅ **Projects are created**, and **analysis commands** are ready for Jenkins execution.

Now, whenever Jenkins runs the pipeline, **SonarQube will analyze the code and report quality & security issues**. 🎯 ✅

---

## **Step 6: Create an ECR Repository for Docker Images**

Amazon **Elastic Container Registry (ECR)** will store the **frontend and backend Docker images** used for deployment. Let's configure it and store necessary credentials in Jenkins.

### 1\. Create Private ECR Repositories

1. **Open AWS Console** and navigate to the **ECR service**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442220525/01fc1f1c-87ba-481f-bc52-37717f214dec.png align="center")
    
    Click **Create Repository**.
    
3. Choose **Private Repository**.
    
4. **Repository Name:** `frontend`.
    
5. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741443658443/3719eea2-68ba-4d84-a437-39e024f0875b.png align="center")
    
    Repeat the same steps to create a `backend` repository.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741443722005/889f0766-ee66-4c40-8247-240a2fc1414e.png align="center")
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741443777588/da0bef4d-f75a-4e3f-8503-77c4e04d0345.png align="center")

### 2\. Store Credentials in Jenkins

To integrate Jenkins with SonarQube, AWS ECR, and GitHub, we need to store various credentials securely.

#### **a) Store SonarQube Token in Jenkins**

1. Go to **Jenkins Dashboard** → **Manage Jenkins** → **Credentials**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442565891/30a3a863-56b1-4ba7-bae6-802cfc51b33c.png align="center")
    
    Select the appropriate **Global credentials domain**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442659623/d4bb19fc-b819-4bb5-8ab4-f9ef07dc8dfb.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442716197/ce323ecc-98db-4565-8960-f3206d918614.png align="center")
    
3. Click **Add Credentials** and fill in the details:
    
    * **Kind:** Secret Text
        
    * **Scope:** Global
        
    * **Secret:** `<sonar-qube-token>`
        
    * **ID:** `sonar-token`
        
    * Click **Create**.
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442811289/49f8c587-6987-41e4-a94b-7aaa306dbfb5.png align="center")
    

#### **b) Store AWS Account ID in Jenkins**

1. Go to **Credentials** → **Add Credentials**.
    
2. Enter the details:
    
    * **Kind:** Secret Text
        
    * **Scope:** Global
        
    * **Secret:** `<AWS-Account-ID>`
        
    * ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442935298/8ccf9024-a7d8-4c90-be74-f0ffc99c050b.png align="center")
        
        **ID:** `Account_ID`
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741442974476/8dd3ecb2-c9df-496d-b91d-ad1722a9300f.png align="center")
        
    * Click **Create**.
        

#### **c) Store ECR Repository Names in Jenkins**

For **Frontend Repository**:

1. **Add New Credential**:
    
    * **Kind:** Secret Text
        
    * **Scope:** Global
        
    * **Secret:** `frontend`
        
    * **ID:** `ECR_REPO1`
        
    * Click **Create**.
        

For **Backend Repository**:

1. **Add New Credential**:
    
    * **Kind:** Secret Text
        
    * **Scope:** Global
        
    * **Secret:** `backend`
        
    * **ID:** `ECR_REPO2`
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741443151975/365596a2-0c1c-479e-ac8d-3133e227739e.png align="center")
        
    * Click **Create**.
        

#### **d) Store GitHub Credentials in Jenkins**

1. **Add New Credential**:
    
    * **Kind:** Username with Password
        
    * **Scope:** Global
        
    * **Username:** `<GitHub-Username>`
        
    * **Password:** `<Personal-Access-Token>`
        
    * **ID:** `GITHUB-APP`
        

#### **e) Store GitHub Personal Access Token in Jenkins**

1. **Add New Credential**:
    
    * **Kind:** Secret Text
        
    * **Scope:** Global
        
    * **Secret:** `<Personal-Access-Token>`
        
    * **ID:** `github`
        

> here all required credentials are get added

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741444543186/f9e07190-22d1-4b76-a67e-9a96183e840b.png align="center")

### **Final Confirmation**

✅ **ECR Repositories Created**  
✅ **SonarQube Token Stored in Jenkins**  
✅ **AWS Account ID Saved**  
✅ **ECR Repository Names Stored**  
✅ **GitHub Credentials & Token Added**

With these credentials configured, **Jenkins can authenticate and push Docker images to AWS ECR** seamlessly. 🎯

## Install and Configure Essential Plugins & Tools in Jenkins

To ensure seamless **containerized builds, security analysis, and CI/CD automation**, install and configure the necessary **Jenkins plugins and tools**.

### 1\. Install Required Plugins

Navigate to **Jenkins Dashboard** → **Manage Jenkins** → **Plugins** → **Available Plugins**, then search and install the following:

✅ **Docker** – Enables Docker integration.  
✅ **Docker Pipeline** – Provides Docker support in Jenkins pipelines.  
✅ **Docker Commons** – Manages shared Docker images.  
✅ **Docker API** – Allows interaction with Docker daemon.  
✅ **NodeJS** – Supports Node.js builds.  
✅ **OWASP Dependency-Check** – Detects security vulnerabilities in dependencies.  
✅ **SonarQube Scanner** – Enables code quality analysis with SonarQube.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741444870296/d0b8da8d-096d-44cd-a8ee-84b080bbb2ab.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741444907427/8a66ee60-47ba-4106-9c3c-e2fb62d3966f.png align="center")

Once installed, **restart Jenkins** to apply changes.

### 2\. Configure Essential Tools in Jenkins

#### **a) NodeJS Installation**

1. Go to **Manage Jenkins** → **Tools**
    
2. Under **NodeJS**, click **Add NodeJS**.
    
3. Fill in the required details.
    
4. Check the box **Install Automatically**.
    
5. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741445221523/b0af3629-63dc-4aa2-a342-b66df19acb0a.png align="center")
    
    Click **Save**.
    

#### **b) SonarQube Scanner Installation**

1. Under **Tools Configuration**, go to **SonarQube Scanner**.
    
2. Click **Add SonarQube Scanner**.
    
3. Fill in the required details.
    
4. Check **Install Automatically**.
    
5. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741445339243/557241fb-587f-4235-b5c2-063b83875f16.png align="center")
    
    Click **Save**.
    

#### **c) OWASP Dependency Check Installation**

1. Under **Tools Configuration**, go to **Dependency Check**.
    
2. Click **Add Dependency Check**.
    
3. Check **Install Automatically from GitHub**.
    
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741445517523/5686bd62-1514-4a3f-bbe7-469a0e69f84d.png align="center")
    
    Click **Save**.
    

#### **d) Docker Installation**

1. Under **Tools Configuration**, go to **Docker**.
    
2. Click **Add Docker**.
    
3. Fill in the required details.
    
4. Check **Install Automatically from** **Docker.com**.
    
5. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741445599671/c0845040-5c52-499e-9664-fb1e8d6bb39a.png align="center")
    
    Click **Save & Apply**.
    

### 3\. Configure SonarQube Webhook in Jenkins

To enable **SonarQube notifications in Jenkins**, configure the webhook.

#### **Add SonarQube Server in Jenkins**

1. Navigate to **Manage Jenkins** → **Configure System**.
    
2. Scroll to the **SonarQube installation** section.
    
3. Click **Add SonarQube** and enter:
    
    * **Name:** `sonar-server`
        
    * **Server URL:** `http://<public-ip-of-jenkins-server>:9000`
        
    * **Server Authentication Token:** `<sonar-qube-token-credential-name>`
        
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741445857812/239f98f1-91ed-4f26-86e8-92e751f460e4.png align="center")
    
    Click **Apply & Save**.
    

> Jenkins is now **fully equipped** to handle **Docker builds, security analysis, and SonarQube scanning** in the DevSecOps pipeline. 🚀

---

## Step 7: Create a Jenkins Pipeline for Frontend

This pipeline automates the **frontend build, security analysis, Docker image creation, and deployment updates** for the **DevSecOps pipeline**.

### 1\. Create a New Pipeline in Jenkins

1. Navigate to **Jenkins Dashboard** → **New Item**.
    
2. Enter a **Pipeline Name** (e.g., `frontend-pipeline`).
    
3. Select **Pipeline** as the item type.
    
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741456092289/fcb015a5-9fb4-401c-801d-0ab19218063b.png align="center")
    
    Click **OK** to proceed.
    
5. Scroll down to the **Pipeline** section and choose **Pipeline script**.
    

### 2\. Add the Pipeline Script

Copy and paste the following **Jenkinsfile**:

```bash
pipeline {
    agent any 
    tools {
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('Account_ID')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO1')
        AWS_DEFAULT_REGION = 'ap-south-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git credentialsId: 'GITHUB-APP', url: 'https://github.com/praduman8435/DevSecOps-in-Action.git', branch: 'main'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('frontend') {
                    withSonarQubeEnv('sonar-server') { // Use withSonarQubeEnv wrapper
                        sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=frontend \
                        -Dsonar.projectKey=frontend \
                        -Dsonar.sources=.
                        '''
                    }
                }
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('OWASP Dependency-Check Scan') {
            steps {
                dir('Application-Code/backend') {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('Trivy File Scan') {
            steps {
                dir('frontend') {
                    sh 'trivy fs . > trivyfs.txt'
                }
            }
        }
        stage('Docker Image Build') {
            steps {
                script {
                    dir('frontend') {
                        sh 'docker system prune -f'
                        sh 'docker container prune -f'
                        sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
                    }
                }
            }
        }
        stage('ECR Image Pushing') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                    sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER} > trivyimage.txt'
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "DevSecOps-in-Action"
                GIT_USER_NAME = "praduman8435"
            }
            steps {
                dir('k8s-manifests/frontend') {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                        git config user.email "praduman.cnd@gmail.com"
                        git config user.name "praduman"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        imageTag=$(grep -oP '(?<=frontend:)[^ ]+' deployment.yaml)
                        sed -i "s/${AWS_ECR_REPO_NAME}:${imageTag}/${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}/" deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741457839893/15fd831a-2071-4a69-894f-b36de7ce4c24.png align="center")

### 3\. Build the Pipeline

1. Click **Save & Apply**.
    
2. Click **Build Now** to start the pipeline.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741458808440/e91d9923-d0da-4dfb-a11a-6dcc8430992c.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741514552441/eae55dca-acc9-4bc8-b15c-44b3e89ea063.png align="center")
    

### 4\. Verify SonarQube Analysis

1. Open SonarQube UI:
    
    ```bash
    http://<public-ip-of-jenkins-server>:9000
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741514588913/1d911ec7-71a7-4880-b4b4-c4c0ec6bff94.png align="center")
    
2. Check if the **SonarQube scan results** appear in the UI under the **frontend project**.
    

### **Pipeline Workflow Summary**

✅ **Code Checkout** from GitHub.  
✅ **SonarQube Scan** for code quality analysis.  
✅ **Security Scans** using OWASP Dependency-Check and Trivy.  
✅ **Docker Build & Push** to Amazon ECR.  
✅ **Deployment Update** in Kubernetes manifests.

The frontend pipeline is now **fully automated and integrated into the DevSecOps workflow! 🚀**

---

## **Step 8: Create a Jenkins Pipeline for Backend**

This pipeline automates the **backend build, security scanning, Docker image creation, and Kubernetes deployment updates** for the **DevSecOps pipeline**.

### 1\. Create a New Pipeline in Jenkins

1. Go to **Jenkins Dashboard** → **New Item**.
    
2. Enter a **Pipeline Name** (e.g., `backend-pipeline`).
    
3. Select **Pipeline** as the item type.
    
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741518531141/eb049dd3-8936-4810-8f66-028378f8f12c.png align="center")
    
    Click **OK** to proceed.
    
5. Scroll to the **Pipeline** section and choose **Pipeline script**.
    

### 2\. Add the Pipeline Script

Copy and paste the following **Jenkinsfile**:

```bash
pipeline {
    agent any 
    tools {
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('Account_ID')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO2')
        AWS_DEFAULT_REGION = 'ap-south-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git credentialsId: 'GITHUB-APP', url: 'https://github.com/praduman8435/DevSecOps-in-Action.git', branch: 'main'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('sonar-server') { // Use withSonarQubeEnv wrapper
                        sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=backend \
                        -Dsonar.projectKey=backend \
                        -Dsonar.sources=.
                        '''
                    }
                }
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('OWASP Dependency-Check Scan') {
            steps {
                dir('backend') {
                    dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }
        stage('Trivy File Scan') {
            steps {
                dir('backend') {
                    sh 'trivy fs . > trivyfs.txt'
                }
            }
        }
        stage('Docker Image Build') {
            steps {
                script {
                    dir('backend') {
                        sh 'docker system prune -f'
                        sh 'docker container prune -f'
                        sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
                    }
                }
            }
        }
        stage('ECR Image Pushing') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                    sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER} > trivyimage.txt'
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "DevSecOps-in-Action"
                GIT_USER_NAME = "praduman8435"
            }
            steps {
                dir('k8s-manifests/backend') {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                        git config user.email "praduman.cnd@gmail.com"
                        git config user.name "praduman"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        imageTag=$(grep -oP '(?<=backend:)[^ ]+' deployment.yaml)
                        sed -i "s/${AWS_ECR_REPO_NAME}:${imageTag}/${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}/" deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741519197297/4015381c-9824-4414-985e-a85be33e690c.png align="center")

### 3\. Build the Pipeline

1. Click **Save & Apply**.
    
2. Click **Build Now** to trigger the pipeline.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741523884768/3cc037bb-99a5-4435-b006-c6e9e343f254.png align="center")
    

### 4\. Verify SonarQube Analysis

1. Open SonarQube UI:
    
    ```bash
    http://<public-ip-of-jenkins-server>:9000
    ```
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741519441252/9599b465-e09d-45ad-a104-a407ea1cb14c.png align="center")
    
    Check the **SonarQube scan results** under the **backend project**.
    

### **Pipeline Workflow Summary**

✅ **Code Checkout** from GitHub.  
✅ **SonarQube Scan** for code quality analysis.  
✅ **Security Scans** using OWASP Dependency-Check and Trivy.  
✅ **Docker Build & Push** to Amazon ECR.  
✅ **Deployment Update** in Kubernetes manifests.

The **backend pipeline** is now fully automated and integrated into the **DevSecOps workflow! 🚀**

---

## **Step 9: Setup Application in ArgoCD**

In this step, we will **deploy the application (frontend, backend, database, and ingress) to the EKS cluster** using **ArgoCD**.

### 1\. Open ArgoCD UI

1. Get the ArgoCD **server External-IP**:
    
    ```bash
    kubectl get svc -n argocd argocd-server
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741520697491/d405f4f6-cb99-448f-94e5-8f6779bd3fc1.png align="center")
    
2. Access the ArgoCD UI using **EXTERNAL-IP** in the output.
    
3. Login using `username` and `password` created by you previously.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741524032969/bf38e285-4f43-41c4-811c-f583c35a4b0b.png align="center")

### 2\. Connect GitHub Repository to ArgoCD

1. Go to **Settings** → **Repositories**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741524080333/523cc402-8152-4c35-a966-7e7a478060e9.png align="center")
    
    Click **"Connect Repository using HTTPS"**.
    
3. Enter:
    
    * **Project:** `default`
        
    * **Repository URL:** [`https://github.com/praduman8435/DevSecOps-in-Action.git`](https://github.com/praduman8435/DevSecOps-in-Action.git)
        
    * **Authentication:** None (if public repo)
        
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741524351951/0745cb43-eb83-460d-8e02-bc5bfea1a9a9.png align="center")
    
    Click **"Connect"**.
    

### 3\. Create Kubernetes Namespace for Deployment

1. Open **terminal** and run:
    
    ```bash
    kubectl create namespace three-tier
    ```
    
2. Verify the namespace:
    
    ```bash
    kubectl get namespaces
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741524473226/95f4a026-bc5b-45b1-88dc-e7dbd5bec1e5.png align="center")
    

### 4\. Deploy Database in ArgoCD

1. In ArgoCD UI, go to **Applications** → Click **New Application**.
    
2. Fill in the following details:
    
    * **Application Name:** `three-tier-database`
        
    * **Project Name:** `default`
        
    * **Sync Policy:** `Automatic`
        
    * **Repository URL:** [`https://github.com/praduman8435/DevSecOps-in-Action.git`](https://github.com/praduman8435/DevSecOps-in-Action.git)
        
    * **Path:** `k8s-manifests/database`
        
    * **Cluster URL:** `https://kubernetes.default.svc`
        
    * **Namespace:** `three-tier`
        
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741525023900/7b80d291-5161-419d-8e73-dfec2028f5af.png align="center")
    
    Click **Create**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741525162975/5aa54702-f575-4ba2-9323-7e9fa53ab19b.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741525188668/f927c4f4-1cdc-42a4-ba2b-f25c99a0cc35.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741525425034/c790456f-eb2f-4990-aa21-b6fa0c7db513.png align="center")

### 5\. Deploy Backend in ArgoCD

1. Go to **Applications** → Click **New Application**.
    
2. Fill in:
    
    * **Application Name:** `three-tier-backend`
        
    * **Project Name:** `default`
        
    * **Sync Policy:** `Automatic`
        
    * **Repository URL:** [`https://github.com/praduman8435/DevSecOps-in-Action.git`](https://github.com/praduman8435/DevSecOps-in-Action.git)
        
    * **Path:** `k8s-manifests/backend`
        
    * **Cluster URL:** `https://kubernetes.default.svc`
        
    * **Namespace:** `three-tier`
        
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741525596484/fab7203e-ff2b-47fd-876c-708fa6efddb2.png align="center")
    
    Click **Create**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741525631628/3daaeb8d-27ec-4be0-bf13-841029f078c4.png align="center")

### 6\. Deploy Frontend in ArgoCD

1. Go to **Applications** → Click **New Application**.
    
2. Fill in:
    
    * **Application Name:** `three-tier-frontend`
        
    * **Project Name:** `default`
        
    * **Sync Policy:** `Automatic`
        
    * **Repository URL:** [`https://github.com/praduman8435/DevSecOps-in-Action.git`](https://github.com/praduman8435/DevSecOps-in-Action.git)
        
    * **Path:** `k8s-manifests/frontend`
        
    * **Cluster URL:** [`https://kubernetes.default.svc`](https://kubernetes.default.svc)
        
    * **Namespace:** `three-tier`
        
3. Click **Create**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741607070003/87b23be1-15eb-4b78-a1a1-cd4105982a2c.png align="center")

### 7\. Deploy Ingress in ArgoCD

1. Go to **Applications** → Click **New Application**.
    
2. Fill in:
    
    * **Application Name:** `three-tier-ingress`
        
    * **Project Name:** `default`
        
    * **Sync Policy:** `Automatic`
        
    * **Repository URL:** [`https://github.com/praduman8435/DevSecOps-in-Action.git`](https://github.com/praduman8435/DevSecOps-in-Action.git)
        
    * **Path:** `k8s-manifests`
        
    * **Cluster URL:** `https://kubernetes.default.svc`
        
    * **Namespace:** `three-tier`
        
3. Click **Create**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741607042824/e0df1c07-151f-4fa6-992e-1d4227732cb1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741607128084/8a7fb643-b1d7-4999-9e49-d0539ce1fb27.png align="center")

### 8\. Verify Deployment in ArgoCD

1. Go to **Applications** in ArgoCD UI.
    
2. Check if all applications are **Synced and Healthy**.
    
3. If needed, **Manually Sync** any pending application.
    

> 🎉 **Congratulations! Your application is now fully deployed using ArgoCD!** 🚀 and can be accessed at [`http://3-111-158-0.nip.io/`](http://3-111-158-0.nip.io/)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741607218351/449d87b3-f39f-481b-a957-e79966c78210.png align="center")

---

## **Step 10: Configure Monitoring using Prometheus and Grafana**

In this step, we will **install and configure Prometheus and Grafana** using **Helm charts** to monitor the Kubernetes cluster.

### 1\. Add Helm Repositories for Prometheus & Grafana

Run the following commands to add and update the Helm repositories:

```bash
helm repo add stable https://charts.helm.sh/stable
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741610771165/040d9108-e8da-488f-915b-e33384af1489.png align="center")

### 2\. Install Prometheus and Grafana using Helm

```bash
helm install prometheus prometheus-community/kube-prometheus-stack \
  --set prometheus.server.persistentVolume.storageClass=gp2 \
  --set alertmanager.alertmanagerSpec.persistentVolume.storageClass=gp2 \
```

### 3\. Access Prometheus UI

* Get the **Prometheus service details**:
    
    ```bash
    kubectl get svc 
    #look for prometheus-kube-prometheus-prometheus svc
    ```
    
* Change the serveice type from `ClusterIP` to `Loadbalancer`
    
    ```bash
    kubectl edit svc  prometheus-kube-prometheus-prometheus
    ```
    
    * Find the line `type: ClusterIP` and change it to `type: LoadBalancer`.
        
* You can now access the prometheus server using external-IP of `prometheus-kube-prometheus-prometheus` svc
    
    ```bash
    kubectl get svc prometheus-kube-prometheus-prometheus
    ```
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741627422238/9e1a26e9-3e30-40d9-b6c6-3cd72e148a32.png align="center")
    
    Open `<External-IP>:9999` in your browser.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741628374258/3c4d69c4-d8ec-46cc-8fed-7f3fc4cd0e9e.png align="center")

* Click on **Status** and select **Target**. You'll see a list of Targets displayed. In Grafana, we'll use this as a data source.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741630034240/7a33de79-caf9-4cef-8ad7-582efb0d30ba.png align="center")
    

### 4\. Access Grafana UI

* Get the **Grafana service details**
    
    ```bash
    kubectl get svc
    #look for the prometheus-grafana svc
    ```
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741628868815/1a4c02b0-036d-4b32-8036-c62d55e03ca6.png align="center")
    
* By default, it uses `ClusterIP`. Change it to `LoadBalancer`:
    
    ```bash
    kubectl edit svc prometheus-grafana
    ```
    
    * Find the line `type: ClusterIP` and change it to `type: LoadBalancer`.
        
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741629034772/e0f43fd3-15f2-4056-aaaf-b9f7c16cc5e2.png align="center")
    
* Get the **external IP** of Grafana:
    
    ```bash
    kubectl get svc prometheus-grafana
    ```
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741629165298/fc3afc55-7cfe-4f7b-a359-8f88153cf603.png align="center")
    
* Open `<EXTERNAL-IP>` in your browser.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741629578205/00861a5b-b4d5-4951-94f0-8acdc3db0739.png align="center")

### 5\. Get Grafana Admin Password

```bash
kubectl get secret grafana -n default -o jsonpath="{.data.admin-password}" | base64 --decode
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741629668671/dc7e1e63-f33c-4a5b-9d19-dfb15311f8da.png align="center")

* **Username:** `admin`
    
* **Password:** (output from the above command)
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741629732834/b9b95613-41e7-4c98-8860-a2a354019aa6.png align="center")

### **6\. Configure Prometheus as a Data Source in Grafana**

1. Login to **Grafana UI**.
    
2. Go to **Connections** → **Data Sources**.
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741630661899/20d56a67-d85c-4208-bef6-69e9fa3cf162.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741630724688/e305815a-4d60-4696-982e-b392dce4314d.png align="center")
    
    Click **Data source** → Select **Prometheus**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741630757616/4d990978-9a0d-4c0c-bbb0-198364c40ea5.png align="center")
    
4. Provide the **Prometheus URL** (&lt;prometheus-loadbalancer-dns&gt;:9090) if not get provided.
    
5. Click **Save & Test**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741630997721/a5105f26-a119-4d4a-bc96-69cb99e34e2f.png align="center")
    

> 🎉 **Congratulations! Your Kubernetes cluster is now being monitored using Prometheus & Grafana!**

### **7\. Setting Up Dashboards in Grafana for Kubernetes Monitoring**

Grafana allows us to visualize Kubernetes cluster and resource metrics effectively. We’ll set up two essential dashboards to monitor our cluster using Prometheus as the data source.

#### **Dashboard 1: Kubernetes Cluster Monitoring**

This dashboard provides an overview of the Kubernetes cluster, including node health, resource usage, and workload performance.

**Steps to Import the Dashboard:**

* Open the **Grafana UI** and navigate to **Dashboards**.
    
* ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741631187960/c29b5cde-c389-45da-b79c-56cb8dccfaa6.png align="right")
    
    Click on **New** → **Import**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741631232457/f4d81a20-7b49-4a3a-8ba7-1def38368e4b.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741631372426/938341f1-9559-4014-8ea9-391776f58245.png align="center")
    
* In the **Import via** [**Grafana.com**](http://Grafana.com) field, enter **6417** (Prometheus Kubernetes Cluster Monitoring Dashboard).
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741631528244/d4e7e50a-dce5-4c54-84e1-a29aa143b0d5.png align="center")
    
* Click **Load**.
    
* Select **Prometheus** as the data source.
    
* Click **Import**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741631806997/c4c07457-de58-4c21-acd1-ed86ec1085d8.png align="center")
    

> You should now see a comprehensive dashboard displaying Kubernetes cluster metrics.

#### **Dashboard 2: Kubernetes Resource Monitoring**

This dashboard provides insights into individual Kubernetes resources such as pods, deployments, and namespaces.

**Steps to Import the Dashboard:**

1. Open the **Grafana UI** and navigate to **Dashboards**.
    
2. Click on **New** → **Import**.
    
3. Enter **17375** (Kubernetes Resources Monitoring Dashboard).
    
4. Click **Load**.
    
5. Select **Prometheus** as the data source.
    
6. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741632190806/881b8686-0d75-41f9-9253-635ba9f9a1fa.png align="center")
    
    select the data source i.e. `prometheus` and click on `import`
    
7. Click **Import**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741632323000/e67a6044-8017-4be3-b0ee-f2249be47c6e.png align="center")
    

> Now, you have two powerful dashboards to monitor both the overall cluster health and specific Kubernetes resources in real-time.

---

## **Conclusion**

This **Ultimate DevSecOps Project** is all about bringing security into the DevOps pipeline while deploying a **scalable, secure, and fully automated three-tier application** on **AWS EKS**. By combining the power of **Jenkins, SonarQube, Trivy, OWASP Dependency-Check, Terraform, ArgoCD, Prometheus, and Grafana**, we've built a **robust CI/CD pipeline** that ensures **code quality, security, and smooth deployments**—without any manual headaches!

With **SonarQube and OWASP Dependency-Check**, we keep our code secure and compliant. **Trivy** scans our Docker images before they even reach **AWS ECR**, blocking vulnerabilities before they hit production. **Jenkins** takes care of automation, while **ArgoCD** ensures our Kubernetes deployments stay in perfect sync. And of course, **Prometheus and Grafana** give us full visibility into system health and performance, so we're always on top of things.

This project isn't just a **DevSecOps tutorial**—it's a **real-world playbook** for modern software delivery. Whether you're a **DevOps pro, security enthusiast, or just diving into cloud automation**, this guide sets you up with the tools and best practices to **master DevSecOps in Kubernetes**.

🚀 **Ready to take your DevSecOps game to the next level? Let’s build, secure, and deploy—without limits!** 🔐🎯
