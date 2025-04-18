# Ansible Learning Repository

Welcome to our team's Ansible learning repository! This repository is designed to help team members understand and effectively use Ansible for configuration management, application deployment, and infrastructure automation.

## What is Ansible?

Ansible is an open-source automation tool created by Michael DeHaan in 2012 and later acquired by Red Hat in 2015. It's designed to simplify complex orchestration and configuration management tasks. Unlike other configuration management tools, Ansible is agentless, using SSH to connect to servers and execute commands.

## Why Use Ansible?

- **Simplicity**: Uses YAML syntax which is easy to read and write
- **Agentless**: No agents to install on managed nodes
- **Idempotent**: Only makes changes when needed
- **Extensible**: Can be extended with modules and plugins
- **Secure**: Uses SSH for secure connections
- **Declarative**: You declare the desired state, Ansible handles the rest

## Repository Structure

This repository is organized to provide both learning materials and practical examples:

```
ansible-learning/
├── README.md                 # This file
├── docs/                     # Documentation and tutorials
│   ├── 01-getting-started.md
│   ├── 02-inventory.md
│   ├── 03-playbooks.md
│   ├── 04-roles.md
│   ├── 05-variables.md
│   └── 06-best-practices.md
├── examples/                 # Example playbooks and roles
│   ├── simple-playbook/
│   ├── roles-example/
│   └── github-actions/
├── inventories/              # Example inventory structures
│   ├── development/
│   └── production/
└── roles/                    # Reusable roles
    ├── common/
    └── webserver/
```

## Getting Started

1. **Prerequisites**:
   - Basic understanding of Linux/Unix systems
   - Familiarity with YAML syntax
   - Python installed on your control node

2. **Installation**:
   ```bash
   # For Ubuntu/Debian
   sudo apt update
   sudo apt install ansible

   # For macOS
   brew install ansible
   
   # Using pip
   pip install ansible
   ```

3. **Verification**:
   ```bash
   ansible --version
   ```

4. **First Steps**:
   - Review the documentation in the `docs/` directory
   - Try running the example playbooks

## Learning Path

1. Start with `docs/01-getting-started.md`
2. Understand inventory structures in `docs/02-inventory.md`
3. Learn how to write playbooks in `docs/03-playbooks.md`
4. Explore roles in `docs/04-roles.md`
5. Master variables and facts in `docs/05-variables.md`
6. Follow best practices from `docs/06-best-practices.md`

## Contributing

Feel free to contribute to this repository by adding examples, improving documentation, or fixing issues. Please follow these steps:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## Resources

- [Official Ansible Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/) - Community repository for roles
- [Ansible Blog](https://www.ansible.com/blog)

## License

This repository is licensed under the MIT License - see the LICENSE file for details.
