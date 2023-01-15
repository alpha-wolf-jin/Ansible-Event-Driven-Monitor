# Ansible-Event-Driven-Monitor

```
echo "# Ansible-Event-Driven-Monitor" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/alpha-wolf-jin/Ansible-Event-Driven-Monitor.git
git config --global credential.helper 'cache --timeout 72000'
git push -u origin main

git add . ; git commit -a -m "update README" ; git push -u origin main
```

# Install ansible-rulebook

**Install the latest RHEL 9**

**Install ansible-rulebook**

Reference: https://github.com/ansible/ansible-rulebook/blob/main/docs/installation.rst

```
dnf --assumeyes install gcc java-17-openjdk maven python3-devel python3-pip
export JDK_HOME=/usr/lib/jvm/java-17-openjdk
export JAVA_HOME=$JDK_HOME
pip3 install -U Jinja2
pip3 install ansible ansible-rulebook ansible-runner wheel
pip3  install aiokafka
```

**Install collection ansible.eda**

```
ansible-galaxy collection install community.general ansible.eda
```

