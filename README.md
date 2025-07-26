# Ansible Quickstart

Demo ansible repo following the [official documentation](https://docs.ansible.com/ansible/latest/getting_started/get_started_ansible.html).

## Step 1 : invertory

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

After reading associated documentation, I ignored the warning for the second host since it wouldn't affect normal use of ansible.