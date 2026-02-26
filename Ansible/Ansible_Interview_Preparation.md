# Ansible Interview Preparation Guide
## Based on Resume: Harshal Kalamkar - DevOps Engineer

---

## SECTION 1: BASIC LEVEL QUESTIONS

### 1. What is Ansible and Why Use It?

**Q: Explain Ansible and its advantages over other configuration management tools.**

**Answer:**
"Ansible is an agentless, open-source automation tool used for configuration management, application deployment, and orchestration. I've used it extensively at both Birlasoft and Infosys.

**Key Characteristics:**

1. **Agentless Architecture**
   - Uses SSH for Linux (no agent installation needed)
   - Uses WinRM for Windows
   - Just needs Python on target machines

2. **Simple & Readable**
   - Uses YAML for playbooks (human-readable)
   - Declarative syntax - you specify what you want, not how to do it
   - Easy learning curve compared to Puppet/Chef

3. **Idempotent**
   - Safe to run multiple times
   - Only makes changes when needed
   - Example: Installing a package that's already installed = no action

**Why I Choose Ansible:**

**vs Puppet/Chef:**
- No master-slave architecture needed
- No agents to maintain and update
- Simpler to get started

**vs Scripts:**
- Idempotent (scripts often aren't)
- Structured and maintainable
- Built-in modules for common tasks
- Error handling and rollback capabilities

**Real Example from Birlasoft:**
At Birlasoft, we used Ansible to configure 100+ servers. With scripts, we had issues with:
- Scripts running multiple times causing problems
- Hard to track what's configured where
- No rollback capability

Ansible solved all these issues with playbooks that were idempotent, version-controlled, and easy to understand."

---

### 2. Ansible Architecture & Components

**Q: Explain Ansible architecture and its core components.**

**Answer:**

```
┌─────────────────────────────────────────────────┐
│         CONTROL NODE (Your Laptop/AWX)          │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌──────────────┐      ┌──────────────┐        │
│  │  Inventory   │      │  Playbooks   │        │
│  │  (hosts)     │      │  (tasks)     │        │
│  └──────────────┘      └──────────────┘        │
│                                                 │
│  ┌──────────────┐      ┌──────────────┐        │
│  │   Modules    │      │    Roles     │        │
│  │ (yum,copy..) │      │ (organize)   │        │
│  └──────────────┘      └──────────────┘        │
│                                                 │
└────────────┬────────────────────────────────────┘
             │
             │ SSH (Linux) / WinRM (Windows)
             │
    ┌────────┼────────┬────────────┐
    │        │        │            │
┌───▼────┐ ┌─▼──────┐ ┌──▼──────┐ ┌──▼──────┐
│ Node 1 │ │ Node 2 │ │  Node 3 │ │  Node N │
│ (web)  │ │  (db)  │ │ (cache) │ │  (app)  │
└────────┘ └────────┘ └─────────┘ └─────────┘
```

**Core Components:**

**1. Control Node:**
- Machine where Ansible is installed
- Runs playbooks and commands
- Can be your laptop, Jenkins server, or AWX

**2. Managed Nodes:**
- Target servers you're configuring
- No agent needed - just SSH access and Python
- Can be on-prem, cloud, or hybrid

**3. Inventory:**
```ini
# /etc/ansible/hosts
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[webservers:vars]
ansible_user=deploy
ansible_port=22
```
- List of managed nodes
- Can be static (file) or dynamic (script/plugin)
- Groups hosts for easier management

**4. Modules:**
- Reusable code units that do specific tasks
- Examples: `yum`, `apt`, `copy`, `service`, `user`
- Over 3000+ modules available
```yaml
- name: Install nginx
  yum:
    name: nginx
    state: present
```

**5. Playbooks:**
- YAML files defining automation tasks
- Contain plays (groups of tasks)
- Declarative - describe desired state

**6. Roles:**
- Way to organize playbooks
- Reusable, shareable units
- Standard directory structure

**7. Plugins:**
- Extend Ansible functionality
- Connection plugins, lookup plugins, filter plugins

**At Birlasoft, our setup:**
- AWX as control node (Kubernetes deployment)
- 200+ managed nodes (VMware VMs + cloud instances)
- Dynamic inventory syncing from VMware vCenter and AWS
- Custom roles for application deployment
- Centralized credential management through AWX vault"

---

### 3. Inventory Management

**Q: Explain static vs dynamic inventory. How did you use them?**

**Answer:**
"I've used both static and dynamic inventories extensively:

**Static Inventory:**

```ini
# inventory/production.ini

# Individual hosts
web1.example.com
web2.example.com

# Groups
[webservers]
web[1:3].example.com  # Expands to web1, web2, web3

[databases]
db1.example.com
db2.example.com

[loadbalancers]
lb1.example.com
lb2.example.com

# Group of groups
[production:children]
webservers
databases
loadbalancers

# Variables for a group
[webservers:vars]
ansible_user=deploy
ansible_port=22
http_port=80
app_env=production

# Variables for individual host
[databases]
db1.example.com mysql_role=primary
db2.example.com mysql_role=replica
```

**Pros:**
- Simple to understand
- Easy to maintain for small environments
- Version controlled with Git

**Cons:**
- Manual updates when infrastructure changes
- Doesn't scale well
- Out of sync if VMs are created/destroyed

---

**Dynamic Inventory:**

At Birlasoft with AWX, we used dynamic inventory:

**1. AWS EC2 Dynamic Inventory:**
```yaml
# aws_ec2.yml (inventory plugin)
plugin: aws_ec2
regions:
  - us-east-1
  - us-west-2

# Group by tags
keyed_groups:
  - key: tags.Environment
    prefix: env
  - key: tags.Application
    prefix: app
  - key: instance_type
    prefix: instance_type

# Filter instances
filters:
  instance-state-name: running
  tag:ManagedBy: Ansible

# Compose variables
compose:
  ansible_host: public_ip_address
```

**Usage:**
```bash
# Automatically discovers all running EC2 instances
ansible-inventory -i aws_ec2.yml --graph

# Groups created automatically:
# - env_production
# - env_staging
# - app_webapp
# - app_database
# - instance_type_t3_medium
```

**2. VMware Dynamic Inventory (Birlasoft):**
```yaml
# vmware.yml
plugin: vmware_vm_inventory
hostname: vcenter.company.com
username: ansible@vsphere.local
password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          ...

validate_certs: false

keyed_groups:
  - key: config.guestId
    prefix: os
  - key: summary.runtime.powerState
    prefix: state
  - key: customValue  # Custom attributes in vCenter
```

At Birlasoft, our vCenter had custom attributes like:
- `Application`: webapp, database, cache
- `Environment`: prod, staging, dev
- `Team`: backend, frontend, data

Ansible automatically created groups based on these!

**3. Custom Dynamic Inventory Script (Python):**

```python
#!/usr/bin/env python3
# dynamic_inventory.py

import json
import requests

def get_inventory():
    # Query CMDB API
    response = requests.get('https://cmdb.company.com/api/servers')
    servers = response.json()
    
    inventory = {
        '_meta': {'hostvars': {}},
        'all': {'hosts': []},
        'webservers': {'hosts': []},
        'databases': {'hosts': []}
    }
    
    for server in servers:
        hostname = server['hostname']
        inventory['all']['hosts'].append(hostname)
        
        # Group by role
        if server['role'] == 'web':
            inventory['webservers']['hosts'].append(hostname)
        elif server['role'] == 'db':
            inventory['databases']['hosts'].append(hostname)
        
        # Host variables
        inventory['_meta']['hostvars'][hostname] = {
            'ansible_host': server['ip_address'],
            'ansible_user': server['ssh_user'],
            'environment': server['environment']
        }
    
    return inventory

if __name__ == '__main__':
    print(json.dumps(get_inventory(), indent=2))
```

**Usage:**
```bash
ansible-playbook -i dynamic_inventory.py site.yml
```

---

**AWX Dynamic Inventory (Production at Birlasoft):**

In AWX, we configured:

1. **Inventory Sources:**
   - VMware vCenter (synced every 30 minutes)
   - AWS EC2 (synced every 15 minutes)
   - Azure (synced hourly)
   - Custom script (synced on-demand)

2. **Benefits:**
   - Always up-to-date
   - No manual maintenance
   - Automatic grouping
   - Multi-source consolidation

3. **Example AWX Inventory Sync:**
```yaml
# AWX automatically creates groups like:
os_ubuntu2004  # All Ubuntu 20.04 VMs
state_poweredOn  # All running VMs
app_webapp  # All webapp servers
env_production  # All production servers

# Can target with:
ansible webservers:&env_production  # Webservers AND production
ansible all:!env_production  # All EXCEPT production
```

**Real Production Example:**

At Birlasoft, we had a nightly patching job:

```yaml
# patch-servers.yml
- name: Patch staging servers nightly
  hosts: env_staging:&state_poweredOn  # Only running staging VMs
  serial: 5  # Patch 5 at a time
  
  tasks:
    - name: Update all packages
      yum:
        name: '*'
        state: latest
        update_cache: yes
    
    - name: Reboot if needed
      reboot:
        reboot_timeout: 600
      when: ansible_facts.pkg_mgr == 'yum'
```

Dynamic inventory ensured:
- Only running VMs were patched
- New VMs automatically included
- Decommissioned VMs automatically excluded
- No manual inventory updates needed

This saved us hours of manual inventory maintenance every week!"

---

### 4. Playbooks, Plays, and Tasks

**Q: Explain the structure of an Ansible playbook. What are plays and tasks?**

**Answer:**

```yaml
---
# Playbook (YAML file containing one or more plays)
# File: webserver-setup.yml

# PLAY 1: Configure web servers
- name: Setup web servers              # Play name
  hosts: webservers                    # Target hosts
  become: yes                          # Run as sudo
  become_user: root                    # Sudo to root
  vars:                                # Play variables
    http_port: 80
    doc_root: /var/www/html
  
  tasks:                               # List of tasks
    
    # TASK 1
    - name: Install nginx
      yum:
        name: nginx
        state: present
    
    # TASK 2
    - name: Start nginx service
      service:
        name: nginx
        state: started
        enabled: yes
    
    # TASK 3
    - name: Copy website files
      copy:
        src: files/index.html
        dest: "{{ doc_root }}/index.html"
        owner: nginx
        group: nginx
        mode: '0644'

# PLAY 2: Configure databases
- name: Setup database servers
  hosts: databases
  become: yes
  
  tasks:
    - name: Install PostgreSQL
      yum:
        name: postgresql-server
        state: present
    
    - name: Initialize database
      command: postgresql-setup initdb
      args:
        creates: /var/lib/pgsql/data/PG_VERSION
```

**Hierarchy Breakdown:**

```
Playbook (webserver-setup.yml)
│
├── Play 1: "Setup web servers"
│   ├── hosts: webservers
│   ├── vars: http_port, doc_root
│   └── tasks:
│       ├── Task 1: Install nginx
│       ├── Task 2: Start nginx
│       └── Task 3: Copy files
│
└── Play 2: "Setup database servers"
    ├── hosts: databases
    └── tasks:
        ├── Task 1: Install PostgreSQL
        └── Task 2: Initialize database
```

---

**Components Explained:**

**1. Playbook:**
- YAML file (starts with `---`)
- Contains one or more plays
- Defines entire automation workflow
- Can include multiple plays for different host groups

**2. Play:**
- Maps hosts to tasks
- Defines what should run where
- Can have variables, handlers, roles
- Runs tasks in order (top to bottom)

**Example:**
```yaml
- name: This is a play
  hosts: webservers     # WHO to configure
  tasks:                # WHAT to do
    - name: Task 1
    - name: Task 2
```

**3. Task:**
- Single unit of work
- Calls an Ansible module
- Has a name (for readability)
- Idempotent by default

**Example:**
```yaml
- name: Install nginx        # Human-readable description
  yum:                       # Module name
    name: nginx              # Module parameters
    state: present
```

---

**Real Production Example (from Infosys):**

```yaml
---
# deploy-webapp.yml
# Complete application deployment playbook

# Play 1: Prepare infrastructure
- name: Prepare servers for deployment
  hosts: appservers
  become: yes
  
  tasks:
    - name: Create application user
      user:
        name: webapp
        state: present
        shell: /bin/bash
    
    - name: Create application directories
      file:
        path: "{{ item }}"
        state: directory
        owner: webapp
        group: webapp
        mode: '0755'
      loop:
        - /opt/webapp
        - /opt/webapp/logs
        - /var/log/webapp

# Play 2: Install dependencies
- name: Install application dependencies
  hosts: appservers
  become: yes
  
  tasks:
    - name: Install Java
      yum:
        name: java-11-openjdk
        state: present
    
    - name: Install required packages
      yum:
        name:
          - python3
          - git
          - curl
        state: present

# Play 3: Deploy application
- name: Deploy application
  hosts: appservers
  become: yes
  become_user: webapp
  vars:
    app_version: "1.0.0"
    artifact_url: "https://artifactory.company.com/webapp-{{ app_version }}.jar"
  
  tasks:
    - name: Download application artifact
      get_url:
        url: "{{ artifact_url }}"
        dest: /opt/webapp/webapp.jar
        mode: '0644'
    
    - name: Copy application config
      template:
        src: templates/application.properties.j2
        dest: /opt/webapp/application.properties
        mode: '0600'
    
    - name: Copy systemd service file
      template:
        src: templates/webapp.service.j2
        dest: /etc/systemd/system/webapp.service
        mode: '0644'
      become: yes
      become_user: root
    
    - name: Reload systemd
      systemd:
        daemon_reload: yes
      become: yes
      become_user: root
    
    - name: Start application
      systemd:
        name: webapp
        state: restarted
        enabled: yes
      become: yes
      become_user: root
    
    - name: Wait for application to start
      wait_for:
        port: 8080
        delay: 10
        timeout: 60
    
    - name: Health check
      uri:
        url: http://localhost:8080/health
        status_code: 200
      retries: 5
      delay: 10
```

**Running the playbook:**
```bash
# Syntax check
ansible-playbook deploy-webapp.yml --syntax-check

# Dry run (see what would change)
ansible-playbook deploy-webapp.yml --check

# Run for real
ansible-playbook deploy-webapp.yml

# Run with extra variables
ansible-playbook deploy-webapp.yml -e "app_version=1.0.1"

# Run only specific tags
ansible-playbook deploy-webapp.yml --tags "deploy"

# Verbose output
ansible-playbook deploy-webapp.yml -vvv
```

**Output looks like:**
```
PLAY [Prepare servers for deployment] ****************************************

TASK [Gathering Facts] *******************************************************
ok: [web1.example.com]
ok: [web2.example.com]

TASK [Create application user] ***********************************************
changed: [web1.example.com]
changed: [web2.example.com]

TASK [Create application directories] ****************************************
changed: [web1.example.com] => (item=/opt/webapp)
changed: [web2.example.com] => (item=/opt/webapp)
changed: [web1.example.com] => (item=/opt/webapp/logs)
changed: [web2.example.com] => (item=/opt/webapp/logs)

PLAY RECAP *******************************************************************
web1.example.com    : ok=10   changed=5    unreachable=0    failed=0
web2.example.com    : ok=10   changed=5    unreachable=0    failed=0
```

**Key Points:**
- Multiple plays for logical separation
- Tasks run in order
- Idempotent - safe to run multiple times
- Clear naming for debugging
- Uses variables for flexibility
- Includes error handling (wait_for, health checks)

This structure made our deployments consistent, repeatable, and easy to debug at Infosys."

---

### 5. Modules in Ansible

**Q: What are Ansible modules? Name some commonly used modules you've worked with.**

**Answer:**
"Ansible modules are reusable, standalone scripts that do specific tasks. They're the building blocks of Ansible automation.

**Key Characteristics:**
- Idempotent (most of them)
- Return JSON output
- Can be written in any language (but Python is standard)
- 3000+ modules available

**Modules I Use Daily:**

**1. Package Management:**

```yaml
# YUM (RHEL/CentOS)
- name: Install nginx
  yum:
    name: nginx
    state: present

# APT (Ubuntu/Debian)
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

# Multiple packages
- name: Install multiple packages
  yum:
    name:
      - nginx
      - postgresql
      - redis
    state: present

# Specific version
- name: Install specific version
  yum:
    name: nginx-1.18.0
    state: present

# Update all packages
- name: Update all packages
  yum:
    name: '*'
    state: latest
```

---

**2. File Operations:**

```yaml
# Copy files
- name: Copy config file
  copy:
    src: files/nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
    backup: yes  # Backup existing file

# Create directories
- name: Create directory
  file:
    path: /opt/webapp
    state: directory
    owner: webapp
    group: webapp
    mode: '0755'

# Create symlinks
- name: Create symlink
  file:
    src: /opt/webapp/current
    dest: /var/www/html
    state: link

# Delete files/directories
- name: Remove old logs
  file:
    path: /var/log/oldlogs
    state: absent

# Set permissions
- name: Set permissions on script
  file:
    path: /opt/scripts/deploy.sh
    mode: '0755'
```

---

**3. Service Management:**

```yaml
# Start/stop services
- name: Start nginx
  service:
    name: nginx
    state: started
    enabled: yes  # Enable on boot

# Restart service
- name: Restart nginx
  service:
    name: nginx
    state: restarted

# Reload without restart
- name: Reload nginx config
  service:
    name: nginx
    state: reloaded

# Using systemd directly
- name: Restart using systemd
  systemd:
    name: nginx
    state: restarted
    daemon_reload: yes
```

---

**4. Template (with Jinja2):**

```yaml
# Template file with variables
- name: Deploy nginx config from template
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
    validate: 'nginx -t -c %s'  # Test config before replacing
  notify: restart nginx
```

**Template file (nginx.conf.j2):**
```jinja2
worker_processes {{ ansible_processor_vcpus }};

http {
    server {
        listen {{ http_port }};
        server_name {{ ansible_hostname }};
        
        root {{ doc_root }};
        
        {% if enable_ssl %}
        listen 443 ssl;
        ssl_certificate {{ ssl_cert_path }};
        ssl_certificate_key {{ ssl_key_path }};
        {% endif %}
    }
}
```

---

**5. User Management:**

```yaml
# Create user
- name: Create deploy user
  user:
    name: deploy
    state: present
    shell: /bin/bash
    groups: wheel  # Add to sudo group
    append: yes
    create_home: yes

# Set password (hashed)
- name: Set user password
  user:
    name: deploy
    password: "{{ 'mypassword' | password_hash('sha512') }}"

# Create SSH key
- name: Generate SSH key
  user:
    name: deploy
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_rsa
```

---

**6. Command/Shell Execution:**

```yaml
# Run command
- name: Check Java version
  command: java -version
  register: java_version
  changed_when: false  # Doesn't change system state

# Run shell command (with pipes, redirects)
- name: Get running processes
  shell: ps aux | grep nginx | grep -v grep
  register: nginx_processes
  failed_when: false  # Don't fail if not found

# Run command only if file doesn't exist
- name: Initialize database
  command: /usr/bin/postgresql-setup initdb
  args:
    creates: /var/lib/pgsql/data/PG_VERSION
```

---

**7. Git Operations:**

```yaml
# Clone repository
- name: Clone application repo
  git:
    repo: https://github.com/company/webapp.git
    dest: /opt/webapp
    version: main  # Branch or tag
    force: yes

# Update existing clone
- name: Pull latest code
  git:
    repo: https://github.com/company/webapp.git
    dest: /opt/webapp
    update: yes
    version: "{{ app_version }}"
```

---

**8. Download/Upload:**

```yaml
# Download file
- name: Download artifact
  get_url:
    url: https://artifactory.company.com/webapp-1.0.0.jar
    dest: /opt/webapp/webapp.jar
    checksum: sha256:abc123...
    mode: '0644'

# Upload file to S3 (AWS)
- name: Upload backup to S3
  aws_s3:
    bucket: my-backups
    object: /backups/db-backup-{{ ansible_date_time.date }}.sql
    src: /tmp/backup.sql
    mode: put
```

---

**9. Database Operations:**

```yaml
# PostgreSQL
- name: Create database
  postgresql_db:
    name: webapp_db
    state: present

- name: Create database user
  postgresql_user:
    name: webapp_user
    password: secret
    db: webapp_db
    priv: ALL
    state: present

# MySQL
- name: Create MySQL database
  mysql_db:
    name: webapp_db
    state: present

- name: Create MySQL user
  mysql_user:
    name: webapp_user
    password: secret
    priv: 'webapp_db.*:ALL'
    state: present
```

---

**10. Cloud Modules (AWS):**

```yaml
# EC2 instance
- name: Launch EC2 instance
  ec2:
    key_name: my-key
    instance_type: t3.medium
    image: ami-12345678
    wait: yes
    group: webserver-sg
    vpc_subnet_id: subnet-abc123
    assign_public_ip: yes
    instance_tags:
      Name: webserver-1
      Environment: production

# Security group
- name: Create security group
  ec2_group:
    name: webserver-sg
    description: Web server security group
    vpc_id: vpc-abc123
    rules:
      - proto: tcp
        from_port: 80
        to_port: 80
        cidr_ip: 0.0.0.0/0
      - proto: tcp
        from_port: 443
        to_port: 443
        cidr_ip: 0.0.0.0/0

# S3 bucket
- name: Create S3 bucket
  s3_bucket:
    name: my-webapp-assets
    state: present
    region: us-east-1
```

---

**11. Wait/Assertion Modules:**

```yaml
# Wait for port
- name: Wait for application to start
  wait_for:
    port: 8080
    delay: 5
    timeout: 60
    state: started

# Wait for file
- name: Wait for log file
  wait_for:
    path: /var/log/webapp/startup.log
    search_regex: "Application started"
    timeout: 120

# Assert condition
- name: Verify nginx is running
  assert:
    that:
      - ansible_facts.services['nginx.service'].state == 'running'
    fail_msg: "Nginx is not running!"
```

---

**12. Debug/Output:**

```yaml
# Print message
- name: Print variable
  debug:
    msg: "Application version is {{ app_version }}"

# Print variable value
- name: Show disk space
  debug:
    var: ansible_facts.mounts

# Fail with message
- name: Fail if not enough disk space
  fail:
    msg: "Not enough disk space: {{ ansible_facts.mounts[0].size_available }}"
  when: ansible_facts.mounts[0].size_available < 1000000000
```

---

**Real Production Example (from Birlasoft):**

```yaml
---
# Application deployment with multiple modules
- name: Deploy microservice
  hosts: appservers
  become: yes
  vars:
    app_version: "1.2.0"
    app_name: "user-service"
    artifact_url: "https://artifactory.company.com/{{ app_name }}-{{ app_version }}.jar"
  
  tasks:
    # 1. Prepare environment
    - name: Create app directory
      file:
        path: /opt/{{ app_name }}
        state: directory
        owner: appuser
        group: appuser
        mode: '0755'
    
    # 2. Stop existing application
    - name: Stop application if running
      systemd:
        name: "{{ app_name }}"
        state: stopped
      ignore_errors: yes
    
    # 3. Download new version
    - name: Download application artifact
      get_url:
        url: "{{ artifact_url }}"
        dest: /opt/{{ app_name }}/{{ app_name }}.jar
        owner: appuser
        group: appuser
        mode: '0644'
    
    # 4. Deploy configuration
    - name: Deploy application config
      template:
        src: templates/application.yml.j2
        dest: /opt/{{ app_name }}/application.yml
        owner: appuser
        group: appuser
        mode: '0600'
    
    # 5. Deploy systemd service
    - name: Deploy systemd service file
      template:
        src: templates/app.service.j2
        dest: /etc/systemd/system/{{ app_name }}.service
        mode: '0644'
      notify: reload systemd
    
    # 6. Start application
    - name: Start application
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes
    
    # 7. Wait for startup
    - name: Wait for application port
      wait_for:
        port: 8080
        delay: 10
        timeout: 120
    
    # 8. Health check
    - name: Check application health
      uri:
        url: http://localhost:8080/actuator/health
        status_code: 200
        return_content: yes
      register: health_check
      retries: 5
      delay: 10
      until: health_check.status == 200
    
    # 9. Verify
    - name: Display health status
      debug:
        msg: "Application {{ app_name }} v{{ app_version }} is healthy!"
  
  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes
```

**Module Categories Summary:**

| Category | Common Modules | Use Case |
|----------|----------------|----------|
| Package | yum, apt, pip | Install software |
| Files | copy, template, file | Manage files/directories |
| System | service, systemd, cron, user | System configuration |
| Commands | command, shell, script | Execute commands |
| Database | mysql_db, postgresql_db | Database management |
| Cloud | ec2, s3_bucket, azure_rm | Cloud resources |
| Networking | uri, get_url | HTTP operations |
| Source Control | git, svn | Code management |

**Module Selection Tips:**

1. **Always prefer modules over shell/command:**
   ```yaml
   # Bad
   - shell: yum install -y nginx
   
   # Good
   - yum:
       name: nginx
       state: present
   ```

2. **Check module documentation:**
   ```bash
   ansible-doc yum
   ansible-doc -l  # List all modules
   ```

3. **Use check mode:**
   ```bash
   ansible-playbook playbook.yml --check  # Dry run
   ```

This module knowledge has been crucial in my automation work at both Birlasoft and Infosys!"

---

### 6. Variables in Ansible

**Q: How do you use variables in Ansible? Explain different ways to define and use them.**

**Answer:**
"Variables make playbooks flexible and reusable. I use variables extensively for environment-specific configurations.

**Ways to Define Variables:**

**1. Playbook Variables:**
```yaml
---
- name: Example playbook
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
    app_env: production
  
  tasks:
    - name: Show variables
      debug:
        msg: "Running on port {{ http_port }} in {{ app_env }}"
```

**2. External Variable Files:**
```yaml
# vars/production.yml
---
http_port: 80
https_port: 443
app_env: production
db_host: prod-db.company.com
db_port: 5432
cache_enabled: true
```

```yaml
# playbook.yml
- name: Deploy app
  hosts: webservers
  vars_files:
    - vars/production.yml
    - vars/secrets.yml
  
  tasks:
    - name: Configure application
      template:
        src: app.conf.j2
        dest: /etc/app/app.conf
```

**3. Inventory Variables:**

```ini
# inventory/production.ini
[webservers]
web1.example.com http_port=80 ssl_enabled=true
web2.example.com http_port=8080 ssl_enabled=false

[webservers:vars]
ansible_user=deploy
ansible_python_interpreter=/usr/bin/python3
app_env=production
```

**4. Group Variables (group_vars):**
```
project/
├── inventory/
│   └── production.ini
├── group_vars/
│   ├── all.yml          # Variables for all hosts
│   ├── webservers.yml   # Variables for webservers group
│   └── databases.yml    # Variables for databases group
└── playbook.yml
```

```yaml
# group_vars/all.yml
---
ntp_server: ntp.company.com
dns_servers:
  - 8.8.8.8
  - 8.8.4.4
company_domain: company.com
```

```yaml
# group_vars/webservers.yml
---
http_port: 80
https_port: 443
max_connections: 1000
worker_processes: 4
```

**5. Host Variables (host_vars):**
```
project/
├── inventory/
│   └── production.ini
├── host_vars/
│   ├── web1.example.com.yml
│   └── web2.example.com.yml
└── playbook.yml
```

```yaml
# host_vars/web1.example.com.yml
---
server_id: 1
is_primary: true
backup_schedule: "0 2 * * *"
```

**6. Command Line Variables:**
```bash
# Using -e or --extra-vars (highest precedence)
ansible-playbook deploy.yml -e "app_version=1.2.0"
ansible-playbook deploy.yml -e "app_version=1.2.0 app_env=staging"

# From JSON file
ansible-playbook deploy.yml -e "@vars/prod.json"

# From YAML file
ansible-playbook deploy.yml -e "@vars/prod.yml"
```

**7. Registered Variables:**
```yaml
tasks:
  - name: Get current time
    command: date +%Y-%m-%d
    register: current_date
  
  - name: Show date
    debug:
      msg: "Today is {{ current_date.stdout }}"
  
  - name: Check if nginx is installed
    command: which nginx
    register: nginx_check
    failed_when: false
    changed_when: false
  
  - name: Install nginx if not present
    yum:
      name: nginx
      state: present
    when: nginx_check.rc != 0
```

**8. Facts (Automatically Gathered):**
```yaml
tasks:
  # Ansible automatically gathers facts
  - name: Show OS information
    debug:
      msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"
  
  - name: Show hostname
    debug:
      msg: "Hostname: {{ ansible_hostname }}"
  
  - name: Show IP address
    debug:
      msg: "IP: {{ ansible_default_ipv4.address }}"
  
  - name: Show CPU count
    debug:
      msg: "CPUs: {{ ansible_processor_vcpus }}"
  
  - name: Show memory
    debug:
      msg: "Memory: {{ ansible_memtotal_mb }} MB"
```

**Commonly Used Facts:**
```yaml
ansible_hostname          # server1
ansible_fqdn              # server1.company.com
ansible_distribution      # Ubuntu, CentOS, RedHat
ansible_distribution_version  # 20.04, 7, 8
ansible_os_family         # Debian, RedHat
ansible_architecture      # x86_64
ansible_processor_vcpus   # 4
ansible_memtotal_mb       # 16384
ansible_default_ipv4.address  # 192.168.1.100
ansible_all_ipv4_addresses    # ['192.168.1.100', '10.0.0.1']
ansible_mounts            # List of mounted filesystems
ansible_date_time.date    # 2024-02-13
```

---

**Variable Precedence (Lowest to Highest):**

```
1.  role defaults (lowest)
2.  inventory file or script group vars
3.  inventory group_vars/all
4.  inventory group_vars/*
5.  inventory file or script host vars
6.  inventory host_vars/*
7.  playbook group_vars/all
8.  playbook group_vars/*
9.  playbook host_vars/*
10. host facts
11. play vars
12. play vars_files
13. role vars
14. block vars (only for tasks in block)
15. task vars (only for the task)
16. include_vars
17. set_facts / registered vars
18. role params
19. include params
20. extra vars (-e, highest precedence)
```

---

**Variable Usage in Templates:**

```jinja2
{# templates/nginx.conf.j2 #}
worker_processes {{ ansible_processor_vcpus }};

events {
    worker_connections {{ worker_connections | default(1024) }};
}

http {
    server {
        listen {{ http_port }};
        server_name {{ ansible_fqdn }};
        
        {% if ssl_enabled %}
        listen {{ https_port }} ssl;
        ssl_certificate {{ ssl_cert_path }};
        ssl_certificate_key {{ ssl_key_path }};
        {% endif %}
        
        {% for upstream in app_upstreams %}
        upstream {{ upstream.name }} {
            {% for server in upstream.servers %}
            server {{ server }};
            {% endfor %}
        }
        {% endfor %}
    }
}
```

---

**Real Production Example (from Birlasoft):**

```
project/
├── inventory/
│   ├── production.ini
│   └── staging.ini
├── group_vars/
│   ├── all.yml
│   ├── production/
│   │   ├── vars.yml
│   │   └── vault.yml  # Encrypted
│   └── staging/
│       ├── vars.yml
│       └── vault.yml
├── host_vars/
│   └── app-server-1.yml
└── deploy.yml
```

```yaml
# group_vars/all.yml
---
company_name: "Birlasoft"
ntp_servers:
  - ntp1.company.com
  - ntp2.company.com
log_retention_days: 30
backup_enabled: true
```

```yaml
# group_vars/production/vars.yml
---
environment: production
app_replicas: 5
db_pool_size: 100
cache_ttl: 3600
monitoring_enabled: true
debug_mode: false

db_host: prod-db.company.internal
db_port: 5432
db_name: webapp_prod

redis_host: prod-redis.company.internal
redis_port: 6379
```

```yaml
# group_vars/production/vault.yml (encrypted with ansible-vault)
---
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62383437373936393834376338646534653935643766393336346264336233623362323861396134
          ...

api_key: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          39373236343835363734363836353934653932643736393336346264336233623362323861396134
          ...
```

```yaml
# deploy.yml
---
- name: Deploy application
  hosts: appservers
  vars:
    app_version: "{{ lookup('env', 'APP_VERSION') | default('1.0.0') }}"
    artifact_url: "https://artifactory.company.com/webapp-{{ app_version }}.jar"
  
  tasks:
    - name: Show deployment info
      debug:
        msg: |
          Deploying to: {{ environment }}
          Version: {{ app_version }}
          DB Host: {{ db_host }}
          Replicas: {{ app_replicas }}
    
    - name: Deploy config
      template:
        src: templates/application.yml.j2
        dest: /opt/webapp/application.yml
        mode: '0600'
```

```yaml
# templates/application.yml.j2
---
spring:
  application:
    name: webapp
  
  datasource:
    url: jdbc:postgresql://{{ db_host }}:{{ db_port }}/{{ db_name }}
    username: {{ db_username }}
    password: {{ db_password }}
    hikari:
      maximum-pool-size: {{ db_pool_size }}
  
  redis:
    host: {{ redis_host }}
    port: {{ redis_port }}

server:
  port: 8080

logging:
  level:
    root: {% if debug_mode %}DEBUG{% else %}INFO{% endif %}

app:
  environment: {{ environment }}
  replicas: {{ app_replicas }}
  monitoring:
    enabled: {{ monitoring_enabled }}
```

**Running with different environments:**
```bash
# Production
ansible-playbook -i inventory/production.ini deploy.yml \
  --ask-vault-pass \
  -e "app_version=1.2.0"

# Staging
ansible-playbook -i inventory/staging.ini deploy.yml \
  --ask-vault-pass \
  -e "app_version=1.2.0-rc1"
```

**Variable Best Practices:**

1. **Use group_vars for environment-specific configs**
2. **Use host_vars only when truly host-specific**
3. **Encrypt sensitive data with ansible-vault**
4. **Use defaults in templates:** `{{ variable | default('default_value') }}`
5. **Use meaningful variable names:** `db_host` not `h1`
6. **Document variables** in README or comments
7. **Keep vars files in version control** (except secrets)

This variable structure allowed us to manage 200+ servers across dev, staging, and production environments at Birlasoft with a single codebase!"

---

### 7. Ansible Roles

**Q: What are Ansible roles? How do you structure and use them?**

**Answer:**
"Roles are Ansible's way of organizing playbooks into reusable, modular components. They follow a standardized directory structure.

**Role Directory Structure:**

```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml          # Main task file
    ├── handlers/
    │   └── main.yml          # Handlers (triggered by notify)
    ├── templates/
    │   ├── nginx.conf.j2     # Jinja2 templates
    │   └── site.conf.j2
    ├── files/
    │   ├── index.html        # Static files
    │   └── ssl-cert.pem
    ├── vars/
    │   └── main.yml          # Role variables
    ├── defaults/
    │   └── main.yml          # Default variables (lowest precedence)
    ├── meta/
    │   └── main.yml          # Role dependencies
    └── README.md             # Documentation
```

**Example Role: webserver**

```yaml
# roles/webserver/defaults/main.yml
---
# Default variables (can be overridden)
http_port: 80
https_port: 443
worker_processes: auto
max_connections: 1024
ssl_enabled: false
```

```yaml
# roles/webserver/vars/main.yml
---
# Role variables (higher precedence than defaults)
nginx_user: nginx
nginx_group: nginx
config_dir: /etc/nginx
log_dir: /var/log/nginx
```

```yaml
# roles/webserver/tasks/main.yml
---
- name: Install nginx
  yum:
    name: nginx
    state: present

- name: Create required directories
  file:
    path: "{{ item }}"
    state: directory
    owner: "{{ nginx_user }}"
    group: "{{ nginx_group }}"
    mode: '0755'
  loop:
    - "{{ config_dir }}/sites-available"
    - "{{ config_dir }}/sites-enabled"
    - "{{ log_dir }}"

- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: "{{ config_dir }}/nginx.conf"
    owner: root
    group: root
    mode: '0644'
    validate: 'nginx -t -c %s'
  notify: restart nginx

- name: Deploy site config
  template:
    src: site.conf.j2
    dest: "{{ config_dir }}/sites-available/default"
    owner: root
    group: root
    mode: '0644'
  notify: reload nginx

- name: Enable site
  file:
    src: "{{ config_dir }}/sites-available/default"
    dest: "{{ config_dir }}/sites-enabled/default"
    state: link
  notify: reload nginx

- name: Start and enable nginx
  service:
    name: nginx
    state: started
    enabled: yes

- name: Configure firewall
  firewalld:
    service: "{{ item }}"
    permanent: yes
    state: enabled
    immediate: yes
  loop:
    - http
    - https
  when: ansible_os_family == "RedHat"
```

```yaml
# roles/webserver/handlers/main.yml
---
- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: reload nginx
  service:
    name: nginx
    state: reloaded

- name: test nginx config
  command: nginx -t
  changed_when: false
```

```jinja2
{# roles/webserver/templates/nginx.conf.j2 #}
user {{ nginx_user }};
worker_processes {{ worker_processes }};
error_log {{ log_dir }}/error.log;
pid /run/nginx.pid;

events {
    worker_connections {{ max_connections }};
}

http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log {{ log_dir }}/access.log main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include {{ config_dir }}/sites-enabled/*;
}
```

```yaml
# roles/webserver/meta/main.yml
---
dependencies:
  - role: common
  - role: firewall
    when: configure_firewall

galaxy_info:
  author: DevOps Team
  description: Nginx web server role
  company: Birlasoft
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: EL
      versions:
        - 7
        - 8
    - name: Ubuntu
      versions:
        - 18.04
        - 20.04
  galaxy_tags:
    - nginx
    - webserver
```

---

**Using Roles in Playbooks:**

```yaml
# Method 1: Simple role inclusion
---
- name: Setup web servers
  hosts: webservers
  roles:
    - webserver

# Method 2: With variables
---
- name: Setup web servers
  hosts: webservers
  roles:
    - role: webserver
      vars:
        http_port: 8080
        ssl_enabled: true

# Method 3: Conditional execution
---
- name: Setup servers
  hosts: all
  roles:
    - role: webserver
      when: "'webservers' in group_names"
    
    - role: database
      when: "'databases' in group_names"

# Method 4: Using include_role (dynamic)
---
- name: Setup servers
  hosts: all
  tasks:
    - name: Include webserver role
      include_role:
        name: webserver
      when: install_webserver | default(false)

# Method 5: Using import_role (static)
---
- name: Setup servers
  hosts: all
  tasks:
    - name: Import webserver role
      import_role:
        name: webserver
      when: server_type == 'web'
```

---

**Real Production Example (from Birlasoft):**

We had multiple roles for complete application stack:

```
roles/
├── common/              # Base configuration for all servers
│   ├── tasks/
│   │   └── main.yml    # NTP, DNS, users, SSH hardening
│   ├── templates/
│   └── defaults/
│
├── java/                # Java installation
│   ├── tasks/
│   │   └── main.yml    # Install OpenJDK, set JAVA_HOME
│   ├── defaults/
│   │   └── main.yml    # java_version: 11
│   └── meta/
│       └── main.yml    # Depends on: common
│
├── app-deploy/          # Application deployment
│   ├── tasks/
│   │   ├── main.yml
│   │   ├── download.yml
│   │   ├── configure.yml
│   │   └── service.yml
│   ├── templates/
│   │   ├── application.yml.j2
│   │   └── app.service.j2
│   ├── handlers/
│   │   └── main.yml
│   ├── defaults/
│   │   └── main.yml
│   └── meta/
│       └── main.yml    # Depends on: common, java
│
├── monitoring/          # Prometheus node-exporter
│   ├── tasks/
│   └── templates/
│
└── nginx-proxy/         # Nginx reverse proxy
    ├── tasks/
    ├── templates/
    └── handlers/
```

**Master playbook:**

```yaml
# site.yml
---
- name: Setup all servers
  hosts: all
  roles:
    - common

- name: Setup application servers
  hosts: appservers
  roles:
    - java
    - app-deploy
    - monitoring

- name: Setup proxy servers
  hosts: proxies
  roles:
    - nginx-proxy
    - monitoring
```

```yaml
# roles/app-deploy/tasks/main.yml
---
- name: Include download tasks
  include_tasks: download.yml

- name: Include configuration tasks
  include_tasks: configure.yml

- name: Include service tasks
  include_tasks: service.yml
```

```yaml
# roles/app-deploy/tasks/download.yml
---
- name: Create application directory
  file:
    path: "{{ app_home }}"
    state: directory
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0755'

- name: Download application artifact
  get_url:
    url: "{{ artifact_url }}/{{ app_name }}-{{ app_version }}.jar"
    dest: "{{ app_home }}/{{ app_name }}.jar"
    owner: "{{ app_user }}"
    group: "{{ app_group }}"
    mode: '0644'
  notify: restart application
```

```yaml
# roles/app-deploy/handlers/main.yml
---
- name: restart application
  systemd:
    name: "{{ app_name }}"
    state: restarted
    daemon_reload: yes

- name: reload application
  systemd:
    name: "{{ app_name }}"
    state: reloaded
```

---

**Role Dependencies (meta/main.yml):**

```yaml
# roles/app-deploy/meta/main.yml
---
dependencies:
  - role: common
  
  - role: java
    vars:
      java_version: 11
  
  - role: monitoring
    when: enable_monitoring | default(true)

galaxy_info:
  author: Harshal Kalamkar
  description: Deploy microservice applications
  company: Birlasoft
  min_ansible_version: 2.9
  platforms:
    - name: EL
      versions:
        - 7
        - 8
```

When you use `app-deploy` role, it automatically includes `common` and `java` roles first.

---

**Creating a New Role:**

```bash
# Initialize role structure
ansible-galaxy init webserver

# Creates:
webserver/
├── README.md
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml
```

---

**Ansible Galaxy (Role Repository):**

```bash
# Search for roles
ansible-galaxy search nginx

# Install role from Galaxy
ansible-galaxy install geerlingguy.nginx

# Install from requirements file
# requirements.yml:
# - src: geerlingguy.nginx
#   version: 3.1.4
# - src: geerlingguy.postgresql

ansible-galaxy install -r requirements.yml

# Install to specific path
ansible-galaxy install geerlingguy.nginx -p ./roles

# List installed roles
ansible-galaxy list
```

---

**Role Best Practices:**

1. **Keep roles focused** - One role, one purpose
2. **Use defaults** - Provide sensible defaults in `defaults/main.yml`
3. **Document** - README with variables and usage examples
4. **Test** - Include test playbook in `tests/`
5. **Version** - Tag roles in Git for versioning
6. **Dependencies** - Declare in `meta/main.yml`
7. **Idempotent** - Ensure tasks can run multiple times safely
8. **Variables** - Use role-specific variable names (prefix with role name)

**Example: Good variable naming**
```yaml
# Bad (generic, might conflict)
port: 80
user: webapp

# Good (role-specific)
nginx_http_port: 80
nginx_user: nginx
```

---

**Complete Project Structure:**

```
ansible-project/
├── inventory/
│   ├── production/
│   │   ├── hosts.ini
│   │   └── group_vars/
│   └── staging/
│       ├── hosts.ini
│       └── group_vars/
│
├── roles/
│   ├── common/
│   ├── webserver/
│   ├── database/
│   └── app-deploy/
│
├── playbooks/
│   ├── site.yml              # Master playbook
│   ├── webservers.yml        # Web server playbook
│   └── deploy.yml            # Deployment playbook
│
├── group_vars/
│   ├── all.yml
│   └── production/
│       └── vault.yml
│
├── host_vars/
│
├── files/
│
├── templates/
│
├── ansible.cfg
├── requirements.yml          # Galaxy role requirements
└── README.md
```

This role structure made our playbooks modular, reusable, and maintainable across 200+ servers at Birlasoft!"

---

(Continuing with more basic questions...)

### 8. Handlers in Ansible

**Q: What are handlers? When and how do you use them?**

**Answer:**
"Handlers are special tasks that run only when notified by other tasks and only run once at the end of a play, even if notified multiple times.

**Why Handlers?**

Without handlers:
```yaml
# BAD: Service restarted multiple times
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: restart nginx  # Triggers restart
  
  - name: Update site config
    copy:
      src: site.conf
      dest: /etc/nginx/sites-available/default
    notify: restart nginx  # Triggers restart again
  
  - name: Update SSL cert
    copy:
      src: cert.pem
      dest: /etc/nginx/ssl/cert.pem
    notify: restart nginx  # Triggers restart again
  
  # Nginx would restart 3 times!
```

With handlers:
```yaml
# GOOD: Service restarted only once at the end
tasks:
  - name: Update nginx config
    copy:
      src: nginx.conf
      dest: /etc/nginx/nginx.conf
    notify: restart nginx
  
  - name: Update site config
    copy:
      src: site.conf
      dest: /etc/nginx/sites-available/default
    notify: restart nginx
  
  - name: Update SSL cert
    copy:
      src: cert.pem
      dest: /etc/nginx/ssl/cert.pem
    notify: restart nginx

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
  
  # Nginx restarts only ONCE at the end!
```

---

**Handler Examples:**

```yaml
---
- name: Configure web server
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
    
    - name: Copy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        validate: 'nginx -t -c %s'
      notify:
        - reload nginx
        - send slack notification
    
    - name: Copy site config
      copy:
        src: default.conf
        dest: /etc/nginx/conf.d/default.conf
      notify: reload nginx
    
    - name: Ensure nginx is started
      service:
        name: nginx
        state: started
        enabled: yes
  
  handlers:
    - name: reload nginx
      service:
        name: nginx
        state: reloaded
    
    - name: restart nginx
      service:
        name: nginx
        state: restarted
    
    - name: send slack notification
      slack:
        token: "{{ slack_token }}"
        msg: "Nginx configuration updated on {{ inventory_hostname }}"
        channel: "#deployments"
```

---

**Handler Execution Order:**

```yaml
tasks:
  - name: Task 1
    debug: msg="Task 1"
    notify: handler A
  
  - name: Task 2
    debug: msg="Task 2"
    notify: handler B
  
  - name: Task 3
    debug: msg="Task 3"
    notify: handler A  # Same handler again

handlers:
  - name: handler A
    debug: msg="Handler A"
  
  - name: handler B
    debug: msg="Handler B"

# Execution:
# Task 1
# Task 2
# Task 3
# Handler A (runs once, not twice!)
# Handler B
```

---

**Force Handler Execution:**

```bash
# Handlers run even if play fails
ansible-playbook playbook.yml --force-handlers
```

```yaml
# In playbook
- name: Configure servers
  hosts: all
  force_handlers: yes  # Run handlers even on failure
  tasks:
    # ... tasks
```

---

**Flush Handlers (run immediately):**

```yaml
tasks:
  - name: Update config
    copy:
      src: app.conf
      dest: /etc/app/app.conf
    notify: restart app
  
  - name: Flush handlers now
    meta: flush_handlers  # Run pending handlers immediately
  
  - name: Run tests
    command: /opt/app/test.sh
    # This runs AFTER app has been restarted
```

---

**Listen (Multiple tasks trigger same handler):**

```yaml
tasks:
  - name: Update nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart web services
  
  - name: Update php-fpm config
    template:
      src: php-fpm.conf.j2
      dest: /etc/php-fpm.conf
    notify: restart web services

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
    listen: restart web services
  
  - name: restart php-fpm
    service:
      name: php-fpm
      state: restarted
    listen: restart web services
  
  # Both handlers run when notified with "restart web services"
```

---

**Real Production Example (Birlasoft):**

```yaml
# deploy-app.yml
---
- name: Deploy microservice
  hosts: appservers
  become: yes
  
  vars:
    app_name: user-service
    app_version: "1.2.0"
  
  tasks:
    - name: Stop application
      systemd:
        name: "{{ app_name }}"
        state: stopped
    
    - name: Download new version
      get_url:
        url: "https://artifactory/{{ app_name }}-{{ app_version }}.jar"
        dest: "/opt/{{ app_name }}/{{ app_name }}.jar"
      notify:
        - restart application
        - clear cache
        - notify team
    
    - name: Update application config
      template:
        src: application.yml.j2
        dest: "/opt/{{ app_name }}/application.yml"
      notify: restart application
    
    - name: Update log config
      template:
        src: logback.xml.j2
        dest: "/opt/{{ app_name }}/logback.xml"
      notify: restart application
    
    - name: Update systemd service
      template:
        src: app.service.j2
        dest: "/etc/systemd/system/{{ app_name }}.service"
      notify:
        - reload systemd
        - restart application
  
  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes
    
    - name: restart application
      systemd:
        name: "{{ app_name }}"
        state: restarted
    
    - name: wait for app
      wait_for:
        port: 8080
        delay: 10
        timeout: 120
      listen: restart application
    
    - name: health check
      uri:
        url: http://localhost:8080/actuator/health
        status_code: 200
      retries: 5
      delay: 10
      listen: restart application
    
    - name: clear cache
      command: redis-cli FLUSHALL
      delegate_to: "{{ redis_host }}"
    
    - name: notify team
      slack:
        token: "{{ slack_token }}"
        channel: "#deployments"
        msg: |
          ✅ {{ app_name }} v{{ app_version }} deployed to {{ inventory_hostname }}
          Environment: {{ app_env }}
```

**Execution flow:**
1. All tasks run
2. If any task with `notify` reports "changed"
3. Handlers run once at the end
4. In order they're defined (not notified)

This prevents service restarts during config updates and ensures the service restarts only once after all changes!"

---

(Due to length, I'll continue with advanced/hardcore questions in the next section...)

## SECTION 2: HARDCORE/CHALLENGING QUESTIONS

### 1. AWX/Ansible Tower Administration

**Q: You deployed AWX on Kubernetes at Birlasoft. Walk me through the complete setup, challenges you faced, and how you managed it in production.**

**Answer:**
"This was one of my key projects at Birlasoft. Let me walk you through the complete setup:

**Architecture:**

```
┌─────────────────────────────────────────────────┐
│           Kubernetes Cluster                    │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─────────────────────────────────────┐       │
│  │     AWX Namespace                   │       │
│  ├─────────────────────────────────────┤       │
│  │                                     │       │
│  │  ┌──────────┐  ┌────────────┐      │       │
│  │  │ AWX Web  │  │ AWX Task   │      │       │
│  │  │   Pod    │  │    Pod     │      │       │
│  │  └────┬─────┘  └─────┬──────┘      │       │
│  │       │              │             │       │
│  │  ┌────▼──────────────▼──────┐      │       │
│  │  │   PostgreSQL Pod         │      │       │
│  │  │   (StatefulSet)          │      │       │
│  │  └──────────────────────────┘      │       │
│  │                                     │       │
│  │  ┌──────────────────────────┐      │       │
│  │  │   Redis Pod              │      │       │
│  │  │   (for task queue)       │      │       │
│  │  └──────────────────────────┘      │       │
│  │                                     │       │
│  │  ┌──────────────────────────┐      │       │
│  │  │   PersistentVolumes      │      │       │
│  │  │   - PostgreSQL data      │      │       │
│  │  │   - Project files        │      │       │
│  │  │   - Job outputs          │      │       │
│  │  └──────────────────────────┘      │       │
│  └─────────────────────────────────────┘       │
│                                                 │
│  ┌─────────────────────────────────────┐       │
│  │   Ingress Controller                │       │
│  │   awx.company.com → AWX Web         │       │
│  └─────────────────────────────────────┘       │
└─────────────────────────────────────────────────┘
```

---

**Deployment Process:**

**1. Prerequisites:**
```bash
# Kubernetes cluster (we used on-prem cluster)
# kubectl configured
# Storage class for dynamic PV provisioning

# Install AWX Operator
kubectl apply -f https://raw.githubusercontent.com/ansible/awx-operator/2.7.2/deploy/awx-operator.yaml

# Verify operator is running
kubectl get pods -n awx
```

**2. AWX Instance CR:**
```yaml
# awx-instance.yml
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
  namespace: awx
spec:
  # Service type
  service_type: ClusterIP
  
  # Ingress configuration
  ingress_type: ingress
  ingress_tls_secret: awx-tls-secret
  hostname: awx.company.com
  
  # PostgreSQL configuration
  postgres_storage_class: fast-ssd
  postgres_storage_requirements:
    requests:
      storage: 50Gi
  postgres_resource_requirements:
    requests:
      memory: "4Gi"
      cpu: "2000m"
    limits:
      memory: "8Gi"
      cpu: "4000m"
  
  # AWX Web
  web_resource_requirements:
    requests:
      memory: "2Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "2000m"
  
  # AWX Task
  task_resource_requirements:
    requests:
      memory: "2Gi"
      cpu: "1000m"
    limits:
      memory: "4Gi"
      cpu: "2000m"
  
  # Projects storage
  projects_persistence: true
  projects_storage_class: fast-ssd
  projects_storage_size: 20Gi
  
  # Admin credentials secret
  admin_user: admin
  admin_password_secret: awx-admin-password
  
  # Enable metrics
  metrics_utility_enabled: true
  
  # Auto-upgrade
  auto_upgrade: false
```

**3. Create Secrets:**
```bash
# Admin password
kubectl create secret generic awx-admin-password \
  --from-literal=password='SuperSecretPassword123!' \
  -n awx

# TLS certificate
kubectl create secret tls awx-tls-secret \
  --cert=awx.crt \
  --key=awx.key \
  -n awx
```

**4. Deploy AWX:**
```bash
kubectl apply -f awx-instance.yml

# Monitor deployment
kubectl logs -f deployments/awx-operator-controller-manager -n awx

# Wait for all pods to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=awx -n awx --timeout=600s
```

---

**Configuration & Setup:**

**1. Initial Login:**
```bash
# Get admin password
kubectl get secret awx-admin-password -n awx -o jsonpath='{.data.password}' | base64 -d

# Access: https://awx.company.com
# Login: admin / <password>
```

**2. Organization Structure:**
```
Organization: Birlasoft
├── Teams
│   ├── DevOps Team (Admin)
│   ├── Development Team (Execute)
│   └── QA Team (Execute)
│
├── Users
│   ├── harshal.kalamkar (Admin)
│   ├── dev-user-1 (Normal User)
│   └── qa-user-1 (Normal User)
│
├── Projects (Git repositories)
│   ├── ansible-playbooks (main playbooks)
│   ├── ansible-roles (custom roles)
│   └── ansible-inventories (inventory sources)
│
├── Inventories
│   ├── Production (VMware + AWS)
│   ├── Staging (VMware + AWS)
│   └── Development (VMware)
│
├── Credentials
│   ├── SSH Keys (for managed nodes)
│   ├── VMware vCenter
│   ├── AWS Access Keys
│   ├── Git Credentials
│   └── Vault Passwords
│
└── Job Templates
    ├── Deploy Application
    ├── Patch Servers
    ├── Backup Databases
    └── Security Hardening
```

**3. Setting up Dynamic Inventory:**

```yaml
# In AWX UI:
# Inventory: Production
# Sources:
# 1. VMware vCenter

Name: VMware Production
Source: VMware vCenter
Credential: vCenter-Production
Variables:
  validate_certs: false
  with_tags: true
  keyed_groups:
    - key: config.guestId
      prefix: os
    - key: summary.runtime.powerState
      prefix: state
    - key: customValue
      prefix: app
  
Update Options:
  ✓ Overwrite
  ✓ Overwrite Variables
  ✓ Update on Launch
  
Update on Project Update: No
Cache Timeout: 0
Verbosity: 1

# 2. AWS EC2

Name: AWS Production
Source: Amazon EC2
Credential: AWS-Production
Regions: us-east-1, us-west-2
Instance Filters: tag:Environment=production
Variables:
  keyed_groups:
    - key: tags.Application
      prefix: app
    - key: tags.Environment
      prefix: env
    - key: placement.availability_zone
      prefix: az
  compose:
    ansible_host: public_ip_address
```

**4. Job Template Example:**

```yaml
Name: Deploy Microservice
Job Type: Run
Inventory: Production
Project: ansible-playbooks
Playbook: playbooks/deploy-app.yml
Credentials:
  - SSH-Production-Key
  - AWS-Production (for fetching from S3)
  - Ansible-Vault-Password

Variables:
  app_name: user-service
  app_version: "{{ version }}"  # Survey variable
  environment: production

Options:
  ✓ Prompt on launch: Limit
  ✓ Prompt on launch: Extra Variables
  ✓ Enable Concurrent Jobs
  ✓ Enable Fact Storage
  
Survey:
  - Name: Application Version
    Variable: version
    Type: Text
    Required: Yes
    Default: 1.0.0
  
  - Name: Target Servers
    Variable: target_servers
    Type: Multiple Choice
    Choices:
      - All Servers
      - app-server-1
      - app-server-2
    Required: Yes

Notifications:
  On Success: Slack-Deployments
  On Failure: Slack-Alerts, PagerDuty
```

---

**Challenges & Solutions:**

**Challenge 1: Persistent Storage Issues**

**Problem:**
- PostgreSQL pod kept crashing
- Data loss on pod restarts

**Root Cause:**
- PersistentVolume using slow network storage
- Database couldn't handle I/O load

**Solution:**
```yaml
# Changed storage class to local SSD
postgres_storage_class: local-ssd-fast

# Added resource limits
postgres_resource_requirements:
  requests:
    memory: "8Gi"  # Increased from 4Gi
    cpu: "4000m"   # Increased from 2000m

# Enabled PostgreSQL performance tuning
postgres_configuration_secret: postgres-tuning-config
```

```yaml
# postgres-config.yml (Secret)
apiVersion: v1
kind: Secret
metadata:
  name: postgres-tuning-config
  namespace: awx
stringData:
  postgresql.conf: |
    max_connections = 200
    shared_buffers = 2GB
    effective_cache_size = 6GB
    work_mem = 10MB
    maintenance_work_mem = 512MB
    checkpoint_completion_target = 0.9
    wal_buffers = 16MB
    default_statistics_target = 100
    random_page_cost = 1.1
```

---

**Challenge 2: Job Execution Slowness**

**Problem:**
- Jobs taking 2-3x longer than command line
- Task pod CPU throttling

**Root Cause:**
- Insufficient task pod resources
- Too many concurrent jobs
- Inventory sync blocking job execution

**Solution:**
```yaml
# Scaled task pods
task_replicas: 3  # Instead of 1

# Increased resources
task_resource_requirements:
  requests:
    memory: "4Gi"
    cpu: "2000m"
  limits:
    memory: "8Gi"
    cpu: "4000m"

# Limited concurrent jobs per pod
task_resource_requirements:
  limits:
    storage: 10Gi

# Settings in AWX UI
Settings → Jobs:
  Maximum Concurrent Jobs: 15 (was 50)
  Maximum Active Jobs Per Instance: 5
  Job Event Buffer Size: 100000
```

```bash
# Also added node affinity to schedule task pods on high-CPU nodes
kubectl patch awx awx -n awx --type merge -p '
spec:
  task_args:
    - "--node-selector"
    - "node-type=compute-optimized"
'
```

---

**Challenge 3: Credential Management**

**Problem:**
- Team members sharing credentials
- No audit trail for credential access
- Credentials hardcoded in playbooks

**Solution:**

1. **RBAC Configuration:**
```
Team: DevOps (Admin)
  - Can create/edit all credentials
  - Can see all playbooks

Team: Development (Execute)
  - Can use staging credentials
  - Cannot see credential values
  - Can only run specific job templates

Team: QA (Execute)
  - Can use QA credentials
  - Read-only access to playbooks
```

2. **Credential Types:**
```yaml
# Custom Credential Type: Artifactory
Name: Artifactory
Input Configuration:
  fields:
    - id: artifactory_url
      type: string
      label: Artifactory URL
    - id: artifactory_user
      type: string
      label: Username
    - id: artifactory_token
      type: string
      label: API Token
      secret: true

Injector Configuration:
  env:
    ARTIFACTORY_URL: "{{ artifactory_url }}"
    ARTIFACTORY_USER: "{{ artifactory_user }}"
    ARTIFACTORY_TOKEN: "{{ artifactory_token }}"
```

3. **External Secrets (HashiCorp Vault integration):**
```yaml
# AWX retrieves secrets from Vault at runtime
Credential Type: HashiCorp Vault Signed SSH
Vault Server URL: https://vault.company.com
Vault Token: <service-token>
Path: secret/data/ssh/production
```

---

**Challenge 4: Multi-Environment Management**

**Problem:**
- Accidentally running prod playbooks in staging
- Variable conflicts between environments

**Solution:**

1. **Color-coded Environments:**
```yaml
# In AWX UI settings
Settings → UI:
  Custom Logo
  Custom Login Info:
    "🔴 PRODUCTION - Exercise Caution!"
  
# Separate organizations for critical environments
Organization: Production (Red theme)
Organization: Non-Production (Blue theme)
```

2. **Approval Workflow:**
```yaml
# Production deployments require approval
Workflow Template: Deploy to Production
  
  Step 1: Deploy to Staging
    ├─ Success → Manual Approval
    └─ Failure → Notify Team
  
  Step 2: Approval Node
    ├─ Approved → Deploy to Production
    └─ Denied → Stop
  
  Step 3: Deploy to Production
    ├─ Success → Notify Success
    └─ Failure → Rollback

Approvers: DevOps Lead, Engineering Manager
Timeout: 2 hours
```

3. **Instance Groups:**
```yaml
# Separate execution environments
Instance Group: Production-Execution
  - Runs only production jobs
  - Has access to production network
  - Higher resource allocation

Instance Group: Non-Production-Execution
  - Runs dev/staging jobs
  - Isolated from production network
  - Standard resources
```

---

**Challenge 5: Logging & Debugging**

**Problem:**
- Difficult to debug failed jobs
- No centralized logging
- Job outputs too verbose

**Solution:**

1. **Centralized Logging:**
```yaml
# Configured ELK stack integration
Settings → System:
  Logging Aggregator: Enabled
  Logging Aggregator Type: logstash
  Logging Aggregator Host: logstash.company.com
  Logging Aggregator Port: 5044
  
  Log Format:
    json
  
  Logging Aggregator Level: INFO
```

2. **Job Output Control:**
```yaml
# In playbooks
- name: Task with controlled output
  command: /opt/scripts/long-running.sh
  no_log: true  # Don't log output
  register: result

- name: Show summary only
  debug:
    msg: "Task completed with {{ result.stdout_lines | length }} lines of output"
```

3. **Fact Caching:**
```yaml
Settings → Jobs:
  Enable Fact Cache: Yes
  Per-Host Ansible Fact Cache Timeout: 3600

# This speeds up playbooks by caching facts
# Reduces gather_facts time from 30s to <1s
```

---

**Production Metrics (After Optimization):**

```yaml
Infrastructure:
  - AWX Pods: 7 (2 web, 3 task, 1 postgres, 1 redis)
  - CPU Usage: ~40% average, 80% peak
  - Memory Usage: ~60% average
  - Storage: 120GB (postgres 50GB, projects 20GB, job outputs 50GB)

Job Statistics (Monthly):
  - Total Jobs: ~2,500
  - Successful: 96.5%
  - Failed: 2.5%
  - Cancelled: 1%
  - Average Duration: 8 minutes
  - Concurrent Jobs (peak): 12

Managed Infrastructure:
  - Inventories: 3 (prod, staging, dev)
  - Total Hosts: 250+
  - Dynamic Inventory Sources: 4 (2x VMware, 2x AWS)
  - Job Templates: 35
  - Workflow Templates: 12
  - Projects: 8
  - Credentials: 50+
```

---

**Backup & DR:**

```bash
# Automated backup script (runs daily)
#!/bin/bash
# backup-awx.sh

NAMESPACE="awx"
BACKUP_DIR="/mnt/backups/awx"
DATE=$(date +%Y%m%d-%H%M%S)

# Backup PostgreSQL
kubectl exec -n $NAMESPACE awx-postgres-0 -- \
  pg_dumpall -U awx > $BACKUP_DIR/postgres-$DATE.sql

# Backup AWX configuration
kubectl get -n $NAMESPACE awx -o yaml > $BACKUP_DIR/awx-config-$DATE.yaml
kubectl get -n $NAMESPACE secrets -o yaml > $BACKUP_DIR/secrets-$DATE.yaml
kubectl get -n $NAMESPACE configmaps -o yaml > $BACKUP_DIR/configmaps-$DATE.yaml

# Backup PVCs
kubectl exec -n $NAMESPACE awx-postgres-0 -- tar czf - /var/lib/postgresql/data \
  > $BACKUP_DIR/postgres-data-$DATE.tar.gz

# Upload to S3
aws s3 cp $BACKUP_DIR/ s3://company-backups/awx/$DATE/ --recursive

# Cleanup old backups (keep 30 days)
find $BACKUP_DIR -mtime +30 -delete
```

This AWX deployment has been running production workloads for 18+ months at Birlasoft, managing 250+ servers across multiple environments with 99.8% uptime!"

---

### 2. Complex Ansible Scenario

**Q: You need to perform a zero-downtime deployment of a microservice across 100 servers behind a load balancer. 5 servers should be updated at a time, health-checked, and only then move to next batch. If any batch fails, rollback all changes. How would you implement this with Ansible?**

**Answer:**
"This is a classic rolling deployment with automated rollback. Here's my complete solution:

```yaml
# deploy-rolling-with-rollback.yml
---
- name: Zero-downtime rolling deployment with rollback
  hosts: appservers
  serial: 5  # Update 5 servers at a time
  max_fail_percentage: 0  # Fail entire deployment if any server fails
  
  vars:
    app_name: user-service
    new_version: "{{ version }}"
    old_version: "{{ current_version | default('unknown') }}"
    lb_backend_group: "app-{{ app_name }}"
    health_check_url: "http://localhost:8080/actuator/health"
    health_check_retries: 10
    health_check_delay: 10
    
  tasks:
    # Pre-flight checks
    - name: Get current version before deployment
      shell: cat /opt/{{ app_name }}/VERSION
      register: version_check
      failed_when: false
      changed_when: false
      tags: preflight
    
    - name: Set current version fact
      set_fact:
        current_version: "{{ version_check.stdout | default('unknown') }}"
      tags: preflight
    
    - name: Verify sufficient disk space
      assert:
        that:
          - item.size_available > 1000000000  # 1GB free
        fail_msg: "Insufficient disk space on {{ item.mount }}"
      loop: "{{ ansible_mounts }}"
      when: item.mount == '/'
      tags: preflight
    
    - name: Create backup directory
      file:
        path: /opt/{{ app_name }}/backups/{{ ansible_date_time.epoch }}
        state: directory
        owner: "{{ app_user }}"
        mode: '0755'
      tags: backup
    
    # Backup current version
    - name: Backup current application
      copy:
        src: /opt/{{ app_name }}/{{ app_name }}.jar
        dest: /opt/{{ app_name }}/backups/{{ ansible_date_time.epoch }}/{{ app_name }}.jar
        remote_src: yes
      tags: backup
    
    - name: Backup current config
      copy:
        src: /opt/{{ app_name }}/application.yml
        dest: /opt/{{ app_name }}/backups/{{ ansible_date_time.epoch }}/application.yml
        remote_src: yes
      tags: backup
    
    # Remove from load balancer
    - name: Remove server from load balancer
      uri:
        url: "http://{{ load_balancer_host }}:{{ load_balancer_port }}/api/backend/{{ lb_backend_group }}/servers/{{ ansible_default_ipv4.address }}"
        method: DELETE
        status_code: [200, 204]
        headers:
          Authorization: "Bearer {{ lb_api_token }}"
      delegate_to: localhost
      tags: lb
    
    - name: Wait for active connections to drain
      wait_for:
        timeout: 30
      tags: lb
    
    - name: Verify server removed from LB
      uri:
        url: "http://{{ load_balancer_host }}:{{ load_balancer_port }}/api/backend/{{ lb_backend_group }}/servers"
        method: GET
        return_content: yes
      register: lb_servers
      delegate_to: localhost
      failed_when: ansible_default_ipv4.address in lb_servers.content
      retries: 3
      delay: 5
      tags: lb
    
    # Stop application
    - name: Stop application service
      systemd:
        name: "{{ app_name }}"
        state: stopped
      tags: deploy
    
    # Deploy new version
    - name: Download new application version
      get_url:
        url: "{{ artifactory_url }}/{{ app_name }}-{{ new_version }}.jar"
        dest: /opt/{{ app_name }}/{{ app_name }}.jar.new
        username: "{{ artifactory_user }}"
        password: "{{ artifactory_password }}"
        mode: '0644'
        timeout: 300
      tags: deploy
    
    - name: Verify checksum
      stat:
        path: /opt/{{ app_name }}/{{ app_name }}.jar.new
        checksum_algorithm: sha256
      register: new_jar_stat
      tags: deploy
    
    - name: Fail if checksum mismatch
      fail:
        msg: "Checksum mismatch! Expected: {{ expected_checksum }}, Got: {{ new_jar_stat.stat.checksum }}"
      when: new_jar_stat.stat.checksum != expected_checksum
      tags: deploy
    
    - name: Move new JAR to production location
      command: mv /opt/{{ app_name }}/{{ app_name }}.jar.new /opt/{{ app_name }}/{{ app_name }}.jar
      tags: deploy
    
    - name: Update application config
      template:
        src: templates/application.yml.j2
        dest: /opt/{{ app_name }}/application.yml
        owner: "{{ app_user }}"
        mode: '0600'
        backup: yes
      tags: deploy
    
    - name: Save version file
      copy:
        content: "{{ new_version }}"
        dest: /opt/{{ app_name }}/VERSION
        owner: "{{ app_user }}"
        mode: '0644'
      tags: deploy
    
    # Start application
    - name: Start application service
      systemd:
        name: "{{ app_name }}"
        state: started
      tags: deploy
    
    - name: Wait for application port
      wait_for:
        port: 8080
        delay: 10
        timeout: 120
        state: started
      tags: healthcheck
    
    # Health check
    - name: Perform health check
      uri:
        url: "{{ health_check_url }}"
        status_code: 200
        return_content: yes
      register: health_check
      retries: "{{ health_check_retries }}"
      delay: "{{ health_check_delay }}"
      until: health_check.status == 200
      tags: healthcheck
    
    - name: Verify health response
      assert:
        that:
          - health_check.json.status == "UP"
        fail_msg: "Health check failed: {{ health_check.json }}"
      tags: healthcheck
    
    # Smoke tests
    - name: Run smoke tests
      uri:
        url: "http://localhost:8080{{ item }}"
        status_code: 200
      loop:
        - /api/users/1
        - /api/health/db
        - /api/health/cache
      tags: smoke_test
    
    # Add back to load balancer
    - name: Add server back to load balancer
      uri:
        url: "http://{{ load_balancer_host }}:{{ load_balancer_port }}/api/backend/{{ lb_backend_group }}/servers"
        method: POST
        body_format: json
        body:
          address: "{{ ansible_default_ipv4.address }}"
          port: 8080
          weight: 1
          max_fails: 3
        status_code: [200, 201]
        headers:
          Authorization: "Bearer {{ lb_api_token }}"
      delegate_to: localhost
      tags: lb
    
    - name: Wait for LB health check to pass
      wait_for:
        timeout: 20
      tags: lb
    
    - name: Verify server is healthy in LB
      uri:
        url: "http://{{ load_balancer_host }}:{{ load_balancer_port }}/api/backend/{{ lb_backend_group }}/servers/{{ ansible_default_ipv4.address }}/stats"
        method: GET
        return_content: yes
      register: lb_health
      delegate_to: localhost
      failed_when: lb_health.json.status != 'healthy'
      retries: 5
      delay: 10
      tags: lb
    
    - name: Log successful deployment
      lineinfile:
        path: /var/log/deployments.log
        line: "{{ ansible_date_time.iso8601 }} - Successfully deployed {{ app_name }} {{ new_version }} (previous: {{ current_version }})"
        create: yes
      tags: logging

  rescue:
    # Rollback on failure
    - name: ROLLBACK - Remove failed server from LB
      uri:
        url: "http://{{ load_balancer_host }}:{{ load_balancer_port }}/api/backend/{{ lb_backend_group }}/servers/{{ ansible_default_ipv4.address }}"
        method: DELETE
        status_code: [200, 204, 404]
        headers:
          Authorization: "Bearer {{ lb_api_token }}"
      delegate_to: localhost
      ignore_errors: yes
      tags: rollback
    
    - name: ROLLBACK - Stop failed application
      systemd:
        name: "{{ app_name }}"
        state: stopped
      ignore_errors: yes
      tags: rollback
    
    - name: ROLLBACK - Restore previous JAR
      copy:
        src: /opt/{{ app_name }}/backups/{{ ansible_date_time.epoch }}/{{ app_name }}.jar
        dest: /opt/{{ app_name }}/{{ app_name }}.jar
        remote_src: yes
      tags: rollback
    
    - name: ROLLBACK - Restore previous config
      copy:
        src: /opt/{{ app_name }}/backups/{{ ansible_date_time.epoch }}/application.yml
        dest: /opt/{{ app_name }}/application.yml
        remote_src: yes
      tags: rollback
    
    - name: ROLLBACK - Restore version file
      copy:
        content: "{{ current_version }}"
        dest: /opt/{{ app_name }}/VERSION
      tags: rollback
    
    - name: ROLLBACK - Start application with old version
      systemd:
        name: "{{ app_name }}"
        state: started
      tags: rollback
    
    - name: ROLLBACK - Wait for port
      wait_for:
        port: 8080
        delay: 10
        timeout: 120
        state: started
      tags: rollback
    
    - name: ROLLBACK - Health check
      uri:
        url: "{{ health_check_url }}"
        status_code: 200
      retries: 10
      delay: 10
      tags: rollback
    
    - name: ROLLBACK - Add back to LB
      uri:
        url: "http://{{ load_balancer_host }}:{{ load_balancer_port }}/api/backend/{{ lb_backend_group }}/servers"
        method: POST
        body_format: json
        body:
          address: "{{ ansible_default_ipv4.address }}"
          port: 8080
          weight: 1
        status_code: [200, 201]
        headers:
          Authorization: "Bearer {{ lb_api_token }}"
      delegate_to: localhost
      tags: rollback
    
    - name: ROLLBACK - Log rollback
      lineinfile:
        path: /var/log/deployments.log
        line: "{{ ansible_date_time.iso8601 }} - ROLLBACK: Failed to deploy {{ app_name }} {{ new_version }}, restored {{ current_version }}"
        create: yes
      tags: rollback
    
    - name: ROLLBACK - Send alert
      slack:
        token: "{{ slack_token }}"
        channel: "#alerts"
        msg: |
          🚨 DEPLOYMENT FAILED - ROLLED BACK
          Service: {{ app_name }}
          Server: {{ inventory_hostname }}
          Failed Version: {{ new_version }}
          Restored Version: {{ current_version }}
          Error: {{ ansible_failed_result.msg }}
      delegate_to: localhost
      tags: rollback
    
    - name: ROLLBACK - Fail playbook
      fail:
        msg: "Deployment failed and rolled back on {{ inventory_hostname }}"

# Post-deployment verification
- name: Post-deployment verification
  hosts: appservers
  gather_facts: no
  
  tasks:
    - name: Verify all servers are at new version
      command: cat /opt/{{ app_name }}/VERSION
      register: deployed_version
      changed_when: false
    
    - name: Assert version consistency
      assert:
        that:
          - deployed_version.stdout == new_version
        fail_msg: "Version mismatch! Expected {{ new_version }}, got {{ deployed_version.stdout }}"
    
    - name: Final health check across all servers
      uri:
        url: "{{ health_check_url }}"
        status_code: 200
      register: final_health
    
    - name: Deployment summary
      debug:
        msg: |
          ✅ DEPLOYMENT SUCCESSFUL
          Service: {{ app_name }}
          Version: {{ new_version }}
          Servers Updated: {{ ansible_play_hosts | length }}
          Duration: {{ ansible_play_duration }}

# Notifications
- name: Send deployment notifications
  hosts: localhost
  gather_facts: no
  
  tasks:
    - name: Send success notification
      slack:
        token: "{{ slack_token }}"
        channel: "#deployments"
        msg: |
          ✅ DEPLOYMENT SUCCESSFUL
          Service: {{ app_name }}
          Version: {{ new_version }}
          Servers: {{ groups['appservers'] | length }}
          Duration: {{ ansible_play_duration }}
          
          Deployed to: {{ groups['appservers'] | join(', ') }}
      when: deployment_failed is not defined
```

**Key Features:**

1. **Serial Execution**: `serial: 5` - only 5 servers at a time
2. **Zero Fail Tolerance**: `max_fail_percentage: 0` - any failure aborts
3. **Load Balancer Integration**: Removes/adds servers via API
4. **Health Checks**: Multiple levels (port, HTTP, smoke tests)
5. **Automated Rollback**: rescue block restores previous version
6. **Version Tracking**: Saves version file for future reference
7. **Comprehensive Logging**: Tracks all deployments
8. **Notifications**: Slack alerts on success/failure

**Usage:**
```bash
# Deploy new version
ansible-playbook deploy-rolling-with-rollback.yml \
  -e "version=1.2.0" \
  -e "expected_checksum=abc123..." \
  --limit "appservers"

# Dry run
ansible-playbook deploy-rolling-with-rollback.yml \
  -e "version=1.2.0" \
  --check --diff

# Deploy to specific batch first (canary)
ansible-playbook deploy-rolling-with-rollback.yml \
  -e "version=1.2.0" \
  --limit "app-server-[1:5]"

# Then to all
ansible-playbook deploy-rolling-with-rollback.yml \
  -e "version=1.2.0"
```

This approach has successfully deployed 100+ applications at Birlasoft with zero production incidents!"

---

(Continuing with final questions...)

### 3. Ansible Performance & Optimization

**Q: Your Ansible playbooks are taking too long to run. A playbook that should take 10 minutes is taking 45 minutes. How do you troubleshoot and optimize this?**

**Answer:**
"I've faced this exact scenario. Here's my systematic troubleshooting approach:

**Step 1: Enable Profiling**

```ini
# ansible.cfg
[defaults]
callback_whitelist = profile_tasks, timer

# Shows time taken by each task
# Plus total playbook execution time
```

```bash
# Run with timing
ansible-playbook slow-playbook.yml

# Output shows:
# TASK [Install packages] ******* 35.2s
# TASK [Copy files] ************ 120.5s  ← Problem!
# TASK [Start service] ********** 2.1s
```

---

**Common Issues & Solutions:**

**Issue 1: Slow Fact Gathering**

```yaml
# Problem: gather_facts on 100 hosts = 5 minutes
# Facts gathered on EVERY play

# Solution 1: Disable when not needed
- name: Simple task
  hosts: all
  gather_facts: no  # Skip if facts not used
  tasks:
    - command: /some/command

# Solution 2: Gather once, cache facts
# ansible.cfg:
[defaults]
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600

# Solution 3: Gather specific facts only
- name: Need only network facts
  hosts: all
  gather_facts: yes
  gather_subset:
    - '!all'
    - '!min'
    - network  # Only network facts
```

**Improvement: 5 minutes → 30 seconds**

---

**Issue 2: Serial Copy to Many Hosts**

```yaml
# SLOW: Copy file to 100 hosts serially
- name: Copy large file
  copy:
    src: files/app.jar  # 500MB file
    dest: /opt/app/app.jar
  # Takes 500MB × 100 hosts = SLOW!

# SOLUTION 1: Download from central location
- name: Download from artifact server
  get_url:
    url: https://artifactory/app.jar
    dest: /opt/app/app.jar
    # All hosts download in parallel
    # Network bandwidth utilized efficiently

# SOLUTION 2: Use synchronize (rsync)
- name: Sync files efficiently
  synchronize:
    src: files/app.jar
    dest: /opt/app/app.jar
    compress: yes
    # Uses rsync - much faster than copy

# SOLUTION 3: Distribute via CDN/S3
- name: Download from S3
  aws_s3:
    bucket: my-artifacts
    object: app.jar
    dest: /opt/app/app.jar
    mode: get
```

**Improvement: 45 minutes → 3 minutes**

---

**Issue 3: SSH Connection Overhead**

```ini
# Problem: New SSH connection for EVERY task
# 100 hosts × 50 tasks = 5000 SSH connections!

# SOLUTION: Enable SSH pipelining and ControlPersist
# ansible.cfg:
[defaults]
pipelining = True

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
```

**How it works:**
```
Without ControlPersist:
  Task 1: Connect → Execute → Disconnect
  Task 2: Connect → Execute → Disconnect
  Task 3: Connect → Execute → Disconnect

With ControlPersist:
  Task 1: Connect → Execute (keep alive 60s)
  Task 2: Reuse connection → Execute
  Task 3: Reuse connection → Execute
  (Disconnect after 60s idle)
```

**Improvement: 20 minutes → 8 minutes**

---

**Issue 4: Too Many forks (Paradoxically)**

```ini
# Problem: forks = 100, but overwhelming network
# ansible.cfg:
[defaults]
forks = 100  # Default is 5

# CPU spikes, network saturated
# Some tasks timeout

# SOLUTION: Tune based on your environment
forks = 20  # Sweet spot for most setups

# For cloud with good network:
forks = 50

# For slow network / many hosts:
forks = 10
```

**Test different values:**
```bash
time ansible-playbook -f 5 playbook.yml   # 15min
time ansible-playbook -f 10 playbook.yml  # 10min
time ansible-playbook -f 20 playbook.yml  # 8min  ← Best
time ansible-playbook -f 50 playbook.yml  # 12min (too many)
```

---

**Issue 5: Loops with Remote Operations**

```yaml
# SLOW: Loop calls apt for each package
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - postgresql
    - redis
    - git
    - curl
    # 5 separate apt operations!

# FAST: Single apt operation
- name: Install packages
  apt:
    name:
      - nginx
      - postgresql
      - redis
      - git
      - curl
    state: present
    # 1 apt operation!
```

**Improvement: 2 minutes → 20 seconds**

---

**Issue 6: Unnecessary Task Execution**

```yaml
# SLOW: Always runs even if already done
- name: Download artifact
  get_url:
    url: https://artifactory/app-1.0.0.jar
    dest: /opt/app/app.jar
  # Downloads every time!

# FAST: Skip if already correct version
- name: Check current version
  stat:
    path: /opt/app/app.jar
    checksum_algorithm: sha256
  register: current_jar

- name: Download artifact only if needed
  get_url:
    url: https://artifactory/app-1.0.0.jar
    dest: /opt/app/app.jar
    checksum: sha256:abc123...
  when: >
    current_jar.stat.checksum is not defined or
    current_jar.stat.checksum != 'abc123...'
```

---

**Issue 7: Debug Output Overhead**

```yaml
# SLOW: Debug outputs large variables
- name: Get file contents
  slurp:
    src: /var/log/huge.log  # 100MB log file
  register: log_content

- name: Show log
  debug:
    var: log_content  # Prints 100MB to console!
  # Takes forever, clutters output

# FAST: Only show what's needed
- name: Show log summary
  debug:
    msg: "Log size: {{ log_content.content | b64decode | length }} bytes"
```

---

**Issue 8: Serial Package Updates**

```yaml
# SLOW: Update packages one by one
- name: Update package cache
  apt:
    update_cache: yes
  # Runs on all 100 hosts in parallel - OK

- name: Upgrade packages
  apt:
    upgrade: dist
  # Each host downloads updates serially
  # 100 hosts × 500MB updates = SLOW

# SOLUTION: Use local package mirror/cache
# Set up apt-cacher-ng or Squid proxy

# On proxy server:
apt-get install apt-cacher-ng

# In playbook:
- name: Configure apt proxy
  lineinfile:
    path: /etc/apt/apt.conf.d/02proxy
    line: 'Acquire::http::Proxy "http://apt-cache.company.com:3142";'
    create: yes

- name: Update with cached packages
  apt:
    update_cache: yes
    upgrade: dist
  # First host downloads, rest use cache
```

**Improvement: 30 minutes → 5 minutes**

---

**Issue 9: No Parallelization**

```yaml
# SLOW: Serial deployment
- name: Deploy to servers
  hosts: appservers
  serial: 1  # One at a time!
  tasks:
    - include_tasks: deploy-app.yml

# FAST: Parallel with safety
- name: Deploy to servers
  hosts: appservers
  serial: "30%"  # 30% of hosts at a time
  max_fail_percentage: 10  # Stop if >10% fail
  tasks:
    - include_tasks: deploy-app.yml

# Or use strategy:
- name: Deploy to servers
  hosts: appservers
  strategy: free  # Don't wait for slowest host
  tasks:
    - include_tasks: deploy-app.yml
```

---

**Issue 10: Inefficient Templates**

```yaml
# SLOW: Complex Jinja2 processing
- name: Generate config
  template:
    src: complex.j2
    dest: /etc/app/config.xml

# complex.j2:
{% for host in groups['all'] %}
  {% for fact in hostvars[host] %}
    {{ fact }}  # Processes ALL facts for ALL hosts!
  {% endfor %}
{% endfor %}

# FAST: Simplified template
# Only use needed variables
- name: Generate config
  template:
    src: simple.j2
    dest: /etc/app/config.xml

# simple.j2:
server_name: {{ inventory_hostname }}
environment: {{ app_env }}
# Only essential vars
```

---

**Comprehensive Optimization Example:**

```yaml
# optimized-playbook.yml
---
- name: Optimized deployment
  hosts: appservers
  gather_facts: no  # Disable fact gathering
  strategy: free  # Don't wait for slow hosts
  serial: "20%"  # 20% of hosts in parallel
  
  vars:
    app_version: "1.2.0"
  
  tasks:
    # Gather only needed facts
    - name: Gather minimal facts
      setup:
        gather_subset:
          - '!all'
          - '!min'
          - network
          - virtual
    
    # Batch operations
    - name: Install all packages at once
      yum:
        name:
          - java-11-openjdk
          - python3
          - git
        state: present
    
    # Download in parallel
    - name: Download artifact
      get_url:
        url: "{{ artifactory_url }}/app-{{ app_version }}.jar"
        dest: /opt/app/app.jar
        checksum: sha256:{{ artifact_checksum }}
      # All hosts download in parallel
    
    # Conditional execution
    - name: Check if restart needed
      stat:
        path: /opt/app/restart-required
      register: restart_flag
    
    - name: Restart only if needed
      systemd:
        name: app
        state: restarted
      when: restart_flag.stat.exists
    
    # Minimal output
    - name: Health check
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: health
      no_log: true  # Don't spam output
    
    - name: Show health summary
      debug:
        msg: "Health: {{ 'OK' if health.status == 200 else 'FAILED' }}"

# ansible.cfg
[defaults]
forks = 20
pipelining = True
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600
host_key_checking = False
callback_whitelist = profile_tasks, timer

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o ServerAliveInterval=60
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
pipelining = True
```

---

**Profiling Results:**

```bash
# Before optimization:
$ ansible-playbook deploy.yml
PLAY RECAP ****************************************
Completed in: 45m 32s

Tasks breakdown:
  gather_facts: 5m 12s
  Install packages (loop): 8m 45s
  Copy files: 25m 30s
  Update configs: 3m 15s
  Restart services: 2m 50s

# After optimization:
$ ansible-playbook deploy-optimized.yml
PLAY RECAP ****************************************
Completed in: 6m 18s  ← 86% improvement!

Tasks breakdown:
  gather_facts (minimal): 15s
  Install packages (batch): 45s
  Download files (parallel): 2m 10s
  Update configs: 1m 30s
  Restart services: 1m 38s
```

**Optimization Checklist:**

✅ Disable fact gathering when not needed
✅ Enable SSH pipelining and ControlPersist
✅ Tune forks based on infrastructure
✅ Batch operations (packages, files)
✅ Use get_url instead of copy for large files
✅ Implement fact caching
✅ Use 'when' conditionals to skip unnecessary tasks
✅ Minimize debug output
✅ Use 'strategy: free' when appropriate
✅ Parallelize independent operations
✅ Use local mirrors/caches for packages
✅ Profile with callback plugins

This systematic approach helped me reduce playbook runtime from 45 minutes to under 10 minutes at Birlasoft!"

---

(Truncating here due to length - the full document would continue with more advanced questions on Ansible Vault, custom modules, dynamic includes, error handling, testing, etc.)

Would you like me to continue with the rest of the advanced questions and create the complete file?