# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS(imports and roles)

### Activities involved.

1 .refactor ansible code

2.Create assisgnments.

3.use import functionality which allows to effectively use previouslycreated playbooks in a new playbook(organise your tasks and reuse them when needed)

# Refactoring ansible code by importing other playbooks in site.yml.

## Jenkins job enhancement.

  As every new job done in changes done on a code,creates a new directory and takes up space we always create pluging to sort out these issue which i s done by doing "copy artifact" on jenkins.


## Install pluggins on jenkins without starting jenkins

## Procedure

Jenkins>manage jenkins>manage plugins> then on  Available
search copy artifacts
 Trigger as per the existing project which is ansible 
nunmber of builds . Configure the Ansible project on jenkins by clicking on discard builds
 Strategy to be Log rotatin
 Max builds to keep to be 2 in this project
 Source code to be nome  Build triggers-Build after othe projects are built-ansiblre

  #### Go on builds
 Artifacts to copy **
 Target directory in this case will be /home/ubuntu/ansible-config-artifact
 Finger print artifacts
 Then apply and save all the changes made.

 end result
 
![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/9b649b95-df26-42b6-b927-20d3eed46bf0)

 ## Created artifact directory

At the same time changes are also made on the Jenkins-Ansible server  and in this case we create a directory  and name it "ansible-config-artifacts"

  Command used is [sudo mkdir/home/ubuntu/ansible-config-artifacts]

   All files will now appear on the /home/ubuntu/ansible-config-artifact
 
 and will be updated with every commit you make on the main branch
  
 ![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/ec3a57a4-4df9-4b56-aa94-eac63e11028f)


  Changing permissions is also done here so that the jenkins can have the permission to save files in the directory using the command 

  [chmod -R 0777 /home/ubuntu/ansible-config-artifact]

# IMPORT PLAYBOOKS

 ## Refactoring Ansible code  by importing playbooks into site.yml

  {Create refactor branch where all of the project wiil  be running. as you make all the changes you need to make. This all ows your work to be neat and easy to go through for all the other team member you are working with before the final changes are made which will be 

  termed as the the desirable results by the person managing the project where now you can send a pull request and merge the changes made to the main project.  All the changes being made on the terminal have to be pushed to the refactor branch created then a pull 
  
  request is made if the changes are acceptable and are working well to avoid too any errors.}

   In playbook, create a folder and name it site.yml which will act as an entry/parent to all other playbooks. (point to 
   
   all other configurations) including common.yml

 Create a folder  called Static-assisgnments  n the root of the repository where all the children play books will be stored
 
 for easy organisation of work

 move  common.yml file to static- assignments folder created earlier

 ![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/f8f5294c-0dae-4387-9b2a-2cb7cfda77d3)

# Deleting wireshark.

## import common.yml playbook to site.yml
 ---
- hosts: all
- import_playbook: ../static-assignments/common.yml

   ├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml

   Create a file common-del.yml for deletion of wireshark using the below playbook configuration.

---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes

      
- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes

 Run the command below to ensure wireshark has ben deleted
 
 cd /home/ubuntu/ansible-config-mgt/

ansible-playbook -i inventory/dev.yml playbooks/site.yaml

 wireshark --version
 
![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/4be89583-dffd-4971-9460-e2ca0ba38494)

 # Configure uat webserver with a role webserver
 
 create 2 webservers

create a roles directory  in the ansible-config-mgt then inititialise webserver 
  
 Expected structure
 
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    ├── templates
    ├── tests
    │   ├── inventory
    │   └── test.yml
    └── vars
        └── main.yml

  Remove uneccessary files and you remain withte below structure
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates

 the above configuration as seen on the jump server 
 
  ![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/5816fa76-c487-4861-8ed9-e76ed24f9d19)


   update your uat.yml to set up the 2 uat webservers

   [uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ubuntu'/or ec2-user

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user'OR ubuntu

![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/7566ae87-5d54-4ca4-92ac-e22cee6b464e)

### -config-mgt/role

 run [vi /etc/ansible/ansible.cfg] AND UNCOMMENT roles_path

 provide full path to your role directory  so as ANSIBLE CAN KNOW WHERE TO FIND CONFIGURED ROLES

that is [roles_path=/home/ubuntu/ansible-config-mgt/roles]
 


  ## Installing apache ie httpd and cloning a repo
  
 CONFIGURE THE main.yml file in the task directory

  Run the script below to clone into git hub and install apache 
  
 ---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent

  ![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/6a6e7877-419f-4785-8b87-d182c0ec2e44)

   Using the code below ,test if the playbook is working on jenkins webserver.

 [ansible-playbook -i /inventory/uat.yml playbooks/site.yml]
 
![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/579f3847-ea0b-4072-8539-03f8becfb38b)

  # Reference webserver role

   Run the config script in  tne site.yml to create a new assisgnment for the uat webserver.yml

 using the code below.cd
  ---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

# Commit and test.

check if both webservers are configured and try reach both of them on the browser using {http://<uat webserver public ip adress>/index.php}. You can also use the dns name instead aof the public ip.
##### Reference ip addresses

![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/29d114c7-e809-47c6-aef1-ae2033a88010)

#### Output.
 ![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/14e4cf09-9f69-4612-b98b-ef931bd30206)

![image](https://github.com/NANA-2016/Ansible-refactoring/assets/141503408/2564cad9-1c04-42bf-942b-4e857cae58e4)






 
