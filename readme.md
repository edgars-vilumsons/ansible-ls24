## Ping All Hosts in "public" Group

You can use the following Ansible command to ping all hosts in the "public" group using the local connection:


```ansible public -m ping```

## To install and start Nginx

```ansible-playbook playbook.yml -u root --ask-pass```

## To backup

```ansible-playbook backup.yml -u root --ask-pass```

