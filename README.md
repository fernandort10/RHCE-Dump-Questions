# RHCE-Dump-Questions

## Introduction

This comprehensive guide provides a series of practice tasks designed to simulate the Red Hat Certified Engineer (RHCE) exam environment. Each task is crafted to test and reinforce your skills in Ansible automation, system administration, and configuration management.

## Prerequisites

- Basic understanding of Ansible concepts
- Access to a lab environment with multiple nodes (node1, node2, node3, node4, node5)
- Familiarity with RHEL 9 administration

## Task List

1. [Install and Configure Ansible on the Control Node](#task-1-install-and-configure-ansible-on-the-control-node)
2. [Create repo.yml for Configuring Repository in All Nodes](#task-2-create-repoyml-for-configuring-repository-in-all-nodes)
3. [Create Roles Directory and Download Roles](#task-3-create-roles-directory-and-download-roles)
4. [Create Offline Apache Role](#task-4-create-offline-apache-role)
5. [Create Balancer and PHPInfo Roles Playbook](#task-5-create-balancer-and-phpinfo-roles-playbook)
6. [Install Ansible Content Collections](#task-6-install-ansible-content-collections)
7. [Install Packages in Multiple Groups](#task-7-install-packages-in-multiple-groups)
8. [Create Web Content Playbook](#task-8-create-web-content-playbook)
9. [Collect Hardware Report](#task-9-collect-hardware-report)
10. [Replace /etc/issue File](#task-10-replace-etcissue-file)
11. [Configure /etc/myhosts](#task-11-configure-etcmyhosts)
12. [Create and Encrypt Vault Variable File](#task-12-create-and-encrypt-vault-variable-file)
13. [Configure Users Using Variable Files](#task-13-configure-users-using-variable-files)
14. [Rekey Variable File](#task-14-rekey-variable-file)
15. [Create Cronjob for User student](#task-15-create-cronjob-for-user-student)
16. [Create and Use Logical Volume](#task-16-create-and-use-logical-volume)
17. [Use RHEL Timesync System Role](#task-17-use-rhel-timesync-system-role)
18. [Use SELinux Role for All Nodes](#task-18-use-selinux-role-for-all-nodes)

## Detailed Tasks

### Task 1: Install and Configure Ansible on the Control Node

1. Install required Ansible packages.

- Verify if ansible is installed
  
```
ansible --version
```

- If not installed do this:

```
sudo dnf install ansible
```
 
2. Create a static inventory file at `/home/student/ansible/inventory`:
   - node1: dev group
   - node2: test group
   - node3, node4: prod group
   - node5: balancers group
   - prod group is a member of the host group

- Create the /ansible folder
```
mkdir /ansible
```
- Create the inventory file and add the configuration.
```
vim inventory
```
```
[dev]
node1

[test]
node2

[prod]
node3
node4

[balancers]
node5

[host:children]
prod
```

3. Create `ansible.cfg` with the following configurations:
   - Inventory file: `/home/student/ansible/inventory`
   - Roles location: `/home/student/ansible/roles`
   - Collections location: `/home/student/ansible/mycollection`

```
vim ansible.cfg
```
```
[defaults]

remote_user=student
inventory=/home/student/ansible/inventory
roles_path=/home/student/ansible/roles
collection_path=/home/student/ansible/mycollection

[privilege_escalation]

become=true
become_method=sudo
become_user=root
```

### Task 2: Create repo.yml for Configuring Repository in All Nodes

Create `repo.yml` to configure the following repositories on all nodes:

1. baseos-internal:
   - Description: "baseos description"
   - URL: `http://content/rhel9.0/x86_64/dvd/baseos`
   - GPG enabled, key: `http://content.example.com/rhel9.0/x86_64/dvd/rpm-gpg-key-redhat-release`
   - Repository enabled

2. appstream-internal:
   - Description: "app description"
   - URL: `http://content/rhel9.0/x86_64/dvd/appstream`
   - GPG enabled, key: `http://content.example.com/rhel9.0/x86_64/dvd/rpm-gpg-key-redhat-release`
   - Repository enabled
  
```
vim repo.yml
```
```
---
- name: Configuring Repositories in ALL Nodes.
  hosts: all
  tasks:
    - name: BaseOS
      yum_repository:
        name: baseos-internal
        description: baseos description
        baseurl: http://content/rhel9.0/x86_64/dvd/baseos
        gpgkey: http://content.example.com/rhel9.0/x86_64/dvd/rpm-gpg-key-redhat-release
        gpgcheck: yes
        enabled: yes

    - name: AppStream
      yum_repository:
        name: appstream-internal
        description: app description
        baseurl: http://content/rhel9.0/x86_64/dvd/appstream
        gpgkey: http://content.example.com/rhel9.0/x86_64/dvd/rpm-gpg-key-redhat-release
        gpgcheck: yes
        enabled: yes
```

### Task 3: Create Roles Directory and Download Roles

1. Create `/home/student/ansible/roles` directory.
2. Create `requirements.yml` in the roles directory.
3. Download roles using Galaxy command:
   - balancer: `http://content.example.com/rhce/balancer.tar`
   - phpinfo: `http://content.example.com/rhce/phpinfo.tar`

- Download and set the roles:
```
wget http://content.example.com/rhce/balancer.tar
wget http://content.example.com/rhce/phpinfo.tar
```

```
vim requirements.yml
```
```
---
- src: /home/student/ansible/roles/balancer.tar
  name: balancer

- src: /home/student/ansible/roles/phpinfo.tar
  name: phpinfo
```
```
ansible-galaxy install -r requirements.yml
```

- Move the roles created in .ansible to your roles folder:
```
cd /home/student/.ansible/roles
mv * /home/student/ansible/roles/
```

### Task 4: Create Offline Apache Role

1. Create role 'apache' under roles directory:
   - Install and enable httpd package and service
   - Host web page using `template.j2`
   - `template.j2` content: `Welcome to <hostname> on <ipaddress>`
2. Create `apache_role.yml` playbook and run the role in the dev group.

```
ansible-galaxy init apache
cd apache/
```
```
vim /templates/template.j2
```
```
Welcome to {{ ansible_facts['hostname'] }} on {{ ansible_facts[ansible_default_ipv4][address] }}
```
```
vim /tasks/main.yml
```
```
---
- name: Install Apache
  dnf:
    name:
      - httpd
      - firewalld
    state: latest

- name: Start httpd service
service:
  name: httpd
  state started
  enabled: yes

- name: Start firewalld service
service:
  name: firewalld
  state: restarted
  enabled: yes

- name: add http service in firewall rule
firewalld:
  service httpd
  state enabled
  permanent yes
  immediate: yes

- name: copy the template.j2 file to web server directory
template:
  src template.j2
  dest: /var/www/html/index.html
```

- In your /ansible folder
```
vim apache_role.yml
```
```
---
- name: Use apache role
  hosts: dev
  roles:
    - apache
```
```
ansible-playbook apache_role.yml
```

### Task 5: Create Balancer and PHPInfo Roles Playbook

Create `/home/admin/ansible/roles.yml`:

1. Use roles from Ansible Galaxy.
2. Configure load balancing for webserver host group.
3. Implement phpinfo role for webserver host group.
4. Ensure proper output for balancer and phpinfo pages.


### Task 6: Install Ansible Content Collections

Install the following collections in the local collections directory:

1. `http://server.lab.example.com/role-collections/redhat-rhel_system_roles.tar.gz`
2. `http://server.lab.example.com/role-collections/community-general-8.3.0.tar.gz`
3. `http://server.lab.example.com/role-collections/ansible-posix-1.5.4.tar.gz`

```
wget http://server.lab.example.com/role-collections/redhat-rhel_system_roles.tar.gz
wget http://server.lab.example.com/role-collections/community-general-8.3.0.tar.gz
wget http://server.lab.example.com/role-collections/ansible-posix-1.5.4.tar.gz
```

```
ansible-galaxy collection install /home/student/ansible/mycollection/redhat-rhel_system_roles.tar.gz -p /home/student/ansible/mycollection/
ansible-galaxy collection install /home/student/ansible/mycollection/community-general-8.3.0.tar.gz -p /home/student/ansible/mycollection/
ansible-galaxy collection install /home/student/ansible/mycollection/ansible-posix-1.5.4.tar.gz -p /home/student/ansible/mycollection/
```

### Task 7: Install Packages in Multiple Groups

Create `packages.yml`:

1. Install vsftpd and mariadb-server in dev and test groups.
2. Install "RPM Development Tools" group package in prod group.
3. Update all packages in each group.
4. Use separate plays for each task.

```
vim packages.yml
```
```
---
- name: packages installation
  hosts: dev,test
  tasks:
    - name: Install vsftpd and mariadb-server
      dnf:
        name:
          - vsftpd
          - mariadb-server
        state: present

- name: Install "RPM Development Tools" group package
  hosts: prod
  tasks:
  - name: Install RPM
    dnf:
      name: '@RPM Development Tools'
      state: present

- name: update packages
  hosts: all
  tasks:
    - name: updating all
      dnf:
        name: '*'
        state: latest
```
```
ansible-playbook packages.yml
```

### Task 8: Create Web Content Playbook

Create `webcontent.yml` for dev group:

1. Create `/devweb` directory owned by devops group.
2. Set context type as httpd.
3. Set permissions: user=rwx, group=rwx, others=rx, with group special permission.
4. Create `/devweb/index.html` with content "Development".
5. Link `/devweb` to `/var/www/html/devweb`.

```
vim webcontent.yml
```
```
---
- name: Create web content
  hosts: dev
  tasks:
    - name: create a dir
      file:
        path: /devweb
        state: directory
        mode: '2775'
        group: devops
        setype: httpd_sys_content_t

    - name: create a symbolic link
      file:
        src: /devweb
        dest: /var/www/html/devweb
        state: link
        setype: httpd_sys_content_t

    - name: copy using inline content
      copy:
        content:
        dest:
        setype:

    - name: permit traffic in default zone
      firewalld:
        service: http
        permanent: true
        state: enabled
        immediate: true
    ```
```
```
ansible-playbook webcontent.yml
```

### Task 9: Collect Hardware Report

Create `hwreport.yml`:

1. Download `hwreport.txt` from `http://content.example.com/rhce/hwreport.empty`.
2. Save as `/root/hwreport.txt`.
3. Show "none" if no information is available.

```
wget http://content.example.com/rhce/hwreport.empty
```
```
vim hwreport.j2
```
```
Hostname={{ ansible_facts['hostname'] | default('none') }}
Total_memory={{ ansible_facts['memtotal_mb'] | default('none') }}
Bios_version={{ ansible_facts['bios_version'] | default('none') }}
CPU={{ ansible_facts['processor']['processor_cores'] | default('none') }}
vda_size={{ ansible_facts['devices']['vda']['size'] | default('none') }}
vdb_size={{ ansible_facts['devices']['vda']['size'] | default('none') }}
```
```
vim hwreport.yml
```
```
- name: Collect Hardware Report
  hosts: all
  tasks:

    - name: Create template
      template:
        src: hwreport.j2
        dest: /root/hwreport.txt
```
```
ansible-playbook hwreport.yml
```

### Task 10: Replace /etc/issue File

Create `issue.yml`:

1. dev group: "Development"
2. test group: "Test"
3. prod group: "Production"
4. Run on all managed nodes.

```
vim issue.yml
```
```
- name: Replace issue file on hosts
  hosts: all
  tasks:

    - name: Replace template
      template:
        src: issue.j2
        dest: /etc/issue
```
```
vim issue.j2
```
```
{% if 'dev' in group_names %}
"Development"
{% elif 'test' in group_names %}
"Test"
{% elif 'prod' in group_names %}
"Production"
{% endif %}
```
```
ansible-playbook issue.yml
```

### Task 11: Configure /etc/myhosts

Create `hosts.yml`:

1. Download template from `http://content.example.com/rhce/hosts.j2`.
2. Populate with node information (IP, FQDN, hostname).
3. Save as `/etc/myhosts` on all managed nodes.
4. Run in dev group.

```
wget http://content.example.com/rhce/hosts.j2
```
```
vim host.j2
```
add this to the hosts.j2 template
```
...

{% for in groups['all'] %}
{{ hostvars[host].ansible_default_ipv4.address }} {{ hostvars[host].ansible_fqdn }} {{ 
hostvars[host].ansible_hostname }}
{% endfor %}
```
```
vim hosts.yml
```
```
- name: use template hosts.j2
  hosts: all
  tasks:
    - name: execute the template
      template:
        src: ./hosts.j2
        dest: /etc/myhosts
- name: delete from all
  hosts: all, !dev
  tasks:
    - name: delete file
      file:
        path: /etc/myhosts
        state: absent
```
```
ansible-playbook hosts.yml
```

### Task 12: Create and Encrypt Vault Variable File

1. Create `vault.yml` with variables:
   - pw_developer: lamdev
   - pw_manager: lammgr
2. Encrypt using password "P@ssword".
3. Store password in `secret.txt`.

```
vim secret.txt
```
```
P@assword
```
```
ansible-vault create vault.yml --vault-password-file=secret.txt
```
```
pw_developer: lamdev
pw_manager: lammgr
```
```
ansible-vault view vault.yml --vault-password-file=secret.txt
```

- Modify the ansible.cfg and add the vault password file

```
vim ansible.cfg
```
```
[defaults]

remote_user=student
inventory=/home/student/ansible/inventory
roles_path=/home/student/ansible/roles
collection_path=/home/student/ansible/mycollection
vault_password_file=./secret.txt

[privilege_escalation]

become=true
become_method=sudo
become_user=root
```

### Task 13: Configure Users Using Variable Files

Create `users.yml`:

1. Download `user_list.yml` from `http://content.example.com/rhce/user_list.yml`.
2. Use `user_list.yml` and `vault.yml`.
3. Create opsdev and opsmgr groups.
4. Create users based on job roles.
5. Set passwords using SHA512 format.
6. Use when conditions for different host groups.

```
wget http://content.example.com/rhce/user_list.yml
```
```
vim users.yml
```
```
---
- name: user creation
  hosts: dev,test
  vars_files:
    - user_list.yml
    - vault.yml
  tasks:

  - name: add group
    group:
      name: opsdev
      state: present

  - name: adding users developer
    user:
      name: "{{ item.name }}"
      groups: opsdev
      password: "{{ pw_developer | password_hash(''sha512') }}"
    when: item.job=='developer'
    loop: "{{ users }}"

- name: Create users and groups manager
  hosts: prod
  vars_files:
    - user_list.yml
    - vault.yml
  tasks:

   - name: add group
          group:
            name: opsmgr
            state: present
  - name: adding users manager
    user:
      name: "{{ item.name }}"
      groups: opsmgr
      password: "{{ pw_manager | password_hash(''sha512') }}"
    when: item.job=='manager'
    loop: "{{ users }}"
```

```
ansible-playbook users.yml
```

### Task 14: Rekey Variable File

Rekey `http://content.example.com/rhce/salaries.yml`:
- Old password: cisco
- New password: redhat

### Task 15: Create Cronjob for User student

Create `crontab.yml`:
- Set up cron job for user student on all nodes.
- Run every 2 minutes: `logger "EX294 in progress"`

### Task 16: Create and Use Logical Volume

Create `/home/admin/ansible/ansible/lv.yml`:

1. Create logical volume "data" in "research" volume group.
2. Size: 1500 MiB (fallback to 800 MiB if not possible).
3. Format with ext4 filesystem.
4. Handle errors for non-existent volume group.
5. Do not mount the logical volume.

### Task 17: Use RHEL Timesync System Role

Create `/home/admin/ansible/timesync.yml`:

1. Run on all managed nodes.
2. Use timesync role.
3. Configure active NTP provider.
4. Use time server: classroom.lab.example.com.
5. Enable iburst parameter.

### Task 18: Use SELinux Role for All Nodes

Create `selinux.yml`:
- Set SELinux to enforcing mode on all nodes.

## Conclusion

Work through these tasks systematically to prepare for your RHCE exam. Test each playbook in your lab environment and verify the expected outcomes. Remember to consult the official Red Hat documentation for any clarifications. Good luck with your exam preparation!
