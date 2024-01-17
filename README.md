------------------------------------------------
<div align = "center">

# Ansible-CMDB

</div>
 
# References
- [ANSIBLE-CMDB Docs](https://ansible-cmdb.readthedocs.io/en/latest/usage/) 
- [ANSIBLE-CMDB README](https://github.com/fboender/ansible-cmdb/tree/1.31)
- [ANSIBLE-CMDB Use Case](https://www.reddit.com/r/ansible/comments/7op6wq/what_configuration_management_tool/)
- [Push & Pull in Config Man](https://gayatrisajith.medium.com/beginner-fundamentals-push-pull-configuration-management-tools-85eff1b41447)

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

With the primary columns present in the output, I could simply add the new columns from this template to the `csv.tpl` or create a new custom template and referring its absolute path. To add specific data columns from the Ansible-Output file, I first listed the columns that were common with our Inventory and added them into this [Common-Columns Spreadsheet](https://docs.google.com/spreadsheets/d/1c8-KJ2-Nz4_dlwONRyq-JtToTQrJiy2gwPiGwSYIkrc/edit#gid=0). 

The newly created 'ims.tpl' Ansible-CMDB template file that is stored in /usr/local/lib/ansiblecmdb/data/tpl directory now has some of the columns provided below included. Request the complete access of the template if needed.
```
 {"title": "PRIMARY",    "id": "manual",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "SL",         "id": "manual",        "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "CLIENT",     "id": "manual",    "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "PROJECT",    "id": "manual",   "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "APPLICATION_ENVIRONMENT",       "manual": "APPLICATION_ENVIRONMENT",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "SETUP_ENVIRONMENT",       "id": "manual",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "LOCATION",   "id": "manual",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "HOSTNAME",       "id": "name",       "visible": True, "field": lambda h: h.get('name', '')},
  {"title": "IP",         "id": "ip",         "visible": True, "field": lambda h: host['ansible_facts'].get('ansible_default_ipv4', {}).get('address', '')},
  {"title": "CPU",       "id": "cpus",       "visible": True, "field": lambda h: str(host['ansible_facts'].get('ansible_processor_count', 0))},
  {"title": "RAM",        "id": "mem",        "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_memtotal_mb', 0)) / 1000.0)},
  {"title": "MemUsed",    "id": "memused",    "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_memory_mb', {}).get('real', {}).get('used',0)) / 1000.0)},
  {"title": "FREERAM",    "id": "memfree",    "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_memfree_mb', 0)) / 1000.0)},
  {"title": "SwapFree",  "id": "swapfree",  "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_swapfree_mb', 0)) / 1000.0)},
  {"title": "SwapTotal",  "id": "swaptotal",  "visible": True, "field": lambda h: '%0.0fg' % (int(host['ansible_facts'].get('ansible_swaptotal_mb', 0)) / 1000.0)},
  {"title": "TYPE",       "id": "virt",       "visible": True, "field": lambda h: host['ansible_facts'].get('ansible_virtualization_type', 'Unk') + '/' + host['ansible_facts'].get('ansible_virtualization_role', 'Unk')},
  {"title": "WORKER_NODE_INTERNAL_IP",    "id": "manual",       "visible": True, "field": lambda h: h.get(' ', '')},
  {"title": "OWNER_CONCTACT_DETAIL",    "id": "manual",       "visible": True, "field": lambda h: h.get(' ', '')},
...
```

## Adding Missing Columns To Ansible-CMDB
For adding the data of columns that are missing from the original Ansible-Data, I followed the [Custom-Facts](https://ansible-cmdb.readthedocs.io/en/stable/usage/#custom-facts) section of the Ansible Document and followed these steps:
- Created a new directory out_extend in addition to original ansible/ 
- Stored custom-fact file with the following content:
```
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
- ansible-cmdb ansible/ out_extend/ > overview.html which showed all of the data mentioned under the 'Custom Facts' header.
![img](https://i.imgur.com/qLRGQVM.png)

Now I need to understand how to add remaining columns using Playbooks 
