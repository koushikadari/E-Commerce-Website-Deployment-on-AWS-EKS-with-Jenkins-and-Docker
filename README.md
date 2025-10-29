# E-Commerce-Website-Deployment-on-AWS-EKS-with-Jenkins-and-Docker
E-Commerce Website Deployment on AWS EKS with Jenkins and Docker
🧩 1. CLI Tools to Install

🔹 AWS CLI                                                             
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

🔹 kubectl                                                                                 
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.1/2025-09-19/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

🔹 eksctl                
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
eksctl version



☸️ 2. Create EKS Cluster
eksctl create cluster --name=EKS-1 \
--region=us-east-1 \
--zones=us-east-1a,us-east-1b \
--without-nodegroup



🧾 3. Attach IAM OIDC Provider

eksctl utils associate-iam-oidc-provider \
--region us-east-1 \
--cluster EKS-1 \
--approve




🧠 4. Create Node Group

eksctl create nodegroup \
--cluster=EKS-1 \
--region=us-east-1 \
--name=node2 \
--node-type=c6i.large \
--nodes=3 --nodes-min=2 --nodes-max=4 \
--node-volume-size=20 \
--ssh-access --ssh-public-key=git \
--managed --asg-access \
--external-dns-access --full-ecr-access \
--appmesh-access --alb-ingress-access



🧭 5. Configure kubectl to Access Cluster


aws eks update-kubeconfig --name EKS-1 --region us-east-1
kubectl config view


⚙️ 6. Jenkins Setup

🔹 Install Jenkins and Docker                    
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo chmod 777 /var/run/docker.sock


Then install Jenkins and access via:             http://<EC2-Public-IP>:8080


🔹 Jenkins Plugins to Install

Pipeline: Stage View

Docker Pipeline

Kubernetes Plugin

Kubernetes CLI Plugin



🐳 7. Integrate Docker Hub

Go to Manage Jenkins → Credentials → Global

Add new credentials:

      Username: kous763

      Password: Koushikadari@0717

      ID: docker-cred


🧠 8. Integrate Kubernetes (EKS) in Jenkins

Step 1: Create Namespace
        kubectl create ns webapps


Step 2: Create Service Account
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: jenkins
          namespace: webapps


kubectl apply -f service.yml



Step 3: Create Role & RoleBinding

(Allow Jenkins to manage deployments, pods, and services)



role.yml


apiVersion: rbac.authorization.k8s.io/v1                                                     
kind: Role
metadata:
        namespace: webapps
        name: jenkins-role                          
rules:
  - apiGroups: [""]                                                      
    resources: ["pods", "services", "configmaps"]                                  
    verbs: ["get", "list", "create", "delete", "update", "watch"]                                                  
  - apiGroups: ["apps"]                         
    resources: ["deployments"]                                                                  
    verbs: ["get", "list", "create", "delete", "update", "watch"]                                                                     




apiVersion: rbac.authorization.k8s.io/v1                               
kind: RoleBinding
metadata:                       
  name: jenkins-rolebinding
  namespace: webapps                                          
subjects:                          
  - kind: ServiceAccount                                              
    name: jenkins                           
    namespace: webapps                        
roleRef:                                               
  kind: Role                                    
  name: jenkins-role                             
  apiGroup: rbac.authorization.k8s.io                             




kubectl apply -f role.yml
kubectl apply -f rolebinding.yml


Step 4: Create Secret Token for Jenkins

secret.yml

apiVersion: v1
kind: Secret                
metadata:                    
  name: mysecret                 
  namespace: webapps                 
  annotations:                     
    kubernetes.io/service-account.name: jenkins                  
type: kubernetes.io/service-account-token
                
kubectl apply -f secret.yml
kubectl describe secret mysecret -n webapps



Use this token in Jenkins Kubernetes Credentials for accessing your cluster.


🧼 9. Delete EKS Cluster (Cleanup)                                          
eksctl delete cluster --name EKS-1 --region us-east-1

























