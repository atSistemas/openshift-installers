# openshift-installers
This repos contains installation tasks and helpers to provision openshift on-premise and over cloud

## AWS Openshift Installation

Deploys OpenShift in AWS via cloudformation and ansible byo

### Requirements
- AWS Account
- awscli
- Ansible
- AWS key pair configured

### AWS Resource creation using CloudFormation

- 1 CentOS-based master node
- 2 CentOS-based worker nodes
- A separate VPC and subnets to isolate our environment from any other entities
- Security groups that will open the following public ports:
    - 22 SSH for all host
    - 8443 for the OpenShift Web console, master node
    - 10250 master proxy to node hosts, master node

```shell
CF_TEMPLATE_URL=file://openshift-cf.yml

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

### Deploy openshift

Once the template has run successfully, you need to get the public IPs' of all nodes and update the inventory host file

```shell
cd ansible
# change hosts 
[OSEv3:children]
masters
etcd
nodes

[OSEv3:vars]
ansible_ssh_user=ec2-user
ansible_sudo=true
ansible_become=true

deployment_type=origin
os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'
openshift_install_examples=true

openshift_docker_options='--selinux-enabled --insecure-registry 172.30.0.0/16'
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/openshift/openshift-passwd'}]
openshift_disable_check=disk_availability,docker_storage,memory_availability

[masters]
openshift-master	ansible_host=##MASTER IP##

[etcd]
openshift-master	ansible_host=##MASTER IP##

[nodes]
openshift-master	ansible_host=##MASTER IP## openshift_node_labels="{'region':'infra','zone':'east'}" openshift_schedulable=true
openshift-worker1	ansible_host=##WORKER1 IP## openshift_node_labels="{'region': 'primary', 'zone': 'east'}"
openshift-worker2	ansible_host=##WORKER2 IP## openshift_node_labels="{'region': 'primary', 'zone': 'east'}"
```

Update ansible.cfg with your AWS key pem
``` bash
[defaults]
private_key_file=##PATH TO YOUR PEM KEY##
host_key_checking = False
inventory=hosts
```

Run requirements
``` bash
ansible-playbook requirements.yml
```

Checkout openshift-ansible repo
``` bash
git clone https://github.com/openshift/openshift-ansible.git
cd openshift-ansible
git checkout origin/release-3.7
cd ..
ansible-playbook openshift-ansible/playbooks/byo/config.yml
```