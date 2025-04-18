# Ansible Best Practices

This document covers best practices for developing, organizing, and maintaining Ansible automation. Following these guidelines will help you create more effective, maintainable, and scalable infrastructure as code.

## Directory Structure

A well-organized directory structure makes your Ansible content easier to maintain:

```
ansible-project/
├── ansible.cfg               # Ansible configuration file
├── inventories/              # Inventory files
│   ├── production/
│   │   ├── hosts             # Production inventory file
│   │   ├── group_vars/       # Production group variables
│   │   └── host_vars/        # Production host variables
│   └── staging/
│       ├── hosts             # Staging inventory file
│       ├── group_vars/       # Staging group variables
│       └── host_vars/        # Staging host variables
├── playbooks/                # Playbooks directory
│   ├── site.yml              # Main playbook
│   ├── webservers.yml        # Webserver playbook
│   └── dbservers.yml         # Database server playbook
├── roles/                    # Roles directory
│   ├── common/               # Common role
│   ├── webserver/            # Webserver role
│   └── database/             # Database role
├── library/                  # Custom modules
├── module_utils/             # Custom module utilities
├── filter_plugins/           # Custom filters
└── files/                    # Static files
```

## Playbook Organization

### Use a Main Playbook

Create a main playbook that includes other playbooks:

```yaml
# site.yml
---
- import_playbook: playbooks/common.yml
- import_playbook: playbooks/webservers.yml
- import_playbook: playbooks/dbservers.yml
```

### Organize Playbooks by Functionality

Group related tasks into separate playbooks:

```
playbooks/
├── site.yml                  # Main playbook
├── common.yml                # Common setup for all servers
├── webservers.yml            # Webserver configuration
├── dbservers.yml             # Database server configuration
├── monitoring.yml            # Monitoring setup
└── maintenance/              # Maintenance playbooks
    ├── updates.yml           # System updates
    ├── backups.yml           # Backup procedures
    └── security.yml          # Security hardening
```

## Role Development

### Create Focused Roles

Each role should have a single responsibility:

- `common` - Basic system configuration
- `webserver` - Web server installation and configuration
- `database` - Database server installation and configuration
- `monitoring` - Monitoring agent installation and configuration

### Use Role Defaults

Put default variables in `defaults/main.yml`:

```yaml
# roles/webserver/defaults/main.yml
---
http_port: 80
https_port: 443
max_clients: 200
```

### Make Roles Parameterizable

Design roles to be configurable with variables:

```yaml
# Example task in roles/webserver/tasks/main.yml
- name: Generate Nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  vars:
    server_name: "{{ server_name | default(inventory_hostname) }}"
    root_dir: "{{ web_root | default('/var/www/html') }}"
```

### Document Roles

Include a README.md in each role with:

- Purpose of the role
- Required variables
- Optional variables with defaults
- Dependencies
- Example usage
- Author information
- License

### Use Meta Information

Document role dependencies and other metadata:

```yaml
# roles/webserver/meta/main.yml
---
dependencies:
  - role: common

galaxy_info:
  role_name: webserver
  author: Your Name
  description: Installs and configures web server
  license: MIT
  min_ansible_version: 2.9
  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
  galaxy_tags:
    - web
    - nginx
```

## Variable Management

### Variable Precedence

Understand Ansible's variable precedence order:

1. Command line extra vars (`-e`)
2. Task vars
3. Block vars
4. Role and include vars
5. Play vars_files
6. Play vars_prompt
7. Play vars
8. Host facts
9. Playbook host_vars
10. Playbook group_vars
11. Inventory host_vars
12. Inventory group_vars
13. Role defaults

### Variable Naming Conventions

Follow consistent naming conventions:

- Use snake_case (lowercase with underscores)
- Use namespaces for role variables (e.g., `nginx_http_port`)
- Use descriptive names that indicate purpose

### Protect Sensitive Data

Use Ansible Vault for sensitive information:

```bash
# Create encrypted file
ansible-vault create group_vars/all/vault.yml

# Edit encrypted file
ansible-vault edit group_vars/all/vault.yml

# Run playbook with vault password
ansible-playbook site.yml --ask-vault-pass
```

## Task Writing

### Use Module Names Instead of Commands

```yaml
# Bad practice
- name: Start Nginx
  command: systemctl start nginx

# Good practice
- name: Start Nginx
  service:
    name: nginx
    state: started
```

### Use Handlers for Service Restarts

```yaml
tasks:
  - name: Configure Nginx
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart Nginx

handlers:
  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

### Use Loops for Repeated Tasks

```yaml
- name: Install required packages
  package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - openssl
    - python3
```

### Use Conditionals for Platform-Specific Tasks

```yaml
- name: Install Apache (Debian)
  apt:
    name: apache2
    state: present
  when: ansible_facts['os_family'] == "Debian"

- name: Install Apache (RedHat)
  dnf:
    name: httpd
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```

### Use Tags for Task Selection

```yaml
- name: Install packages
  package:
    name: "{{ item }}"
    state: present
  loop: "{{ required_packages }}"
  tags:
    - packages
    - install
```

## Testing and Validation

### Use Check Mode

Verify changes without applying them:

```bash
ansible-playbook site.yml --check
```

### Use Diff Mode

See what would change:

```bash
ansible-playbook site.yml --check --diff
```

### Test in Development First

Follow a testing path:
1. Test on a local VM or container
2. Deploy to development environment
3. Deploy to staging environment
4. Deploy to production environment

### Use Molecule for Role Testing

```bash
# Create a new role with Molecule test framework
molecule init role my_role

# Run tests
cd roles/my_role
molecule test
```

## Performance Optimization

### Optimize Fact Gathering

Disable fact gathering when not needed:

```yaml
- hosts: all
  gather_facts: no
  tasks:
    # Tasks that don't need facts
```

Or gather only specific facts:

```yaml
- hosts: all
  gather_facts: no
  tasks:
    - name: Gather only network facts
      setup:
        gather_subset:
          - '!all'
          - '!min'
          - 'network'
```

### Use Async for Long-Running Tasks

```yaml
- name: Update apt cache
  apt:
    update_cache: yes
  async: 300
  poll: 0
  register: apt_update
  
- name: Wait for apt update to complete
  async_status:
    jid: "{{ apt_update.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 30
  delay: 10
```

### Optimize SSH Connections

In `ansible.cfg`:

```ini
[ssh_connection]
pipelining = True
control_path = /tmp/ansible-ssh-%%h-%%p-%%r
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

## Security Best Practices

### Limit Access to Inventory and Variables

- Store sensitive inventory and variable files in a secure location
- Use version control with appropriate access controls
- Encrypt sensitive data with Ansible Vault

### Use Privileged Access Carefully

- Only use `become: yes` when necessary
- Create dedicated service accounts for Ansible
- Apply the principle of least privilege

### Regularly Update and Audit

- Keep Ansible and its dependencies updated
- Regularly audit playbooks and roles
- Remove unused roles and playbooks

## Documentation Best Practices

### Document Your Ansible Project

Create a comprehensive README.md with:
- Project overview
- Directory structure explanation
- How to use the playbooks
- Prerequisites
- Common commands
- Contribution guidelines

### Add Comments to Playbooks and Roles

```yaml
# This playbook configures web servers
# It installs nginx, sets up SSL, and configures virtual hosts
- name: Configure web servers
  hosts: webservers
  become: yes
  
  # Install base packages
  # These are required for the web server to function
  tasks:
    - name: Install required packages
      package:
        name:
          - nginx
          - openssl
        state: present
```

### Document Variables

```yaml
# Database configuration
# These variables control the database connection parameters
# Required for the application to connect to the database
db_host: localhost
db_port: 3306
db_name: myapp
db_user: appuser
```

## Version Control Integration

### Use Meaningful Commit Messages

```
Add Nginx role for web server configuration

- Installs Nginx package
- Configures SSL certificates
- Sets up virtual hosts
- Enables and starts the service
```

### Tag Releases

Use semantic versioning (e.g., v1.0.0, v1.1.0) for releases and tag them in your version control system.

### Use Branches for Development

- `main` - Stable branch for production use
- `develop` - Development branch for new features
- Feature branches for specific enhancements

## Error Handling

### Use Blocks for Error Handling

```yaml
- name: Handle errors with blocks
  block:
    - name: This might fail
      command: /bin/false
  rescue:
    - name: This runs on failure
      debug:
        msg: "There was an error"
  always:
    - name: This always runs
      debug:
        msg: "This always executes"
```

### Set Failed_When Conditions

```yaml
- name: Check disk space
  command: df -h
  register: df_output
  failed_when: '"100%" in df_output.stdout'
```

## Conclusion

Following these best practices will help you create more maintainable, scalable, and reliable Ansible automation. By organizing your code thoughtfully, documenting your work, and adopting consistent patterns across your organization, you'll build an automation framework that can grow with your infrastructure needs while remaining manageable and understandable by your team.
