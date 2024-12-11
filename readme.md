#Dynamic Inventory

aws_ec2.yaml: It uses plugin aws_ec2 to get EC2 information from AWS. EC2 having tag "is_ansible_managed" will be grouped in group "ansible_true"
```
keyed_groups:
  - key: tags.is_ansible_managed | string
```

#Prod Playbook

The playbook execute tasks defined in deploy_webserver roles folder to install httpd service on hosts in group "ansible_true" 
```
---
- hosts: ansible_true
  gather_facts: True
  become: yes
  vars_files:
   - ../prod/group_vars/prod_vars.yml
  roles:
   - { role: ../roles/deploy_webserver }
```
#Var files

Each environment has its own vars file. Eg: prod env uses prod_vars.yml in /prod/group_vars
```
---
#Declare source of index.html
source_file: ../templates/index.html

#Declare location of index.html in webserver
dest_file  : /var/www/html

#Declare location of private key used in control node to run prod playbook 
ansible_ssh_private_key_file : ../key_pairs/private_key.pem
```

#Private key and control node

Control node: it uses github action runner to deploy control node in workflow
Private key: key value is stored securely in github environment secret. Then it's passed into control node when workflow is triggered. Eventually, it's destroyed after the workflow is completed
```
      - name: setup ssh private key
        working-directory: ./key_pairs
        run: |
         echo "${{ secrets.VOC_KEY }}" > private_key.pem
         sudo chmod 600 private_key.pem
```
