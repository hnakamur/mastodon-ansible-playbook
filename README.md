mastadon-ansible-playbook
=========================

## Prerequisite

### Create the target container

- Ubuntu 16.04 (xenial)
- Install and setup LXD
- Create a LXD container named `mastodon`

```
lxc launche images:ubuntu/xenial mastodon
```

### Install ansible

- Ubuntu 16.04 (xenial)

```
python3 -m venv venv
source venv/bin/active
pip install --upgrade pip
pip install ansible
```

## Run the playbook

```
ansible-playbook playbook.yml -v -D
```
