# Executing Ansible

This guide covers different methods for executing Ansible playbooks, from local execution to automated runs with GitHub Actions.

## Basic Execution: Running Ansible Locally

### Prerequisites

Before running Ansible locally, ensure:

1. Ansible is installed on your control node
2. You have SSH access to target hosts
3. Python is installed on all target hosts

### Running a Simple Playbook

The basic syntax for running a playbook is:

```bash
ansible-playbook playbook.yml
```

To specify an inventory file:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

### Common Options

- `-i, --inventory`: Specify inventory file
- `-l, --limit`: Limit execution to specific hosts
- `-v, --verbose`: Increase verbosity (-v, -vv, -vvv for more detail)
- `--check`: Dry-run mode (doesn't make changes)
- `--diff`: Show differences when changing files
- `-e, --extra-vars`: Set additional variables
- `--tags`: Only run plays and tasks tagged with these values
- `--skip-tags`: Skip plays and tasks tagged with these values

### Example: Basic Execution

```bash
# Run playbook on all hosts in inventory
ansible-playbook -i inventory.ini site.yml

# Run playbook only on webservers
ansible-playbook -i inventory.ini -l webservers site.yml

# Dry run with difference reporting
ansible-playbook -i inventory.ini --check --diff site.yml

# Run with extra variables
ansible-playbook -i inventory.ini -e "env=production version=1.2.3" site.yml
```

## Execution Strategies

Ansible offers different execution strategies:

- **Linear**: Default strategy, runs each task on all hosts before starting the next task
- **Free**: Allows each host to run through the playbook as fast as possible
- **Debug**: Linear strategy with ability to step through tasks interactively

To specify a strategy:

```yaml
# In your playbook
- hosts: all
  strategy: free
  tasks:
    # Your tasks here
```

## SSH Connection Options

Customize SSH connections:

```bash
# Using different SSH port
ansible-playbook -i inventory.ini --ssh-common-args='-o Port=2222' playbook.yml

# Using SSH keys
ansible-playbook -i inventory.ini --private-key=~/.ssh/id_rsa playbook.yml
```

## Running Ansible in CI/CD Pipelines

### GitHub Actions

You can automate Ansible execution using GitHub Actions. Here's an example workflow:

```yaml
# .github/workflows/ansible.yml
name: Ansible Deployment

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install ansible
          
      - name: Set up SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -t rsa ${{ secrets.HOST }} >> ~/.ssh/known_hosts
          
      - name: Run Ansible playbook
        run: |
          ansible-playbook -i inventories/production/hosts site.yml
```

To use this workflow:

1. Store your SSH private key in GitHub Actions secrets as `SSH_PRIVATE_KEY`
2. Store your target host in secrets as `HOST`
3. Configure your inventory file accordingly

### Running Playbooks with Environment-Specific Variables

For different environments:

```bash
# Development environment
ansible-playbook -i inventories/development/hosts site.yml

# Production environment
ansible-playbook -i inventories/production/hosts site.yml
```

## Running Ansible in Docker

You can use Ansible within Docker containers:

```bash
# Pull the Ansible Docker image
docker pull ansible/ansible

# Run a playbook
docker run --rm -v $(pwd):/ansible -v ~/.ssh:/root/.ssh ansible/ansible ansible-playbook -i inventory.ini playbook.yml
```

## Running Ansible with AWX/Tower

For enterprise environments, consider using AWX (open-source) or Red Hat AAP (Red Hat's commercial offering):

1. Schedule playbook runs
2. Manage credentials securely
3. Track execution history
4. Implement role-based access control
5. Event-Driven Ansible

## Running Ansible with Dynamic Inventory

When working with cloud providers or other dynamic environments:

```bash
# Using AWS EC2 dynamic inventory
ansible-playbook -i inventory_aws_ec2.yml site.yml
```

## Common Execution Patterns

### 1. Development Workflow

During development, use:

```bash
# Check syntax
ansible-playbook --syntax-check playbook.yml

# Dry run with diffs
ansible-playbook --check --diff -i inventory.ini playbook.yml

# Limited deployment
ansible-playbook -i inventory.ini -l dev-server playbook.yml
```

### 2. Production Deployment

For production:

```bash
# Validate before deploying
ansible-playbook --check -i inventories/production/hosts site.yml

# Deploy to production
ansible-playbook -i inventories/production/hosts site.yml
```

### 3. Rolling Updates

For zero-downtime deployments:

```yaml
# In your playbook
- hosts: webservers
  serial: 2  # Number of hosts to update simultaneously
  tasks:
    # Your deployment tasks
```

## Troubleshooting Execution Issues

### Common Problems and Solutions

1. **SSH Connection Issues**:
   ```bash
   # Increase verbosity for SSH debugging
   ansible-playbook -vvv -i inventory.ini playbook.yml
   ```

2. **Privilege Escalation**:
   ```bash
   # Prompt for sudo password
   ansible-playbook -i inventory.ini --ask-become-pass playbook.yml
   ```

3. **Execution Timeouts**:
   ```yaml
   # In ansible.cfg
   [defaults]
   timeout = 30  # Seconds
   ```

## Conclusion

Effective Ansible execution involves choosing the right command-line options, execution strategy, and environment for your specific needs. As your Ansible usage matures, you'll likely move from simple local execution to automated pipelines using GitHub Actions or dedicated tools like AWX/Tower.

Remember that automation is an incremental journey. Start with simple manual execution for testing, then progressively add more automation as your confidence and requirements grow.
