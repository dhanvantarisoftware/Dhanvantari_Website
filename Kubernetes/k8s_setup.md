# Kubernetes Setup on Amazon EKS

You can follow the same procedure in k8s official documentation. AWS document [Getting started with Amazon EKS – eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)

#### Pre-Requisites:
  - EC2 Instance (t2.small)
  - Install AWSCLI latest verison in the EC2 Instance

1. Setup kubectl
   a. Download latest kubectl version 1.25
   b. Grant execution permissions for kubectl as a executable
   c. Move kubectl onto /usr/local/bin
   d. Test that your kubectl installation was successful by using the command kubectl version --short --client

   ```sh
   curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.6/2023-01-30/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mv ./kubectl /usr/local/bin
   kubectl version --short --client
   ```
2. Setup eksctl
   a. Download and extract the latest release of eksctl
   b. Move the extracted binary to /usr/local/bin
   c. Test that your eksctl installation was successful by using the command eksctl version

   ```sh
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
   sudo mv /tmp/eksctl /usr/local/bin
   eksctl version
   ```

3. Now create an IAM Role and attache it to EC2 instance

   IAM user should have access to the following services

   IAM, EC2, CloudFormation & AdministorAccess
   
   Note: Check eksctl documentaiton for [Minimum IAM policies](https://eksctl.io/usage/minimum-iam-policies/)

4. Now create your k8s cluster and nodes
   ```sh
   eksctl create cluster --name cluster-name  \
   --region region-name \
   --node-type instance-type \
   --nodes-min 2 \
   --nodes-max 2 \
   --zones <AZ-1>,<AZ-2>

   Example:
   eksctl create cluster --name dhanvantari-cluster \
      --region us-east-2 \
   --node-type t2.medium \
    ```

5. To delete the EKS clsuter execute the below command
   ```sh
   eksctl delete cluster dhanvantari-cluster --region ap-south-1
   ```

6. Validate your cluster using by creating by checking nodes and by creating a pod
   ```sh
   kubectl get nodes
   kubectl run nginx --image=nginx
   ```

   #### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl create deployment nginx-deployment --image=nginx --replicas=4 --port=80
    # kubectl deployment mydeployment --image=dhanvantarisoftware/dhanvantari-image --replicas=4 --port=8080
    kubectl get all
    kubectl get pod
   ```

1. Expose the deployment as service. This will create an ELB in front of those 4 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer
   # kubectl expose deployment mydeployment --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```