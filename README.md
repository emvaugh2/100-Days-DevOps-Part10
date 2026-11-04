# 100 Days of DevOps with KodeKloud - Days 91 - 100

THEE. FINAL. STRETCH. Lets finish up Ansible and Terraform so we'll be officially DONE with the 100 Days of DevOps challenge. 


## Day 100: Create and Configure Alarm Using CloudWatch Using Terraform



## Day 99: Attach IAM Policy for DynamoDB Access Using Terraform



## Day 98: Launch EC2 in Private VPC Subnet Using Terraform



## Day 97: Create IAM Policy Using Terraform



## Day 96: Create EC2 Instance Using Terraform



## Day 95: Create Security Group Using Terraform



## Day 94: Create VPC Using Terraform



## Day 93: Using Ansible Conditionals



## Day 92: Managing Jinja2 Templates Using Ansible

I don't know how to do this lab at all. I'm going to have AI walk me through it and review it another time. So, from what I can tell, the Jinja2 templates are sort of like Helm in a way or Terraform modules. It allows you to break down a large `playbook.yml` file into small parts. You call these smaller parts roles. You can have roles for Apache, NGINX, monitoring, etc. just like you would have with different Terraform files and modules instead of bundling it all into one main.tf file. 


```
---
- name: Configure HTTPD on App Server 3
  hosts: stapp03
  become: true

  roles:
    - role/httpd
```

This playbook says on host `stapp03`, run everything that I defined in my HTTPD role. The file path is underneath the role so the template will know how to find the role. Ansible will go into the file path and find the tasks associated with this role under `tasks/main.yml`. This is the standard task entry point. You can have multiple roles in your playbook file as well such as httpd, docker, monitoring, etc. As long as you have all of these tasks defined in the tasks/main.yml file. 

The template module basically just allows you to use variables. These values will be grabbed by the HOST that you run the playbook against. So for stapp03, if you make hostname, IP address, and like, idk some other type of variable, the template module will return stapp03 for the hostname, lets say 10.0.0.3 for the IP address if that's stapp03's IP, and such and such. The src and dest just says grab your information from this source and insert it into this destination. 

The source Jinja2 files can be found in the `/role/<specific_role>/templates/<file_name?` path. They'll typically have the file extension of `.j2`. The `.j2` extension on the file tells us we're using a Jinja2 template where Ansible can interpret variable information.

To create the directory tree for using Jinja2 templates, use the command `ansible-galaxy role init <role_name>`


## Day 91: Ansible Lineinfile Module

We needed to use the Lineinfile Module to add some additional text to the `index.html` file. We needed to add this text to the beginning of the file as well. Im thinking of the lineinfile module as more like sed and regex. You can look for lines in a file and replace them, add a line before or after the line, etc. This is some strong manipulation and I'll probably never do this by hand. I would have AI create the commands for me. I used the Ansible docs to figure out the linein moduel syntax. I originally had the commands to insert the new line before the initial line since that would be the only line in the file. AI said just use BOF instead to make sure the new line is ALWAYS at the Beginning Of (the) File. Small tune up but as long as it works, I'm fine with that. 

UPDATE: It seems like the validator didn't like that I used the blockin file. We switched this block to the copy block. That fixed it. 

```
---
- name: Testing the linelin module
  hosts: all
  become: true

  tasks:
  - name: Install HTTPD on app servers
    ansible.builtin.yum:
      name: httpd
      state: present

  - name: Start and enable HTTPD
    ansible.builtin.systemd_service:
      name: httpd
      state: started
      enabled: true

  - name: Create index.html with initial content
    ansible.builtin.copy:
      dest: /var/www/html/index.html
      content: |
        This is a Nautilus sample file, created using Ansible!
      owner: apache
      group: apache
      mode: '0655'

  - name: Add welcome message
    ansible.builtin.lineinfile:
      path: /var/www/html/index.html
      line: 'Welcome to xFusionCorp Industries!'
      insertbefore: BOF


```


