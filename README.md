# Ansible Quickstart

My journey on learning the basics of Ansible.

I started following the [official documentation](https://docs.ansible.com/ansible/latest/getting_started/get_started_ansible.html).

## Step 1: invertory

I created an inventory [`myhosts.yml`](./myhosts.yml) then listed hosts inside with the `ansible-inventory` command:

```
delopa@desktop-fedora:~/code/ansible/ansible_quickstart$ ansible-inventory -i myhosts.yml --list
{
    "_meta": {
        "hostvars": {
            "desktop_fedora": {
                "ansible_host": "192.168.1.101"
            },
            "ichiraku": {
                "ansible_host": "192.168.1.10"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "myhosts"
        ]
    },
    "myhosts": {
        "hosts": [
            "desktop_fedora",
            "ichiraku"
        ]
    }
}
```

Then I used the `ping` module on the host group `myhosts` from the inventory file [`myhosts.yml`](./myhosts.yml) with the `ansible` command:

```
delopa@desktop-fedora:~/code/ansible/ansible_quickstart$ ansible myhosts -m ping -i myhosts.yml 
desktop_fedora | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 192.168.1.101 port 22: Connection refused",
    "unreachable": true
}
The authenticity of host '192.168.1.10 (192.168.1.10)' can't be established.
ED25519 key fingerprint is <fingerprint here>
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: ichiraku
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
[WARNING]: Platform linux on host ichiraku is using the discovered Python interpreter at /usr/bin/python3.11, but future installation
of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ichiraku | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.11"
    },
    "changed": false,
    "ping": "pong"
}
```

The first host (my own machine) was unreachable because there's no SSH server running.

After reading associated documentation, I ignored the warning for the second host since it wouldn't affect normal use of Ansible.

## Step 2: First playbook

I created [`playbook.yml`](./playbook.yml) to ping the hosts and print a message. To use the builtin ansible module I had to use their [Fully Qualified Collection Name (FQCN)](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Fully-Qualified-Collection-Name-FQCN) to call them in tasks.

I called the playbook with the `ansible-playbook` command:

```
delopa@desktop-fedora:~/code/ansible/ansible_quickstart$ ansible-playbook -i myhosts.yml -l ichiraku playbook.yml 

PLAY [My first play] *******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
[WARNING]: Platform linux on host ichiraku is using the discovered Python interpreter at /usr/bin/python3.11, but future installation
of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [ichiraku]

TASK [Ping my hosts] *******************************************************************************************************************
ok: [ichiraku]

TASK [Print message] *******************************************************************************************************************
ok: [ichiraku] => {
    "msg": "Hello World!"
}

PLAY RECAP *****************************************************************************************************************************
ichiraku                   : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

I used the `-l` option to limit the hosts since my machine still lacks a SSH server (see step 1).

I'll get rid of the warning message when I reach the part of setting up `ansible.cfg` file.

---

As the official quickstart ends here the steps will be the ones I give myself while reading the documentation as I understand new concepts and discover new potential use cases for Ansible.

---

## Step 3: A custom task

The goal is to create a playbook that updates the `ichiraku` server.

### The theory

`ichiraku` is a Debian based server so I need to find a way to run `apt` commands on the [managed node](https://docs.ansible.com/ansible/latest/getting_started/basic_concepts.html#managed-nodes) in a task, then wrap-up this in a playbook.

The commands I need to use are `apt update` and `apt upgrade -y`. They need to be ran with root privileges.

### What I did

The `ansible.builtin` collection have an `apt` module. I used it as it can provide status codes.

I created the [`update-debian.yml`](./update-debian.yml) playbook to update the APT cache, full-upgrade the system, then remove no longer required dependencies.

I used the `become` property to do a privilege escalation.

```
delopa@desktop-fedora:~/code/ansible/ansible_quickstart$ ansible-playbook -i myhosts.yml -l ichiraku update-debian.yml --ask-become-pass 
BECOME password: 

PLAY [Update Debian] *******************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************
[WARNING]: Platform linux on host ichiraku is using the discovered Python interpreter at /usr/bin/python3.11, but future installation
of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible-
core/2.18/reference_appendices/interpreter_discovery.html for more information.
ok: [ichiraku]

TASK [Upgrade the system] **************************************************************************************************************
changed: [ichiraku]

TASK [Remove dependecies no longer required] *******************************************************************************************
changed: [ichiraku]

PLAY RECAP *****************************************************************************************************************************
ichiraku                   : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

I used the `--ask-become-pass` flag for the `ansible-playbook` command to provide sudo password at runtime.

## Step 4: Secure the become pass

The goal is to store securely the password(s) for the `become` option.

### The theory

Ansible provide a vault feature to store securely some critical stuff. I need to use it to store sudo passwords and map each one to the appropriate host.

### What I did

I used the `host_vars` method to store the `ansible_become_pass` variable.

I created two files under `host_vars/<host>/`: `crypted.yml` and `plain.yml`. The first one is an ansible vault protected with a passphrase to store the become pass with an alias name, the second one is a standard file mapping the encrypted value to `ansible_become_pass`. This way we can know a value is assigned without decrypting the vault.