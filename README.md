# Launching-A-React-Application-Using-Terraform-And-Ansible-s-Dynamic-Inventory-Role-on-AWS

### Hello, automation technicians !!
In this article, i have launched a react application inside a RedHat instance launched on AWS. The respective ec2 instance has been launched using terraform (one of the great provisioning tool) on AWS cloud and created a nodejs application in that respective instances using Ansible (one of the great automation configuration tool). The nodejs application is configured using roles and dynamic inventory.
Why terraform and Ansible only for this. So look at the below graph. you will get some little info.

![alt text](https://miro.medium.com/max/875/1*k39QO_lyJga0ivtR_XEetA.png)

Ansible has taken a lead role in doing the configuration management in the automation world whereas terraform rising on the edge of provisioning operating systems. The graph is outdated but its tells you a truth about the today’s reality.
In this article, all the codes and concepts explained in detailed. So sit at the back with one cup of coffee and read bit by bit.
So, lets start to build this automation. First we have to install terraform from below link.

[`Install Terraform`](https://www.terraform.io/downloads.html)

Now, you need to set the path in the environment variables. After setting the environment variables check the terraform version. you can use `terraform -version` to check the version of terraform.
![Terraform Apply](https://miro.medium.com/max/625/1*thK7hLyvQwUzmN2YFDMPmA.png)

After we need to make directory and create a file with .tf extension. .tf is the extension of terraform.

``` 
  provider "aws" {
  region     = "ap-south-1"
  profile    = "IAM_User_Name"
} 
```
The terraform code will be starting by taking the user name which you have created in your AWS account. You need to give the region also in the provider block. **provier → aws (else your cloud name)**
```
variable "cidr_subnet1" {
  description = "CIDR block for the subnet"
  default = "192.168.1.0/24"
}
variable "availability_zone" {
  description = "availability zone to create subnet"
  default = "ap-south-1"
}
```
Above code will create a variable of different subnets and availability zones. cidr_subnet1, availability_zone are the variables. The rage of IP’s is given to its respective subnets. Also you need to mention your default region *availability_zone* block above.
## Creating VPC
Amazon Virtual Private Cloud **(Amazon VPC)** enables you to launch AWS resources into a virtual network that you’ve defined. This virtual network closely resembles a traditional network that you’d operate in your own data center, with the benefits of using the scalable infrastructure of AWS.
```
resource "aws_vpc" "vpc" {
  cidr_block = "${var.cidr_vpc}"
  enable_dns_support   = true
  enable_dns_hostnames = true
tags ={
    Environment = "${var.environment_tag}"
    Name= "TerraformVpc"
  }
}
```
the above code will create VPC. The above vpc block accept the cidr_block. you can enable dns support given to ec2 instances after launching. You can give the tag name to the above vpc. Here i have given **“TerraformVpc”**.

      ![VPC](https://miro.medium.com/max/875/1*-hQAfEPTD58xg3huSb2dSg.png)

## Creating Subnets
Subnetwork or subnet is a logical subdivision of an IP network. The practice of dividing a network into two or more networks is called subnetting. AWS provides two types of subnetting one is Public which allow the internet to access the machine and another is private which is hidden from the internet.
```
resource "aws_subnet" "subnet_public1_Lab1" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "${var.cidr_subnet1}"
  map_public_ip_on_launch = "true"
  availability_zone = "ap-south-1a"
  tags ={
    Environment = "${var.environment_tag}"
    Name= "TerraformPublicSubnetLab1"
  }
}
```
The above code will create subnet. The subnet block accepts *vpc_id*, *cidr_block*, *map_public_ip_on_launch(assign public IP to instance after launching)*, *availability_zone*. You can give tags for easy recognition after creating subnets.

![Subnets](https://miro.medium.com/max/875/1*EnzVjUm5hArzCgrSMXUbzQ.png)

## Creating Security Group
A security group acts as a virtual firewall for your EC2 instances to control incoming and outgoing traffic. … If you don’t specify a s**ecurity group**, Amazon EC2 uses the default security group. You can add rules to each security group that allow traffic to or from its associated instances.
```
resource "aws_security_group" "TerraformSG" {
  name = "TerraformSG"
  vpc_id = "${aws_vpc.vpc.id}"
  ingress {
      from_port   = 0
      to_port     = 0
      protocol    = "-1"
      cidr_blocks = ["0.0.0.0/0"]
  }
egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags ={
    Environment = "${var.environment_tag}"
    Name= "TerraformSG"
  }
}
```
The above block of code will create a security group. It accepts the respective parameters name(name of the security group), vpc_id, ingress(inbound rule) here i have given “all traffic”. *“-1” means all*. **from_port= 0 to_port=0 (0.0.0.0)** that means we have disabled the firewall. you need to mention the range of IP’s you want have in inbound rule.

The egress rule is the outbound rule. I have taken *(0.0.0.0/0)* means all traffic i can able to access from this outbound rule. You can give the name of respective Security Group.

![SecurityGroup](https://miro.medium.com/max/875/1*cL4s0FCPaOyfvAJEjJNJ1Q.png)

## Creating InternetGateway
An internet gateway serves two purposes: to provide a target in your VPC route tables for internet-routable traffic, and to perform network address translation (**NAT**) for instances that have been assigned public IPv4 addresses.

```
resource "aws_internet_gateway" "gw" {
  vpc_id = "${aws_vpc.vpc.id}"
tags = {
    Name = "Terraform_IG"
  }
}
```
the above code will create you respective internet gateway. you need to specify on which vpc you want to create internet gateway. Also you can give name using tag block.

![InterNetGateway](https://miro.medium.com/max/875/1*FL2U8yjnr2-eY0loqcnSOQ.png)

## Creating Route Table
A route table contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.
```
resource "aws_route_table" "r" {
  vpc_id = "${aws_vpc.vpc.id}"
route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
  tags = {
    Name = "TerraformRoteTable"
  }
}
```
You need to create a route table for the internet gateway you have created above. Here, i am allowing all the IP rage. So my ec2 instances can connect to the internet world. we need to give the **vpc_id** so that we can easily allocate the routing table to respective vpc. You can specify the name of the routing table using tag block.

![Route Table](https://miro.medium.com/max/875/1*Zrob2Zj5o1fwWffYJnCBtw.png)
## Route Table Association To Subnets
We need to connect the route table created for internet gateways to the respective subnets inside the vpc.
```
resource "aws_route_table_association" "public" {
  subnet_id = "${aws_subnet.subnet_public1_Lab1.id}"  
  route_table_id = "${aws_route_table.r.id}"
}
```

You need to specify which subnets you want to take to the public world. As if the subnets gets associated(connected) to the Internet Gateway it will be a public subnet. But if you don’t associate subnets to the Internet gateway routing table then it will known as private subnets. The instances which is launched in the private subnet is not able to connected from outside as it will not having public IP, also it will not be connected to the Internet Gateway.

You need to specify the routing table for the association of the subnets. If you don’t specify the routing table in the above association block then subnet will take the vpc’s route table. So if you want to take the ec2 instances to the public world then you need to specify the router in the above association block. Its upon you which IP range you want you ec2 instances to connect. Here i have give 0.0.0.0/0 means i can access any thing from the ec2 instances.

## Creating Ec2 Instances
An EC2 instance is nothing but a virtual server in Amazon Web services terminology. It stands for Elastic Compute Cloud. It is a web service where an AWS subscriber can request and provision a compute server in AWS cloud. … AWS provides multiple instance types for the respective business needs of the user.
```
resource "aws_instance" "testInstance1" {
  ami           = "ami-052c08d70def0ac62"
  instance_type = "t2.micro"
  subnet_id = "${aws_subnet.subnet_public1_Lab1.id}"
  vpc_security_group_ids = ["${aws_security_group.TerraformSG.id}"]
  key_name = "ansiblekey"
 tags ={
    Environment = "${var.environment_tag}"
    Name= "RedhatNodeJs"
  }
}
```
In the above code, it will create a new instance. It accepts the following parameters.
```
ami-> image name
instance_type-> type of instances
subnet_id-> In which subnet you want to launch instance
vpc_security_group_ids->  Security Group ID
key_name-> key name of instance
Name-> name of the instance

```

![RedHatNodeJs](https://miro.medium.com/max/875/1*QC_XdhuJSjx7mOGpXkZJTQ.png)
Now we need to run the terraform code to get the above output
## Initializing terraform code
The terraform init command is used to initialize a working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration or cloning an existing one from version control. It is safe to run this command multiple times. You can initialize using `terraform-init`.

![terraform initialize](https://miro.medium.com/max/875/1*-veaPigw_55_Ye3ZX-Je3w.png)
## Running Terraform Apply
The terraform apply command is used to apply the changes required to reach the desired state of the configuration, or the pre-determined set of actions generated by a terraform plan execution plan. You can apply the terraform code using `terraform-apply`.

![terraform apply](https://miro.medium.com/max/875/1*lUiIqyBBCx_Vs8mEiecT8Q.png)
You need to give yes/no for for making the changes in AWS.

![yes/no](https://miro.medium.com/max/875/1*mhYDxjxc3pZG7bpsmrEy5w.png)
As you can see that 9 resources has been added to the AWS. The above is the changes done in AWS. You can also destroy the changes you made using following command.

![changes done](https://miro.medium.com/max/875/1*Fb_6WB7M4ia7IbIgLKo_Ow.png)
What we have created its looks like something below architecture.

![vpc architecture](https://miro.medium.com/max/875/1*CRTpWdrpbUub-wW0XnEM3g.png)
Now the instances is launched successfully, So we can now use Ansible for the Configuration of Apache webserver in that respective instance.
## What is Ansible ??
Ansible is a software tool that provides simple but powerful automation for cross-platform computer support. It is primarily intended for IT professionals, who use it for application deployment, updates on workstations and servers, cloud provisioning, configuration management, intra-service orchestration, and nearly anything a systems administrator does on a weekly or daily basis. Ansible doesn’t depend on agent software and has no additional security infrastructure, so it’s easy to deploy.
## How Ansible works
In Ansible, there are two categories of computers: the control node and managed nodes. The control node is a computer that runs Ansible. There must be at least one control node, although a backup control node may also exist. A managed node is any device being managed by the control node.

Ansible works by connecting to nodes (clients, servers, or whatever you’re configuring) on a network, and then sending a small program called an Ansible module to that node. Ansible executes these modules over SSH and removes them when finished. The only requirement for this interaction is that your Ansible control node has login access to the managed nodes. SSH Keys are the most common way to provide access, but other forms of authentication are also supported.

## Ansible playbooks
While modules provide the means of accomplishing a task, the way you use them is through an Ansible playbook. A playbook is a configuration file written in YAML that provides instructions for what needs to be done in order to bring a managed node into the desired state. Playbooks are meant to be simple, human-readable, and self-documenting. They are also idempotent, meaning that a playbook can be run on a system at any time without having a negative effect upon it. If a playbook is run on a system that’s already properly configured and in its desired state, then that system should still be properly configured after a playbook runs.

## Modules in Ansible
Modules (also referred to as “task plugins” or “library plugins”) are discrete units of code that can be used from the command line or in a playbook task. Ansible executes each module, usually on the remote managed node, and collects return values.

## Variables in Ansible:
Ansible uses variables to manage differences between systems. With Ansible, you can execute tasks and playbooks on multiple different systems with a single command. … You can define these variables in your playbooks, in your inventory, in re-usable files or roles, or at the command line.

## Ansible Installation In below Slides

[`Ansible Installation`](https://docs.google.com/presentation/d/1k9v81cVfb96Vxq2ETAg5H7wUq4CXgxJnN74VERNhpX8/edit)
## Dynamic Inventory for AWS
Dynamic inventory is an ansible plugin that makes an API call to AWS to get the instance information in the run time. It gives you the ec2 instance details dynamically to manage the AWS infrastructure.
Create a directory then add that directory in the configuration file of ansible using `mkdir  Dynamic_Inventory_Data_Base`. You can give any name to the directory.

![Ansible Configuration File](https://miro.medium.com/max/875/1*OImAzWtQNGdzCO5VziAJhA.png)
As you can see the configuration file of ansible, In the default section there is **inventory** which will take the data of the instances launched on **AWS**.
As Ansible works on the ssh protocol for linux OS. So we need to *disable* the ssh key, as when you do ssh it asks you for *yes/no*. So to disable that you need to write **host_key_checking=false.**
To avoid some warnings given by the command we can disable that using **command_warnings=false**
To login in that newly launched OS we need to provide its respective key. Note: key with .pem format file will work not with the .ppk format. Also you need to give permission to that key in the read mode.
```
Permissions in linux
0 → No modes
1 → execute mode
2→ write mode
3 → write execute mode
4 → read mode
5 → read write mode
6 → read write mode
7 → read write and execute mode.
```
So, here i am giving the execute permission to that key. `chmod   400   keyname.pem`
The code for the configuration file of ansible is below.
```
[defaults]
inventory=location
host_key_checking=false
ask_pass=false
private_key_file=/path/key.pem
[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```
As i told earlier to create a directory eg. **Dynamic_Inventory_Data_Base**. So you need to install the **dynamic inventory module** inside the folder you have created. First you have install the **wget** software. `yum install wget`.
After downloading this you need to download the dynamic inventory module i.e **ec2.py** and **ec2.ini**. Both file is interdependent. The **ec2.ini** stores the information of that AWS account and **ec2.py** run that modules and collects the information of instances launched on AWS.
```
This 👇 command will create a ec2.py dynamic inventory file 

wget   https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.py

This 👇 command will create a ec2.ini dynamic inventory file

wget   https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.ini


You need to make executable those two above files

chmod  +x ec2.py 

chmod  +x ec2.ini
```
![installing ec2.py and ec2.ini](https://miro.medium.com/max/875/1*a78aGC-1Dz1-SMVBV3zyjA.png)
Making **ec2.py** and **ec2.ini** file in executable mode.

![Making ec2.py and ec2.ini file in executable mode](https://miro.medium.com/max/875/1*hLDpF4p1ZJPEwv0kWMD3Dw.png)

Now you need to update the file of ec2.py. The first line of ec2.py include python environment. we need to update with the python3.

![python module error](https://miro.medium.com/max/875/1*8w2Nu6twIU1Sh5yb0-SLDg.png)

`#!/usr/bin/python3` This is the *shebang* you need to edit inside **ec2.py** file.

![ec2.py](https://miro.medium.com/max/875/1*tc_do-fyVzt76AHKiZgLLA.png)

Now, we need to update the regions and AWS IAM account access_key and secret_access_key inside ec2.ini file.

![updating region](https://miro.medium.com/max/875/1*mmUianUtwW9W_Qy2v6vnYg.png)

Now you need to export the region, acess_key, secret_access_key with the command line.
```
After that export all these commands.
export AWS_REGION='ap-south-1'
export AWS_ACCESS_KEY=XXXX
export AWS_ACCESS_SECRET_KEY=XXXX

```
Now all set, we can check the number of hosts having on the AWS.

![dynamically fetched the ec2 insatnce public IP](https://miro.medium.com/max/875/1*sHYR2qi-P9l_s7V2FrOb_g.png)

### [Complete React Js Code](https://drive.google.com/drive/folders/19z0aSwW4peK9XbcoGm-Khx1FWVNBhDcz?usp=sharing)

## Creating Roles From Ansible Galaxy
Roles let you automatically load related vars_files, tasks, handlers, and other Ansible artifacts based on a known file structure. Once you group your content in roles, you can easily reuse them and share them with other users. you can create role using `ansible-galaxy  init nodejs`.

![nodejs role created](https://miro.medium.com/max/875/1*L3xCQy9dDwiYn6vh0_D7ww.png)

## Creating Tasks of nodejs role
```
- name: "Installing Nodejs"
    shell: "yum install nodejs -y"
- name: "Installing npm"
    shell: "yum install npm -y"
- name: "copying the Challenge3 directory"
    copy:
       src: "/root/Assignment/challenge3"
       dest: "/root/"
    ignore_errors: yes
- name: "moving to the workspace"
    shell: "cd /root/challenge3/"
- name: "Installing required packages using npm"
    shell: "npm i"
```
In the above task playbook, i have used the shell module to download the nodejs and npm. Then i used the copy module the nodejs application content. “npm i” will install the npm dependencies softwares.

## Main Playbook Code
Main playbook are the playbook which contains all the code or we can say it contains all the respective playbooks, roles, variables, etc.

```
- hosts: "localhost"
  roles:
  - name: "nodejs role"
    role: "/root/Assignment/nodejs"
```
![main_playbook.yml](https://miro.medium.com/max/875/1*ACQ_nk8lc_9rYtf1vv-hRw.png)

## Run Main Playbook 
You can run the main playbook using `ansible-playbook main_playbook.yml`.

![main playbook running](https://miro.medium.com/max/875/1*0oSJJjMekESdxQwdTupZCw.png)
Now you need go inside this ec2 instance and go inside challenge3 folder `cd challenge3`. Then you need to run `npm i`.

![installing dependencies](https://miro.medium.com/max/875/1*UNkwBCpAjlfZ_Ul1l4h7FA.png)

This will again refresh the dependencies of the nodejs application. Now you need to run the react application `npm  run dev`.

![running react application](https://miro.medium.com/max/875/1*Qo1mEla8Tg3ACmjI2lJQbA.png)

To see the application running inside ec2 instance you can check using `public_Ip:3000`.
## React Application Deployed
![React Application Deployed](https://miro.medium.com/max/875/1*IPq_KCnIW0tXJdTQeamK-w.png)
