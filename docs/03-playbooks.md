# Writing Ansible Playbooks

## What are Playbooks?

Playbooks are Ansible's configuration, deployment, and orchestration language. They are YAML files that describe the desired state of your systems and how to achieve that state. Playbooks are the building blocks for complex IT automation scenarios.

## Basic Playbook Structure

A basic playbook consists of one or more plays, each defining a set of tasks to be executed on specific hosts.

```yaml
---
# Basic playbook structure
- name: First play
  hosts: webservers
  become: yes
  
  tasks:
    - name: Install Apache
      package:
        name: apache2
        state: present
        
    - name: Start Apache service
      service:
        name: apache2
        state: started
        enabled: yes
        
- name: Second play
  hosts: dbservers
  become: yes
  
  tasks:
    - name: Install MySQL
      package:
        name: mysql-server
        state: present
```

## Key Components of a Playbook

### Plays

A play maps a group of hosts to tasks. Each play includes:

- `name`: A description of what the play does
- `hosts`: Which hosts or groups to target
- `become`: Whether to use privilege escalation
- `vars`: Variables for use in the play
- `tasks`: List of tasks to execute

### Tasks

Tasks are the units of action in Ansible. Each task calls an Ansible module with specific arguments. A task includes:

- `name`: A description of what the task does
- Module name (e.g., `package`, `service`, `file`)
- Module parameters
- Optional control parameters (e.g., `when`, `register`, `loop`)

Example task:

```yaml
- name: Install web servers
  package:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - apache2
  when: install_web_servers | bool
  register: install_result
```

### Handlers

Handlers are special tasks that run only when notified by other tasks. They are typically used for service restarts:

```yaml
tasks:
  - name: Copy Apache configuration
    copy:
      src: apache.conf
      dest: /etc/apache2/apache2.conf
    notify: Restart Apache

handlers:
  - name: Restart Apache
    service:
      name: apache2
      state: restarted
```

## Variables in Playbooks

You can define and use variables in playbooks:

```yaml
- name: Configure web server
  hosts: webservers
  become: yes
  
  vars:
    http_port: 80
    max_clients: 200
    
  tasks:
    - name: Generate configuration
      template:
        src: httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      vars:
        server_name: "{{ inventory_hostname }}"
```

## Conditionals

Use conditionals to execute tasks based on conditions:

```yaml
tasks:
  - name: Install Apache on Debian-based systems
    apt:
      name: apache2
      state: present
    when: ansible_facts['os_family'] == "Debian"
    
  - name: Install Apache on RedHat-based systems
    dnf:
      name: httpd
      state: present
    when: ansible_facts['os_family'] == "RedHat"
```

## Loops

Use loops to perform a task multiple times:

```yaml
tasks:
  - name: Create users
    user:
      name: "{{ item }}"
      state: present
    loop:
      - alice
      - bob
      - charlie
```

## Importing and Including

Split playbooks into smaller, reusable files:

### Import Playbook

```yaml
# main.yml
---
- import_playbook: webservers.yml
- import_playbook: dbservers.yml
```

### Import Tasks

```yaml
# playbook.yml
---
- hosts: webservers
  tasks:
    - import_tasks: common_tasks.yml
    - import_tasks: web_tasks.yml
```

### Include Tasks

```yaml
# playbook.yml
---
- hosts: webservers
  tasks:
    - include_tasks: common_tasks.yml
    - include_tasks: "{{ ansible_facts['os_family'] }}_tasks.yml"
```

## Error Handling

Control how Ansible responds to errors:

```yaml
tasks:
  - name: This might fail
    command: /bin/false
    ignore_errors: yes
    
  - name: Try again with retry
    command: /bin/something
    register: result
    until: result.rc == 0
    retries: 3
    delay: 5
```

## Tags

Use tags to run specific parts of playbooks:

```yaml
tasks:
  - name: Install packages
    package:
      name: "{{ item }}"
      state: present
    loop:
      - nginx
      - mysql
    tags:
      - packages
      
  - name: Configure firewall
    firewalld:
      service: http
      permanent: yes
      state: enabled
    tags:
      - firewall
      - security
```

Run tasks with specific tags:

```bash
ansible-playbook playbook.yml --tags "packages,firewall"
```

Skip tasks with specific tags:

```bash
ansible-playbook playbook.yml --skip-tags "security"
```

## Using Blocks

Blocks let you group tasks and apply certain parameters to all tasks in the group:

```yaml
tasks:
  - name: Configure web server
    block:
      - name: Install Apache
        package:
          name: apache2
          state: present
          
      - name: Start Apache service
        service:
          name: apache2
          state: started
    when: ansible_facts['distribution'] == 'Ubuntu'
    become: yes
    ignore_errors: yes
```

## Using Roles

You can include roles in your playbooks:

```yaml
- hosts: webservers
  roles:
    - common
    - webserver
    
  tasks:
    - name: Additional tasks
      debug:
        msg: "Running after roles"
```

## Example: Complete Playbook

Here's a complete example of a playbook that sets up a web server:

```yaml
---
- name: Configure web server
  hosts: webservers
  become: yes
  
  vars:
    http_port: 80
    https_port: 443
    html_dir: /var/www/html
    
  tasks:
    - name: Install required packages
      package:
        name:
          - nginx
          - openssl
        state: present
      tags: packages
      
    - name: Create web root directory
      file:
        path: "{{ html_dir }}"
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'
      tags: configuration
      
    - name: Copy website content
      copy:
        src: files/index.html
        dest: "{{ html_dir }}/index.html"
        owner: www-data
        group: www-data
        mode: '0644'
      notify: Reload Nginx
      tags: content
      
    - name: Generate Nginx configuration
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Reload Nginx
      tags: configuration
      
    - name: Enable Nginx site
      file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link
      notify: Reload Nginx
      tags: configuration
      
    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: yes
      tags: services
      
  handlers:
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```

## Best Practices for Playbook Development

1. **Use meaningful names** for plays, tasks, and files
2. **Keep playbooks simple** and focused on specific goals
3. **Use variables** to make playbooks more flexible
4. **Use roles** for reusable components
5. **Use tags** to selectively run parts of playbooks
6. **Use version control** to track changes to playbooks
7. **Test playbooks** in development environments before production
8. **Document playbooks** with comments and README files
9. **Use `--check` mode** to test playbooks without making changes
10. **Adopt an idempotent approach** so playbooks can be run multiple times safely

## Conclusion

Playbooks are powerful tools for automating complex IT tasks. By understanding the structure and features of playbooks, you can create robust, reusable automation that simplifies infrastructure management and application deployment.

In the next section, we'll explore Ansible roles, which provide an even more structured way to organize your automation code.
