# 安装

## 1.Yum
epel 
yum install ansible


## 2.Apt(Ubuntu)
apt-get install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible


# 主要命令
1. ansible
Usage: ansible <host-pattern> [options]
默认使用command模块

2. ansible-playbook 读取playbook,执行相应任务
Usage: ansible-playbook playbook.yml

3. ansible-doc
Usage: ansible-doc [options] [module...]
ansible-doc -l
ansible-doc -s apt

4. ansible-galaxy 下载第三方模块
Usage: ansible-galaxy [delete|import|info|init|install|list|login|remove|search|setup] [--help] [options] ...

5. ansible-pull
Usage: ansible-pull -U <repository> [options]

6. ansible-vault 加密playbook文件
Usage: ansible-vault [create|decrypt|edit|encrypt|rekey|view] [--help] [options] vaultfile.yml

# Quick Start
## 1.编辑host inventory文件
/etc/ansible/hosts
ubuntu.test.com
fedora.test.com

## 2.使用SSH keys认证,配置ssh-agent
ssh-agent bash
ssh-add ~/.ssh/id_rsa

## 3.ping all nodes
ansible all -m ping

## 使用指定的远程用户执行任务 -u
ansible all -m ping -u bruce

## 使用bruce,切换到root
ansible -m ping -u bruce -b

## 使用bruce,切换到batman
ansible all -m ping -u bruce -b --become-user batman

## run a live command 
ansible all -a "/bin/echo hello"


# Host and Group

## inventory文件
/etc/ansible/hosts
```
# 指定ssh端口
mail.example.com:2110

# 别名
jumper ansible-port=5555 ansible_host=192.168.2.100

# 分组
[webservers]
www[a:f].example.com
www[01:05].example.com
webserver1
webserver2
webserver3	ansible_connection=ssh 	ansible_user=batman

## Host Variables
[atlanta]
host1	http_port=80	maxRequestsPerChild=808

## Group Variables
[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com

## Groups of Groups, and Group Variables
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southease:vars]
ntp_server=ntp.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest

```

# Default Groups
all
ungrouped


# Splitting Out Host and Group Specific Data


# Patterns
ansible <pattern_goes_here> -m <module_name> -a <arguments>

eg.
ansible webservers -m service -a "name=httpd state=restarted"

all
*

## 组合匹配,逻辑或、与、非
webservers:dbservsers
webservers:&dbserevers
webservers:!dbservers

## 通配符
*.example.com
*.com

one*.com:dbservers

webservers[0]
webservers[0:-1]

## 正则表达式,'~'开头
~(web|db).*\.example\.com

## exclusion criteria
ansible-playbook site.yml --limit datacenter2
ansible-playbook site.yml --limit @retry_hosts.txt


# 并发执行 -f
ansible atlanta -a "/sbin/reboot" -f 10

# shell module
ansible atlanta -m shell -a 'echo $TERM'

# 文件操作
ansible atlanta -m copy -a "src=/etc/hosts dest=/tmp/hosts"

ansible all -m file -a "dest=/tmp/hosts mode=600 owner=batman group=batman"

## 创建目录 "mkdir -p"
ansible all -m file -a "dest=/path/to/c mode=755 owner=batman group=batman state=directory"

## 删除文件和目录(递归)
ansible all -m file -a "dest=/path/to/c state=absent"


# Managing Packages

## ensure package is installed, but don't update
ansible all -m yum -a "name=wget state=present"

## ensure package is installed to a specific version
ansible all -m yum -a "name=wget-1.5 state=present"

## ensure a package is at the latest version
ansible all -m yum -a "name=wget state=latest"

## ensure a package is not installed
ansible all -m yum -a "name=wget state=absent"


# Users and Groups
ansible all -m user -a "name=foo password=test"
ansible all -m user -a "name=foo state=absent"


# Deploying from source control
ansible webserver -m git -a "repo=git://foo.example/org/repo.git dest=/srv/myapp version=HEAD"


# Managing services

## ensure service is started
ansible webserver -m service -a "name=httpd state=started"

## restart a service
ansible webserver -m service -a "name=httpd state=restarted"

## ensure a service is stopped
ansible webserver -m service -a "name=httpd state=stopped"


# Time limited background opreations


# Gathering Facts
ansible all -m setup


# playbook构成
target section 		执行playbook的远程主机组
variable section 	定义playbook运行时用到的变量
task section 		需要执行的任务列表
handler section 	task执行完成后调用的任务

1. Hosts和Users
- hosts: webnodes
  tasks:
    - name: Test ping connection
      remote_user: test
      sudo: yes

2. 任务列表和action: task list中的任务依次在hosts中指定的主机上运行
tasks:
  - name: Make sure apache2 is running
    service: name=httpd state=running

3. handlers: 当关注的资源产生变化时进行的操作
"notify"最好在每个play的最后被触发
- name: template configuration file
  template: src=template.j2 dest=/etc/foo.conf
  notify:
  - restart memcached
  - restart apache

handlers:
  - name: restart memcached
    service: name=memcached state=restarted
  - name: restart apache
    service: name=apache2 state=restarted


# 循环

## 标准循环
- name: add serveral users
  user:
    name: "{{ item }}"
    state: present
    groups: "wheel"
  with_items:
    - testuser1
    - testuser2

- name: add serveral users
  user:
    name: "{{ item.name }}" 
    state: present
    gorups: "{{ item.groups }}"
  with_items:
    - { name: 'testuser1', groups: 'wheel'}
    - { name: 'testuser2', groups: 'root'}

## 嵌套循环
- name: give users access to multiple databases
  mysql_user:
    name: "{{ item[0] }}"
    priv: "{{ item[1] }}.*.:ALL"
    append_privs: yes
    password: "foo"
  with_nested:
    - [ 'alice', 'bob' ]
    - [ 'clientdb', 'employeedb', 'providerdb' ]



# 条件

## when语句
tasks:
  - name: "Shut down Debian flavored systems"
    command: /sbin/shutdown -t now
    when: (ansible_distribution == "Debian" and ansible_distribution_major_version == "6") or 
    	  (ansible_distribution == "CentOS" and ansible_distribution_major_version == "7")

多个条件必须都为True(逻辑与),可以用列表的形式
when:
  - ansible_distribution == "CentOS"
  - ansible_distribution_major_version == "6"

## 循环和条件
tasks:
  - command: echo {{ item }}
    with_items: [ 0, 2, 4, 6, 8, 10 ]
    when: item > 5
