# openshift-installers
This repos contains installation tasks and helpers to provision openshift on-premise and over cloud

## AWS Openshift Installation

Deploys OpenShift in AWS via cloudformation and ansible byo

### Requirements
- AWS Account
- awscli
- Ansible

### AWS Resource creation using CloudFormation

- 1 CentOS-based master node
- 2 CentOS-based worker nodes
- A separate VPC and subnets to isolate our environment from any other entities
- Security groups that will open the following public ports:
    - 22 SSH for all host
    - 8443 for the OpenShift Web console, master node
    - 10250 master proxy to node hosts, master node

 ```shell
CF_TEMPLATE_URL=your-s3-bucket

# Where to place your cluster (ireland)
REGION=eu-west-1
AVAILABILITY_ZONE=eu-west-1b

# What you want to call your CloudFormation stack
STACK_NAME=openshift-cf-stack

# What SSH key you want to allow access to the cluster (must be created ahead of time in your AWS EC2 account)
SSH_KEYNAME=ssh_key_name
    
aws cloudformation create-stack \
 --region $REGION \
 --stack-name $STACK_NAME \
 --template-url $CF_TEMPLATE_URL \
 --parameters \
   ParameterKey=AvailabilityZone,ParameterValue=$AVAILABILITY_ZONE \
   ParameterKey=KeyName,ParameterValue=$SSH_KEYNAME \
 --capabilities=CAPABILITY_IAM
  ```




