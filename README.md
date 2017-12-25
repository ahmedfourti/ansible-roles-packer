Role Name: Packer
=========

This role is used to provision AWE EC2 instances backed by EBS volumes.  
First it makes sure that Packer is present on the local machine.  
Then it can provide two provisioning methods: Shell commands and/or Ansible-local.  

Requirements
------------

Please make sure to configure your AWS Keys.  
You need to create the following file ```~/.aws/credentials```  and then put in your keys.  

```
[default]
aws_access_key_id = XXXXXXX
aws_secret_access_key = XXXXXXX
```

If you use the ansible-local provisioner, please make sure to use also the shell command to install ansible.  
As its name suggests, ansible-local will execute roles locally inside the new EC2 Instance, that means it need python and ansible installed.    

Role Variables
--------------

**Packer variables :**
  
**packer_version**: The desired version of packer  
**system_type**: The system architecture where Packer will be executed (x86 or x64)  
**dest_bin_path**: The path where to move the Packer binary   
**temp_path**: The path where to download the Packer archive   
**checksum**: The checksum of the Packer archive to be sure to get the good one (for security reasons)      


**Builder variables :**  

**source_ami_id**: The AMI source ID  
**source_ami_owner**: The source AMI owner ID  
**ami_instance_type**: The instance type to get used during the build  
**aws_region**: The AWS Region where to launch the build and store the final AMI  
**ssh_username**: The ssh username to be used by Packer (depends on the source AMI choosen before)  
**ami_name**: The name for the final AMI to get registered  


**Shell Provisioners variables :**  

**shell_commands**: This variable is a list of commands to run when building the image (cf Example)  


**Ansible-Local provisioner :**

**roles_base_dir**: The path to roles to play. They will be copied to instance    
**playbook_file**: The playbook to play inside the remote instance (This must be configured locally before building)  
**provisioner_roles**: This is a list of roles to get run when building the image (cf Example)    

Dependencies
------------

N/A

Example Playbook
----------------

In this example; I download Packer 1.1.3 and move it to "/usr/local/bin".  
Then I used the shell-command provisioner to install Python with some of its dependencies and then the ansible-local provisioner to play the given playbook with the listed roles.  


    ---
    - name: Deploying Demo
      hosts:
        - localhost
      roles:
        - packer
      become: true
      vars:
       #Role Packer
        packer_version: 1.1.3
        system_type: amd64
        dest_bin_path: /usr/local/bin
        temp_path: /tmp
        checksum: sha256:b7982986992190ae50ab2feb310cb003a2ec9c5dcba19aa8b1ebb0d120e8686f
        template_name: ec2_instance
        source_ami_id: debian-jessie-amd64-hvm-2017-01-15-1221-ebs
        source_ami_owner: 379101102735
        ami_instance_type: t2.micro
        aws_region: eu-west-1
        ssh_username: admin     
        ami_name: base-ami-jessie-{{ ansible_date_time.epoch }}
        shell_commands:
          - sudo apt-get update -y
          - sudo apt-get upgrade -y
          - sudo apt-get install build-essential libssl-dev libffi-dev python-dev git curl -y
          - sudo curl --silent --show-error --retry 5 https://bootstrap.pypa.io/get-pip.py | sudo python
          - sudo pip install ansible==2.3.2.0          
        roles_base_dir: "roles"
        playbook_file: playbook.yml
        provisioner_roles:
          - utils
          - java
          - apache
          - elasticsearch

Don't forget to configure the ```playbook.yml``` that will be used inside the remote instance if you need ansible-local provisioner. 
To get the listed role above your can use ```requireemnts.yml``` that will download them to your local computer (make sure to specify the right path)       

License
-------

BSD

