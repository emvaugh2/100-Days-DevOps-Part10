# 100 Days of DevOps with KodeKloud - Days 91 - 100

THEE. FINAL. STRETCH. Lets finish up Ansible and Terraform so we'll be officially DONE with the 100 Days of DevOps challenge. 


## Day 100: Create and Configure Alarm Using CloudWatch Using Terraform



## Day 99: Attach IAM Policy for DynamoDB Access Using Terraform



## Day 98: Launch EC2 in Private VPC Subnet Using Terraform

Okay I made a few careless mistakes here. I am a little tired but I didn't attach my subnet to my EC2 instance. I also didn't attach my security group either and I didn't attach my VPC to my security group. Apparently with the security group, my subnet was too restrictive. I used `10.0.01.0/24` for my ingress and egress rule instead of the overall VPC CIDR. Lastly, I got my ports and protocols backwards. That's not something I would care about though since I would google the right syntax. I did that from memory. 

UPDATE: Do not forget to create the variables.tf and outputs.tf files. I didn't do that and I failed the lab. 

```
# Creates the xfusion-priv-vpc VPC
resource "aws_vpc" "xfusion-priv-vpc" {
  cidr_block = var.KKE_VPC_CIDR

  tags = {
    Name = "xfusion-priv-vpc"
  }
}

# Creates a private subnet 
resource "aws_subnet" "xfusion-priv-subnet" {
  vpc_id                  = aws_vpc.xfusion-priv-vpc.id
  cidr_block              = var.KKE_SUBNET_CIDR
  map_public_ip_on_launch = false

  tags = {
    Name = "xfusion-priv-subnet"
  }
}

# Creates the EC2 
resource "aws_instance" "xfusion-priv-ec2" {

  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"

  vpc_security_group_ids = [
    aws_security_group.xfusion-default.id
  ]

  subnet_id = aws_subnet.xfusion-priv-subnet.id

  tags = {
    Name = "xfusion-priv-ec2"
  }
}

# Creates the security group for the Load Balancer
resource "aws_security_group" "xfusion-default" {
  name        = "xfusion-sg"
  description = "Security group for Nautilus App Servers"

  vpc_id      = aws_vpc.xfusion-priv-vpc.id


  ingress {
    cidr_blocks = ["10.0.0.0/16"]
    from_port   = 0
    protocol    = "-1"
    to_port     = 0
  }

  egress {
    cidr_blocks = ["10.0.0.0/16"]
    from_port   = 0
    protocol    = "-1"
    to_port     = 0
  }

  tags = {
    Name = "xfusion-sg"
  }
}
```

```
variable "KKE_VPC_CIDR" {
  type    = string
  default = "10.0.0.0/16"
}

variable "KKE_SUBNET_CIDR" {
  type    = string
  default = "10.0.1.0/24"
}
```

```
output "KKE_vpc_name" {
  value = "xfusion-priv-vpc"
}

output "KKE_subnet_name" {
  value = "xfusion-priv-subnet"
}

output "KKE_ec2_private" {
  value = "xfusion-priv-ec2"
}
```

## Day 97: Create IAM Policy Using Terraform

Okay Im so bad at AWS IAM roles. I'm even worse at creating them in Terraform. As an Azure engineer, I just simply do not want to deal with this. I challenged myself to create this though. I did use some previous code from a CloudWatch resource I created using AI. I swapped everything out and focused on creating read-only access to the EC2 console. So the users will be able to view all instances, AMIs, and snapshots. I googled IAM role examples which helped me change my `logs:*` to `ec2:`. It also helped me with the Resource line which is pretty open. I found an S3 policy and it had a bunch of List actions. I copied it and basically made up my own actions for the EC2. I then googled what actions can I actually perform and AWS showed me the describe actions. I found one literally for instances and snapshots. I didn't find one for AMI but I found an images one. I'm going to run this through AI and see what feedback they give me. 

UPDATE: Okay AI had me alter my resource line to simply put an asterisk. Oh wow that's all I had to change! Ya boy is COOKIN'. Alright, I still never want to do this again but that shows some level of improvement. Most likely I would create the rule manually in the AWS console, extract the JSON, and then feed this to AI to create the Terraform blocks for me in the future. 

```
resource "aws_iam_policy" "iampolicy_javed" {
  name        = "iampolicy_javed"
  path        = "/"
  description = "Allow read-only access to the EC2 console"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = [
          "ec2:DescribeInstances",
          "ec2:DescribeImages",
          "ec2:DescribeSnapshots"
        ]
        Effect   = "Allow"
        Resource = "*"
      },
    ]
  })

  tags = {
    Name = "iampolicy_javed"
  }

}
```

## Day 96: Create EC2 Instance Using Terraform

Same as the last lab. I had all of this outlined in a previous lab. I needed to add the name of the key they wanted me to generate and `terraform validate` pointed out I needed quotations around the name. I also couldn't find how to assign the default security group to this VM so I just erased that line completely. That did the job. 

```
# Creates the EC2 
resource "aws_instance" "jenkins_server" {

  ami           = "ami-0c101f26f147fa7fd"
  instance_type = "t2.micro"

  key_name = "devops-kp"

  tags = {
    Name = "devops-ec2"
  }
}
```

## Day 95: Create Security Group Using Terraform

Also another easy lab. Same as the last lab. Used some previous code an edited it. I removed the `vpc.id` line so I'm assuming the lab already pre-provisioned a VPC and the security group (SG) knew exactly which VPC to attach itself to. Here's the code below. I literally just swapped out the values and erased the egress rule. 

```
# Creates the security group for the Load Balancer
resource "aws_security_group" "xfusion-sg" {
  name        = "xfusion-sg"
  description = "Security group for Nautilus App Servers"

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 80
    protocol    = "tcp"
    to_port     = 80
  }

  ingress {
    cidr_blocks = ["0.0.0.0/0"]
    from_port   = 22
    protocol    = "tcp"
    to_port     = 22
  }

  tags = {
    Name = "xfusion-sg"
  }
}
```

## Day 94: Create VPC Using Terraform

Oh this was one was really easy. The thing that threw me off the most was Kode Kloud switched up the format after all this time. They launched me into VS Code and had me open up a new terminal. Outside of that, I just needed to create a VPC with a specific name. I literally copied my code from my previousl projects. I didn't see an actual place where you can name your VPC so I created a tag for it. The provider.tf file also specified the region so I took a chance and didn't put the region here. Terraform didn't like that I added a region in the code anyway. I ran the usual commands: terraform init, fmt -recursive, validate, plan, and apply. Then I hit submit and got the green check. 

```
# Creates the Kode Kloud VPC
resource "aws_vpc" "xfusion-vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "xfusion-vpc"
  }
}
```

## Day 93: Using Ansible Conditionals

This was also pretty new to me although way easier than the previous lab. We needed to make a conditional statement that said run this task if the hostname is so and so. AI generated me the solution because honestly I'm just a bit tired today but I glanced at it and tried to recreate it from memory. It basically had three pieces of structure: 3 different tasks (one for each server), the copy module, and the conditional statement at the end. Knowing that format, I create one task and copied this 3 times once I got the format down. For the copy module, Ansible has clear documentation on how to get the source file to the destination path. It also shows the `owner:` and `group:` lines as well. Very easy. As far as the conditional was concerned, Ansible also had documentation for this so I copied it and made it my own. The lab told me the name of the variable so I swapped that out and gave my task the the variable value. As previously stated, I copied this task 3 times and swapped out the server information. 

The playbook ran and I did some ad hoc commands to verify the files were on each server. That completed this lab. 

```
---
- name: Create files on app servers based on the hostname
  hosts: all
  become: true

  tasks:
  - name: Copy file from jumpbox to App Server 1
    ansible.builtin.copy:
      src: /usr/src/itadmin/blog.txt
      dest: /opt/itadmin/blog.txt
      owner: tony
      group: tony
      mode: '0744'
    when: ansible_nodename == "stapp01"

  - name: Copy file from jumpbox to App Server 2
    ansible.builtin.copy:
      src: /usr/src/itadmin/story.txt
      dest: /opt/itadmin/story.txt
      owner: steve
      group: steve
      mode: '0744'
    when: ansible_nodename == "stapp02"

  - name: Copy file from jumpbox to App Server 3
    ansible.builtin.copy:
      src: /usr/src/itadmin/media.txt
      dest: /opt/itadmin/media.txt
      owner: banner
      group: banner
      mode: '0744'
    when: ansible_nodename == "stapp03"
```

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

To create the directory tree for using Jinja2 templates, use the command `ansible-galaxy role init <role_name>`. 

I learned a lot here. Definitely will need to revisit this in the future. 


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


