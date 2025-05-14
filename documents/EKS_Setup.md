# step 1 : create EKS management Host in AWS
---------------------------------------
**1.**  **Launch new ubuntu vm using aws ec2(t2.micro).**

**2.** 
 **connect to machine and install kubectl using below command.**
~~~
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.0/2024-09-12/bin/linux/amd64/kubectl


chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin

kubectl version --client
~~~
**3.** **install AWS CLi latest Version using below command.**

~~~
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
~~~
**4.** **install eks using below command.**
~~~
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
eksctl version
~~~

# STEP-2: Create IAM role & Attach to EKS Management Host
--------------------------------------
**1. create new role using iam service(select Usercase ec2)**

**2. add below permission for the role.**
   * **IAM - fullaccess**
   * **VPC - fullaccess**
   * **ec2 - fullaccess**
   * **cloudFormation- fullaccess**
   * **Administrator - Access**
**3. enter role name (eksroleec2).**
**4. Attach created role to EKS Management Host (Select ec2-->click on security--->modify iam role--->attach iam role we have created.)**
# step -3 : Create EKS Cluster using eksctl
**syntax:**

~~~
eksctl create cluster --name cluster-name
--region region-name
--node-type instance-type
--nodes-min 2
--nodes-max 2
--zones <zone name>
~~~
## Example:
**Mumbai:**

~~~
eksctl create cluster --name shaif-cluster --region ap-south-1 --node-type t2.medium --zones ap-south-1a,ap-south-1b
~~~
 **cluster creation take 5 to 15 min**


# delete cluster

~~~
eksctl delete cluster --name shaifcluster --region ap-south-1
~~~


#  ************** EKS cluster setup completed *********