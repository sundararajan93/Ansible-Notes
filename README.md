ANSIBLE
========

#### Ansible 
- Opensource
- Agentless
- Python / Yaml based
- Highly flexible configuration management systems
- Large number of in-built use modules for system managements
- custom modules can be added if needed
- Configuration roll-back in case of error
- simple and human readable


#### Ansible Components

- Ansible Configuration - Configuration ansible settings
- Host Inventory - The inventory files with Hostnames 
- Core Modules - Built in modules
- Custom Modules - Modules we can write 
- Playbooks - Playbook which have plays / tasks
- Connection plugins - Connectivity plugins other than ssh
- plugin - Enhances Ansible Functionality 


#### Two kinds of Nodes 

- Control Node - The Master node
- Managed Hosts - Client machines

**Control Node prerequisites**

- SSH connectivity
- Python above 2.6
- Should be Linux / Unix based OS

**Managed hosts Prerequisites**

- Client node 
- SSH connectivity between control node and managed nodes
- python 2.4 or above 

#### Setup

**Control Node**
server1.example.com

**Managed hosts**
client1.example.com
client2.example.com
client3.example.com


#### Creating Inventory file

```
/etc/ansible/hosts
```

The above is the inventory file

```
[Prod]
server1.example.com
server2.example.com

[Dev]
dev1.example.com
dev2.example.com
dev3.example.com

[Uat]
uat1.example.com
uat2.example.com

# If ssh runs in different port and to login using different username
# We can specify like the below in hostgroup
[Test]
client.example.com:2233 ansible_connection=ssh ansible_user=john

#We can specify the IP Range / Naeme like below 
[webservers]
192.168.2.[10:100]
hosts[01:99].example.com
```

The file looks something like this
grouped with [group_name]

**Note:**
```
# ansible --list-hosts myser
```

To view the hostnames in particular group


#### Adhoc Command

```
ansible -m command -a "uptime" Dev
```

The above command used to view uptime of all the servers in Dev group  

* ansible - keyword
* -m - Specifying that we gonna use a module
* command - Module Name
* "uptime" - OS command to check uptime of the node
* Dev - Target server group


#### Ping module to check the connectivity

```
# ansible -m ping Prod
```

#### Customizing Ansible

All the configuration of Ansible lies in **/etc/ansible/ansible.cfg**

For Example: Enable priviliage escalation by below settings

```
[previliage_escalation]
become=True
```

This will become root when you perform any ansible task

examples: become_method=sudo, become_user=root, become_ask_pass=True


$ ansible -m file -a "dest=/tmp/test mode=777 owner=testuser group=testgroup" dev --become --become-method=sudo --become-user=root -K


#### Ad-hoc Commands

File module
  
To change the ownership of a file
$ ansible -m file -a "dest=/tmp/filetotest mode=777 owner=newuser" dev --become -K

To delete the file

$ ansible -m file -a "dest=/tmp/filetotest state=absent" dev --become -K

Copy Module
 
$ ansible -m copy -a "src=/etc/hosts dest=/tmp/filetotest" dev


$ansible -m copy -a 'content="Sent from Ansible" dest=/tmp/hosts' dev


#### Playbooks

- Playbooks are files where Ansible code is written 
- They executes Plays which we write 
- Playbooks are YAML files
- YAML Syntax is very easy
- There can be more that one play inside a playbook

##### YAML SYNTAX

```
---
- hosts: dev
    become: true
    become_user: sundar
    tasks:
        - name: run the script
            script: /home/sundar/ansible/script.sh
            when: ansible_distribution == "RedHat" and ansible_distribution_major_version == '7'
---
```

#### Playbook to check and start the service - Service module

```
- hosts: dev
    become: true
    become_user: root
    tasks:
        - name: Check and start service - Cron
        service:
            name: cron.service
            state: started
```

* The above Playbook has to be saved as any filename with .yml extension
* -hosts - Tag is to list of hosts to be applied 
* become, become_user - For privilege escalation
* tasks - are the actual tasks to be performed
* -name - This is the title of the task
* service - The module name (which is builtin to integrate with service) 
* name - Name of the service to be checked
* state - To check the status of the service and apply if condition failed (started, stopped, restarted, reloaded)

#### To Copy a file using copy module

```
- hosts: dev
    become: true
    become_user: root
    tasks:
        - name: Copy the File /etc/hosts
        copy:
            src: /etc/hosts
            dest: /tmp/fromAnsible
            owner: sundar
            group: sundar
            mode: 0777
        when: ansible_distribution == "Ubuntu" and ansible_distribution_major_version == '18.04'
```

#### Modules

- Programs that is designed to perform specific operations 
- can be executed from ansible cli or playbooks
- when run, modules are copied to managed host and execute there
- comes with 500 modules built in
    - core modules - Built In (build by Ansible Team)
    - Extra modules - External modules (built by communities ex: openstack)
    - Custom modules -  Developed by end users

##### Library    
/usr/share/my_modules/


##### Module categories

* cloud
* files
* Notification
* packaging
* source control
* system
* utilities
* web infrastructure
* windows
* clustering
* Inventory
* messaging
* monitoring
* Network
* commands
* Database

```
$ ansible-doc <modulename>

Example:
$ ansible-doc apt
```
To Learn about the module from documentation with Examples 

```
$ansible -m apt -a "name=tree state=present" dev --become -K
```

(state = present/absent)

```
$ ansible -m apt -a "name=apache2 state=present" dev --become -K
  
#To start the service and enable
$ ansible -m service -a "name=apache2 state=started enabled=yes" dev --become -K

```

#### Tips to write playbook

* Do not use command, shell and raw modules in playbook since they are non-idempotent (They will rewrite the file)
* use copy module instead of shell: echo "nameserver " > /etc/resolv.conf
* you can write multiple task in a playbook

**example:**

```
---
- hosts: all
  become: true
  become_user: root
  tasks:
      - name: "Title of task one"
      copy:
          src: /etc/hosts
          dest: /tmp/test
          
       -name: "Start and enable the apache service"
       service:
           name: apache2
           state: started
           enabled: yes
```
The above playbook will be perform two tasks. 
1. copy a file using copy module
2. start and enable service apache2 using service module

This is handy when you perform more than one play

#### Dry run

It will display the actions happen when you run the playbook 

```
$ ansible-playbook playbook.yml -K -C
```

The above command will show all the changes which will be done if the playbook executed. The changes will not be done to the hosts in Dry run. Its just displaying purpose


#### Syntax check of playbook

To check the syntax of the playbook 
```
$ ansible-playbook --syntax-check <playbook_name>.yml 
```

#### Step by step execution

To execute the playbook step by step with yes/no prompt
```
$ ansible-playbook --step <playbook_name>.yml 
```

#### Line in file module

To add some text between lines we can use "lineinfile" module

```
- name: update Entry 
  lineinfile:
      path: /etc/httpd/conf/httpd.conf
      line: ServerName www.client1.example.com:80
      insertafter: "#ServerName"
```


#### Managing Variables

* Ansible supports variables that used to store values 
* users to create, packages to install, services to restart, files to remove
* Variable scopes
    * Global scope: Variables set from CLI of Ansible Config
    * Play Scope: Variables set in play and related structures
    * Host Scope: Variables set on host groups and individual hosts by the inventory, fact gathering, registered task
* Administrators can use their own variables in playbook

##### Play Scope

```
---
- name: Install Apache and start service
  hosts: all
  vars:
      web_pkg: apache2
      web_service: apache2
   tasks:
       - name: Install the requirement
        apt:
            name:
                - "{{ web_pkg }}"
                - "{{ webservice }}"
                state: latest
```

##### Host Variables and group variables

* Variables specified in inventory file which apply to hosts/ group
* Example:
/etc/ansible/hosts
```
[servers]
demo.example.com ansible_user=sundar
```

ansible_user is the variable defined for only the server

```
[server1]
demo1.com
demo2.com
[server2]
demo3.com
demo3.com

# grouping two groups  to single group name 'servers'
[servers:children]
server1
server2

# Variable user with value 'sundar' applied for group servers
[servers:vars]
user=sundar
```

##### Variable precedence (priority)

1. Playbook variable
2. Host variable
3. Group Variable

#### Ansible Facts

* Ansible facts are variables that are automatically discovered by ansible from a managed host
* 'setup' module can be useful to pull those values of variables

Example usage: 

- server can be restarted depending on the current kernel version
- users can be created depending on the host name

Ansible facts are convenient way to retrieve the state of a managed node and decide which action to be taken based on its state

- The host name
- The kernel version
- The network interfaces
- The IP address
- The version of the Operating system
- Various environment variables
- The number of CPUs
- The available or Free memory

**__To get facts with ansible__**
```
$ ansible <groupname/hostname> -m setup

$ansible dev -m setup
```

**__To get only certain value you can use filter like below__**


```
$ ansible dev -m setup -a "filter=ansible_hostname"
```

Using Facts in Playbook
```
---
- hosts : all
  become: true
  become_user: root
  tasks:
      - name: Gather Ansible Facts about node
         debug:
             msg:
                 - Distro {{ ansible_distribution }}
                 - Release {{ ansible_distribution_release }}
                 - Package Manager {{ ansible_pkg_mgr }}
                 - Free memory {{ ansible_memfree_mb }} Mb
```


#### Conditionals in Ansible

- Ansible can use conditionals to execute task when certain condition met
- we can use 'when' statement
- sometimes we want to skip a particular step based on os version
- cleanup file system is getting full

example:

To shutdown Debian based system  

```
tasks:
    - name: "Shut down Debian based system"
      command: /sbin/shutdown -t now
      when: ansible_os_family == "Debian"      
```

#### Ansible Loop (with_items)

* We can perform loop operation ansible using with_items keyword
* For example if you want to install multiple packages (postfix, dovecot, apache2) in single playbook you need to write multiple tasks. But this can easily be done like below
  

```
apt:
   name: "{{ item }}"
   state: latest
 with_items:
     - postfix
     - dovecot
     - apache2   
```

Another approach with variables

```
vars:
    mail_services:
        - postfix
        - dovecot

tasks:
    apt:
        name: " {{ item }} "
        state: latest
     with_items: " {{ mail_services }}"
```


#### Ansible Handler

- Handler is exactly same as a task, but it will run when called by another task
- Action when called by an event 
- They are pretty much like a function

```
tasks:
- name: copy demo.example.conf conf template
  copy:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
   notify:
       - restart_mysql
       - restart_apache
#Handlers
 handlers:
    - name: restart_mysql
       service:
           name: mariadb
           state: restarted
     - name: restart_apache
        service:
            name: apache2
            state: restarted
```

#### Ansible Roles

1. Roles consists of many playbooks, similar to modules in puppet and cook book in chef
2. Roles are a way to group multiple tasks together in one container to automate effectively
3. Roles are set of tasks and additional files for a certain role which will allow  you to breakup configuration
4. Easy to reuse code if role is suitable to someone
5. Easy to modify and reduce syntax errors

To create Ansible role:

```
$ ansible-galaxy init /etc/ansible/roles/apache --offline
```
This command would create a module like architecture for the ansible role apache in /etc/ansible/roles/apache

```
master@master:/etc/ansible/roles/apache$ ls -ltr
total 36
drwxr-xr-x 2 root root 4096 Feb  6 09:14 templates
-rw-r--r-- 1 root root 1328 Feb  6 09:14 README.md
drwxr-xr-x 2 root root 4096 Feb  6 09:14 files
drwxr-xr-x 2 root root 4096 Feb  6 09:14 defaults
drwxr-xr-x 2 root root 4096 Feb  6 09:14 tasks
drwxr-xr-x 2 root root 4096 Feb  6 09:14 meta
drwxr-xr-x 2 root root 4096 Feb  6 09:14 handlers
drwxr-xr-x 2 root root 4096 Feb  6 09:14 tests
drwxr-xr-x 2 root root 4096 Feb  6 09:14 vars
```


Please refer to the code at /etc/ansible/roles/apache for detailed overview in ansible master


#### Patching

- Verify the app/database process are running or not
- Decision point to start patchin
- copy the required Repo file to the managed hosts
- upgrade the kernel or packages
- check if reboot is required
- Reboot system
- wait for few mins so the server come up after patchin
- Debug a message with new kernel version


-------------------------X------------------------------------

Password less ssh authentication

1. copy contents from .ssh/id_rsa.pub from jumpserver to client .ssh/authorized_keys
2. After this the ssh connection from jumpserver to client machine would be passwordless

User administration

1. User creation
2. User Removal
3. Group add
4. Password reset
5. Add entry in sudoers file

Health Check statistics

1. Memory Utilization
2. cpu utilization
3. overall I/O Activities
4. Reports run queue and load average on the server

