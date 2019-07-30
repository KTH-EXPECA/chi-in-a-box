# chi-in-a-box

![chi-in-a-box](./chi-in-a-box.png)

Ansible playbooks for deploying Chameleon operations systems.

### Deployment node setup

The first step is to provision the Ansible deployment node (which should be localhost, effectively) with Ansible. It's Ansible all the way down! This will install any other python modules and packages which are necessary for the operation of the Ansible tasks. It's the expectation that as new modules are introduced to this repo, their dependencies are properly bootstrapped by the `ansible` role. Use the `cc-ansible` tool for this.

```bash
export CC_ANSIBLE_SITE=/path/to/site/config
./cc-ansible --playbook playbooks/ansible.yml
# Or, in one command:
./cc-ansible --site /path/to/site/config --playbook playbooks/ansible.yml
```

### Applying playbooks

Playbooks are set up to target a host group with the same name. This means the `grafana` playbook will target the `grafana` host group etc. To run a playbook, you can use the `./cc-ansible` wrapper script, which just sets up two important parameters for you: the Ansible Vault configuration, and the inventory path.

```bash
# Run the 'grafana' playbook (deploys/updates grafana)
./cc-ansible --playbook playbooks/grafana.yml
# Run only the tasks tagged 'configuration' in the 'grafana' playbook
# (Any arguments normally passed to ansible-playbook can be passed here.)
./cc-ansible --playbook playbooks/grafana.yml --tags configuration
```

### Running Kolla-Ansible playbooks

Much of the deployment is ultimately controlled by Kolla-Ansible. To invoke, you can use the `./cc-ansible` tool much like you could use `kolla-ansible`:

```bash
# Deploy the Neutron components
./cc-ansible deploy --tags neutron
# Pull latest images for all components
./cc-ansible pull
# Upgrade Nova and Ironic
./cc-ansible upgrade --tags nova,ironic
```

### Post-deployment

There is a `post-deploy` script you can run to finish things up. This will install compatible versions of all OpenStack clients for your deployment and set up some OpenStack entities needed to do bare metal provisioning.

```bash
./cc-ansible post-deploy
```

Finally, consider adding the following to your .bashrc or similar:

```bash
# Pre-set site so you don't have to type it each time
export CC_ANSIBLE_SITE=/opt/config/<your_site>

# Source OpenStack client environment automatically
if [ -f "$CC_ANSIBLE_SITE/admin-openrc.sh" ]; then
  source "$CC_ANSIBLE_SITE/admin-openrc.sh"
fi

# Source virtualenv to have access to OpenStack clients installed
# in virtualenv (assumes this repo is installed at /etc/ansible)
if [ -f /etc/ansible/venv/bin/activate ]; then
  export VIRTUAL_ENV_DISABLE_PROMPT=1
  source /etc/ansible/venv/bin/activate
fi
```

## Configuration secrets

Secrets like database and user passwords or sensitive API keys should be encrypted with [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) in a `passwords.yml` file located in the site configuration. This is encrypted with a symmetrical cipher key (`vault_password`). This key should **never be stored in source control**. You can edit or view the encrypted contents with the `./cc-ansible` tool:

```bash
# Opens an interactive editor for editing passwords
./cc-ansible edit_passwords
# Prints unencrypted passwords to stdout
./cc-ansible decrypt_passwords
```
