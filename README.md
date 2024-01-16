<div align = "center">

Ansible-CMDB

</div>
 
# References
- [ANSIBLE-CMDB Docs](https://ansible-cmdb.readthedocs.io/en/latest/usage/) 
- [ANSIBLE-CMDB README](https://github.com/fboender/ansible-cmdb/tree/1.31)
- [ANSIBLE-CMDB Use Case](https://www.reddit.com/r/ansible/comments/7op6wq/what_configuration_management_tool/)
- [Push & Pull in Config Man](https://gayatrisajith.medium.com/beginner-fundamentals-push-pull-configuration-management-tools-85eff1b41447)

# Setting up servers
- We create 2 VMs // not per my req
- download package using wget & then install ansible server'spackage
- epel = extra paxkages for enterprise linux 
- To create shahee paneer/a dish we bought the ingredients by installing the main ingredient/paneer/ansible
- we need to install other dependencies/components to create a dish - these dishes are part of the package
- How will ansible server know what nodes are connected with it - what nodes it has to send updates to
- In ansible server, find ivnentory file/host default file, enter pvt ip of nodes for saving it to let server know who it is connected with "vi /etc/ansible/hosts"
- Inside the file, create a "demoByYash group" and paste node1 & 2 pvt ip
- another file in ansible: config file where commands are written but //ed, remove // before inventory & inventory file activated! also remove // before sudo-user = root (this allows future's ansible user to have the right privelege

- connection still not established b/w nodes & ansible server - we create 3 ansible users in the nodes as root user won't be known for each node practically
- "adduser user/ansible" in 3 servers, setting passwords using "passwd user/ansible"
- switching users in servers with switch user with su - ansible
- giving root privelege to ansible by editing visudo file, add:
ansible ALL=(ALL) NOPASSWD:ALL
- STILL have to write commands w/ sudo since you are not root user - ansible user is only a sudo user not root user
- NO communication b/w servers through ssh, come bacck to ansible server and we need to install ssh 
- make changes to /etc/ssh/sshd_config & node 1 and node 2 should be allowed to connect
- create the connection in the server by creating 
- sudo apt-get install ansible
got erro "sudo ansible --version
ERROR: Ansible requires the locale encoding to be UTF-8; Detected ISO8859-1." after installing
solved by https://www.reddit.com/r/ansible/comments/z66t0l/comment/jwenrji/?utm_source=share&utm_medium=web2x&context=3 adding following to .bashrc:
```
export LC_ALL=en_US.UTF-8
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
```

- work as an ansible user and create necessary 
- ansible --version shows "config file = /etc/ansible/ansible.cfg" 
- the hosts file will not work until we make chanes to ansible.cfg

- nano /etc/ansible/hosts
created a group including my vm called "demoByYash" using:
[demoByYash]
192.168.122.128

- vi /etc/ansible/ansible.cfg and uncommented "$ ansible-config init --disabled > ansible.cfg"

- there are only 2 files in /etc/ansible. my ansible.cfg says:
```
# Since Ansible 2.12 (core):
# To generate an example config file (a "disabled" one with all default settings, commented out):
#               $ ansible-config init --disabled > ansible.cfg
#
# Also you can now have a more complete file by including existing plugins:
# ansible-config init --disabled -t all > ansible.cfg

# For previous versions of Ansible you can check for examples in the 'stable' branches of each version
# Note that this file was always incomplete  and lagging changes to configuration settings

# for example, for 2.9: https://github.com/ansible/ansible/blob/stable-2.9/examples/ansible.cfg
```
so i ran ansible-config init --disabled -t all > ansible.cfg

- adduser ansible in both machines

- su - ansible for switching to the new user

- i got an error when trying to test install httpd "sudo apt install httpd
[sudo] password for ansible: 
ansible is not in the sudoers file.
This incident has been reported to the administrator."

- to create it same as root user, i logged out of ansible user on BOTH SERVERS and ran "visudo", under the "allow root to run any commands anywhere", I addded the following:
```
ansible ALL=(ALL)   NOPASSWD: ALL
```

- Trust relationship: root w/ root & sser

- TO CHECK if connection has been established b/w node & server, come to server and run su - ansible and establish ssh between the 2 USERS

- when i ran ssh 192.168.122.128, i was able to connect with the node from my server

optional i think:
- changes made to /etc/ssh/sshd_config in both servers:
1. added this line: PermitRootLogin yes
2. uncommented PasswordAuthentication yes
3. outside of the file ran service sshd restart

- to verify the connection, turn ansible user first, establish ssh connection

- if i had another node, i'd have to connect to 2nd node from main server again - since i have to enter password again and again while trying to establish connection with ssh, I have to remove password authentication - not practical since we have to update 100s of server, automation with password is not feasible

- solution, we gen a key inside ansible server using ssh-keygen, one pub and one pvt, it is imp to be signed in as same level user in all servers - when creating trust relation, you need gen ssh-keygen seen using ls -a

- inside .ssh i'd see id_rsa and id_rsa_pub and when you connect using ansible@ip again, you'll be asked to enter passwd first only - 

- inside the server, add the public key on the node so that node always knows that the server is authenticate - everything to be done as an ansible user 

- first in server, run "ssh-keygen" - do ls -a and you'll find .ssh, inside is id_rsa and id_rsa.pub, copy the public key

- to copy public key to node, we run "ssh-copy-id ansible@192.168.122.12 - for the last time passwd is asked and once done, connection will be set w/o passwd

- in the nodes server, ~/.ssh/authorized_keys get created and the
id_rsa.pub is added form the server - automatically after running ssh-copy-id on the server

# Ansible-CMDB Intro

> "This tool takes the output of Ansible’s fact-gathering as its input and converts it into a static HTML overview page with all gathered system configuration information ... supports multiple types of output (HTML, csv, sql, etc) and extending information gathered by Ansible with custom data." 

- CMDB = Config Management DB
- It takes output of ansible's fact gathering
- converts it into static html overview, containing config info
- It supports html, csv, txt, sql, etc. outputs
- Extension of Ansible

# Ansible-CMDB Installation

Step 1: Download Ansible-CMDB: 
- su - root 
- cd /opt
- wget https://github.com/fboender/ansible-cmdb/releases/download/1.30/ansible-cmdb-1.30.zip -O ansible-cmdb-1.30.zip

Step 2: Install Ansible-CMDB:
- unzip ansible-cmdb-1.30.zip
- cd ansible-cmdb-1.30
- yum install python-pip -y

- pip install ansible-cmdb --break-system-packages -->install

**NOTE**: I referred to the GitHub ReadMe from [here](https://github.com/fboender/ansible-cmdb/tree/1.31)

# Ansible-CMDB Demonstration 
## Viewing all configurations as HTML
- `mkdir ansible` --> `cd ansible` --> `mkdir out`
- `ansible -m setup --tree ~/ansible/out/ all` for sending the nodes conf into out direction
- It is also possible to run as a specific user: `
ansible -m setup --user=root --tree output/ all --ask-pass`
- `ansible-cmdb ~/ansible/out >  html.html` for converting the entire content of out directory into an html file
- Potential error to be faced:
  - `error: No suitable python version found (v2.7 or higher required). Aborting`
- Ran `sudo ln -s /usr/bin/python3 /usr/bin/python` (https://unix.stackexchange.com/questions/673847/how-to-set-python-3-as-default-interpreter-in-linux) for setting system link (sym)
- Reran `ansible-cmdb ~/ansible/out >  html.html` - which generated the html file available for viewing configurations.
- html overview was generated in out directory, which was then moved to my host user using `sudo scp -r html.html /home/user/imsnov` 
- I then switched to the host user and ran `google-chrome html.html` inside the imsnov directory, which gave the following output:
![img](https://i.imgur.com/6f4b4V8.png)
- By default, the `html_fancy` format is used for generating fancy html overview of configurations from out directory.
- When we don't specify the format, ansible-cmdb generates overview automatically as `html_fancy` using `ansible-cmdb --template html_fancy out/ > abcd.html` - `--template html_fancy` secretly added auto if not mentioned.
- 

## Using Templates For Other Formats
From the [Ansible-CMDB Documentation](https://ansible-cmdb.readthedocs.io/en/latest/usage/) command:
```
html_fancy: A dynamic, modern HTML page containing all hosts. **USED AS DEFAULT!!**
html_fancy_split: A dynamic, modern HTML page with each host's details in a separate file.
txt_table: A quick text table summary of the available hosts with some minimal information.
json: Dumps all hosts including groups, variable, custom info in JSON format.
csv: The CSV template outputs a CSV file of your hosts.
markdown: The Markdown template generates host information in the Markdown format.
sql: The SQL template generates an .sql file that can be loaded into an SQLite or MySQL database.
```
. html_fancy_split
```s
> ansible-cmdb -t html_fancy_split out/

> cd cmdb

> 192.168.122.159.html  192.168.122.6.html  index.html
```
**index.html**
![img](https://i.imgur.com/6f4b4V8.png)

**192.168.122.6.html**
![img](https://i.imgur.com/6f4b4V8.png)

**192.168.122.159.html**
![img](https://i.imgur.com/6f4b4V8.png)

. txt_table

```
ansible-cmdb --template txt_table out/

Name             OS            IP               Mac                Arch           Mem  MemFree  MemUsed  CPUs  Virt       Disk avail          
---------------  ------------  ---------------  -----------------  -------------  ---  -------  -------  ----  ---------  ------------------  
192.168.122.6    Ubuntu 23.04  192.168.122.6    52:54:00:80:60:d6  x86_64/x86_64  3g   1g       2g       2     kvm/guest  5.5g, 5.5g          
192.168.122.159  Ubuntu 22.04  192.168.122.159  52:54:00:a4:fe:5d  x86_64/x86_64  4g   2g       2g       2     kvm/guest  15.0g, 15.0g, 0.5g 
```

. markdown

```
ansible-cmdb --template markdown out/ > markdown.md

# <a name="host_overview"></a> Host overview

* <a href="#192.168.122.159">192.168.122.159</a>
* <a href="#192.168.122.6">192.168.122.6</a>

# Host details

## <a name="192.168.122.159"></a> 192.168.122.159

<a href="#host_overview">Back to top</a>

### General

* **Node name**: inventory
* **Form factor**: Other
* **Virtualization role**: guest
* **Virtualization type**: kvm
. . .
```

. ansible-cmdb --template markdown_split out/
```==
ansible-cmdb --template markdown_split out/

cd cmdb

192.168.122.159.md 192.168.122.6.md overview.md
```

. txt_table

```
ansible-cmdb --template txt_table out/

Name             OS            IP               Mac                Arch           Mem  MemFree  MemUsed  CPUs  Virt       Disk avail          
---------------  ------------  ---------------  -----------------  -------------  ---  -------  -------  ----  ---------  ------------------  
192.168.122.6    Ubuntu 23.04  192.168.122.6    52:54:00:80:60:d6  x86_64/x86_64  3g   1g       2g       2     kvm/guest  5.5g, 5.5g          
192.168.122.159  Ubuntu 22.04  192.168.122.159  52:54:00:a4:fe:5d<div align = "center">
```

. csv
```
ansible-cmdb -t csv out/ > csv.csv

"Name","OS","IP","Arch","Mem","MemFree","MemUsed","CPUs","Virt","Disk avail"
"192.168.122.6","Ubuntu 23.04","192.168.122.6","x86_64/x86_64","3g","1g","2g","2","kvm/guest","5.5g, 5.5g"
"192.168.122.159","Ubuntu 22.04","192.168.122.159","x86_64/x86_64","4g","2g","2g","2","kvm/guest","15.0g, 15.0g, 0.5g"
```

. json
```
ansible-cmdb -t json out/ > json.json

{
  "192.168.122.6": {
    "name": "192.168.122.6",
    "hostvars": {},
    "ansible_facts": {
      "ansible_all_ipv4_addresses": [
        "192.168.122.6"
      ],
      "ansible_all_ipv6_addresses": [
        "fe80::d5eb:7044:7a87:d59b"
      ],
      "ansible_apparmor": {
        "status": "enabled"
      },
...
```

## Limiting Hosts
- You can limit the number servers you want data from.
- If want data from all hosts in a group:
ansible-cmdb -i hosts -l 'mygroup' ~/ansible/ > cmdb.html
- If want data from specific hosts of a group
ansible-cmdb -i hosts -l 'all:!mygroup:192.x.x.x' ~/ansible/ > cmdb.html

## Adding Columns To Be Displayed FOR SPECIFIC TEMPLATE
- Do not leave any space
```
ansible-cmdb -t txt_table --columns name,os,ip,mem,cpus ~/ansible/
Name             OS            IP               Mem  CPUs  
---------------  ------------  ---------------  ---  ----  
192.168.122.6    Ubuntu 23.04  192.168.122.6    3g   2     
192.168.122.159  Ubuntu 22.04  192.168.122.159  4g   2     
ansible@yashanand:~$ ansible-cmdb -t txt_table --columns name,os,ip,mem ~/ansible/
Name             OS            IP               Mem  
---------------  ------------  ---------------  ---  
192.168.122.6    Ubuntu 23.04  192.168.122.6    3g   
192.168.122.159  Ubuntu 22.04  192.168.122.159  4g   
```

## Extending Facts
> You can specify multiple directories that need to be scanned for facts. This lets you override, extend and fill in missing information on hosts. You can also use this to create completely new hosts or to add custom facts to your hosts.

## Override and fill in facts
> Sometimes Ansible doesn't properly gather certain facts for hosts. You can add it manually to a host.

1. Create directory for storing extended facts - have 2 dir: out & out-extended
2. Create a file for a host/server - . The file must be named the same as it appears in your hosts file. Its content:
```
{
  "ansible_facts": {
      "ansible_userspace_architecture": "x86_64"
  }
}
```

3. Specify both directories when generating the output:
```
./ansible-cmdb out/ out_extend/ > overview.html
```

## Manual hosts
- Similar to overriding facts, we can manually add all details of a server's inventory by running creating output file liek 192.x.x.x like:
```
1. Create a directory for you custom facts:

$ mkdir out_manual

2. Create a file in it for your windows host:

$ vi out_manual/win.dev.local
{
  "groups": [
  ],
 "ansible_facts": {
    "ansible_all_ipv4_addresses": [
      "10.10.0.2",
      "191.37.104.122"
    ], 
    "ansible_default_ipv4": {
      "address": "191.37.104.122"
    }, 
    "ansible_devices": {
    }, 
    "ansible_distribution": "Windows", 
    "ansible_distribution_major_version": "2008"
      }, 
  "changed": false
}

3. Now you can create the overview including the windows host by specifying two fact directories:

./ansible-cmdb out/ out_manual/ > overview.html
```

## Custom Facts 
custom facts added will be displayed under 'custom facts' header in html fancy. Not same as host local facts which are read by ansible from /etc/ansible/facts.d directory. custom facts explained here are manually defined on the host where you run ansible-cmdb, and have little to do with Ansible itself.

UseCase: 
Let's say you want to add information about installed software to your facts.
1. Create a directory for you custom facts:
```
$ mkdir out_custom
```
2. reate a file in it for the host where you want to add the custom facts:
```
$ vi custfact.test.local
{
  "custom_facts": {
    "software": {
      "apache": {
        "version": "2.4",
        "install_src": "backport_deb"
      },
      "mysql-server": {
        "version": "5.5",
        "install_src": "manual_compile"
      },
      "redis": {
        "version": "3.0.7",
        "install_src": "manual_compile"
      }
    }
  }
}
```
For this to work the facts must be listed under the custom_facts key.

Generate the overview:

3. The software items will be listed under the "Custom facts" heading.
```
./ansible-cmdb out/ out_custom/ > overview.html
```

-------------------------------

# Bash Scripting For Ansible-CMDB

## Reference
[YouTube Tutorial](https://www.youtube.com/watch?v=PPQ8m8xQAs8&ab_channel=NullByte)

## Introduction

- which bash
/usr/bin/bash

- nano bash.sh
```
#! /bi/bash #Tells what kind of file this is
STRING="Null Byte"
echo "I just wrote a string = $STRING" # $ are like ampersand
```
If I run `bash bash.sh`, then I would get `My name is Yash Anand` as output.

- nano variables.sh
```
#! /bin/bash # telling terminal to use bash to execute this file when running ./variables.sh
echo "My name is $1 and surname is $2"
```
if `bash variables.sh "Yash" "Anand"` is run then the $1 and $2 will be 
replaced by the arguments passed 

- nano interacting.sh
```
#! /bin/bash
echo $(whoami)
```
For printing the output of the command in after $ directly, run `bash interacting.sh` which gave the output `user`.

- nano ifelse.sh

```
#! /bin/bash

echo "Enter something"
read something
if [ $something ]; then
        echo "$something sure is something"
else
        echo "Why didn't you enter anything?"
fi
```
if last line is not error then you get "syntax error: unexpected end of file" since loops dont end automatically, they have to be ended manually.

- Aliasing & ifconfig | grep "broadcast" | awk '{print $2}'

User may not want to run long commands over and over again so aliasing can be used. The mentioned command can be defined to a specific command:
```
alias whatismyipaddress="echo $(ifconfig | grep "broadcast" | awk '{print $2}')"
```

When running `whatismyipaddress`, the second argument will only be displayed without running the entire command

## Ansible-CMDB CSV Script
```
#! /bin/bash

group="mygroup"

echo "This script assumes you already have Ansible & Ansible-CMDB set-up."
echo "What is the name of your group?"
read group
echo "Enter path for saving ansible facts"
read factpath
echo "Enter name to save csv output as"
read csvname

ansible -m setup --tree $factpath $group

ansible-cmdb $factpath > $csvname
```

## How Ansible-CMDB Adds Columns In Templates

It is possible to display more columns in the output of templates using the '--column' flag and mentioning them. **Use-case**: I want to only display 3 specific columns in the output of txt_table template:
```
ansible-cmdb -t txt_table --column os,ip,mem ansible/ 
```

The output of the above command would be as follows:
```
OS            IP               Mem  
------------  ---------------  ---  
Ubuntu 23.04  192.168.122.6    3g   
Ubuntu 22.04  192.168.122.159  4g 
```

| **NOTE: Adding a column that doesn't exist in the `/usr/local/lib/ansiblecmdb/data/tpl/txt_table.tpl` file (like kernel) OR A DIFFERENT COLUMN ID THAN THE ONE MENTIONED IN TPL FILE, then a blank will be shown** |
|---------------|

To create a CSV file according to the format chosen for IMS Inventories, `/usr/local/lib/ansiblecmdb/data/tpl/csv.tpl` will be modified. This way, when `ansible-cmdb -t csv ansible/ > csv` is run, the required columns will be retrieved. The default `csv.tpl` is as follows:
```
cols = [
  {"title": "Name",       "id": "name",       "visible": True, "field": lambda h: h.get('name', '')},
  {"title": "OS",         "id": "os",         "visible": True, "field": lambda h: h['ansible_facts'].get('ansible_distribution', '') + ' ' + h['ansible_facts'].get('ansible_distribution_version', '')},
  {"title": "IP",         "id": "ip",         "visible": True, "field": lambda h: host['ansible_facts'].get('ansible_default_ipv4', {}).get('address', '')},
  {"title": "Arch",       "id": "arch",       "visible": True, "field": lambda h: host['ansible_facts'].get('ansible_architecture', 'Unk') + '/' + host['ansible_facts'].get('ansible_userspace_architecture', 'Unk')},
  {"title": "Mem",        "id": "mem",        "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_memtotal_mb', 0)) / 1000.0)},
  {"title": "MemFree",    "id": "memfree",    "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_memfree_mb', 0)) / 1000.0)},
  {"title": "MemUsed",    "id": "memused",    "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_memory_mb', {}).get('real', {}).get('used',0)) / 1000.0)},
  {"title": "CPUs",       "id": "cpus",       "visible": True, "field": lambda h: str(host['ansible_facts'].get('ansible_processor_count', 0))},
  {"title": "Virt",       "id": "virt",       "visible": True, "field": lambda h: host['ansible_facts'].get('ansible_virtualization_type', 'Unk') + '/' + host['ansible_facts'].get('ansible_virtualization_role', 'Unk')},
  {"title": "Disk avail", "id": "disk_avail", "visible": True, "field": lambda h: ', '.join(['{0:0.1f}g'.format(i['size_available']/1048576000) for i in host['ansible_facts'].get('ansible_mounts', []) if 'size_available' in i and i['size_available'] > 1])},
]
```
In the above default template that is used for displaying tables:
- "title": "Name": Title of the column in output after displayed
- "id": "name": ID is identifier of column, title can be different
- "visible": True: If turned false then we will not see column
- "field": lambda h: h.get('name', ''): 
- - Field is the value that is being used for adding values by making  parameter h (for ANSIBLE output file!) get value related to id 'name' from host (ANSIBLE output file saved in out/). 
- - If 'name' not present, empty string is shown as value.

## Adding Columns To Ansible-CMDB
**To add more columns from the Ansible-Output file, I will need to add it in the ID field and value in h.get**

The following columns that are useful to us are already retrieved by Ansible from the Host file:
groups: A list of Ansible groups the host belongs to.
dtap: Whether a host is a development, test, acceptance or production system.
comment: A comment for the host.
ext_id: An external unique identifier for the host.

For my own undestanding, I first added the following columns as per the original Inventory to the `txt_table.tpl`template:
```
  {"title": "Primary",      "id": "primary",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "SL",           "id": "sl",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "Client",       "id": "client",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "Project",       "id": "project",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "Application Environment",       "id": "APPLICATION_ENVIRONMENT",       "visible": True, "field": lambda h: h>
  {"title": "Setup Environment",       "id": "SETUP_ENVIRONMENT",       "visible": True, "field": lambda h: h.get(' ', ''>
  {"title": "Location",     "id": "location",       "visible": True, "field": lambda h: h.get(' ', '')},
```

Now the output of running `ansible-cmdb -t txt_table ansible/` is as follows:
```
|Primary | SL |  Client | Project | Application Environment | Setup Environment | Location | Name            |  OS          |  IP             |  Mac              |  Arch         |  Mem | MemFree | MemUsed | CPUs | Virt      |  Disk avail |          
|-------|  -- | ------ |  ------- | ----------------------- | ----------------- | -------- | --------------- | ------------ | --------------- | ----------------- | ------------- | --- | ------- | ------- | ----  | --------- |  ------------------ |  
|       |     |        |           |                        |                   |          |192.168.122.6    |Ubuntu 23.04 | 192.168.122.6    |52:54:00:80:60:d6  |x86_64/x86_64  |3g   |1g       |2g       |2     |kvm/guest  |5.6g, 5.6g|          
```

With the primary columns present in the output, I could simply add the new columns from this template to the `csv.tpl` or create a new custom template and referring its absolute path. To add specific data columns from the Ansible-Output file, I first listed the columns that were common with our Inventory:
```
HOSTNAME
IP
CPU
RAM
MemUsed_GB
FREE_RAM_GB
SwapFree_GB
SwapTotal_GB
WORKER_NODE_INTERNAL_IP (Private)
LOG_PATH
MAC_ADD
NETMASK
OS
OS_VERSION
KERNEL_VERSION
GATEWAY
WWN_STORAGE
WWN_NAME
WWIN
LOCAL_DISK_GB
DISK_AVAILABLE_GB
LOCAL_DISK_PARTITION
```
