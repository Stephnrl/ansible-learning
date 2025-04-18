# Mastering Ansible Variables and Facts

Variables and facts are essential components in Ansible that help you create flexible, reusable playbooks. This guide will explain how to effectively use variables and gather facts in your Ansible automation.

## Understanding Variables in Ansible

Variables in Ansible store values that can be used throughout playbooks and roles. They allow you to write more adaptable and reusable automation.

## Variable Naming

Valid variable names in Ansible:
- Must start with a letter
- Can contain letters, numbers, and underscores
- Should use snake_case (lowercase with underscores)

Examples:
```
http_port
app_version
max_connections
```

## Variable Definition Locations

Variables can be defined in multiple places, with different precedence levels:

### 1. Inventory Variables

#### In inventory file:
```ini
[webservers]
web1.example.com http_port=80 app_env=production

[webservers:vars]
http_port=80
app_env=production
```

#### In group_vars and host_vars directories:
```
inventories/
├── production/
│   ├── hosts
│   ├── group_vars/
│   │   ├── all.yml         # Variables for all hosts
│   │   └── webservers.yml  # Variables for webservers group
│   └── host_vars/
│       └── web1.yml        # Variables for web1.example.com
```

Example content of `group_vars/webservers.yml`:
```yaml
---
http_port: 80
https_port: 443
max_clients: 200
```

### 2. Playbook Variables

#### In a play:
```yaml
- hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  tasks:
    # Tasks here
```

#### In a separate file included with vars_files:
```yaml
- hosts: webservers
  vars_files:
    - vars/common.yml
    - vars/web.yml
  tasks:
    # Tasks here
```

### 3. Task Variables

```yaml
- name: Template configuration file
  template:
    src: template.j2
    dest: /etc/config.conf
  vars:
    http_port: 8080  # Overrides any previously defined http_port
```

### 4. Role Variables

Default variables in `roles/myapp/defaults/main.yml`:
```yaml
---
http_port: 80
max_clients: 100
```

Role variables in `roles/myapp/vars/main.yml`:
```yaml
---
required_packages:
  - nginx
  - openssl
```

### 5. Extra Variables (highest precedence)

Passed on the command line:
```bash
ansible-playbook playbook.yml -e "http_port=8080 env=staging"
```

Or using a JSON/YAML file:
```bash
ansible-playbook playbook.yml -e @extra_vars.json
```

## Variable Precedence

Variable precedence in Ansible determines which variable definition takes priority when the same variable is defined in multiple locations. Understanding this precedence is crucial for effective variable management in complex Ansible projects.

## Complete Precedence Order (Lowest to Highest)

1. **Role defaults** (`roles/x/defaults/main.yml`)
   * Intentionally lowest precedence
   * Easily overridden baseline values

2. **Inventory-defined variables**
   * Defined directly in inventory files
   ```ini
   [webservers]
   web1.example.com http_port=80
   
   [webservers:vars]
   http_port=80
   ```

3. **Inventory group_vars/all**
   * Variables in `inventory_directory/group_vars/all.yml`
   * Apply to all hosts in a specific inventory (environment)

4. **Playbook group_vars/all**
   * Variables in `playbook_directory/group_vars/all.yml`
   * Apply to all hosts across all inventories
   * Override inventory-level all variables

5. **Inventory group_vars/specific_group**
   * Variables in `inventory_directory/group_vars/webservers.yml`
   * Apply to specific groups in a specific inventory

6. **Playbook group_vars/specific_group**
   * Variables in `playbook_directory/group_vars/webservers.yml`
   * Apply to specific groups across all inventories
   * Override inventory-level group variables

7. **Inventory host_vars/specific_host**
   * Variables in `inventory_directory/host_vars/web1.yml`
   * Apply to specific hosts in a specific inventory

8. **Playbook host_vars/specific_host**
   * Variables in `playbook_directory/host_vars/web1.yml`
   * Apply to specific hosts across all inventories
   * Override inventory-level host variables

9. **Host facts**
   * Automatically gathered information about hosts

10. **Play variables**
    * Defined directly in the play
    ```yaml
    - hosts: webservers
      vars:
        http_port: 8080
    ```

11. **Task variables**
    * Variables defined for a specific task
    * Only apply to that task

12. **Role variables** (`roles/x/vars/main.yml`)
    * Higher-precedence role variables

13. **Block variables**
    * Variables defined for a block of tasks
    * Only apply to tasks within that block

14. **Extra variables**
    * Command line variables using `-e` or `--extra-vars`
    * Highest precedence, override everything else

## Inventory vs. Playbook Variables

### Inventory Variables (Environment-Specific)
Variables stored within inventory directories are tied to specific environments:

```
inventories/
├── production/           # Production environment
│   ├── hosts
│   ├── group_vars/
│   │   ├── all.yml       # Variables for all production hosts
│   │   └── webservers.yml # Variables for production webservers
│   └── host_vars/
│       └── web1.yml      # Variables for specific production host
├── staging/              # Staging environment
│   ├── hosts
│   ├── group_vars/
│   │   ├── all.yml       # Variables for all staging hosts
│   │   └── webservers.yml # Variables for staging webservers
│   └── host_vars/
│       └── web1.yml      # Variables for specific staging host
```

**Key characteristics:**
- Organized by environment first
- Good for environment-specific settings (DB credentials, server addresses)
- Lower precedence than playbook variables

### Playbook Variables (Code-Specific)
Variables stored alongside playbooks apply across environments:

```
ansible-project/
├── site.yml              # Main playbook
├── group_vars/           # Variables for groups across environments
│   ├── all.yml           # Variables for all groups
│   └── webservers.yml    # Variables for webservers group
└── host_vars/            # Variables for hosts across environments
    └── web1.yml          # Variables for specific host
```

**Key characteristics:**
- Organized by function first
- Good for application-specific settings (ports, standard configurations)
- Higher precedence than inventory variables
- Override environment-specific variables when needed

## Practical Example

If you define the same variable in multiple places:

```yaml
# roles/webserver/defaults/main.yml (precedence: 1)
http_port: 80

# inventories/production/group_vars/all.yml (precedence: 3)
http_port: 8080

# group_vars/webservers.yml (precedence: 6)
http_port: 8000

# playbook.yml (precedence: 10)
- hosts: webservers
  vars:
    http_port: 9000
```

The effective value would be `9000` because play variables (10) have higher precedence than playbook group variables (6), which have higher precedence than inventory group variables (3), which have higher precedence than role defaults (1).

## Best Practices

1. **Use role defaults** for baseline configuration that should be easily overridable
2. **Use inventory variables** for environment-specific settings
3. **Use playbook variables** for consistent settings across environments
4. **Use extra vars** for one-off overrides during execution
5. **Document your variable precedence strategy** for team clarity

## Using Variables

### In Tasks

```yaml
- name: Create application directory
  file:
    path: "/opt/{{ app_name }}/data"
    state: directory
    mode: '0755'
```

### In Templates

Using Jinja2 syntax in template files (e.g., `templates/nginx.conf.j2`):

```jinja
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
    location / {
        root {{ web_root }};
        index index.html;
    }
}
```

### Accessing Dictionary Values

```yaml
vars:
  webapp:
    name: myapp
    version: 1.2.3
    port: 8080

tasks:
  - name: Show application info
    debug:
      msg: "App {{ webapp.name }} version {{ webapp.version }} runs on port {{ webapp.port }}"
```

### Accessing List Values

```yaml
vars:
  fruits:
    - apple
    - banana
    - cherry

tasks:
  - name: Show first fruit
    debug:
      msg: "First fruit is {{ fruits[0] }}"
```

## Registered Variables

Capture the output of a task:

```yaml
- name: Check service status
  command: systemctl status nginx
  register: service_status
  ignore_errors: yes
  
- name: Show service status
  debug:
    msg: "Service status: {{ service_status.stdout }}"
  when: service_status.rc == 0
```

## Facts

Facts are variables automatically discovered by Ansible when it connects to a host.

### Gathering Facts

Facts are gathered automatically at the beginning of a play. You can disable this for faster execution:

```yaml
- hosts: all
  gather_facts: no
  tasks:
    # Tasks here
```

Or gather specific facts:

```yaml
- hosts: all
  gather_facts: no
  tasks:
    - name: Gather only specific facts
      setup:
        gather_subset:
          - '!all'
          - '!min'
          - 'network'
          - 'hardware'
```

### Using Facts

```yaml
- name: Show OS information
  debug:
    msg: "Host {{ inventory_hostname }} runs {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}"
```

Common facts include:
- `ansible_facts['distribution']` - OS distribution name
- `ansible_facts['distribution_version']` - OS version
- `ansible_facts['processor_count']` - Number of processors
- `ansible_facts['memtotal_mb']` - Total memory in MB
- `ansible_facts['default_ipv4']['address']` - Default IPv4 address

### Custom Facts

Create custom facts using the `ansible.facts` namespace:

1. Create a directory structure on managed hosts:
```
/etc/ansible/facts.d/
```

2. Add custom fact files with `.fact` extension:
```bash
cat > /etc/ansible/facts.d/customfact.fact << EOF
[application]
name=MyApp
version=1.2.3
environment=production
EOF
```

3. Access custom facts:
```yaml
- name: Show custom facts
  debug:
    msg: "App {{ ansible_facts['ansible_local']['customfact']['application']['name'] }} version {{ ansible_facts['ansible_local']['customfact']['application']['version'] }}"
```

## Filters and Transformations

Jinja2 filters allow you to transform variable values:

```yaml
vars:
  username: "John Smith"
  
tasks:
  - name: Display username in lowercase
    debug:
      msg: "Username: {{ username | lower }}"
      
  - name: Create safe filename
    debug:
      msg: "Filename: {{ username | replace(' ', '_') | lower }}.txt"
```

Common filters:
- `default('value')` - Use default value if variable is undefined
- `upper` / `lower` - Convert to uppercase/lowercase
- `trim` - Remove whitespace
- `join(',')` - Join list items with a separator
- `split(',')` - Split string into a list
- `to_yaml` / `to_json` - Convert to YAML/JSON

## Variable Best Practices

1. **Use meaningful names** that describe what the variable contains
2. **Document variables** with comments
3. **Use default values** for optional variables
4. **Structure complex data** using dictionaries and lists
5. **Override variables** at the appropriate level
6. **Use variable files** for better organization
7. **Limit variable scope** to where they're needed
8. **Ensure consistency** in naming conventions
9. **Define defaults in roles** for reusability
10. **Use `vars_prompt`** for interactive input when needed

## Example: Comprehensive Variable Usage

```yaml
---
- name: Configure web application
  hosts: webservers
  gather_facts: yes
  
  vars:
    deployment_env: production
    common_packages:
      - curl
      - vim
      - htop
  
  vars_files:
    - vars/{{ deployment_env }}.yml
  
  pre_tasks:
    - name: Load environment variables
      include_vars: "vars/{{ deployment_env }}_secrets.yml"
  
  tasks:
    - name: Install common packages
      package:
        name: "{{ common_packages }}"
        state: present
      
    - name: Check if application is already deployed
      stat:
        path: "/opt/{{ app_name }}/current"
      register: app_stat
      
    - name: Show OS and application info
      debug:
        msg: >
          Deploying {{ app_name }} version {{ app_version }} on 
          {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}.
          Application {{ 'already exists' if app_stat.stat.exists else 'will be installed' }}.
      
    - name: Create application directories
      file:
        path: "/opt/{{ app_name }}/{{ item }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_group }}"
        mode: '0755'
      loop:
        - releases
        - shared
        - logs
      
    - name: Generate configuration
      template:
        src: "templates/{{ app_name }}/config.yml.j2"
        dest: "/opt/{{ app_name }}/shared/config.yml"
      vars:
        db_host: "{{ db_config[deployment_env].host }}"
        db_name: "{{ db_config[deployment_env].name }}"
        environment_type: "{{ deployment_env }}"
      notify: Restart application
  
  handlers:
    - name: Restart application
      service:
        name: "{{ app_name }}"
        state: restarted
```

## Conclusion

Effective use of variables and facts makes your Ansible playbooks more dynamic, reusable, and maintainable. By understanding variable precedence and best practices, you can create flexible automation that works across different environments and scenarios.

As you develop more complex playbooks, proper variable management becomes increasingly important. Take time to design your variable structure and follow consistent naming conventions to make your automation more robust and easier to understand.
