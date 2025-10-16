# 🏠 Home Automation Deployment with Ansible & Docker

This project uses **Ansible** to define, deploy, and manage a suite of self-hosted applications, primarily focused on **Home Automation** and supporting services. It provides an Infrastructure-as-Code (IaC) approach to ensure consistent and repeatable deployments across your infrastructure.

## 🚀 Getting Started

These instructions will get a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

You'll need the following tools installed on your control machine (the machine you run Ansible from):

* **Ansible**: `sudo pip install ansible` (or via your distribution's package manager)
* **Target Hosts**: The servers you are deploying to must be accessible via SSH and have **Python** installed.

### Installation & Setup

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/odykyi-dev/ansible-homelab
    cd ansible-homelab
    ```

2.  **Configure Inventory:**
    Modify the `inventory.yml` file to list your target hosts. A typical setup might look like this:
    ```yaml
    all:
      hosts:
        homeserver:
          ansible_host: 192.168.1.100
          ansible_user: deployer
    ```

3.  **Configure Variables:**
    Review and customize the variables in the `group_vars/docker_images.yml` file for docker image versions. You may also need to set up variables for secrets, paths, and configurations specific to each role (e.g., in `group_vars/all.yml` or using Ansible Vault for sensitive data).

    **Important:** For services like `homeassistant` and `zigbee2mqtt`, update the files in the `templates/` and `files/` directories within their respective roles to match your desired configurations before deployment.

## 🛠️ Usage

The project uses two main playbooks:

### 1. The Main Deployment Playbook (`playbook.yml`)

This is the primary playbook to deploy and configure all services on your target hosts.

```bash
# Dry run (highly recommended for first time):
ansible-playbook -i inventory.yml playbook.yml --check

# Run the deployment:
ansible-playbook -i inventory.yml playbook.yml