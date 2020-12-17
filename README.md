# Launching-A-React-Application-Using-Terraform-And-Ansible-s-Dynamic-Inventory-Role-on-AWS

#Hello, automation Engineers !!
In above repo, i have launched a react application inside a RedHat instance on AWS. The respective ec2 instance has been launched using terraform (one of the great provisioning tool) on AWS cloud and created a nodejs application in that respective instances using Ansible (one of the great automation configuration tool). The nodejs application is configured using roles and dynamic inventory.

# Terraform 
Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions. Configuration files describe to Terraform the components needed to run a single application or your entire datacenter.

# Dynamic inventory
Dynamic inventory is an ansible plugin that makes an API call to AWS to get the instance information in the run time. It gives you the ec2 instance details dynamically to manage the AWS infrastructure.

# Roles
Roles let you automatically load related vars_files, tasks, handlers, and other Ansible artifacts based on a known file structure. Once you group your content in roles, you can easily reuse them and share them with other users.

# Main Playbook
Main playbook are the playbook which contains all the code or we can say it contains all the respective playbooks, roles, variables, etc.
