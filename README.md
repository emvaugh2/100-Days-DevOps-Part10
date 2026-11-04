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


