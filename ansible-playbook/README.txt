# Commands for Ansible Playbook
# Assumption: All commands are run in userspace, therefore with 'sudo'.

# Run playbook inside a Ubuntu 22.04 vm/system:
sudo ansible-playbook playbook.yml

# Just run the ufw(firewall) role:
sudo ansible-playbook /roles/ufw/tasks/main.yml

# Just run the cronjob role:
sudo ansible-playbook /roles/cron/tasks/main.yml

# Just run the ssh role:
sudo ansible-playbook /roles/ssh/tasks/main.yml
