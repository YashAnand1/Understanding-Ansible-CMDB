<div align = "center">

# Ansible-CMDB

</div>
 
#References
- [ANSIBLE-CMDB Docs](https://ansible-cmdb.readthedocs.io/en/latest/usage/) 
- [ANSIBLE-CMDB README](https://github.com/fboender/ansible-cmdb/tree/1.31)
- [ANSIBLE-CMDB Use Case](https://www.reddit.com/r/ansible/comments/7op6wq/what_configuration_management_tool/)
- [Push & Pull in Config Man](https://gayatrisajith.medium.com/beginner-fundamentals-push-pull-configuration-management-tools-85eff1b41447)

# Ansible-CMDB Intro
- CMDB = Config Management DB
- It takes output of ansible's fact gathering
- converts it into static html overview, containing config info
- It supports html, csv, txt, sql, etc. outputs
- Extension of Ansible

installed ansible cmdb:

Step 1: Download Ansible-CMDB: 
# su - root 
Make sure your ansible system having the internet connectivity
# cd /opt
# wget https://github.com/fboender/ansible-cmdb/releases/download/1.30/ansible-cmdb-1.30.zip -O ansible-cmdb-1.30.zip

Step 2: Install Ansible-CMDB:
# unzip ansible-cmdb-1.30.zip
# cd ansible-cmdb-1.30
# yum install python-pip -y

# pip install ansible-cmdb --break-system-packages -->install

referred to the GitHub ReadMe from [here](https://github.com/fboender/ansible-cmdb/tree/1.31)
entered as ansible user and created out directory

ansible -m setup --tree out/ all 
for sending the nodes conf into out dir

- ansible-cmdb out/ >  all_hosts.html
error: No suitable python version found (v2.7 or higher required). Aborting
Ran "sudo ln -s /usr/bin/python3 /usr/bin/python" (https://unix.stackexchange.com/questions/673847/how-to-set-python-3-as-default-interpreter-in-linux) for solving error

- ansible-cmdb out/ >  all_hosts.html now generates the html file available for viewing
